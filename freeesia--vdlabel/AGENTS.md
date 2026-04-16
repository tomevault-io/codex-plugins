# VdLabel — Copilot 開発ガイド

VdLabel は Windows 仮想デスクトップの拡張ユーティリティ（.NET 8 / WPF）です。デスクトップへのカスタム名・画像・コマンド出力の表示、ウィンドウの自動移動などを提供します。

## コーディング規約

- **コードコメントは日本語で記述する。**
- **PR の説明（タイトル・本文）は日本語で記述する。**

## ビルド・公開コマンド

```bash
# 依存関係の復元とビルド
dotnet restore
dotnet build VdLabel -c Debug
dotnet build VdLabel -c Release

# 自己完結型で公開（CI と同様）
dotnet publish VdLabel -c Release -o publish --sc true -p:Version=X.X.X.X

# サードパーティライセンスファイルの生成
dotnet tool restore
dotnet nuget-license -t -ignore ignore-packages.json -override package-information.json -i VdLabel/VdLabel.csproj -d licenses -fo licenses/third-party-licenses.txt

# MSI インストーラーのビルド（VdLabel.Wix/ ディレクトリで実行）
cd VdLabel.Wix
dotnet wix extension -g add WixToolset.Util.wixext WixToolset.UI.wixext
dotnet run -c Release
```

## アーキテクチャ

ソリューションは 2 プロジェクト構成：
- **VdLabel** — WPF アプリケーション本体（net8.0-windows, WinExe）
- **VdLabel.Wix** — MSI 生成用コンソールアプリ（.NET Framework 4.7.2, WixSharp 使用）

### MVVM + サービス層

```
Program.cs          → DI / ホスティング設定（Microsoft.Extensions.Hosting + Kamishibai.Hosting）
Views (XAML)        → UI のみ。コードビハインドにはイベントハンドラーのみ記述する
ViewModels          → CommunityToolkit.Mvvm のソース生成を使った partial クラス
Services            → IHostedService / BackgroundService の実装
Models (Config.cs)  → レコード型の設定データ。%AppData%\VdLabel\config.json に保存される
```

**主要サービスと責務：**
| サービス | 役割 |
|---|---|
| `VirtualDesktopService` | Windows 仮想デスクトップ API のラッパー。`DesktopChanged` イベントを発行。`OverlayWindow` のライフサイクルを管理 |
| `CommandLabelService` | タイマーでシェルコマンドを実行し、結果をキャッシュして ViewModel に提供 |
| `WindowMonitor` | `BackgroundService`。新規・フォーカスウィンドウを監視し、設定済みデスクトップへ自動移動 |
| `UpdateChecker` | `BackgroundService`。Octokit で GitHub リリースをポーリング。トースト通知を発行 |
| `ConfigStore` | `System.Text.Json` と `ColorJsonConverter` を使った非同期 JSON 読み書き |
| `PresentationService` | Kamishibai ベースのウィンドウ・ダイアログナビゲーター。`IPresentationService` として注入 |

`IHostedService` も実装するサービスは、コンクリートシングルトンとホステッドサービスの両方として同一インスタンスで登録される：
```csharp
.AddSingleton<VirtualDesktopService>()
.AddHostedService(sp => sp.GetRequiredService<VirtualDesktopService>())
.AddSingleton<IVirualDesktopService>(sp => sp.GetRequiredService<VirtualDesktopService>())
```

## 主要な規約・パターン

### ViewModel
すべての ViewModel は `ObservableObject`（CommunityToolkit.Mvvm）を継承した `partial class`：

```csharp
partial class DesktopConfigViewModel(DesktopConfig config, ...) : ObservableObject
{
    [ObservableProperty]
    [NotifyCanExecuteChangedFor(nameof(RemoveWindowCommand))]
    private WindowConfigViewModel? selectedWindowConfig;

    [RelayCommand(CanExecute = nameof(CanRemoveWindow))]
    private void RemoveWindow() { ... }

    private bool CanRemoveWindow() => SelectedWindowConfig is not null;
}
```

- プロパティは `[ObservableProperty]` を使う（バッキングフィールドと `INotifyPropertyChanged` が自動生成される）。
- コマンドは `[RelayCommand]` を使う（`IRelayCommand` プロパティが自動生成される）。
- 依存関係は `[NotifyCanExecuteChangedFor]` / `[NotifyPropertyChangedFor]` で宣言的に記述する。
- `DesktopConfigViewModel`・`WindowConfigViewModel`・`BadgeConfigViewModel` などのネストした ViewModel は `MainViewModel.cs` 内に partial クラスとして定義する。

### ファイルスコープ名前空間
すべての `.cs` ファイルはファイルスコープ名前空間を使う：
```csharp
namespace VdLabel;
```

### 非同期パターン
- サービスの I/O メソッドは `ValueTask` または `ValueTask<T>` を返す。
- バックグラウンドループは `PeriodicTimer` と `CancellationToken` を使う。
- バックグラウンドスレッドから UI バインドの `ObservableCollection` を更新する場合は `BindingOperations.EnableCollectionSynchronization` を使う。

### モデル
設定モデルは `record` 型（プロパティは `{ get; set; }` または `init` 専用コレクション）：
```csharp
record DesktopConfig
{
    public Guid Id { get; set; }
    public IReadOnlyList<WindowConfig> TargetWindows { get; init; } = [];
}
```
設定は `%AppData%\VdLabel\config.json` に JSON で保存される。`System.Drawing.Color` のシリアライズには `ColorJsonConverter` を使う。

### Windows API
- P/Invoke バインディングは **CsWin32**（`Microsoft.Windows.CsWin32`）のソース生成を優先して使う。
- バージョン固有 API（例：デスクトップ名変更は Windows 11 以降）は `OperatingSystem.IsWindowsVersionAtLeast()` でチェックする。
- 仮想デスクトップ API へのアクセスは `IVirualDesktopService` 経由で行う（インターフェイス名のスペルミスは意図的なので維持すること）。

### UI
- ウィンドウの基底クラスは **WPF-UI** の `FluentWindow`（標準 WPF の `Window` ではない）。
- View は `AddPresentation<TWindow, TViewModel>()`（Kamishibai）で登録し、ウィンドウと DataContext の両方に DI コンストラクタ注入が行われる。
- カスタムスタイルは `Themes/DefaultStyles.xaml` に定義し、`App.xaml` にマージする。

## バージョン管理と CI

バージョンは **GitVersion** が Git タグ（`v*.*.*`）から自動生成する。CI パイプライン（`.github/workflows/dotnet-desktop.yml`）がタグプッシュ時にビルド・ライセンス生成・MSI 作成・GitHub Release 公開を自動的に行う。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Freeesia) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->

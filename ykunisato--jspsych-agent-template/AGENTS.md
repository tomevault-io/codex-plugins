# jsPsych Experiment — Copilot Instructions

## Project Overview

This repository contains a browser-based behavioral experiment built with **jsPsych 8.x**.
The experiment is hosted via **GitHub Pages** from the **root** directory (not `docs/`).

## Tech Stack

- **jsPsych 8.x** — loaded from local `{repo}/jspsych/dist/` (no CDN)
- Vanilla HTML / CSS / JavaScript (no build step, no bundler)
- GitHub Pages for hosting (source: root of `main` branch)

## Project Structure

```
{repo_name}/               ← 課題フォルダ（repo名と同じ）
├── plugins.js             ← 使用プラグインを有効化する（コメントアウト解除）
├── task.js                ← 実験タイムラインを定義する（編集対象）
├── stimuli/               ← 画像・動画などの実験刺激（編集対象）
├── jspsych/dist/          ← jsPsych 8.x ライブラリ本体（変更しない）
└── init_run/              ← initJsPsych / jsPsych.run を担うファイル（変更しない）
    ├── demo_jspsych_init.js   ← initJsPsych と fullscreen トライアルを定義
    ├── demo_jspsych_run.js    ← jsPsych.run(timeline) を実行
    ├── cema_jspsych_init.js
    ├── cema_jspsych_run.js
    ├── jatos_jspsych_init.js
    └── jatos_jspsych_run.js
{repo_name}.html           ← JATOS 用（変更しない）
demo_{repo_name}.html      ← デモ・ローカル確認用（変更しない）
cema_{repo_name}.html      ← CEMA 用（変更しない）
```

## Critical Rules

1. **HTML ファイルは絶対に変更しない**
   - `{repo_name}.html`, `demo_{repo_name}.html`, `cema_{repo_name}.html` は触らない
2. **編集してよいファイルは以下の3つだけ**
   - `{repo_name}/task.js` — タイムラインの要素を書く
   - `{repo_name}/plugins.js` — 使うプラグインのコメントアウトを解除する
   - `{repo_name}/stimuli/` — 実験刺激ファイルを置く
3. **CDN を使わない** — jsPsych は `{repo_name}/jspsych/dist/` から読み込まれる
4. **jsPsych 8.x の API を使うこと** — v7 の書き方（`initJsPsych` の直接呼び出し等）と混同しない

## task.js の書き方

`init_run/demo_jspsych_init.js` で `jsPsych` オブジェクトと `fullscreen` トライアルが定義済みなので、
`task.js` では **これらを使ってタイムライン要素だけを書く**。
`initJsPsych()` と `jsPsych.run()` は不要。

```javascript
/* 課題に関するコードを以下に書く */

const instructions = {
  type: jsPsychHtmlKeyboardResponse,
  stimulus: "<p>教示文をここに書く。</p><p>キーを押して開始してください。</p>"
};

const trial = {
  type: jsPsychHtmlKeyboardResponse,
  stimulus: "刺激テキスト",
  choices: ['f', 'j'],
  data: { task: 'test', condition: 'example' }
};

/* タイムラインの設定（fullscreen は init_run で定義済み） */
const timeline = [fullscreen, instructions, trial];
```

## plugins.js の使い方

使用するプラグインの行頭にある `// ` を削除してコメントアウトを解除する。

```javascript
// 変更前（無効）
// document.write('<script src="name_of_repository/jspsych/dist/plugin-instructions.js"></script>');

// 変更後（有効）
document.write('<script src="name_of_repository/jspsych/dist/plugin-instructions.js"></script>');
```

## 刺激ファイルの配置とパス

画像・動画等は `{repo_name}/stimuli/` に置き、`task.js` からは以下のパスで参照する:

```javascript
stimulus: '<img src="{repo_name}/stimuli/image.png">'
```

## jsPsych 8.x プラグインリファレンス

| Plugin | 用途 |
|--------|------|
| `jsPsychHtmlKeyboardResponse` | テキスト刺激 + キー入力 |
| `jsPsychHtmlButtonResponse` | テキスト刺激 + ボタン入力 |
| `jsPsychImageKeyboardResponse` | 画像刺激 + キー入力 |
| `jsPsychInstructions` | 複数ページの教示文 |
| `jsPsychFullscreen` | フルスクリーン制御（init_run で使用済み） |
| `jsPsychPreload` | メディアファイルの事前読み込み |
| `jsPsychSurveyLikert` | リッカート尺度 |
| `jsPsychSurveyMultiChoice` | 多肢選択 |

## How to Test

1. `demo_{repo_name}.html` をブラウザで直接開いて動作確認する
2. GitHub Pages の URL (`https://{owner}.github.io/{repo}/demo_{repo}.html`) でも確認する
3. 確認ポイント:
   - 教示文が正しく表示されるか
   - すべての試行が実行されるか
   - キー入力 / ボタン入力が正しく記録されるか
   - ブラウザのコンソールにエラーが出ていないか

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ykunisato) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->

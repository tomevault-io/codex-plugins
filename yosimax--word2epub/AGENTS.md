# GitHub Copilotへの指示書

このファイルは、GitHub CopilotやAIアシスタント、開発者がプロジェクトの構造・コーディング規約・命名規則を理解し、一貫したスタイルでコード生成・修正を行うためのガイドです。

<!-- このファイルは https://tech.tokiraku.com/archives/483 を参考に作成しました。 -->


## 1. レビューおよびコメント
- レビューは日本語で記述してください。
- ソース内にコメントを記述する際は英語と日本語の両方で記述してください。

## 3. コーディング規約

### 命名規則

- **言語**: ファイル名、ディレクトリ名、変数名などは全て英語で記述してください。
- **基本**: `user_service.py` のように、役割が明確にわかる名前を付けます。
- **ファイル名**:
    - **形式**: スネークケース (`snake_case`) に統一します（例: `my_file.py`）。
    - **命名**: 所属するディレクトリの役割を表す名前を含めます（例: `app/domain/model/` 配下なら `user_model.py`）。
- **クラス名**: キャメルケース (`CamelCase`) を使用します（例: `UserService`）。
- **関数名・変数名**: スネークケース (`snake_case`) を使用します（例: `get_user_by_id`, `total_price`）。
- **定数名**: 大文字のスネークケース (`UPPER_SNAKE_CASE`) を使用します（例: `MAX_RETRIES`）。

#### 良い例・悪い例

- **ファイル名**
    - 良い例: `user_service.py`, `order_model.py`, `user_model.py`（`app/domain/model/`配下）、`order_repository.py`（`app/infrastructure/repository/`配下）
    - 悪い例: `UserService.py`, `OrderModel.PY`, `userservice.py`, `user.py`、`repository.py`、`model.py`（役割や所属が曖昧なもの）
- **クラス名**
    - 良い例: `UserService`, `OrderModel`
    - 悪い例: `user_service`, `order_model`, `userservice`
- **関数名・変数名**
    - 良い例: `get_user_by_id`, `total_price`
    - 悪い例: `GetUserById`, `TotalPrice`, `getUserById`
- **定数名**
    - 良い例: `MAX_RETRIES`, `DEFAULT_TIMEOUT`
    - 悪い例: `maxRetries`, `defaultTimeout`, `Max_Retries`

### 可読性

- 複雑なロジックやコードの意図を明確にするため、コメントを追加してください。
- ファイルや関数の冒頭には、その役割を簡潔に説明するコメントを記述します。
- 適切なインデントと空白を使い、コードの構造を視覚的に分かりやすくしてください。

### ドキュメンテーション (Docstrings)

- 全ての公開されているクラスと関数には、引数、戻り値、送出する例外を明記したDocstringを追加してください。
- コードを修正した際は、関連するDocstringも必ず最新の状態に更新してください。
- **良い例 (Do):**
    ```python
    def calculate_total_price(items: List[Item]) -> float:
        """Calculate the total price of a list of items.

        Args:
            items (List[Item]): A list of items to calculate the total price for.

        Returns:
            float: The total price of the items.

        Raises:
            ValueError: If the items list is empty.
        """
        if not items:
            raise ValueError("Items list cannot be empty.")
        return sum(item.price for item in items)
    ```

---
> Source: [yosimax/word2epub](https://github.com/yosimax/word2epub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-17 -->

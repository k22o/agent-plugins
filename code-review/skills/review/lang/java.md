# Java 固有の観点

### Java-1. null 安全性

- `NullPointerException` が発生しうる箇所（null チェック漏れ）がないか
- `Optional<T>` が適切に使われているか（`get()` を直接呼ぶ前に `isPresent()` 確認、または `orElse` / `orElseThrow` 使用）
- メソッド引数・戻り値の null 許容性が Javadoc や `@NonNull` / `@Nullable` で明示されているか
- コレクション操作（`stream().findFirst()` など）で Optional の安全な取り出しが行われているか

### Java-2. 型安全性・ジェネリクス

- raw 型（`List` の代わりに `List<String>` など）が使われていないか
- 無検査キャスト（`@SuppressWarnings("unchecked")`）の使用に正当な理由があるか
- ジェネリクスの境界（`<T extends Comparable<T>>` など）が適切に設定されているか
- `instanceof` チェック後のキャストが安全か（Java 16+ のパターンマッチング `instanceof` の活用）
- `var` の使用箇所で型が自明かどうか
- `equals()` をオーバーライドしている場合、`hashCode()` も一貫して実装されているか
- コレクションに入れるオブジェクトで `equals()` / `hashCode()` が未定義のまま同一性比較に使われていないか
- フィールドを `final` にしてイミュータブルにできるケースで可変になっていないか（`record` の活用も検討）
- ローカル変数・メソッド引数に `final` を付けて再代入を防げるケースで省略されていないか
- コレクションを返す・渡す場合に `List.of()` / `List.copyOf()` / `Collections.unmodifiableList()` でイミュータブルにしているか

### Java-3. 並行処理・リソース管理

並行処理・外部リソースアクセスが変更に含まれる場合のみ審査する:

- スレッドセーフでないコレクション（`ArrayList`, `HashMap` など）がマルチスレッド環境で使われていないか
- `synchronized` / `volatile` / `java.util.concurrent` の使い分けが適切か
- `CompletableFuture` のチェーンでエラーハンドリングが行われているか（`.exceptionally()` / `.handle()` の有無）
- ファイル・DB 接続・ストリームが `try-with-resources` で確実にクローズされているか
- デッドロックが発生しうるロック取得順序がないか
- タイムアウト値に根拠があるか

### Java-4. 例外・異常系対応

- チェック例外と非チェック例外の使い分けが適切か
- 例外が握りつぶされていないか（`catch (Exception e) {}` のような空キャッチ、またはログのみで再スローなし）
- `Exception` / `Throwable` を丸ごとキャッチしていないか（具体的な例外型を使う）
- ログ出力に機密情報（パスワード・トークン・個人情報）が含まれていないか
- 例外メッセージがデバッグに十分な情報を含んでいるか（スタックトレース・コンテキスト）
- API 層でのエラーレスポンス形式が統一されているか

### Java-5. Javadoc

- `public` / `protected` なクラス・メソッド・フィールドに Javadoc があるか
- `@param`・`@return`・`@throws` が実態と一致しているか（引数名・型・説明の正確性）
- 実装の「何をするか」ではなく「呼び出し側が知るべき契約（前提条件・保証）」が書かれているか

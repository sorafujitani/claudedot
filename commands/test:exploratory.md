# Exploratory Testing Command

あなたは探索的テストの専門家です。プロジェクトのインターフェースを自動判定し、包括的なテストを実施してください。

## フェーズ0: 初期設定

**まず最初に、AskUserQuestion ツールを使って以下を質問してください:**

### 質問1: 成果物の配置場所

```
質問: "探索的テストの成果物をどこに配置しますか？"
選択肢:
1. "docspriv/" - プライベートドキュメント用ディレクトリ（推奨）
2. "test/exploratory/" - テストディレクトリ配下
3. "tmp/exploratory_test/" - 一時ディレクトリ（Git管理外）
4. "カスタムパス" - 任意のパスを指定
```

ユーザーの回答に基づいて、以下の変数を設定してください:
- `TEST_OUTPUT_DIR`: 成果物の配置ディレクトリ（例: "docspriv", "test/exploratory"）
- `TEST_CASES_DIR`: テストケースのディレクトリ（例: "{TEST_OUTPUT_DIR}/test_cases"）
- `TEST_RESULTS_DIR`: テスト結果のディレクトリ（例: "{TEST_OUTPUT_DIR}/test_results"）
- `SCRIPTS_DIR`: スクリプトのディレクトリ（例: "{TEST_OUTPUT_DIR}/scripts"）

### 質問2: .gitignore への追加

```
質問: "テスト成果物を .gitignore に追加しますか？"
選択肢:
1. "はい" - Git管理から除外する
2. "いいえ" - Git管理に含める
```

「はい」の場合、`.gitignore` に以下を追加:
```
# Exploratory testing
{TEST_OUTPUT_DIR}/
```

### 質問3: 自動承認設定

```
質問: ".claude/settings.json に自動承認設定を追加しますか？"
選択肢:
1. "はい" - テスト実行時の承認を不要にする（推奨）
2. "いいえ" - 各操作ごとに確認する
```

「はい」の場合、`.claude/settings.json` に以下を追加:
```json
{
  "approvalRequired": {
    "bash": {
      "patterns": [
        {
          "pattern": "^ruby {TEST_OUTPUT_DIR}/.*$",
          "behavior": "accept",
          "reason": "Exploratory testing scripts"
        },
        {
          "pattern": "^python {TEST_OUTPUT_DIR}/.*$",
          "behavior": "accept",
          "reason": "Exploratory testing scripts"
        },
        {
          "pattern": "^node {TEST_OUTPUT_DIR}/.*$",
          "behavior": "accept",
          "reason": "Exploratory testing scripts"
        }
      ]
    },
    "write": {
      "patterns": [
        {
          "pattern": "^{TEST_OUTPUT_DIR}/.*$",
          "behavior": "accept",
          "reason": "Exploratory testing files"
        }
      ]
    },
    "edit": {
      "patterns": [
        {
          "pattern": "^{TEST_OUTPUT_DIR}/.*$",
          "behavior": "accept",
          "reason": "Exploratory testing files"
        }
      ]
    }
  }
}
```

**これらの質問に対する回答を取得してから、次のフェーズに進んでください。**

---

## フェーズ1: プロジェクト分析と判定

以下を調査してプロジェクトの性質を判定してください:

### 1.1 プロジェクト構造の把握
- README.mdを読んでプロジェクトの概要を理解
- ディレクトリ構造を確認（lib/, src/, app/, bin/, exe/ 等）
- package.json, Gemfile, Cargo.toml, setup.py 等の設定ファイルを確認

### 1.2 インターフェース判定

以下のパターンでインターフェースタイプを判定してください:

#### CLI (Command Line Interface)
**判定基準:**
- `bin/` または `exe/` ディレクトリにコマンドファイルがある
- `cli.rb`, `cli.py`, `cli.js` 等のファイルが存在
- `Thor`, `Commander`, `Click`, `argparse`, `clap` 等のCLIライブラリを使用
- README に CLI 使用例（`$ command --option` 形式）がある

**テスト対象:**
- コマンド引数のバリエーション
- オプションフラグの組み合わせ
- 標準入力/出力/エラー出力
- 終了コード
- ヘルプメッセージ
- エラーメッセージ

#### API (Library/Module Interface)
**判定基準:**
- `lib/` または `src/` ディレクトリに公開APIがある
- クラス、モジュール、関数が export/require されている
- README に API 使用例（`require`, `import`, `use` 等）がある
- テストファイルに API 呼び出しがある

**テスト対象:**
- 公開メソッド/関数のすべての引数パターン
- 型境界値（数値の最小/最大、空文字列、null/nil）
- 不正な型や値でのエラー処理
- 戻り値の型と内容
- 副作用（ファイル操作、状態変更）

#### Web API (HTTP/REST/GraphQL)
**判定基準:**
- `app/`, `routes/`, `controllers/` ディレクトリがある
- `express`, `sinatra`, `flask`, `actix-web` 等のWebフレームワークを使用
- API エンドポイント定義ファイルがある
- `swagger.yml`, `openapi.json` がある

**テスト対象:**
- 各HTTPメソッド（GET, POST, PUT, DELETE等）
- リクエストボディのバリエーション
- クエリパラメータ
- ヘッダー
- 認証/認可
- レスポンスコード
- レスポンス形式（JSON, XML等）

#### GUI/TUI (Graphical/Terminal User Interface)
**判定基準:**
- HTML/CSS/JSファイルがある（GUI）
- `ncurses`, `blessed`, `ink` 等のTUIライブラリを使用（TUI）
- ブラウザベースのインターフェース

**テスト対象:**
- ユーザー操作のシナリオ
- 画面遷移
- 入力検証
- エラー表示

#### 複合型
複数のインターフェースが混在している場合、すべてを検出してください。

## フェーズ2: テスト計画の作成

判定したインターフェースに基づいて、`{TEST_OUTPUT_DIR}/exploratory_testing_plan.md` を作成してください。

### 計画に含めるべき内容:

1. **プロジェクト概要**
   - プロジェクト名
   - 検出されたインターフェースタイプ
   - 主要な機能

2. **テスト戦略**
   - インターフェースごとのテスト方針
   - 優先順位（コア機能 → エッジケース → 統合）

3. **テストカテゴリ**
   - 基本機能テスト
   - エッジケーステスト
   - エラー処理テスト
   - パフォーマンステスト
   - 統合テスト

4. **成功基準**
   - 実行すべきテスト数
   - カバレッジ目標

## フェーズ3: テスト実装

`{SCRIPTS_DIR}/run_exploratory_tests.rb` (または適切な言語) を作成してください。

### スクリプトの要件:

1. **自動テストケース生成**
   - インターフェースタイプに応じたテストケース
   - 正常系と異常系の両方
   - 境界値テスト

2. **実行エンジン**
   - テストケースを順次実行
   - 結果を記録（成功/失敗/エラー）
   - 実行時間を測定

3. **結果記録**
   - `{TEST_RESULTS_DIR}/summary.json` - 統計
   - `{TEST_RESULTS_DIR}/all_results.json` - 全結果
   - `{TEST_RESULTS_DIR}/failures.json` - 失敗詳細
   - `{TEST_RESULTS_DIR}/bugs.md` - バグレポート
   - `{TEST_RESULTS_DIR}/improvements.md` - 改善提案

### CLI テストの例:

```ruby
def test_cli_command(command, args, expected_exit_code: 0)
  output = `#{command} #{args} 2>&1`
  exit_code = $?.exitstatus

  {
    command: "#{command} #{args}",
    exit_code: exit_code,
    output: output,
    passed: exit_code == expected_exit_code
  }
end

# テストケース
test_cli_command('rfmt', '--help', expected_exit_code: 0)
test_cli_command('rfmt', 'format file.rb', expected_exit_code: 0)
test_cli_command('rfmt', 'nonexistent', expected_exit_code: 1)
```

### API テストの例:

```ruby
def test_api_method(method_name, args, expected_result: nil, should_error: false)
  result = begin
    Object.const_get(class_name).send(method_name, *args)
  rescue => e
    { error: e.message, error_class: e.class.name }
  end

  {
    method: method_name,
    args: args,
    result: result,
    passed: should_error ? result.is_a?(Hash) && result[:error] : !result.is_a?(Hash)
  }
end

# テストケース
test_api_method('format', ['code'], expected_result: String)
test_api_method('format', [nil], should_error: true)
test_api_method('format', [''], expected_result: String)
```

## フェーズ4: テスト実行

作成したスクリプトを実行してください。

```bash
cd [project_root]
[実行コマンド - 言語に応じて]
```

例:
- Ruby: `ruby {SCRIPTS_DIR}/run_exploratory_tests.rb`
- Python: `python {SCRIPTS_DIR}/run_exploratory_tests.py`
- Node.js: `node {SCRIPTS_DIR}/run_exploratory_tests.js`
- Rust: `cargo run --bin exploratory_tests`

## フェーズ5: レポート生成と分析

### 自動生成すべきレポート:

1. **summary.json**
```json
{
  "project": "project_name",
  "interface_types": ["CLI", "API"],
  "total_tests": 150,
  "passed": 145,
  "failed": 5,
  "errors": 2,
  "coverage_percentage": 87.5,
  "execution_time_ms": 1234.56,
  "timestamp": "2025-01-25T12:34:56Z"
}
```

2. **bugs.md**
```markdown
# Bugs Found - [Project Name]

## Summary
- Total bugs: X
- Critical: X
- High: X
- Medium: X
- Low: X

## Critical Bugs

### Bug 1: [Interface] [Component] - [Brief Description]
**Severity:** Critical
**Interface:** CLI
**Component:** format command
**Description:** Crashes when input file contains null bytes
**Reproduction:**
```bash
echo -e "x = 1\x00" > test.rb
rfmt format test.rb  # => Segmentation fault
```
**Expected:** Graceful error message
**Actual:** Segmentation fault

---
```

3. **improvements.md**
```markdown
# Improvement Suggestions

## Performance
- Command X takes 500ms, consider caching

## User Experience
- Error messages could be more specific
- Add progress bar for large files

## Test Coverage
- Missing tests for:
  - Unicode edge cases
  - Concurrent access
  - Large file handling (> 10MB)

## Security
- Input validation needed for:
  - File paths (path traversal)
  - Command injection risks
```

## フェーズ6: 結果の要約

テスト完了後、以下の形式でユーザーに報告してください:

```
================================================================================
探索的テスト完了
================================================================================

プロジェクト: [name]
検出されたインターフェース: [CLI, API, etc.]

テスト結果:
  総テスト数: XXX
  成功: XXX (XX%)
  失敗: XXX (XX%)
  エラー: XXX

発見されたバグ: XX件
  - Critical: X
  - High: X
  - Medium: X
  - Low: X

実行時間: X.XX秒

詳細レポート:
  - docspriv/test_results/summary.json
  - docspriv/test_results/bugs.md
  - docspriv/test_results/improvements.md

主な発見:
1. [重要な発見1]
2. [重要な発見2]
3. [重要な発見3]

推奨アクション:
1. [アクション1]
2. [アクション2]

================================================================================
```

## 重要な注意事項

### 自動承認設定の確認
`.claude/settings.json` に以下が設定されていることを確認してください:

```json
{
  "approvalRequired": {
    "bash": {
      "patterns": [
        {
          "pattern": "^ruby docspriv/.*$",
          "behavior": "accept"
        },
        {
          "pattern": "^python docspriv/.*$",
          "behavior": "accept"
        },
        {
          "pattern": "^node docspriv/.*$",
          "behavior": "accept"
        }
      ]
    },
    "write": {
      "patterns": [
        {
          "pattern": "^docspriv/.*$",
          "behavior": "accept"
        }
      ]
    }
  }
}
```

### 非破壊的テスト
- 既存のソースコードを変更しない
- すべての操作を `docspriv/` 配下で実行
- 読み取り専用でプロジェクトを分析

### セキュリティ
- 外部ネットワークへのアクセスは避ける
- システムコマンドの実行は最小限に
- 機密情報（トークン、パスワード等）を含めない

## エラー処理

テスト実行中にエラーが発生した場合:

1. **環境エラー**: 依存関係、コンパイルエラー等
   - エラー内容を記録
   - 可能であれば代替方法を試行
   - ユーザーに報告

2. **テストケースエラー**: テスト自体のバグ
   - スキップして次のテストへ
   - エラーを記録

3. **システムエラー**: 予期しないクラッシュ
   - 安全に中断
   - これまでの結果を保存
   - ユーザーに報告

## カスタマイズのヒント

ユーザーがテストをカスタマイズできるよう、以下を `docspriv/README.md` に記載してください:

- テストケースの追加方法
- 特定カテゴリのみ実行する方法
- 実行時間の調整方法
- レポートフォーマットの変更方法

## 実行開始

上記のフェーズに従って、探索的テストを自律的に実施してください。
質問や不明点があれば、ユーザーに確認してください。

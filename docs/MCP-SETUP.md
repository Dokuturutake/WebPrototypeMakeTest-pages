# MCP Servers Configuration Guide

このプロジェクトでは 5 つの MCP サーバーを使用します。**すべて npx/uvx 経由で自動実行されるため、事前インストールは不要です。**

## 📦 使用 MCP サーバー

### 1. Kiri MCP Server

**用途**: コード検索、ファイル検索、コンテキストバンドル取得

**提供ツール**:

- `context_bundle` - タスクに関連するコードを検索
- `files_search` - ファイルパターン検索
- `snippets_get` - コードスニペット取得
- `deps_closure` - 依存関係解析
- `semantic_rerank` - 検索結果の再ランク付け

**実行方法**: `npx kiri-mcp-server@latest`  
**GitHub**: https://github.com/CAPHTECH/kiri

### 2. Serena MCP Server

**用途**: 記憶・知識管理、プロジェクトコンテキストの永続化

**提供機能**:

- プロジェクト進捗の保存
- 要件定義・計画の記憶
- ベストプラクティスの蓄積
- シンボルレベルでのコード理解と編集（29 ツール）
- 30 以上のプログラミング言語サポート

**実行方法**: `uvx --from git+https://github.com/oraios/serena serena start-mcp-server`  
**GitHub**: https://github.com/oraios/serena

### 3. Chrome DevTools MCP Server

**用途**: ブラウザ自動化、QA テスト、スクリーンショット取得

**提供ツール**:

- **入力自動化** (8 ツール): `click`, `drag`, `fill`, `fill_form`, `handle_dialog`, `hover`, `press_key`, `upload_file`
- **ナビゲーション** (6 ツール): `close_page`, `list_pages`, `navigate_page`, `new_page`, `select_page`, `wait_for`
- **エミュレーション** (2 ツール): `emulate`, `resize_page`
- **パフォーマンス** (3 ツール): `performance_analyze_insight`, `performance_start_trace`, `performance_stop_trace`
- **ネットワーク** (2 ツール): `get_network_request`, `list_network_requests`
- **デバッグ** (5 ツール): `evaluate_script`, `get_console_message`, `list_console_messages`, `take_screenshot`, `take_snapshot`

**実行方法**: `npx chrome-devtools-mcp@latest`  
**GitHub**: https://github.com/ChromeDevTools/chrome-devtools-mcp

### 4. Filesystem MCP Server

**用途**: ファイルシステム操作（公式 MCP サーバー）

**実行方法**: `npx @modelcontextprotocol/server-filesystem@latest <directory>`

### 5. Memory MCP Server

**用途**: 永続的ナレッジグラフ（公式 MCP サーバー）

**実行方法**: `npx @modelcontextprotocol/server-memory@latest`

## 🚀 セットアップ方法

### DevContainer を使用する場合（推奨）

1. VS Code で "Reopen in Container" を実行
2. コンテナが起動し、環境設定が自動実行されます
3. VS Code ウィンドウをリロード（`Ctrl+Shift+P` > "Developer: Reload Window"）
4. GitHub Copilot Chat を開き、MCP ツールが利用可能か確認

**重要**: MCP サーバーは初回使用時に自動的にダウンロードされます。事前インストールは不要です。

### 設定の仕組み

`.vscode/settings.json`で以下のように設定されています：

```json
{
  "chat.mcp.gallery.enabled": true,
  "chat.mcp.discovery.enabled": true,
  "chat.mcp.access": "all",
  "chat.mcp.autostart": true,
  "mcp.servers": {
    "kiri": {
      "command": "npx",
      "args": [
        "-y",
        "kiri-mcp-server@latest",
        "--repo",
        "${workspaceFolder}",
        "--db",
        "${workspaceFolder}/.kiri/index.duckdb",
        "--socket-path",
        "/tmp/kiri-webprototype.sock"
      ]
    },
    "serena": {
      "command": "uvx",
      "args": [
        "--from",
        "git+https://github.com/oraios/serena",
        "serena",
        "start-mcp-server"
      ],
      "env": {
        "SERENA_STORAGE_PATH": "${workspaceFolder}/.serena-data"
      }
    },
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    },
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem@latest",
        "${workspaceFolder}"
      ]
    },
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory@latest"]
    }
  }
}
```

## 💡 使用方法

### GitHub Copilot Chat での使用

MCP サーバーのツールは、GitHub Copilot Chat で自動的に利用可能になります。

**例**:

```
@workspace Kiriを使ってエラーハンドリングのコードを検索してください
```

```
@workspace Serenaにこのプロジェクトの要件定義を保存してください
```

```
@workspace Chrome DevToolsでindex.htmlのパフォーマンスをテストしてください
```

### 利用可能なツールの確認

Copilot Chat で以下のように質問すると、利用可能な MCP ツールの一覧が表示されます：

```
使用可能なMCPツールを教えてください
```

## 🔧 トラブルシューティング

### MCP サーバーが表示されない

1. **VS Code をリロード**: `Ctrl+Shift+P` > `Developer: Reload Window`
2. **設定を確認**: `.vscode/settings.json`の`mcp.servers`セクション
3. **Copilot Chat 拡張機能のバージョン確認**: 0.35.3 以降が必要

### Kiri のインデックス作成に時間がかかる

Kiri は初回起動時にリポジトリをインデックス化します。大規模なプロジェクトでは数分かかる場合があります。

`.kiri/index.duckdb.daemon.log`でログを確認できます：

```bash
cat .kiri/index.duckdb.daemon.log
```

### Serena の起動が遅い

初回起動時は依存関係のダウンロードに時間がかかります。ログを確認：

```bash
tail -f ~/.serena/logs/*/mcp_*.txt
```

### Chrome DevTools でブラウザが起動しない

1. Chrome がインストールされているか確認
2. `--headless`オプションを追加して、ヘッドレスモードで実行

設定例：

```json
"chrome-devtools": {
  "command": "npx",
  "args": [
    "-y",
    "chrome-devtools-mcp@latest",
    "--headless=true"
  ]
}
```

## 📚 詳細ドキュメント

### Kiri

- [公式ドキュメント](https://github.com/CAPHTECH/kiri#readme)
- [ツールリファレンス](https://github.com/CAPHTECH/kiri/blob/main/docs/tools-reference.md)
- [設定オプション](https://github.com/CAPHTECH/kiri/blob/main/docs/setup.md)

### Serena

- [ユーザーガイド](https://oraios.github.io/serena/02-usage/000_intro.html)
- [ツール一覧](https://oraios.github.io/serena/01-about/035_tools.html)
- [設定方法](https://oraios.github.io/serena/02-usage/050_configuration.html)

### Chrome DevTools MCP

- [公式ドキュメント](https://github.com/ChromeDevTools/chrome-devtools-mcp#readme)
- [ツールリファレンス](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/tool-reference.md)
- [トラブルシューティング](https://github.com/ChromeDevTools/chrome-devtools-mcp/blob/main/docs/troubleshooting.md)

## 🎯 ベストプラクティス

1. **Kiri**: コードベースの理解や特定機能の検索に使用
2. **Serena**: プロジェクトの知識を蓄積し、長期的なコンテキストを保持
3. **Chrome DevTools**: QA テスト、パフォーマンステスト、スクリーンショット取得
4. **Filesystem**: ファイル操作が必要な場合
5. **Memory**: セッションを超えた記憶が必要な場合

## 🔄 更新方法

すべての MCP サーバーは`@latest`を指定しているため、VS Code のリロード時に自動的に最新版が使用されます。

手動で特定のバージョンを指定する場合：

```json
"kiri": {
  "command": "npx",
  "args": [
    "-y",
    "kiri-mcp-server@0.26.1",  // 特定バージョン
    "--repo",
    "${workspaceFolder}",
    "--db",
    "${workspaceFolder}/.kiri/index.duckdb",
    "--watch"
  ]
}
```

## ❓ よくある質問

**Q: MCP サーバーのインストールが必要ですか？**  
A: いいえ。npx/uvx が初回使用時に自動的にダウンロードします。

**Q: インターネット接続は必要ですか？**  
A: 初回使用時のみ必要です。一度ダウンロードされれば、オフラインでも動作します。

**Q: 複数のプロジェクトで設定を共有できますか？**  
A: 各プロジェクトの`.vscode/settings.json`に同じ設定をコピーしてください。

**Q: カスタム MCP サーバーを追加できますか？**  
A: はい。`mcp.servers`セクションに新しいエントリを追加してください。

## 🔧 トラブルシューティング

### Kiri MCP が `ENOTSUP` エラーを出す場合

**症状**: ログに `listen ENOTSUP: operation not supported on socket` エラーが表示される

**原因**: DevContainer のファイルシステムが Unix ソケットをサポートしていない（Windows ホスト等）

**解決方法**: ソケットパスを `/tmp` に変更する

```json
"kiri": {
  "command": "npx",
  "args": [
    "-y",
    "kiri-mcp-server@latest",
    "--repo",
    "${workspaceFolder}",
    "--db",
    "${workspaceFolder}/.kiri/index.duckdb",
    "--socket-path",
    "/tmp/kiri-webprototype.sock"
  ]
}
```

または `--watch` オプションを削除してソケットを使用しないモードで実行します。

### MCP サーバーが起動しない場合

1. VS Code ウィンドウをリロード (`Ctrl+Shift+P` > "Developer: Reload Window")
2. 古いプロセスを停止: `pkill -f kiri-mcp-server`
3. ロックファイルを削除: `rm -f .kiri/*.sock .kiri/*.lock`
4. DevContainer を再起動

## 🆘 サポート

問題が解決しない場合：

1. [Kiri Issues](https://github.com/CAPHTECH/kiri/issues)
2. [Serena Issues](https://github.com/oraios/serena/issues)
3. [Chrome DevTools MCP Issues](https://github.com/ChromeDevTools/chrome-devtools-mcp/issues)
4. [VS Code GitHub Copilot Discussions](https://github.com/microsoft/vscode-copilot-release/discussions)

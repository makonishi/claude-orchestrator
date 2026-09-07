# インストールと更新

Codexが設計・合格条件・受入検証を担当し、Claude Codeへ調査や実装を委譲するプラグインです。

## 前提条件

- ローカルプラグインを利用できるCodexと、`codex` コマンド
- インストール・認証済みのClaude Code（`claude` コマンド）
- Python 3とPyYAML（検証ツール用）
- Codexの `plugin-creator` スキルに付属する作成・検証・更新ツール

以下はmacOS・LinuxなどのPOSIXシェル向け手順です。リポジトリを取得し、そのルートディレクトリで実行してください。付属ツールの配置場所が異なる場合は、`plugin_tools` を実際の場所へ変更します。

## 初回インストール

### 1. パスを設定する

```sh
plugin_source="$PWD"
plugin_tools="${CODEX_HOME:-$HOME/.codex}/skills/.system/plugin-creator/scripts"
plugin_parent="$HOME/.agents/plugins/plugins"
plugin_target="$plugin_parent/claude-orchestrator"
```

### 2. 個人マーケットプレイスへ登録する

以下のコマンドは、プラグインの配置先を作成し、`~/.agents/plugins/marketplace.json` に登録します。既に同名プラグインを登録済みの場合は、後述の更新手順を使ってください。

```sh
python3 "$plugin_tools/create_basic_plugin.py" claude-orchestrator \
  --path "$plugin_parent" --with-skills --with-marketplace
cp -R "$plugin_source/skills/claude-orchestrator" "$plugin_target/skills/"
cp "$plugin_source/.codex-plugin/plugin.json" "$plugin_target/.codex-plugin/plugin.json"
python3 "$plugin_tools/validate_plugin.py" "$plugin_target"
python3 "$plugin_tools/read_marketplace_name.py"
```

各コマンドが成功したことを確認してから次へ進みます。最後に出力されたマーケットプレイス名を、次のコマンドの `personal` に使用します。出力が別の名前なら置き換えてください。

### 3. インストールする

```sh
codex plugin add claude-orchestrator@personal
```

新しいCodexタスクを開いて依頼します。

> claude-orchestratorを使って、この変更を実装して。設計と受入検証はCodex、調査と実装はClaude Codeに任せて。品質を優先して、Codexの消費を抑えて。

## Gitでの継続開発

取得したリポジトリを編集元とし、スキルやマニフェストを変更してコミットします。インストール先のコピーやCodexのキャッシュは編集元にしません。

変更を反映する際は、リポジトリのルートで「パスを設定する」の変数を再設定してから、以下を実行します。

```sh
python3 "$plugin_tools/read_marketplace_name.py"
cp -R "$plugin_source/skills/claude-orchestrator/." "$plugin_target/skills/claude-orchestrator/"
cp "$plugin_source/.codex-plugin/plugin.json" "$plugin_target/.codex-plugin/plugin.json"
python3 "$plugin_tools/update_plugin_cachebuster.py" "$plugin_target"
python3 "$plugin_tools/validate_plugin.py" "$plugin_target"
```

原本でファイルを削除・改名した場合は、コピー先に残った旧ファイルも確認して取り除きます。各コマンドの成功を確認後、実際のマーケットプレイス名で再インストールします。

```sh
codex plugin add claude-orchestrator@personal
```

反映確認は新しいCodexタスクで行います。

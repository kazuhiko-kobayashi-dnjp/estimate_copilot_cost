# GitHub Copilot 消費量モニター

[![Platform](https://img.shields.io/badge/platform-Windows-blue)](https://github.com/kazuhiko-kobayashi-dnjp/estimate_copilot_cost)
[![PowerShell](https://img.shields.io/badge/PowerShell-5.1%2B-blue)](https://github.com/PowerShell/PowerShell)
[![GitHub CLI](https://img.shields.io/badge/requires-GitHub_CLI-black)](https://cli.github.com/)

VS Code の Copilot Chat 応答末尾に、**毎回自動で消費量を表示**するツールです。  
月次上限に対する残量をセッション単位でトラッキングします。

```
---
📊 消費量モニター
- 開始時残: 8000.0 (40.0%) @ 2026-06-01 09:00:00
- 現在残:   7950.0 (39.8%)
- 累積消費: 50.0 units
- 月次枠:   20000 units
- リセット: 2026-07-01
```

---

## 必要な環境

| 項目 | 内容 |
|------|------|
| OS | Windows 10 / 11 |
| PowerShell | 5.1 以上（標準搭載） |
| [GitHub CLI](https://cli.github.com/) | 要インストール |
| GitHub Copilot ライセンス | 有効なサブスクリプション |

---

## セットアップ（3ステップ）

### Step 1: GitHub CLI をインストール

https://cli.github.com/ からインストーラーをダウンロードして実行。

```powershell
# インストール確認
gh --version
```

### Step 2: GitHub PAT（Personal Access Token）を取得

1. https://github.com/settings/tokens/new を開く
2. **Note**: 任意（例: `copilot-monitor`）
3. **Expiration**: 任意（推奨: 90日）
4. **Scopes**: `copilot` にチェック ← これだけでOK
5. **Generate token** → 表示されたトークンをコピー（1度しか表示されません）

### Step 3: セットアップスクリプトを実行

```powershell
powershell -ExecutionPolicy Bypass -File Setup-CopilotMonitor.ps1
```

対話形式でPATとプロキシ設定を入力するだけで完了します。

**VS Code を再起動** すると、Copilot Chat の応答末尾に消費量モニターが表示されます。

---

## 動作の仕組み

```
VS Code Copilot Chat
      │
      ▼
copilot-instructions.md ─── Copilotに「毎回モニターを表示せよ」と指示
      │
      ▼
Get-CopilotQuota.ps1 ─── GitHub API を呼び出して消費量を取得
      │
      ▼
gh_token.sec ─────────── DPAPI暗号化トークン（そのPCのそのユーザーのみ復号可能）
```

---

## 生成されるファイル

セットアップ後、以下のファイルが自動生成されます。

| ファイル | 説明 | 配布 |
|---------|------|------|
| `%USERPROFILE%\copilot_billing\gh_token.sec` | DPAPI暗号化トークン | ✗ 共有不可 |
| `%USERPROFILE%\copilot_billing\config.json` | プロキシ等の設定 | ✗ 個人設定 |
| `%USERPROFILE%\copilot_billing\Get-CopilotQuota.ps1` | 消費量取得スクリプト | △ 自動生成 |
| `%USERPROFILE%\.copilot\copilot-instructions.md` | VS Code Copilot 指示ファイル | △ 自動生成 |

---

## セキュリティ

- **トークンは Windows DPAPI で暗号化**されます
- `gh_token.sec` は「そのPCの、そのWindowsユーザー」のみ復号可能
- 他のPCにコピーしても解読できません
- PAT はセットアップ中も画面に表示されません
- API 呼び出し時のみ一時的に環境変数にセットされ、終了後即座に削除されます

---

## 再設定・更新

```powershell
# PAT の更新や設定変更（既存ファイルの上書き確認あり）
powershell -ExecutionPolicy Bypass -File Setup-CopilotMonitor.ps1

# 強制上書き（確認なし）
powershell -ExecutionPolicy Bypass -File Setup-CopilotMonitor.ps1 -Force
```

---

## プロキシ環境について

社内ネットワーク等でプロキシが必要な場合、セットアップ時に入力してください。  
環境変数 `HTTPS_PROXY` / `HTTP_PROXY` が設定されている場合は自動検出します。

---

## トラブルシューティング

| 症状 | 原因と対処 |
|------|----------|
| `gh: command not found` | GitHub CLI が未インストール |
| `📊 消費量モニター (取得失敗)` | PAT のスコープ不足 / 有効期限切れ |
| プロキシエラー | プロキシURLが誤っている / 社外ネットワーク |
| `Token file not found` | `Setup-CopilotMonitor.ps1` を再実行 |

詳細は [Wiki](../../wiki) を参照してください。

---

## ライセンス

MIT License

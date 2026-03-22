# MEMORY.md - マルの長期記憶

## OpenClaw Gateway 操作メモ

### Gateway の構成（Mac mini）
- Gatewayは**systemドメインのLaunchDaemon**として動いている
- plist: `/Library/LaunchDaemons/ai.openclaw.gateway.plist`
- `openclaw gateway restart` は**効かない**（LaunchAgentとして認識してしまう）
- `launchctl list | grep openclaw`（sudoなし）は**何も出ない** → 必ず `sudo launchctl list` で確認

### APIキー等の環境変数変更後の正しい再起動手順

```bash
# 1. 停止（サービス名で指定。plistパスはNG）
sudo launchctl bootout system/ai.openclaw.gateway

# 2. 起動
sudo launchctl bootstrap system /Library/LaunchDaemons/ai.openclaw.gateway.plist

# 3. 確認
sudo launchctl list | grep openclaw
# PIDが変わっていれば成功
```

### よくあるハマりポイント
- `bootout` に plistパスを渡すと `Boot-out failed: 5` になる → **サービス名（system/ai.openclaw.gateway）で指定する**
- `bootstrap` が `Bootstrap failed: 5` になっても、bootoutが成功していれば再度bootstrapを試せばOK
- `openclaw gateway restart` はLaunchAgent（gui/UID）ドメイン向けのため、systemドメインのGatewayには効かない

### plist内の環境変数変更方法（インデントに注意）
```bash
sudo python3 << 'EOF'
import plistlib
path = '/Library/LaunchDaemons/ai.openclaw.gateway.plist'
with open(path, 'rb') as f:
    d = plistlib.load(f)
d['EnvironmentVariables']['ANTHROPIC_API_KEY'] = 'NEW_KEY_HERE'
with open(path, 'wb') as f:
    plistlib.dump(d, f)
print('OK')
EOF
```
※ `sudo python3 -c "..."` のヒアドキュメントは**インデントが壊れるので使わない**。`<< 'EOF'` 形式を使う。

### エージェントのAPIキー管理

各エージェントのAPIキーは個別のauth-profilesファイルで管理されている：
- `/Users/openclaw/.openclaw/agents/<agent名>/agent/auth-profiles.json`

APIキーを一括更新する場合はplistではなくこのファイルを直接編集すればよい（sudo不要）。
Gatewayを再起動すれば反映される。

対象エージェント一覧：`main`, `luna`, `chii`, `tabby`, `leo` など

### TelegramとWebchatの連携

2026-03-22 に発覚した問題：`default` Telegram botがmainエージェントに紐付いていなかった。

**原因：** `openclaw.json` の `bindings` に以下が抜けていた：
```json
{
  "type": "route",
  "agentId": "main",
  "match": {
    "channel": "telegram",
    "accountId": "default"
  }
}
```

→ 追加済み。これでTelegramとWebchatが同じマルのセッションに繋がる。

---

_最終更新: 2026-03-22_

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

対象エージェント一覧：`main`, `luna`, `chii`, `tabby`, `leo` など

#### ⚠️ APIキー更新の正しい手順（2026-03-22 血の教訓）

**問題の構造：**
- `chii`, `tabby`, `luna` は `secretref-env:ANTHROPIC_API_KEY` でplist環境変数を参照する設定
- plistを更新しても `pkill` や `bootout→bootstrap` だけでは**launchdが環境変数をキャッシュして古いキーのまま動く**ことがある
- auth-profilesに古いキーのハッシュ（`sha256:154a23a3efe6`）がキャッシュされると、キーを変えても401が続く

**正しい手順：**

1. **plistを正しいキーで更新**（sudo必要）
```bash
sudo python3 << 'EOF'
import plistlib
NEW_KEY = 'ここに正しいキー'
path = '/Library/LaunchDaemons/ai.openclaw.gateway.plist'
with open(path, 'rb') as f:
    d = plistlib.load(f)
d['EnvironmentVariables']['ANTHROPIC_API_KEY'] = NEW_KEY
with open(path, 'wb') as f:
    plistlib.dump(d, f)
print('OK')
EOF
```

2. **全エージェントのauth-profilesを直接キーで完全リセット**（secretref-envは使わない）
```python
import json
WORKING_KEY = 'ここに正しいキー'
for agent in ['main', 'luna', 'chii', 'tabby', 'leo']:
    path = f'/Users/openclaw/.openclaw/agents/{agent}/agent/auth-profiles.json'
    d = {
        "version": 1,
        "profiles": {
            "anthropic:default": {
                "type": "api_key",
                "provider": "anthropic",
                "key": WORKING_KEY
            }
        },
        "lastGood": {"anthropic": "anthropic:default"},
        "usageStats": {"anthropic:default": {"errorCount": 0}}
    }
    with open(path, 'w') as f:
        json.dump(d, f, indent=2)
    print(f'{agent}: OK')
```

3. **bootout → bootstrap でGateway再起動**
```bash
sudo launchctl bootout system/ai.openclaw.gateway
sudo launchctl bootstrap system /Library/LaunchDaemons/ai.openclaw.gateway.plist
# Bootstrap failed: 5 が出ても、もう一度bootstrapを実行すればOK
sudo launchctl bootstrap system /Library/LaunchDaemons/ai.openclaw.gateway.plist
```

4. **環境変数が反映されているか確認**
```bash
sudo launchctl print system/ai.openclaw.gateway | grep ANTHROPIC_API_KEY
# 新しいキーが表示されればOK
```

5. **pkillは使わない** → launchdキャッシュが残るため不完全

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

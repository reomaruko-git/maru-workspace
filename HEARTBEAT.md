# HEARTBEAT.md

## Luce監視

`ps aux | grep auto_trader | grep -v grep` でLuceの起動確認を行う。

状態は `memory/luce-state.json` で管理：
- `status`: "running" or "down"
- `downSince`: 最初にdownを確認したUnixタイムスタンプ（running中はnull）
- `alerted`: 1時間アラートを送信済みかどうか（bool）

### ルール
1. プロセスが見つかれば status="running"、downSince=null、alerted=false にリセット
2. プロセスが見つからなければ:
   - 初回: status="down"、downSince=現在時刻 を記録
   - downSince から60分以上経過 かつ alerted=false なら → NAOKOさんに「Luceが1時間以上止まっています！確認をお願いします 🚨」と通知し、alerted=true に更新
   - alerted=true なら再通知しない（次にrunningになったらリセット）

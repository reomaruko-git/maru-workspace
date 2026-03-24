# TOOLS.md - Local Notes

## Web検索
- **SerpApi** を使用（Google検索ラッパー）
- 日本語対応あり
- 月100回無料
- 呼び出し方: `web_fetch("https://serpapi.com/search.json?q=検索語&api_key=SERPAPI_KEY&gl=jp&hl=ja&num=5")`
- APIキー: `f478611ba8ccb960878f5ee32995781e4ada6ae4120d8441e876234a11a4d497`

## 案件CSVの置き場所

NAOKOさんがスクレイピングしたCSVは **`/Users/openclaw/Public/input_candidates/`** に置く。
マルが自動でコピーして `anken-asp.py` で選定する。

## Luce監視
- luceユーザーのプロセス: `ps aux | grep auto_trader`
- ログ（同期済み）: `/Users/openclaw/.openclaw/workspace/luce_auto_trader.log`
- ポジション（同期済み）: `/Users/openclaw/.openclaw/workspace/luce_positions.json`
- 統計（同期済み）: `/Users/openclaw/.openclaw/workspace/luce_trade_stats.json`
- 5分ごと自動同期: `/usr/local/bin/luce_sync.sh`

## インフラ
- Mac mini（adminnoMac-mini）
- openclawユーザー: OpenClaw Gateway
- luceユーザー: FX Trade Luce
- LaunchDaemon: `/Library/LaunchDaemons/com.cathackstudio.fx-luce.plist`

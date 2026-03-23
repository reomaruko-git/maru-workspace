# レオ セットアップ計画

## レオとは
好奇心旺盛なアビシニアン。Cat Hack Studioの雑談・リサーチ担当。
NAOKOさんの気軽な話し相手として活躍予定。

---

## セットアップ手順

### 1. ワークスペース作成
```bash
mkdir -p ~/.openclaw/workspace-leo
```

### 2. エージェント登録
```bash
openclaw agents add leo --workspace ~/.openclaw/workspace-leo
```

### 3. レオのSOUL.md作成
`~/.openclaw/workspace-leo/SOUL.md` に性格・役割を定義

### 4. Telegramのバインド（別チャット or グループ）
レオ専用のTelegramボットを作るか、グループチャットに招待する形になる

```bash
# 例: 特定のチャットIDをレオに割り当て
openclaw agents bind --agent leo --bind telegram:<chat_id>
```

### 5. アイデンティティ設定
```bash
openclaw agents set-identity --agent leo --name "レオ" --emoji "🐱"
```

---

## レオの性格（SOUL.md 案）

- **明るくフレンドリー**。雑談が大好き
- **好奇心旺盛**。何でも「それ面白い！」と食いつく
- **アビシニアンらしく活発**。テンション高め
- **話題転換が得意**。飽きっぽいけど愛嬌がある
- NAOKOさんの話をよく聞いて共感する

---

## 実現方法の選択肢

### A. レオ専用Telegramボットを作る（おすすめ）
- BotFatherで新しいボットを作成
- OpenClawにbotトークンを追加
- そのボットをレオエージェントにバインド
- メリット: マル（元ジジ）とレオが別々のTelegramアカウントになる

### B. グループチャットにマル（元ジジ）とレオを招待
- マル（元ジジ）とレオが同じグループにいる形
- 会話の流れで役割分担できる

---

## 優先度
- [ ] NAOKOさんにBotFather用のTelegramを操作してもらう
- [ ] レオ用ボット作成
- [ ] ワークスペース・SOUL.md設定
- [ ] バインド・テスト

---

_作成: 2026-03-18 by マル（元ジジ）_

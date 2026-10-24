# おでかけ帳

イベントやお店の新着情報を読み込んで眺めつつ、行きたい場所・行った場所をメモしておく PWA。GitHub Pages で動く。ビルド不要で、`index.html` をそのまま配信する。

- 「新着」タブ: 設定した GitHub リポジトリの JSON ファイルを読み込んで表示する。既読・非表示・行きたいに追加をこの端末に記録する
- 「行きたい」「行った」タブ: 自分で追加したメモ。新着から「行きたいに追加」もできる
- ホーム画面に追加するとアプリとして起動し、オフラインでも前回の内容を表示できる

## 新着の読み込み先

アプリ本体には情報源を持たせず、新着の JSON は別のリポジトリ（private でよい）に置く。「新着」タブの下にある「新着の読み込み先を設定」から次を入力する。

| 項目 | 内容 |
| --- | --- |
| リポジトリ | `owner/name` |
| ファイル | JSON ファイルのパス（既定は `news.json`） |
| 読み取り用トークン | private リポジトリの場合に必要。そのリポジトリだけを対象に、Contents を Read-only にした [fine-grained personal access token](https://github.com/settings/personal-access-tokens/new) |

GitHub の API（`GET /repos/{owner}/{repo}/contents/{path}`）で取得する。設定とトークンは端末の localStorage にだけ保存し、どこにも送らない（GitHub への取得リクエストを除く）。

### JSON の形式

```json
{
  "updatedAt": "2026-10-24T09:00:00.000Z",
  "items": [
    {
      "id": "n-example-event",
      "title": "イベント名",
      "category": "event",
      "source": "情報源のサイト名",
      "url": "https://example.com/event",
      "publishedAt": "2026-10-20",
      "eventDate": "2026-11-03",
      "eventEnd": "",
      "area": "会場",
      "summary": "日時・料金・予約などの要点",
      "kidFriendly": true,
      "collectedAt": "2026-10-24T09:00:00.000Z"
    }
  ]
}
```

`category` は `event`（イベント）/ `open`（開店）/ `close`（閉店）/ `community`（地域のお知らせ）。

## 端末に保存するデータ

| キー | 内容 |
| --- | --- |
| `odekake.items` | 行きたい・行ったのメモ（配列） |
| `odekake.newsState` | 新着ごとの既読・非表示・追加済み（`{ [id]: { read, hidden, saved } }`） |
| `odekake.source` | 新着の読み込み先とトークン |
| `odekake.newsCache` | 最後に読み込んだ新着（オフライン用） |

メモは「行きたい」「行った」タブの一番下の「書き出す」で JSON ファイルにでき、「読み込む」で取り込める（`id` が重複しないものだけ追加）。

```json
{ "app": "odekake", "version": 1, "exportedAt": "...", "places": [ ... ], "newsState": { ... } }
```

### 注意点

- iPhone ではホーム画面に追加したアプリと Safari で保存場所が別になる
- localStorage はオリジン（`https://<user>.github.io`）単位なので、同じドメインの他のアプリと領域を共有する。キー名を `odekake.` で始めて区別している
- ブラウザのサイトデータ削除やアプリの削除でメモは消えるので、ときどき書き出しておく

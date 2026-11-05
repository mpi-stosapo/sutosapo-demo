# ストサポの修正と公開のルール

ストマ装具をお使いの方向けのアプリ「ストサポ」（https://stosapo.jp/ ）を、担当者のClaude Codeから直すためのルールです。このファイルは誰でも読める場所にあるので、鍵や個人情報を書きません。

## 大事なこと

- 医療に関わる画面です。頼まれた箇所だけを直し、頼まれていない文言や見た目を変えません。
- 公開するのは、担当者が確認用ページを見て「公開して」と言ったときだけです。
- 患者さんやお客様の個人情報を、ファイルにも修正の説明にも入れません。このリポジトリの中身と修正の履歴は、誰でも見られます。
- 鍵はホームの `.stosapo/token` にあります。画面に出さず、ファイルや修正の説明にも書きません。

## 直してよい範囲

- 直してよいもの：`index.html`、`book.html`、`assets/` の中
- 触らないもの：`CNAME`、`.gitignore`、`.rebuild`、`CLAUDE.md`
- `test/` は確認用ページ専用です。手順2以外で触りません。

Q&Aの文章は、このアプリではなくマスターのスプレッドシートで直します。シートを直すと、アプリを開き直した時点で反映されます。画面の修正で直そうとせず、担当者にそう伝えます。

`index.html` の次の部分は毎朝の自動更新が書き換えるので、形を変えません。

- `/*SS_YOMI_BEGIN*/` から `/*SS_YOMI_END*/` まで
- `/*SS_VTITLE_BEGIN*/` から `/*SS_VTITLE_END*/` まで
- `window.SS_KB = [...]` の行、`function _bg(s)`、`window.ssApplyKB`、`var VIDS`

## やりとりの方法

GitHubのAPIを使います。gitは要りません。リポジトリは `mpi-stosapo/sutosapo-demo`、枝は `main` です。

- 取得：`GET https://api.github.com/repos/mpi-stosapo/sutosapo-demo/contents/<ファイル>`。`sha` はJSONの返事にあり、本文は `Accept: application/vnd.github.raw` で取ります。
- 送信：同じURLへ `PUT`。JSONは `{"message": 説明, "content": 本文のbase64, "sha": 取得したときのsha}` です。新しいファイルなら `sha` を付けません。
- 認証：`Authorization: Bearer <鍵>`

本文は必ずバイトのままbase64にします（UTF-8、改行も変えません）。大きいファイルをコマンドの引数に入れず、ファイル経由で送ります。

## 修正の手順

1. **最新を取る。** 毎回、GitHubから `index.html` とその `sha` を取り直します。手元の古いファイルを土台にしません。直すのはこの取得した本文です。
2. **確認用ページに出す。** 直した本文に次の4つだけ手を加え、`test/index.html` へ送ります。
   - `"assets/` を `"/assets/` に、`'assets/` を `'/assets/` に、`"book.html` を `"/book.html` に、`'book.html` を `'/book.html` に置き換える
   - `window.SS_LOG_URL = '...';` を `window.SS_LOG_URL = '';` に置き換える（確認用の操作をチャットの記録に残さないため。この影響でシートの最新は読まず、アプリの中の控えで動きます）
   - `<head>` の直後に `<meta name="robots" content="noindex,nofollow">` を入れる
   - `</body>` の直前に `<div style="position:fixed;left:0;bottom:0;z-index:2147483647;background:#c00;color:#fff;font-size:11px;padding:2px 6px;pointer-events:none">確認用（本番ではありません）</div>` を入れる

   1分ほどで https://stosapo.jp/test/ に出ます。出たことを確かめてから、担当者にスマホで見てもらいます。
3. **公開する。** 担当者が「公開して」と言ったら、手を加えていない直した本文を、手順1の `sha` を付けて `index.html` へ送ります。
4. **確かめる。** https://stosapo.jp/ を取得し、送った本文と同じになるまで見ます（最大5分）。同じになってから「公開しました」と報告します。

送信で409が返ったら、その間に別の公開がありました。上書きせず、手順1からやり直します。

## 元に戻すとき

1. `GET .../commits?path=index.html&per_page=10` で履歴を見て、戻したい版を担当者と決めます。
2. `GET .../contents/index.html?ref=<その版のsha>` で本文を取ります。
3. 今の `index.html` の `sha` を付けて送り、手順4で確かめます。

## 報告の仕方

担当者はプログラミングをしない方かもしれません。何を直して、どこで見られるかを普通の言葉で伝えます。

# サービス概要
こどもの作品を残し、家族みんなで共有できるWebサービスです。<br>
PCはもちろん、スマホからも扱いやすいUIにしています。<br>
<br>
サービス名は「tottoko（トットコ）」<br>
写真を「撮る」という意味と、残こしておくという意味での「とっておく」から命名。<br>

~~<a href="https://tottoko.su-dx.com" target="_blank">https://tottoko.su-dx.com</a>~~ ※2022/10/2 サービス停止


# システム全体構成
![System configuration diagram](images/tottoko-AWS.drawio.png)

# 使用技術

## フロントエンド
React + Typescriptを使用したシングルページアプリケーション（以下、SPA）を採用。<br>
本番環境ではCDN（Cloud Front）を使用し、SPAで不利となる初期表示の遅延を改善。<br>

### 使用した主要ライブラリ
| ライブラリ名 | バージョン | 説明 |
|----------------|------------|------|
| react | 17.02 |  |
| typescript | 4.5.5 |  |
| chakra-ui | 1.8.5 | UIコンポーネントライブラリ |
| react-router-dom | 6.2.1 | ルーティングライブラリ |
| ky | 0.29.0 | FetchベースのHTTPクライアントライブラリ |
| react-query | 3.34.16 | データフェッチングライブラリ<br>サーバーデータのキャッシュ機構として使用|
| react-hook-form | 7.27.1 | フォームバリデーションライブラリ <br>登録・編集系画面で使用|
| reduxjs/toolkit | 1.7.2 | 状態管理ライブラリ<br>認証情報とログインユーザーの情報のみ管理 |

### アプリ特徴
- バックエンドで生成したアクセストークン(JWT)やリフレッシュトークン(Cookie,JWT)を用いてリソースのアクセス制限やセッション維持を実現。<br>
→ サイレントリフレッシュ機能や起動時自動ログイン機能を実装。
- `chakra-ui`を使用し、チェックボックスカードのUIコンポーネントを実装。
- `keen-slider`、`chakra-ui`を使用し、カルーセルUIコンポーネントを実装。
- `react-query`を使用し、コメントの「もっと見る」機能を実装。

### 開発環境特徴
- `create-react-app`を使用し、プロジェクトテンプレートを作成。
- エディタは`VSCode`を使用し、VSCodeからブラウザ起動・デバッグできるよう設定。
- `ESLint`、`StyleLint`、`Prettier`を導入・設定し、VSCodeと連携。
- `simple-git-hooks`、`lint-staged`を導入し、コミット時にコード解析＆テストを自動実行できるよう設定。

### CI / CD
GitHubのmainリポジトリにコードがマージされた際、GitHub Actionsで以下内容を実施。
- `ESLint`, `StyleLint`を実行し、テストを実施。
- 不備がなければProduction Buildし、本番環境用のコードを生成、S3に配備。
- 配備したコードをユーザーが取得できるよう、Cloud Frontのキャッシュを無効化。

## バックエンド
RailsのAPIモードを使用。
Dockerを使用したコンテナベースで開発し、REST APIを提供。<br>
本番環境でもそのままコンテナを使用できるようAWSのECS（Fargate）を使用。<br>
負荷分散装置（Application Load Balancer）を使用することでスケールアウト可能な構成を実現。<br>
画像はS3に保存し、メール配信はAWS SESを使用。

### 使用した主要ライブラリ
| ライブラリ名 | バージョン | 説明 |
|----------------|------------|------|
| rails | 6.1.4.1 | APIモードで使用 |
| bcrypt | 3.1.16 | パスワードのハッシュ化ライブラリ |
| jwt | 2.3.0 | JWT作成・検証ライブラリ |
| rack-cors | 1.1.1 | CORS設定ライブラリ |
| image_processing | 1.12.1 | 画像処理ライブラリ<br>サブネイル作成に使用 |
| jbuilder | 2.11.2 | JSON形式のデータを出力するライブラリ |
| kaminari | 1.2.2 | ページングライブラリ<br>一覧系を出力する機能で使用 |

### アプリ特徴
- 以下機能をJWTを用いて実現。
  - ユーザー認証（アクセストークン、リフレッシュトークンの実装）
  - アカウントアクティブ化<br>※ JWT付きURLをログインIDのメールアドレスに送ることでメールアドレスの有効性を検証
  - パスワードリセット
  - メールアドレス変更
- 画像保存はRailsの`Active Storage`を使用。
- マスターデータはDBを使用せず`active_hash`を使用してメモリ管理。

### 開発環境特徴
- `docker`及び、`docker-compose`を用いて、コンテナ開発環境を構築。<br>
※イメージ軽量化のためベースイメージは`alpine`を使用。
- `rspec`、`factory_bot`を使用してテスト実施。
- デバックは`pry`を使用。

### CI / CD
GitHubのmainリポジトリにコードがマージされた際、GitHub Actionsで以下内容を実施。
- マージしたリポジトリの内容からdockerイメージを作成。
- `docker-compose`を使用し、作成したdockerイメージをテスト(rspec)。
- 不備がなければAWS ECRにdockerイメージを登録。
- 登録したdockerイメージを使用し、ECS(Fargate)のタスクを新しく起動。<br>
※既存タスクは停止させる

# アプリ機能
## 機能一覧
| 機能分類 | 機能名| 説明 |
|----------|-------|------|
| ユーザー管理 | 新規登録 | ユーザーの新規登録を行う。 |
| | アクティベイト | 登録したユーザーを有効化する。登録したメールアドレスにアクティベイト用URLを送付することで実現。（※JWT利用） |
| | ログイン | ユーザーの認証を行う。アクティベイトしていないユーザーはログイン不可。 |
| | パスワードリセット | ユーザーのパスワードをリセットする。<br>ログインしている場合、ログインIDのメールアドレスにパスワードリセット用URLを送付することで実現。<br>ログインしていない場合（パスワードを忘れた場合）、ログインIDのメールアドレスを指定することで、パスワードリセット用URLを送付。（※いずれもJWT利用） |
| | メールアドレス変更 | ユーザーのメールアドレスを変更する。<br>変更対象のメールアドレスに変更用URLを送付することで実現。（※JWT利用） |
| | プロフィール表示 | ユーザーのプロフィールを表示する。 |
| | プロフィール編集 | ユーザーのプロフィール（表示名やアバター）を変更する。 |
| 家族管理 | お子さま追加 | お子さまを追加する。|
| | お子さま削除 | お子さまを削除する。お子さまに紐づく作品も削除される。|
| | お子さま編集 | お子さまの登録情報を変更する。 |
| | 家族追加 | お子さまの作品を閲覧できる家族（ユーザー）を追加する。<br>（※ 追加する家族が「パパ」、「ママ」、「子ども自身」の場合、作品の投稿・編集・削除が可能。） |
| | 家族解除 | お子さまの作品を閲覧できる家族（ユーザー）を解除する。  |
| 作品管理 | 作品投稿 | お子さまの作品を新規投稿する。 |
| | 作品編集 | 投稿した作品を変更する。 |
| | 作品削除 | 投稿した作品を削除する。 |
| | 作品一覧表示 | お子さまの作品を一覧表示する。<br>ページネーション可能。また、お子さまで作品を絞り込み可能。 |
| | 作品詳細表示 | 作品の詳細を表示する。<br>複数画像のカルーセル表示、画像最大化表示が可能。また、複数画像の一括ダウンロードが可能。 |
| コメント管理 | コメント追加 | 作品にコメントを追加する。 |
| | コメント削除 | 作品のコメントを削除する。 |
| いいね管理 | いいね追加 | 作品に「いいね」を追加する。 |
| | いいね削除 | 作品の「いいね」を削除する。 |
| | いいねカウント | 作品の「いいね」をカウントする。 |

<br>

## 動作イメージ（抜粋）

### 作品一覧表示
お子さまによる作品絞り込みと、ページネーションを実現。<br>
（※ 絞り込みはURLと紐付かせているため、ブラウザバック可能）<br>

![作品一覧表示](images/作品一覧表示.gif)
<br>
<br>

### 作品詳細表示
カルーセルUI、画像最大表示、いいねボタン、画像ダウンロード、コメントの「もっと見る」を実現。<br>

![作品詳細表示](images/作品詳細表示.gif)
<br>
<br>


### 作品投稿
画像プレビュー＆削除、画像圧縮、画像種類（png, jpeg）チェック等を実現。<br>

![作品投稿](images/作品投稿.gif)
<br>
<br>


### お子さま追加
![お子さまの追加](images/お子さまの追加.gif)
<br>
<br>


### 家族の追加
![家族の追加](images/家族の追加.gif)
<br>
<br>


### スマホ表示
![スマホ表示](images/スマホ表示.gif)
<br>
<br>

# ER図
![ER図](images/ER図.png)


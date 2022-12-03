# contentful-good-ref-lambda

Contentfulでいいね数を管理している場合に、いいね数を参照する処理です。  
LocalでのCLI実行、AWS LambdaでAPI Gatewayを経由した実行を可能にしています。  
Goで記述されています。

下記のリポジトリも合わせて参照ください。

- [いいねボタンをWeb上からクリック可能にするフロントエンドの実装](https://github.com/datsukan/good-button)
- [いいね数をインクリメントする実装](https://github.com/datsukan/contentful-good-increment-lambda)
- [いいね数をデクリメントする実装](https://github.com/datsukan/contentful-good-decrement-lambda)

## Settings

### 環境変数の設定

#### Localの場合

```sh
cp .env.example .env
```

`.env`の環境変数にContentfulの値を設定する。


#### AWS Lambdaの場合

Web上で管理コンソールから [ Lambda > 関数 > contentful-good-ref > 設定 > 環境変数 > 編集 ] を開き、下記の通り設定する。

| Key | Value |
| --- | --- |
| CONTENTFUL_SPACE_ID | Contentfulの`Space ID`の値 |
| CONTENTFUL_ACCESS_TOKEN | Contentfulの`Generate personal token`の値 |
| ENV | `production` |

#### CONTENTFUL_SPACE_ID

[ Contentful dashboard > Settings > General settings > Space ID ]

#### CONTENTFUL_ACCESS_TOKEN

[ Contentful dashboard > Settings > API Keys > Content management tokens > Generate personal token ]


### Contentfulの設定

### いいね数の枠を用意

`Content model`を新規作成して`Field`として`goods`を追加する。  
これがいいね数を保持する箇所となる。

その上でContentから新規でgoodsのentryを作成する。

### Localの確認・変更

[ Contentful dashboard > Settings > Locales ] で`Locale`を`Japanese (ja)`にする。  
デフォルト値の`English (en)`で使用したい場合は本リポジトリの`response/lang.go`の`LangNum`を`Ja`→`En`＆`ja`→`en`、およびこの構造体を参照する箇所をすべて`En`に修正する。

## How to use

### Localでの実行

```sh
go run main.go -local -id=xxxxxxxxxxxxxxxxxxxxxxxx
```

`id`はContentfulのContentからいいね数のentryを開き、右サイドバーのinfoタブに記載されている`ENTRY ID`の値を使用する。

実行した結果、いいね数が標準出力にJSONで表示されれば問題ない。  
ContentfulのWebアプリ上の表示と一致していることを確認する。

### API Gateway / AWS Lambdaでの実行

#### 1. AWS Lambdaで関数を新規作成する

- 関数名は`contentful-good-ref`とする（任意）
- ランタイムは`Go 1.x`

#### 2. ランタイム設定からハンドラを設定する

Web上で管理コンソールから [ Lambda > 関数 > contentful-good-ref > コード > ランタイム設定 > 編集 ] を開き、ハンドラを`contentful-good-ref-lambda`に設定する。

#### 3. 実行ファイルをアップロードする

本リポジトリのソースコードをLocalにcloneする。  
cloneしたディレクトリ内でビルド（`go build`）を実行する。  
生成された`contentful-good-ref-lambda`ファイルをZipに圧縮する。  
Web上で管理コンソールから [ Lambda > 関数 > contentful-good-ref > コード > コードソース > アップロード先 > .zipファイル ] を開き、先程生成したZipファイルを選択してアップロードする。

#### 4. API GatewayでREST APIを作成する

APIを作成で`REST API`を選択して構築する。  
API名は`contentful-good`とする。（任意）  

リソースのアクションで`リソースの作成`を選択して`{article_id}`を作成する。  

`{article_id}`に対してアクションで`メソッドの作成`を選択してGETリクエストを選択する。  
統合タイプは`Lambda 関数`、Lambda プロキシ統合の使用はON、Lambda関数は`contentful-good-ref`（任意で変更した場合は合わせて入力）を設定して保存する。  

アクションで`APIのデプロイ`を選択して、ステージの`URLの呼び出し`に記載されているURLを控えておく。

#### 5. WebAPIへリクエストを行う

API Gatewayでデプロイ時に控えたURLに対してリクエストを行い、インクリメントを実行する。

```
GET {url}/{article_id}
```

`{article_id}`の箇所はContentfulで作成したContentのentryのIDを入れる。

実行した結果、ステータス：200でいいね数がJSONでレスポンスされれば問題ない。  
ContentfulのWebアプリ上の表示と一致していることを確認する。

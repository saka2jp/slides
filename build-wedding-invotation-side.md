---
marp: true
theme: gaia
class: invert
footer: © 2023 @saka2jp
---

## エンジニア夫婦で

## 結婚式の WEB 招待状を作った話

---

## 自己紹介

- 🪪 坂津潤平
- 🧑‍💻 フリーランスエンジニア
  - Go/TypeScript
- 🧖‍♂️ サウナ
  - 「サ活だいすき坂津」
- 🎮 スプラトゥーン、APEX
- Follow me!!
  - Twitter, GitHub, etc...

![bg right width:50%](../images/profile-icon.png)

---

## 結婚しました

- 今年の 5 月に挙式予定
  - https://www.shihantei.com/
- WEB 招待状の共作にチャンレジ！

---

![bg](../images/shihantei.jpg)

---

## デモ

- 実際に作成した WEB 招待状をお見せします

---

## 役割分担

- 奥さん：現マネージャー、自分：現アプリエンジニア
- よい機会なので普段の職域から離れてそれぞれチャレンジ！
  - デザイン・コンテンツ：奥さん
  - インフラ：自分

---

## 技術選定

1. Netlify/Vercel/Cloudflare Pages などの静的サイトホスティングサービス
2. AWS Amplify
3. Firebase Hosting
4. GitHub Pages

---

## モチベーション駆動開発

- 「AWS（インフラ）に詳しくなりたい！」
- マネージドサービスは確かに便利だけどブラックボックス
- 良くも悪くも仕組みをわかっていなくても動いてくれる
- AWS エンジニアとして成長するために、CloudFront+S3 で静的サイトホスティングにチャレンジ！
- Not Amplify！

---

## Amazon S3 - 静的ウェブサイトホスティング

- オブジェクトストレージサービスでありながら、静的ウェブサイトをホスティングできる神サービス
- 以下の 3Step でお手軽にホスティングができる
  1. 静的ウェブサイトホスティングの有効化
  2. インデックスドキュメントの設定
  3. アクセス許可の設定
- ただし、Amazon S3 単体では HTTPS をサポートしていない

<span style="font-size: 22px">参考文献: https://docs.aws.amazon.com/ja_jp/AmazonS3/latest/userguide/WebsiteHosting.html</span>

---

## CloudFront - SSL 対応

- CDN（Contents Delivery Network）のマネージドサービス
- ディストリビューションのオリジンに S3 を指定する
- `*.cloudfront.net` ドメイン名で HTTPS を使用できる
  - ディストリビューション用に既に選択されている CloudFront のデフォルト SSL/TLS 証明書
- 味気ないのでどうせならカスタムドメインにしたい！

<br />

<span style="font-size: 22px">参考文献: https://aws.amazon.com/jp/premiumsupport/knowledge-center/cloudfront-https-requests-s3/</span>

---

## Route53 - カスタムドメイン

- DNS（Domain Name System）サーバーのマネージドサービス
- ドメインはお名前.com で無料で取得済み
- Route53 で作成したホストゾーンのネームサーバーをお名前.com に登録し、名前解決できるように設定

<br />
<br />

<span style="font-size: 22px">参考文献: https://dev.classmethod.jp/articles/route53-domain-onamae/</span>

---

## AWS Certificate Manager - SSL/TLS 証明書の発行

- SSL/TLS 証明書のマネージドサービス
- ACM を利用することで、CloudFront や ELB などの他の AWS サービスとシームレスに連携することができる
- Route53 で作成したカスタムドメインの証明書を発行

<br />

<span style="font-size: 22px">参考文献: https://aws.amazon.com/jp/premiumsupport/knowledge-center/install-ssl-cloudfront/</span>

---

## 小ネタ - 自動デプロイ構築してみた ①

- GitHub Actions で特定のブランチにマージされたときに自動デプロイ
- デプロイ用の IAM ユーザーを作成
- aws-cli で S3 にコンテンツをアップロード

---

## 小ネタ - 自動デプロイ構築してみた ②

```yaml
on:
  pull_request:
    branches:
      - main
    types:
      - closed
jobs:
  deploy-s3:
    name: deploy s3
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-1
      - name: s3 sync
        working-directory: ./
        run: aws s3 sync . s3://***
```

---

## 感想 ①

- どのサービスを使えば実現ができるのか？なぜそのサービスが必要なのか？を考えながら構築することでサービスに対する理解が深まった！
- Amplify を使わずに自分で構築するという目的だったが、S3 も CloudFront も Route53 も ACM も便利なマネージドサービスだということを改めて認識

---

## 感想 ②

- マネージドサービスはすごく便利だが、一人のエンジニアとしてはその中身までしっかり理解したい
- 仕組みを理解するには自分で 1 から作ってみることが 1 番！
- 次は DNS サーバーを自前で構築してみるかも...？

---

## 参考文献

- クラスメソッドの数々の記事に助けられました
  - [CloudFront と S3 で作成する静的サイト構成の私的まとめ | DevelopersIO](https://dev.classmethod.jp/articles/s3-cloudfront-static-site-design-patterns-2022/)
  - [CloudFront で素早くコンテンツを更新させたい場合に TTL を短くし Invalidation を行わないキャッシュ戦略を考える | DevelopersIO](https://dev.classmethod.jp/articles/cloudfront-update-content-quickly-using-short-ttl/)
  - [S3 静的ウェブサイトにサーバーレスなお問い合わせフォームを実装してみた（Amazon SES + AWS Lambda + API Gateway） | DevelopersIO](https://dev.classmethod.jp/articles/serverless-mailform-s3-website-hosting/)

---

## まとめ

- AWS、ハンパない
- クラスメソッド、ハンパない

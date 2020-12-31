# Aweti.me

Awetimeの総合プロダクト．

このリポジトリの中身をそのままホスティングサービスに上げたい．

## 現在の構成

本システムは現在defaultサービスと他2つのサービスで構成されています，

- default
    - Landing Pages
    - オリジン : aweti.me
    - 定義ファイル : app.yaml
    - ソースリポジトリ : kaniyama-t/awetime-landing
    - サブモジュール名: www/
- front
    - フロント側TimesWiki(分報文化まとめwiki)
    - オリジン : times.aweti.me
    - 定義ファイル: dispatch.yaml, front/front.yaml
    - ソースリポジトリ: kaniyama-t/awetime-frontend
    - サブモジュール名: front/
- back
    - API(バック)側TimesWiki(分報文化まとめwiki)
    - オリジン : api.aweti.me
    - 定義ファイル: dispatch.yaml, back/back.yaml
    - ソースリポジトリ: kaniyama-t/awetime-backend
    - サブモジュール名: back/

## とりあえずのタスク

- API定義
- バックエンド側をGinで開発
- フロントエンド側をAxiosで開発
- GAEにテストデプロイ
- 構成を変更
- 変更した構成でテスト

## 変更したい構成

本システムは複数のサービスをローンチしたいので，将来的に以下の構成にしたいと思っています．

- default
    - Landing Pages & Root Local Storage Hub
    - オリジン : aweti.me
    - 定義ファイル: app.yaml
    - ソースリポジトリ: kaniyama-t/awetime
    - サブモジュール名: www/
- Accounts
    - ログインシステム
    - オリジン : accounts.aweti.me
    - 定義ファイル: dispatch.yaml, Accounts/Accounts.yaml
    - ソースリポジトリ: kaniyama-t/awetime-accounts
    - サブモジュール名: Accounts
- API
    - バック側システム(全サービス)・子APIモジュールのインクルード
    - オリジン : api.aweti.me
    - 定義ファイル: dispatch.yaml, API/API.yaml
    - ソースリポジトリ: kaniyama-t/awetime-api
    - サブモジュール名: API <br /> <br />
    - TimesWikiAPI
        - API(バック)側TimesWiki(分報文化まとめwiki)・自発的ホスティングはしない
        - エンドポイント : api.aweti.me/times/*
        - 定義ファイル: main.go
        - サブモジュール名: TimesWikiAPI
        - ソースリポジトリ: kaniyama-t/awetime-TimesWikiAPI
- TimesWikiApp
    - フロント側TimesWiki(分報文化まとめwiki)
    - オリジン : times.aweti.me
    - 定義ファイル: dispatch.yaml, TimesWikiApp/TimesWikiApp.yaml
    - ソースリポジトリ: kaniyama-t/awetime-TimesWikiApp
    - サブモジュール名: TimesWikiApp

以下に適当に作った `dispatch.yaml` をはっつけます(ここにメモってごめんなさい)

```
dispatch:
  - url: "*times.aweti.me/*"
    service: TimesWikiApp

  - url: "*accounts.aweti.me/*"
    service: Accounts

  - url: "*api.aweti.me/*"
    service: API
```

### 新構成切り替え後のログイン方法

クライアント側は， `accounts.aweti.me/login?redirect=https://XXX.aweti.me/XXXX` にPOSTすることで認証出来ます．

具体的な方法は以下の通りです．

1. RequireAuthなページから accounts.aweti.me/login?redirect=https://XXX.aweti.me/XXXX に `303 See Other` で転送する.
2. accounts.aweti.me/login はログイン処理をする．
    1. `200 OK` でログインページを提供する
    2. ログインによる認証はクライアント側で行われる，
    3. ローカルストレージに認証情報が入る
    4. aweti.me/sync?redirect=... に `303 See Other` で転送する．
3. aweti.me/sync は，accountsのローカルストレージにある認証情報を保存する．
4. パラメータにあったredirectのオリジンが正常なら，当該ページにリダイレクトする．

### 新構成切り替え後のサービスの追加方法

#### フロント側

以下の通り規則を設けます．

- リリースtagを付与し，`awetime(リポジトリ)` のサブモジュールのコミットの向きを変更する．

以下に `awetime(リポジトリ)` 内の立ち位置を示します．

- AnotherServiceApp
    - オリジン : another-service.aweti.me
    - 定義ファイル : dispatch.yaml AnotherService/AnotherService.yaml
    - ソースリポジトリ: 任意
    - サブモジュール名 : AnotherServiceApp(サービス名と同一・awetimeリポジトリにはサブモジュールでインクルード)

#### API側

以下の通り規則を設けます．

- リリースtagを付与し，`awetime-api(リポジトリ)` のサブモジュールのコミットの向きを変更する．
- 各APIごとにdocsディレクトリまたはリファレンスが書かれたマークダウンファイルを追加します．

以下に `awetime-api(リポジトリ)` 内の立ち位置を示します．

- AnotherServiceAPI
    - エンドポイント : api.aweti.me/AnotherService/*
    - 定義ファイル : main.go
    - ソースリポジトリ: 任意
    - サブモジュール名 : AnotherServiceAPI(サービス名と同一・awetimeリポジトリにはサブモジュールでインクルード)

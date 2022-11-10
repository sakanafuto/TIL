# BFF と GraphQL

よく同じ文脈で語られるため整理する。

## BFF

> Backend For Frontend

MicroServices の文脈で語られる。様々な Micro Service が存在する際に、それら多数のバックエンドとフロントエンドをつなぐ役割を果たす。

例）

1. Client がブログページをリクエストする。
1. BFF がそれを請け負う。
1. BFF が 著者 と記事、コメントそれぞれの責務を担う Micro Service に Request を送信する。
1. レスポンスを BFF が受け取ると、それぞれを 1 つの JSON にしてフロントエンドに返却する。

- クライアントサイドは UI に注力できる。
- BFF はキャッシュを保持できるためパフォーマンスもいい。

## GraphQL

> クエリ型の Web API

- RESTful でありがちなムダなリクエスト/レスポンスのやりとりを解決できる。
- 仕様として定められている。
- 強力な型システム（GraphQLType）がある。
- JSON を返却する。
- 基本的に`/graphql`のような単一のエンドポイントで、REST で表現すると単一のエンドポイントに SQL をポストするようなイメージ。

## 結局なに？

API の Presentation-Domain Separation ができる。BFF としての GraphQL。

### Client

- 単一のエンドポイント、単一の仕様。
- ユースケースごとに自分でクエリを作成する。

### Server Side

- ドメイン API から微細なユースケースの対応を減らせる。
- クライアント都合の変更が減る。API 仕様を伝えるコミュニケーションが減る。
- Schema がドキュメントなる。
- SS 都合で自由に API を作成・統合・撤廃できる。

### CQRS

> 読み込み系と書き込み系の API を分けるデザインパターン

- そもそも GraphQL は Query と Mutation というように CQRS でデザインされている。

## 参考

- [GraphQL が解決する問題とその先のユースケース](https://zenn.dev/saboyutaka/articles/07f1351a6b0049)
- [Node.js + GraphQL で BFF を作った話](https://qiita.com/qsona/items/2c906a4c736f9a10a2c5)
- [GraphQL と BFF について学ぶ](https://github.com/Shinyaigeek/shinyaigeek.dev/issues/66)

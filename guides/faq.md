---
title: FAQ
summary: Frequently Asked Questions about ATP
---

# FAQ

Authenticated Transfer Protocol (ATP)に関するよくある質問です。

## ATPはブロックチェーンですか？

いいえ、ATPは[federated protocol](https://en.wikipedia.org/wiki/Federation_(information_technology))の一つです。ブロックチェーンではありませんし、ブロックチェーンを使うわけでもありません。

## なぜActivityPubを使用しないのですか？

[ActivityPub](https://en.wikipedia.org/wiki/ActivityPub)は、[Mastodon](https://joinmastodon.org/)によって普及した連合型ソーシャルネットワーキング技術です。

アカウントのポータビリティは、私たちが別のプロトコルを構築することを選択した主な理由です。ポータビリティは、突然の禁止、サーバーの停止、ポリシーの不一致からユーザーを保護するため、非常に重要だと考えています。ポータビリティのための我々のソリューションは、[署名付きデータリポジトリ](/guides/data-repos.md)と[DID](/guides/identity.md)の両方を必要としますが、どちらもActivityPubに後付けすることは容易ではありません。ActivityPubの移行ツールは比較的限定的で、元のサーバーがリダイレクトを提供する必要があり、ユーザーの以前のデータを移行することはできません。

その他にも、スキーマの扱い方についての見解の違い、APのdouble-@のメールユーザー名ではなくドメインユーザー名の優先、大規模な検索と発見を目指す（ActivityPubが好むハッシュタグスタイルの発見ではなく）、などの小さな違いがあります。

## なぜJSON-LDやRDFを使わずLexiconを作成するのですか？

ATPでは、組織間でデータやRPCコマンドをやり取りしています。データやRPCを便利に使うためには、別々のチームが作成したスキーマをソフトウェアが正しく扱う必要があります。これが[Lexicon](/guides/lexicon.md)の目的です。

エンジニアには安心してスキーマを作成できるように、開発者にはシステムのDXを楽しんでもらえるようにしたいです。Lexiconは、開発者にとって非常に馴染みやすい強い型付けのAPIを作成し、（分散システムで重要な）様々な実行時の正しさをチェックするのに役立っています。

[RDF](https://en.wikipedia.org/wiki/Resource_Description_Framework)は、システムがほとんどインフラを共有しない、極めて一般的なケースを対象としています。コンセプト的にはエレガントですが、使い方が難しく、開発者が理解できない多くの構文が追加されることが多いです。JSON-LDは、RDF語彙を消費するタスクを単純化しますが、それは、RDFをより読みやすくするのではなく、基本的な概念を隠すことで実現します。

私たちはRDFを使うことをよく検討しましたが、DXやツールが気に入りませんでした。

## XRPCとは何ですか、なぜ___を使わないんですか？

[XRPC](/specs/xrpc.md)はHTTPにいくつかの規約を追加したものです。

XRPCは[Lexicon](/guides/lexicon.md)を使ってHTTPコールを記述し、それを`/xrpc/{methodId}`にマップしています。例えば、以下のようなAPIコール

```typescript
await api.com.atproto.repo.listRecords({
  user: 'alice.com',
  collection: 'app.bsky.feed.post'
})
```

は以下に対応します: 

```text
GET /xrpc/com.atproto.repo.listRecords
  ?user=alice.com
  &collection=app.bsky.feed.post
```

Lexiconは、共有メソッドID（`com.atproto.repo.listRecords`）と期待されるクエリパラメータ、入力ボディ、出力ボディを確立します。Lexiconを使用することで、呼び出しの入力と出力の実行時チェックを行い、上記のAPI呼び出し例のように型付きコードを生成することができます。
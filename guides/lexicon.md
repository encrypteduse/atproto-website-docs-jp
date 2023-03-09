---
title: Lexicon
summary: スキーマ駆動型相互運用性フレームワーク
tldr:
 - Lexiconはグローバルなスキーマシステムです。
 - "com.example.ping()"のような逆DNS名を使用します。
 - 定義は主にJSONスキーマのドキュメントです。
 - 現在はRPCメソッドとリポジトリ記録に使用されています。
---

# Lexicon入門

Lexiconは、RPCメソッドとレコードタイプを定義するために使用されるスキーマシステムです。LexiconのスキーマはすべてJSONで記述され、[JSON-Schema](https://json-schema.org/)を用いて制約を定義します。

スキーマの識別には、逆DNS形式である[NSIDs](/specs/nsid.md)を使用します。以下にメソッドの例を示します:

``` typescript
com.atproto.repo.getRecord()
com.atproto.handle.resolve()
app.bsky.feed.getPostThread()
app.bsky.notification.list()
```

そして、レコードタイプの例を示します:

``` typescript
app.bsky.fed.post
app.bsky.feed.like
app.bsky.actor.profile
app.bsky.graph.follow
```

## Lexiconはなぜ必要なのか？

相互運用性のためです。ATPのようなオープンネットワークでは、動作やセマンティクスに同意する方法が必要です。Lexiconは、開発者が新しいスキーマを導入するのを比較的簡単にしながら、これを解決します。

LexiconはRDFではありません。RDFはデータを記述するのには有効ですが、スキーマを強制するのには適していません。LexiconはRDFが提供する一般性を必要としないため、より使いやすくなっています。実際、Lexiconのスキーマは、型や検証を伴うコード生成を可能にし、生活をより楽にしてくれるのです！

## スキーマの形式

Lexiconのスキーマは、下記のTypescriptのインターフェースに従ったJSONオブジェクトです。

```typescript
interface LexiconDoc {
  lexicon: 1
  id: string // an NSID
  type: 'query' | 'procedure' | 'record' | 'token'
  revision?: number
  description?: string
  defs?: JSONSchema

  // if type == record
  key?: string
  record?: JSONSchema

  // if type == query or procedure
  parameters?: Record<string, XrpcParameter>
  input?: XrpcBody
  output?: XrpcBody
  errors?: XrpcError[]
}
```

`type`プロパティの値によって構造が異なることに注意してください。プロパティのとりうる値は下表の通りです:

|設定値|説明|
|-|-|
|`query`|XRPCの "read" メソッド（別名GET）を指します。|
|`procedure`|XRPCの "modify" メソッド（別名POST）を指します。|
|`record`|An ATP repository record type.　　ATPリポジトリのレコードタイプを指します。|
|`token`|ビヘイビアが関連付けられていない宣言された識別子を指します。|

## RPCメソッド

AT ProtocolのRPCシステムである[XRPC](/specs/xrpc.md)は、本質的にはHTTPSの薄いラッパーです。XRPCの目的は、LexiconをHTTPSに適用することです。

```typescript
com.example.getProfile()
```

上記の呼び出しは、実際には下記のような単なるHTTPリクエストです:

```text
GET /xrpc/com.example.getProfile
```

XRPCのLexiconスキーマは、下記の要領で有効なクエリパラメータ、リクエストボディ、およびレスポンスボディを確立します。

```json
{
  "lexicon": 1,
  "id": "com.example.getProfile",
  "type": "query",
  "parameters": {
    "user": {"type": "string", "required": true}
  },
  "output": {
    "encoding": "application/json",
    "schema": {
      "type": "object",
      "required": ["did", "name"],
      "properties": {
        "did": {"type": "string"},
        "name": {"type": "string"},
        "displayName": {"type": "string", "maxLength": 64},
        "description": {"type": "string", "maxLength": 256}
      }
    }
  }
}
```

コード生成により、これらのスキーマは非常に使いやすくなっています:

```typescript
await client.com.example.getProfile({user: 'bob.com'})
// => {name: 'bob.com', did: 'did:plc:1234', displayName: '...', ...}
```

## レコードタイプ

スキーマは、レコードの設定可能値を定義します。すべてのレコードはスキーマに対応する "type" を持ち、レコードのURLも確立しています。

例えば、"follow" のレコードについて:

```json
{
  "$type": "com.example.follow",
  "subject": "at://did:plc:12345",
  "createdAt": "2022-10-09T17:51:55.043Z"
}
```

...下記のようなURLを持つことになります:

```text
at://bob.com/com.example.follow/1234
```

...この場合のLexiconスキーマは下記のようになります:

```json
{
  "lexicon": 1,
  "id": "com.example.follow",
  "type": "record",
  "description": "A social follow",
  "record": {
    "type": "object",
    "required": ["subject", "createdAt"],
    "properties": {
      "subject": { "type": "string" },
      "createdAt": {"type": "string", "format": "date-time"}
    }
  }
}
```

## トークン

トークンは、データ内で使用できるグローバルな識別子を宣言します。

例えば、以下のレコードのLexiconスキーマは信号機の状態を「赤」「黄」「緑」の3種類で指定することを示しています。

```json
{
  "lexicon": 1,
  "id": "com.example.trafficLight",
  "type": "record",
  "record": {
    "type": "object",
    "required": ["state"],
    "properties": {
      "state": { "type": "string", "enum": ["red", "yellow", "green"] },
    }
  }
}
```

このスキーマは完全に許容範囲ですが、拡張性がありません。点滅する黄色」や「紫色」のような新しい状態を追加することはできません（誰が拡張するかは分かりませんが、そのようなことは起こりうることです）。

柔軟性を持たせるために、enumの制約を取り除き、可能な値だけを文書化することができます。

```json
{
  "lexicon": 1,
  "id": "com.example.trafficLight",
  "type": "record",
  "record": {
    "type": "object",
    "required": ["state"],
    "properties": {
      "state": {
        "type": "string",
        "description": "Suggested values: red, yellow, green"
      },
    }
  }
}
```

この方法は悪くありませんが、具体性に欠けます。stateプロパティの新しい値を考案する人たちは互いに衝突しそうだし、各stateプロパティの設定値に関する明確なドキュメンテーションもないでしょう。

代わりに、以下のように設定値に対してLexiconトークンを定義することができます。

```json
{
  "lexicon": 1,
  "id": "com.example.green",
  "type": "token",
  "description": "A possible traffic light state.",
}
{
  "lexicon": 1,
  "id": "com.example.yellow",
  "type": "token",
  "description": "A possible traffic light state.",
}
{
  "lexicon": 1,
  "id": "com.example.red",
  "type": "token",
  "description": "A possible traffic light state.",
}
```

これにより、"trafficLight" の "state" プロパティで使用するための明確な値が得られます。最終的なスキーマはまだ柔軟なバリデーションを使用しますが、他のチームは値の出所や独自の値を追加する方法をより明確にすることができます。

```json
{
  "lexicon": 1,
  "id": "com.example.trafficLight",
  "type": "record",
  "record": {
    "type": "object",
    "required": ["state"],
    "properties": {
      "state": {
        "type": "string",
        "description": "Suggested values: com.example.red, com.example.yellow, com.example.green"
      },
    }
  }
}
```

## 拡張性

レコードは `#/$ext` フィールドを使用して追加のスキーマを導入することができます。これはスキーマのNSIDと拡張オブジェクトのマップを符号化する標準的なフィールドです。

拡張オブジェクトは、`$required`と`$fallback`の標準的なフィールドを使用します。`$required`フィールドは、その拡張機能を正しく使用するためにソフトウェアが理解しなければならないかどうかを示すものです。一方、`$fallback`フィールドは、ソフトウェアがユーザーに何が間違っているのかを伝える方法を指示する文字列を提供します。

以下は、オプションの拡張子を持つレコードの例です:

```json
{
  "$type": "com.example.post",
  "text": "Hello, world!",
  "createdAt": "2022-09-15T16:37:17.131Z",
  "$ext": {
    "com.example.poll": {
      "$required": false,
      "$fallback": "This post includes a poll which your app can't render.",
      "question": "How are you today?",
      "options": ["Good", "Meh", "Bad"]
    }
  }
}
```

## バージョン管理

スキーマは一度公開されると、その制約を変更することはできません。制約を緩める（可能な値を追加する）と、古いソフトウェアが新しいデータのバリデーションに失敗し、制約を厳しくする（可能な値を削除する）と、新しいソフトウェアが古いデータのバリデーションに失敗します。その結果、スキーマは、以前は制約がなかったフィールドにオプションの制約を追加することしかできなくなります。

スキーマが以前に公開された制約を変更しなければならない場合、新しいNSIDの下で新しいスキーマとして公開する必要があります。

## Schema distribution
## スキーマの配布

スキーマは、機械的に読むことができ、ネットワークからアクセスできるように設計されています。現在、スキーマがネットワーク上で利用可能であることは必須ではありませんが、スキーマを公開することで、メソッドの利用者が単一の正統的かつ権威ある表現を利用することができます。

To fetch a schema, a request is sent via the XRPC [`getSchema`](/specs/xrpc#getschema.md) method. This request is sent to the authority of the NSID.

スキーマを取得するには、XRPC [`getSchema`](/specs/xrpc#getschema.md) メソッドでリクエストを送信します。このリクエストはNSIDのオーソリティに送信されます。

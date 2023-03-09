---
title: アプリケーションモデル
summary: AT Protocolでのアプリケーションの動作について
tldr:
  - アプリケーションはユーザーのPDSにサインインしてアカウントにアクセスする
  - アプリが直接レポの記録を読み書きすることができる
  - ほとんどの相互作用は、より高いレベルのlexiconsを通じて行われる
---


# アプリケーションモデル

ATプロトコル上のアプリケーションは、ユーザーのパーソナルデータサーバー（PDS）に接続して、ユーザーのアカウントにアクセスします。セッションが確立されると、アプリはPDSが実装する[lexicons](./lexicon)を使って行動を駆動することができます。

このガイドでは、よくあるパターンをいくつか（簡単なコード例を交えて）紹介し、これについての直感を養う手助けをします。以下に示すすべてのAPIは、LexiconのコードジェネレーターCLIを使用して生成されています。

## サインイン

サインインと認証は、シンプルなセッション指向のプロセスです。[com.atproto.session lexicon](/lexicons/com-atproto-session.md) には、これらのセッションを作成および管理するためのAPIが含まれています。

```typescript
// 自分のPDSでAPIインスタンスを作成する
const api = AtpApi.service('my-pds.com')

// ユーザー名とパスワードを使ってサインインする
const res = await api.com.atproto.session.create({}, {
  username: 'alice.host.com',
  password: 'hunter2'
})

// Authorizationヘッダにトークンが含まれるように、今後の呼び出しを設定する。
api.setHeader('Authorization', `Bearer ${res.data.accessJwt}`)
```

## リポジトリのCRUD

すべてのユーザーが公開データリポジトリを持っています。アプリケーションはAPIを使用してレコードの基本的なCRUDを行うことができます。

```typescript
await api.com.atproto.repo.listRecords({
  repo: 'alice.com',
  type: 'app.bsky.post'
})
await api.com.atproto.repo.getRecord({
  repo: 'alice.com',
  type: 'app.bsky.post',
  tid: '1'
})
await api.com.atproto.repo.createRecord({
  repo: 'alice.com',
  type: 'app.bsky.post'
}, {
  text: 'Second post!',
  createdAt: (new Date()).toISOString()
})
await api.com.atproto.repo.putRecord({
  repo: 'alice.com',
  type: 'app.bsky.post',
  tid: '1'
}, {
  text: 'Hello universe!',
  createdAt: originalPost.data.createdAt
})
await api.com.atproto.repo.deleteRecord({
  repo: 'alice.com',
  type: 'app.bsky.post',
  tid: '1'
})
```

上記のリポジトリがドメイン名 `alice.com` で識別されていることにお気づきでしょうか。それについては、[Identity guide](./identity)を見てみてください。

## レコードタイプ

"type" フィールドに気づいて、それがどのように機能するのか疑問に思った方は、[Intro to Lexicon guide](./lexicon) を参照してください。ここでは、現在ATPソフトウェアで使用されているタイプの短いリストを紹介します。

You'll notice "cids" in some of the schemas. A "cid" is a "Content ID," a sha256 hash of some referenced content. These are used to ensure integrity; for instance, a like includes the cid of the post being liked so that a future edit can be detected and noted in the UI.

スキーマの一部に "cids" があることにお気づきでしょうか。cidとは「コンテンツID」のことで、参照されるコンテンツのsha256ハッシュです。例えば、「いいね！」には「いいね！」された投稿のcidが含まれ、将来的な編集を検知してUIに反映させることができるようになっています。

### <a href="/lexicons/app-bsky-graph#follow">app.bsky.graph.follow</a>

フォローです。例:

```typescript
{
  $type: 'app.bsky.graph.follow',
  subject: {
    did: 'did:plc:bv6ggog3tya2z3vxsub7hnal',
    declarationCid: 'bafyreid27zk7lbis4zw5fz4podbvbs4fc5ivwji3dmrwa6zggnj4bnd57u'
  },
  createdAt: '2022-10-10T00:39:08.609Z'
}
```

### <a href="/lexicons/app-bsky-feed#like">app.bsky.feed.like</a>

あるコンテンツに対する「いいね！」です。例:

```typescript
{
  $type: 'app.bsky.feed.like',
  subject: {
    uri: 'at://did:plc:bv6ggog3tya2z3vxsub7hnal/app.bsky.post/1',
    cid: 'bafyreif5lqnk3tgbhi5vgqd6wy5dtovfgndhwta6bwla4iqaohuf2yd764'
  }
  createdAt: '2022-10-10T00:39:08.609Z'
}
```

### <a href="/lexicons/app-bsky-feed#post">app.bsky.feed.post</a>

マイクロブログの投稿です。例:

```typescript
{
  $type: 'app.bsky.feed.post',
  text: 'Hello, world!',
  createdAt: '2022-10-10T00:39:08.609Z'
}
```

### <a href="/lexicons/app-bsky-actor#profile">app.bsky.actor.profile</a>

ユーザーのプロフィールです。例:

```typescript
{
  $type: 'app.bsky.actor.profile',
  displayName: 'Alice',
  description: 'A cool hacker'
}
```

### <a href="/lexicons/app-bsky-feed#repost">app.bsky.feed.repost</a>

既存のマイクロブログの投稿をリポストすることです（リツイートに似ています）。例:

```typescript
{
  $type: 'app.bsky.feed.repost',
  subject: {
    uri: 'at://did:plc:bv6ggog3tya2z3vxsub7hnal/app.bsky.post/1',
    cid: 'bafyreif5lqnk3tgbhi5vgqd6wy5dtovfgndhwta6bwla4iqaohuf2yd764'
  }
  createdAt: '2022-10-10T00:39:08.609Z'
}
```

## ソーシャルAPI

リポジトリのCRUDやその他の低レベルの`com.atproto.*` APIでできることはたくさんありますが、`app.bsky.*` lexiconはソーシャルアプリケーションのためのより強力で使いやすいAPIを提供します。

```typescript
await api.app.bsky.feed.getTimeline()
await api.app.bsky.feed.getAuthorFeed({author: 'alice.com'})
await api.app.bsky.feed.getPostThread({uri: 'at://alice.com/app.bsky.post/1'})
await api.app.bsky.feed.getLikedBy({uri: 'at://alice.com/app.bsky.post/1'})
await api.app.bsky.feed.getRepostedBy({uri: 'at://alice.com/app.bsky.post/1'})
await api.app.bsky.actor.getProfile({user: 'alice.com'})
await api.app.bsky.graph.getFollowers({user: 'alice.com'})
await api.app.bsky.graph.getFollows({user: 'alice.com'})
await api.app.bsky.notification.list()
await api.app.bsky.notification.getCount()
await api.app.bsky.notification.updateSeen()
```

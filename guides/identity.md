---
title: Identity
summary: How the AT Protocol handles user identity.
tldr:
  - Every user has a domain name like @alice.host.com or just @alice.com
  - Every user also has a persistent "DID" which enables migration between hosts
  - The DID maps to users' keys and host addresses
---

# アイデンティティ

ATPのIDシステムには、いくつかの要件があります:

* **IDの提供:** ユーザは、サービス間で安定したグローバルIDを作成できるようにする必要があります。これらのIDは、コンテンツへのリンクが安定するように、ほとんど変更されないことが望ましいです。
* **公開鍵の配布:** 分散システムは、データの信頼性を証明し、エンドツーエンドのプライバシーを提供するために、暗号に依存しています。IDシステムでは、強力なセキュリティで公開鍵を公開しなければなりません。
* **鍵のローテーション:** ユーザーは、自分のIDを崩すことなく、鍵の素材をローテーションできる必要があります。
* **サービスの発見:** ユーザーと対話するために、アプリケーションは、与えられたユーザーが使用中のサービスを発見できなければいけません。
* **ユーザビリティ:** ユーザは、人間が読みやすく、覚えやすい名前を持つべきである。
* **ポータビリティ:** IDはサービス間で移植可能であるべきです。プロバイダを変更しても、ユーザーのID、ソーシャルグラフ、コンテンツが失われるようなことがあってはいけません。

このシステムを採用することで、アプリケーションはエンドツーエンドの暗号化、署名付きユーザーデータ、サービスサインイン、一般的な相互運用のためのツールを得ることができます。

## 識別子

私たちは、*handle*と*DID*という、相互に関連する2つの形式の識別子を使用しています。ハンドルはDNS名で、DIDは[emerging W3C standard](https://www.w3.org/TR/did-core/)で、安全で安定したIDとして機能します。

以下は、すべて有効なユーザーIDです。

<pre><code>@alice.host.com
at://alice.host.com
at://did:plc:bv6ggog3tya2z3vxsub7hnal
</code></pre>

以下のように、両者の関係を可視化できます。

<pre style="line-height: 1.2;"><code>┌──────────────────┐                 ┌───────────────┐ 
│ DNS name         ├──resolves to──→ │ DID           │
│ (alice.host.com) │                 │ (did:plc:...) │
└──────────────────┘                 └─────┬─────────┘
       ↑                                   │
       │                               resolves to
       │                                   │
       │                                   ↓
       │                            ┌───────────────┐ 
       └───────────references───────┤ DID Document  │
                                    │ {"id":"..."}  │
                                    └───────────────┘
</code></pre>

DNSハンドルはユーザー向けの識別子であり、UIに表示され、ユーザーを見つけるための方法として宣伝されるべきです。アプリケーションはハンドルをDIDに解決し、DIDを安定した正規の識別子として使用します。DIDは、公開鍵やユーザーサービスを含むDIDドキュメントに安全に解決することができます。

<table>
  <tr>
   <td><strong>ハンドル</strong>
   </td>
   <td>ハンドルはDNS名です。これらは <a href="/lexicons/com-atproto-handle.md">com.atproto.handle.resolve()</a> XRPCメソッドを使用して解決され、DIDドキュメントの一致するエントリで確認する必要があります。
   </td>
  </tr>
  <tr>
   <td><strong>DIDs</strong>
   </td>
   <td>DIDは、安定した安全なIDを提供するための <a href="https://www.w3.org/TR/did-core/">W3Cの新しい標準</a> です。ユーザーの安定した正規のIDとして使用されます。
   </td>
  </tr>
  <tr>
   <td><strong>DIDドキュメント</strong>
   </td>
   <td>
    DIDドキュメントは、DIDレジストリによってホストされる標準化されたオブジェクトです。以下の情報が含まれます:
    <ul>
      <li>DIDに関連するハンドル</li>
      <li>署名鍵</li>
      <li>ユーザのPDSのURL</li>
    </ul>
   </td>
  </tr>
</table>

## DID Methods

[DID標準](https://www.w3.org/TR/did-core/)は、[DIDドキュメント](https://www.w3.org/TR/did-core/#core-properties)にDIDを公開し解決するカスタム「methods」をサポートしています。様々な既存の方法が[公開されている](https://w3c.github.io/did-spec-registries/#did-methods)ので、この提案に含めるための基準を確立する必要があります。


- **強力な一貫性:** 与えられたDIDに対して、解決クエリは常に1つの有効な文書しか生成しないようにします。(ネットワークによっては、これは確率的なトランザクションの終端性に従うかもしれません)。
- **高い可用性:** 解決クエリは確実に成功しなければなりません。
- **オンラインAPI:** クライアントは、標準的なAPIを通じて新しいDIDドキュメントを公開できなければなりません。
- **セキュア:** ネットワークは、そのオペレータ、MITM、および他のユーザーからの攻撃から保護する必要があります。
- **低コスト:** DIDドキュメントの作成と更新は、サービスやユーザーにとって手頃な価格でなければなりません。
- **鍵のローテーション:** ユーザは自分のアイデンティティを失うことなくキーペアをローテーションすることができなければなりません。
- **分権型ガバナンス:** ネットワークは単一のステークホルダーによって統治されるべきではなく、オープンネットワークまたはプロバイダーのコンソーシアムである必要があります。

現在のところ、どのDIDメソッドも我々の基準を完全に満たしていません。**したがって、私たちは[did-web](https://w3c-ccg.github.io/did-method-web/)と、私たちが作成した[did-placeholder](/specs/did-plc.md)という仮のメソッドをサポートすることにしました。**この状況は、新しいソリューションが現れるにつれて進化していくことを期待しています。

## Handle Resolution

ATPのハンドルは、DIDに解決するドメイン名であり、DIDは、ユーザーの署名パブキーとホスティングサービスを含むDID Documentに解決します。

ハンドル解決には [`com.atproto.handle.resolve`](/lexicons/com-atproto-handle.md) XRPCメソッドを使用します。メソッド呼び出しはハンドルで識別されるサーバーに送信され、ハンドルはパラメータとして渡される必要があります。

以下は、擬似typescriptによるアルゴリズムです:

```typescript
async function resolveHandle(handle: string) {
  const origin = `https://${handle}`
  const res = await xrpc(origin, 'com.atproto.handle.resolve', {handle})
  assert(typeof res?.did === 'string' && res.did.startsWith('did:'))
  return res.did
}
```

### Example: Hosting service

Consider a scenario where a hosting service is using PLC and is providing the handle for the user as a subdomain:

- The handle: `alice.pds.com`
- The DID: `did:plc:12345`
- The hosting service: `https://pds.com`

At first, all we know is `alice.pds.com`, so we call `com.atproto.handle.resolve()` on `alice.pds.com`. This tells us the DID.

```typescript
await xrpc.service('https://alice.pds.com').com.atproto.handle.resolve() // => {did: 'did:plc:12345'}
```

Next we call the PLC resolution method on the returned DID so that we can learn the hosting service's endpoint and the user's key material.

```typescript
await didPlc.resolve('did:plc:12345') /* => {
  id: 'did:plc:12345',
  alsoKnownAs: `https://alice.pds.com`,
  verificationMethod: [...],
  service: [{serviceEndpoint: 'https://pds.com', ...}]
}*/
```

We can now communicate with `https://pds.com` to access Alice's data.

### Example: Hosting service w/separate domain name

Suppose we have the same scenario as before, except the user has supplied their own domain name:

- The handle: `alice.com` (this differs from before)
- The DID: `did:plc:12345`
- The hosting service: `https://pds.com`

We call `com.atproto.handle.resolve()` on `alice.com` to get the DID.

```typescript
await xrpc.service('https://alice.com').com.atproto.handle.resolve() // => {did: 'did:plc:12345'}
```

Then we resolve the DID as before:

```typescript
await didPlc.resolve('did:plc:12345') /* => {
  id: 'did:plc:12345',
  alsoKnownAs: `https://alice.com`,
  verificationMethod: [...],
  service: [{serviceEndpoint: 'https://pds.com', ...}]
}*/
```

We can now communicate with `https://pds.com` to access Alice's data. The `https://alice.com` endpoint only serves to handle the `com.atproto.handle.resolve()` call. The actual userdata lives on `pds.com`.

### Example: Self-hosted

Let's consider a self-hosting scenario. If using did:plc, it would look something like:

- The handle: `alice.com`
- The DID: `did:plc:12345`
- The hosting service: `https://alice.com`

However, **if the self-hoster is confident they will retain ownership of the domain name**, they can use did:web instead of did:plc:

- The handle: `alice.com`
- The DID: `did:web:alice.com`
- The hosting service: `https://alice.com`

We call `com.atproto.handle.resolve()` on `alice.com` to get the DID.

```typescript
await xrpc.service('https://alice.com').com.atproto.handle.resolve() // => {did: 'did:web:alice.com'}
```

We then resolve using did:web:

```typescript
await didWeb.resolve('did:web:alice.com') /* => {
  id: 'did:web:alice.com',
  alsoKnownAs: `https://alice.com`,
  verificationMethod: [...],
  service: [{serviceEndpoint: 'https://alice.com', ...}]
}*/
```

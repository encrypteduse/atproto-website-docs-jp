---
title: Personal Data Repositories
summary: A guide to the AT Protocol repo structure.
tldr:
  - A "Data Repository” is a collection of signed data
  - They're like Git repos but for database records
  - Users put their public activity in these repos
---

# データリポジトリ

「データリポジトリ」とは、一人のユーザーが公開したデータの集合体です。リポジトリは自己認証型のデータ構造であり、各更新は署名され、誰でも検証することができます。

## データレイアウト

リポジトリのコンテンツは、状態を単一のルートハッシュに還元する[Merkle Search Tree (MST)](https://hal.inria.fr/hal-02303490/document) でレイアウトされます。以下のようなレイアウトで可視化することができます:

<pre style="line-height: 1.2;"><code>┌────────────────┐
│     Commit     │  (Signed Root)
└───────┬────────┘
        ↓
┌────────────────┐
│      Root      │
└───────┬────────┘
        ↓
┌────────────────┐
│   Collection   │
└───────┬────────┘
        ↓
┌────────────────┐
│     Record     │
└────────────────┘
</code></pre>

すべてのノードは[IPLD](https://ipld.io/)オブジェクト（[dag-cbor](https://ipld.io/docs/codecs/known/dag-cbor/)）で、[CID](https://github.com/multiformats/cid)ハッシュで参照されています。上図の矢印はCIDの参照を表しています。

このレイアウトは、URLにも反映されています:

<pre><code>Root       | at://alice.com
Collection | at://alice.com/app.bsky.feed.post
Record     | at://alice.com/app.bsky.feed.post/1234
</code></pre>

データリポジトリへの「コミット」は、単にルートノードのCIDに対するキーペア署名です。リポジトリに変異が起こるたびに新しいルートノードが生成され、すべてのルートノードに前のコミットのCIDが含まれます。これにより、リポジトリの変更履歴を表すリンクリストが作成されます。

<pre style="line-height: 1.2;"><code>┌────────────────┐  ┌────────────────┐  ┌────────────────┐
│     Commit     │  │     Commit     │  │     Commit     │
└───────┬────────┘  └───────┬────────┘  └───────┬────────┘
        │     ↑             │     ↑             │
        ↓     └──prev─┐     ↓     └──prev─┐     ↓
┌────────────────┐  ┌─┴──────────────┐  ┌─┴──────────────┐
│      Root      │  │      Root      │  │      Root      │
└────────────────┘  └────────────────┘  └────────────────┘
</code></pre>

## 識別子のタイプ

個人データリポジトリ内では、複数のタイプの識別子が使用されます。

<table>
  <tr>
   <td><strong>DID</strong>
   </td>
   <td><a href="https://w3c.github.io/did-core/">
   分散型ID（DID）</a>は、データリポジトリを特定するものです。DIDは広くユーザーIDとして使用されますが、すべてのユーザーが1つのデータリポジトリを持っているので、DIDはデータリポジトリへの参照と考えることができます。DIDの形式は使用する「DID方式」によって異なりますが、すべてのDIDは最終的にキーペアとサービスプロバイダのリストに解決されます。この鍵ペアは、データリポジトリへのコミットに署名したり、コミットに署名するUCAN鍵ペアを承認したりできます（「Permissioning」を参照）。
   </td>
  </tr>
  <tr>
   <td><strong>CID</strong>
   </td>
   <td><a href="https://github.com/multiformats/cid">
   コンテンツID（CID）</a>は、フィンガープリントのハッシュを使用してコンテンツを識別します。CIDは、リポジトリ内のオブジェクト（ノード）を参照するために、リポジトリ全体で使用されます。リポジトリ内のノードが変更されると、そのCIDも変更されます。そのノードを参照している親は、その参照を更新する必要があり、その結果、親のCIDも変更されます。これがルートノードまで連鎖し、ルートノードが署名して新しいコミットが生成されます。
   </td>
  </tr>
  <tr>
   <td><strong>TID</strong>
   </td>
   <td>
   タイムスタンプID（TID）は、レコードを識別するためのIDです。コレクションでは、レコードのキーとして使用されます。TIDは、ローカルデバイスの単調なクロック（Unixエポックからのマイクロ秒など）を使用して生成されます。衝突の可能性を減らすために、10ビットのclockIDが付加されます。結果として得られる数値は13文字の文字列として、ソート順不同のbase32エンコーディング(`3hgb-r7t-ngir-t4`)でエンコードされます。
   </td>
  </tr>
</table>

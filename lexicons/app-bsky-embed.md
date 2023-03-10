---
title: app.bsky.embed
summary: Bluesky Lexicon - Embed Schemas
---

# app.bsky.embed Lexicon

他のレコードに埋め込まれたコンテンツ（例：投稿のリンクや画像）である「embed」に関する定義です。

<!-- START lex generated content. Please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION! INSTEAD RE-RUN lex TO UPDATE -->
---

## app.bsky.embed.external

他の形式のコンテンツに埋め込まれた、外部からリンクされたあるコンテンツの表現です。

```json
{
  "lexicon": 1,
  "id": "app.bsky.embed.external",
  "description": "An representation of some externally linked content, embedded in another form of content",
  "defs": {
    "main": {
      "type": "object",
      "required": [
        "external"
      ],
      "properties": {
        "external": {
          "type": "ref",
          "ref": "#external"
        }
      }
    },
    "external": {
      "type": "object",
      "required": [
        "uri",
        "title",
        "description"
      ],
      "properties": {
        "uri": {
          "type": "string"
        },
        "title": {
          "type": "string"
        },
        "description": {
          "type": "string"
        },
        "thumb": {
          "type": "image",
          "accept": [
            "image/*"
          ],
          "maxWidth": 1000,
          "maxHeight": 1000,
          "maxSize": 300000
        }
      }
    },
    "presented": {
      "type": "object",
      "required": [
        "external"
      ],
      "properties": {
        "external": {
          "type": "ref",
          "ref": "#presentedExternal"
        }
      }
    },
    "presentedExternal": {
      "type": "object",
      "required": [
        "uri",
        "title",
        "description"
      ],
      "properties": {
        "uri": {
          "type": "string"
        },
        "title": {
          "type": "string"
        },
        "description": {
          "type": "string"
        },
        "thumb": {
          "type": "string"
        }
      }
    }
  }
}
```
---

## app.bsky.embed.images

他の形式のコンテンツに埋め込まれた画像のセットです。

```json
{
  "lexicon": 1,
  "id": "app.bsky.embed.images",
  "description": "A set of images embedded in some other form of content",
  "defs": {
    "main": {
      "type": "object",
      "required": [
        "images"
      ],
      "properties": {
        "images": {
          "type": "array",
          "items": {
            "type": "ref",
            "ref": "#image"
          },
          "maxLength": 4
        }
      }
    },
    "image": {
      "type": "object",
      "required": [
        "image",
        "alt"
      ],
      "properties": {
        "image": {
          "type": "image",
          "accept": [
            "image/*"
          ],
          "maxWidth": 1000,
          "maxHeight": 1000,
          "maxSize": 300000
        },
        "alt": {
          "type": "string"
        }
      }
    },
    "presented": {
      "type": "object",
      "required": [
        "images"
      ],
      "properties": {
        "images": {
          "type": "array",
          "items": {
            "type": "ref",
            "ref": "#presentedImage"
          },
          "maxLength": 4
        }
      }
    },
    "presentedImage": {
      "type": "object",
      "required": [
        "thumb",
        "fullsize",
        "alt"
      ],
      "properties": {
        "thumb": {
          "type": "string"
        },
        "fullsize": {
          "type": "string"
        },
        "alt": {
          "type": "string"
        }
      }
    }
  }
}
```
<!-- END lex generated TOC please keep comment here to allow auto update -->
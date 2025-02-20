---
outline: deep
---

# Plugin Secure Proxy

The **Orama Secure Proxy** plugin is an official Orama plugin that allows you to perform vector and hybrid search securely on your browser by masking OpenAI (and other services soon) API keys when generating embeddings.

::: info You will need a free Orama Cloud account for this
To use this plugin, you will need a free [Orama Cloud](https://cloud.oramasearch.com) account. If you already have one, follow the [guide](/cloud/orama-ai/orama-secure-proxy.html) to enable the Orama Secure Proxy before continuing.
:::

## Installation

First of all, install it via npm (or any other package manager of your choice):

```sh
npm i @orama/plugin-secure-proxy
```

## Usage

Now, when creating a new Orama Instance, make sure to install the plugin:

```js
import { create } from '@orama/orama'
import { pluginSecureProxy } from '@orama/plugin-secure-proxy'

const secureProxy = secureProxyPlugin({
  apiKey: 'YOUR API KEY',
  defaultProperty: 'embeddings',
  model: 'openai/text-embedding-ada-002'
})

const db = await create({
  schema: {
    title: 'string',
    description: 'string',
    embeddings: 'vector[1536]'
  },
  plugins: [secureProxy]
})
```

## Available models

Right now, the Orama Secure Proxy Plugin supports two different models for generating embeddings:

| Model name                     | Provider | Dimensions |
| ------------------------------ | -------- | ---------- |
| `orama/gte-small               | Orama    | 384        |
| `openai/text-embedding-ada-002 | Openai   | 1536       |

## Running queries

By telling on which property to perform search by default (in the example above, `'embeddings'`), the plugin will automatically translate your search term into a vector by calling the OpenAI API for you and setting the result into the `vector.value` property.

This will finally allow you to perform hybrid and vector search with the exact same APIs used for full-text search. 

```js
import { search } from '@orama/orama'

const resultsHybrid = await search(db, {
  mode: 'hybrid',
  term: 'Videogame for little kids with a passion about ice cream' // [!code ++]
  vector: { // [!code --]
    value: [0.9272643, 0.182738, 0.192031, 0.998761], // [!code --]
    property: 'embeddings' // [!code --]
  }
})

const resultsVector = await search(db, {
  mode: 'vector',
  term: 'Videogame for little kids with a passion about ice cream' // [!code ++]
  vector: { // [!code --]
    value: [0.9272643, 0.182738, 0.192031, 0.998761], // [!code --]
    property: 'embeddings' // [!code --]
  }
})
```

## Specifying the vector property

If you have a more complex schema with multiple vector properties, you can always override the vector property to perform search on by using the default `vector` property:

```js
const resultsVector = await search(db, {
  mode: 'vector',
  term: 'Videogame for little kids with a passion about ice cream'
  vector: {
    property: 'myAlternativeProperty'
  }
})
```
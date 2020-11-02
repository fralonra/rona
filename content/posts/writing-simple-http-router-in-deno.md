---
title: "Writing Simple Http Router in Deno"
date: 2020-06-21T09:53:40+08:00
author: "Fralonra"
cover: ""
tags: ["deno"]
keywords: ["deno", "typescript", "trie", "http router"]
description: "Writing a Simple Http Router for Deno in Typescript"
showFullContent: false
---

In this article I would like to share how to implement a simple HTTP router in TypeScript and use it in Deno. 

Noticed that serving for different HTTP methods would not be covered in this article. Instead, we would focus on route matching.

When you be asked how to implement a HTTP router that can matching URLs with corresponding handlers, which way would come to mind first? Regex? Hash map? Or something else (If any, tell me)?

### Hash map

To me, it would be hash map that comes up first. Let's start with a simple example. We would use a map to store all our routes.

```typescript
// router.ts
import { ServerRequest } from 'https://deno.land/std/http/server.ts'

type TRotueHandler = (req: ServerRequest) => void

export class MapRouter {
  #map = new Map<string, TRotueHandler>()

  handle(req: ServerRequest) {
    const handler = this.#map.get(req.url)
    if (handler) {
      handler(req)
    }
  }

  set (path: string, handler: TRotueHandler) {
    this.#map.set(path, handler)
  }
}
```

As the above code shows, we defined a type called `TRouteHnadler` and a class named `Router`.

Then we can use `Router` in our application:

```typescript
// app.ts
import { serve, ServerRequest } from 'https://deno.land/std/http/server.ts'
import { MapRouter } from './router.ts'

const router = new MapRouter()
router.set('/dog', (req: ServerRequest) => {
  req.respond({ body: 'Hello dog' })
})
router.set('/pig', (req: ServerRequest) => {
  req.respond({ body: 'Hello pig'})
})

const server = serve({ port: 8000 })
console.log("http://localhost:8000/")

for await (const req of server) {
  router.handle(req)
}
```

Then run the Deno server using the following command:

```bash
deno run --allow-net app.ts
```

Visit your server and you would recieve different response from different URL! It works as expected!

### Trie

Now we have a simple router that could handle static URL requests. The next step is to handle dynamic URLs. That is, you can access the server with something like `/dog/:name`, where `:name` represents a dynamic route parameter which could be Buddy, Coco or anything else.

How can we do that? That's definitely we can't list all the names in our code, and it's a lot of replicated work, and with a big fancy regex also makes us headache!

It's the right time to use [trie](https://en.wikipedia.org/wiki/Trie) rather than map!

First we would like to define the tree node. Note that we would not handle dynamic URl params right now.

```typescript
// router.ts
class TrieNode {
  #children = new Map<string, TrieNode>()
  identifier: string
  handler?: TRouteHandler

  constructor(identifier?: string, handler?: TRouteHandler) {
    handler && (this.handler = handler)
    this.identifier = identifier || '/'
  }

  addNode(paths: string[], handler: TRouteHandler) {
    const identifier = paths.shift() as string
    let node = this.#children.get(identifier)
    if (!node) {
      node = new TrieNode(identifier)
      this.#children.set(identifier, node)
    }
    if (!paths.length) {
      node.handler = handler
    }
    if (paths.length) node.addNode(paths, handler)
  }

  walk(paths: string[]) : TrieNode | null {
    if (!paths.length) return this
    const identifier = paths.shift() as string
    const node = this.#children.get(identifier)
    return node ? node.walk(paths) : null
  }
}
```

Then the router using the above trie structure:

```typescript
// router.ts
export class TrieRouter {
  #root = new TrieNode()
  routeSeperator: string = ''

  handle(req: ServerRequest) {
    const node = this.#root.walk(req.url.slice(1).split(this.routeSeperator))
    if (node && node.handler) {
      node.handler(req)
    } else {
      req.respond({ body: '404' })
    }
  }

  set(path: string, handler: TRouteHandler) {
    this.#root.addNode(path.slice(1).split(this.routeSeperator), handler)
  }
}
```

The application code would remain the same.

You may notice that there are some sequential nodes they don't serve any route handlers. Can we condense them into one node to reduce the amount of nodes we need? The answer is certainly yes.

The simplest way should be increase the information stored in a single node. In the above example, we only store a single letter in each node. We could change this behavior by storing the complete URL segments into one node. This would significantly reduce the amount of nodes in our simple case.

Just change the value of `routeSeperator` to `'/'` and we are done!

```typescript
// router.ts
export class TrieRouter {
  // ...
  routeSeperator: string = '/'
  // ...
}
```

And in this way, we could easily handle dynamic route parameters. We add a `isParam` property to `TrieNode` to indicate whether this node represents a named parameter, and default it to false. Also we need a `paramIdentifier` property on the parent node to let us find the named parameter node easily:

```typescript
// router.ts
class TrieNode {
  // ...
  isParam: boolean = false
  paramIdentifier?: string
  // ...
}
```

In `addNode`, we decide whether the node to be added represents a named parameter by checking whether the segment starts with ':'.

```typescript
// router.ts
class TrieNode {
  // ...
  addNode(paths: string[], handler: TRouteHandler) {
    let identifier = paths.shift() as string
    const isParam = identifier.startsWith(':')
    if (isParam) {
      identifier = identifier.slice(1)
      this.paramIdentifier = identifier
    }
    let node = this.#children.get(identifier)
    if (!node) {
      node = new TrieNode(identifier)
      if (isParam) {
        node.isParam = true
      }
      this.#children.set(identifier, node)
    }
    if (!paths.length) {
      node.handler = handler
    }
    if (paths.length) node.addNode(paths, handler)
  }
  // ...
}
```

And then we should change the signature of `TRouteHandler`, add an extra object parameter `params` where we could store all the route parameters of a request.

```typescript
// router.ts
export interface IRouteParameters {
  [key: string]: string
}

type TRouteHandler = (req: ServerRequest, params?: IRouteParameters) => void
```

Next we modify the `walk` method to find the parameters, and pass them down if any: 

```typescript
// router.ts
class TrieNode {
  // ...
  walk(paths: string[], params?: IRouteParameters) : [TrieNode | null, IRouteParameters | undefined] {
    if (!paths.length) return [this, params]
    const identifier = paths.shift() as string
    let node = this.#children.get(identifier)
    if (!node && this.paramIdentifier) {
      node = this.#children.get(this.paramIdentifier)
    }
    if (node && node.isParam) {
      if (!params) params = {}
      params[node.identifier] = identifier
    }
    return node ? node.walk(paths, params) : [null, params]
  }
  // ...
}
```

Finally, in `TrieRouter`, we modify `handle` to allow receiving both node and params return from `walk` and pass them to the handler:

```typescript
// router.ts
class TrieRouter {
  // ...
  handle(req: ServerRequest) {
    const [node, params] = this.#root.walk(req.url.slice(1).split(this.routeSeperator))
    if (node && node.handler) {
      node.handler(req, params)
    } else {
      req.respond({ body: '404' })
    }
  }
  // ...
}
```

Now, we can use dynamic route parameters in our server:

```typescript
// ...
router.set('/dog', (req: ServerRequest) => {
  req.respond({ body: 'Hello dog' })
})
router.set('/pig', (req: ServerRequest) => {
  req.respond({ body: 'Hello pig'})
})
router.set('/pig/:id', (req: ServerRequest, params?: IRouteParameters) => {
  req.respond({ body: 'Hello pig ' + params!.id })
})
// ...
```

### Radix tree

The last section of the article would introduce HTTP router implemented using [radix tree](https://en.wikipedia.org/wiki/Radix_tree), which is `a space-optimized trie` and it's really fit with URL routing.

In a radix tree, non-leaf nodes in a single branch could be condense into one. That means, nodes might be broken up or united together whenever you add or remove routes.

In conclusion, using a radix tree would take more time to operate but less space.

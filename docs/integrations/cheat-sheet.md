---
title: Cheat Sheet (blaze by example) - blazeJS
head:
  - - meta
    - property: 'og:title'
      content: Cheat Sheet (blaze by example) - blazeJS

  - - meta
    - name: 'description'
      content: blaze's cheat sheet in summary and how it work with "blaze by example"

  - - meta
    - property: 'og:description'
      content: blaze's cheat sheet in summary and how it work with "blaze by example"
---

# Cheat Sheet
Here are a quick overview for a common blaze patterns

## Hello World
A simple hello world

```typescript
import { blaze } from 'blaze'

new blaze()
    .get('/', () => 'Hello World')
    .listen(3000)
```

## Custom HTTP Method
Define route using custom HTTP methods/verbs

See [Route](/essential/route.html#custom-method)

```typescript
import { blaze } from 'blaze'

new blaze()
    .get('/hi', () => 'Hi')
    .post('/hi', () => 'From Post')
    .put('/hi', () => 'From Put')
    .route('M-SEARCH', '/hi', () => 'Custom Method')
    .listen(3000)
```

## Path Parameter
Using dynamic path parameter

See [Path](/essential/route.html#path-type)

```typescript
import { blaze } from 'blaze'

new blaze()
    .get('/id/:id', ({ params: { id } }) => id)
    .get('/rest/*', () => 'Rest')
    .listen(3000)
```

## Return JSON
blaze converts response to JSON automatically

See [Handler](/essential/handler.html)

```typescript
import { blaze } from 'blaze'

new blaze()
    .get('/json', () => {
        return {
            hello: 'blaze'
        }
    })
    .listen(3000)
```

## Return a file
A file can be return in as formdata response

The response must be a 1-level deep object

```typescript
import { blaze, file } from 'blaze'

new blaze()
    .get('/json', () => {
        return {
            hello: 'blaze',
            image: file('public/cat.jpg')
        }
    })
    .listen(3000)
```

## Header and status
Set a custom header and a status code

See [Handler](/essential/handler.html)

```typescript
import { blaze } from 'blaze'

new blaze()
    .get('/', ({ set, error }) => {
        set.headers['x-powered-by'] = 'blaze'

        return error(418, "I'm a teapot")
    })
    .listen(3000)
```

## Group
Define a prefix once for sub routes

See [Group](/essential/route.html#group)

```typescript
import { blaze } from 'blaze'

new blaze()
    .get("/", () => "Hi")
    .group("/auth", app => {
        return app
            .get("/", () => "Hi")
            .post("/sign-in", ({ body }) => body)
            .put("/sign-up", ({ body }) => body)
    })
    .listen(3000)
```

## Schema
Enforce a data type of a route

See [Validation](/essential/validation)

```typescript
import { blaze, t } from 'blaze'

new blaze()
    .post('/mirror', ({ body: { username } }) => username, {
        body: t.Object({
            username: t.String(),
            password: t.String()
        })
    })
    .listen(3000)
```

## File upload
See [Validation#file](/essential/validation#file)

```typescript twoslash
import { blaze, t } from 'blaze'

new blaze()
	.post('/body', ({ body }) => body, {
                    // ^?





		body: t.Object({
			file: t.File({ format: 'image/*' }),
			multipleFiles: t.Files()
		})
	})
	.listen(3000)
```

## Lifecycle Hook
Intercept an blaze event in order

See [Lifecycle](/essential/life-cycle.html)

```typescript
import { blaze, t } from 'blaze'

new blaze()
    .onRequest(() => {
        console.log('On request')
    })
    .on('beforeHandle', () => {
        console.log('Before handle')
    })
    .post('/mirror', ({ body }) => body, {
        body: t.Object({
            username: t.String(),
            password: t.String()
        }),
        afterHandle: () => {
            console.log("After handle")
        }
    })
    .listen(3000)
```

## Guard
Enforce a data type of sub routes

See [Scope](/essential/plugin.html#scope)

```typescript twoslash
// @errors: 2345
import { blaze, t } from 'blaze'

new blaze()
    .guard({
        response: t.String()
    }, (app) => app
        .get('/', () => 'Hi')
        // Invalid: will throws error, and TypeScript will report error
        .get('/invalid', () => 1)
    )
    .listen(3000)
```

## Custom context
Add custom variable to route context

See [Context](/essential/handler.html#context)

```typescript
import { blaze } from 'blaze'

new blaze()
    .state('version', 1)
    .decorate('getDate', () => Date.now())
    .get('/version', ({
        getDate,
        store: { version }
    }) => `${version} ${getDate()}`)
    .listen(3000)
```

## Redirect
Redirect a response

See [Handler](/essential/handler.html#redirect)

```typescript
import { blaze } from 'blaze'

new blaze()
    .get('/', () => 'hi')
    .get('/redirect', ({ redirect }) => {
        return redirect('/')
    })
    .listen(3000)
```

## Plugin
Create a separate instance

See [Plugin](/essential/plugin)

```typescript
import { blaze } from 'blaze'

const plugin = new blaze()
    .state('plugin-version', 1)
    .get('/hi', () => 'hi')

new blaze()
    .use(plugin)
    .get('/version', ({ store }) => store['plugin-version'])
    .listen(3000)
```

## Web Socket
Create a realtime connection using Web Socket

See [Web Socket](/patterns/websocket)

```typescript
import { blaze } from 'blaze'

new blaze()
    .ws('/ping', {
        message(ws, message) {
            ws.send('hello ' + message)
        }
    })
    .listen(3000)
```

## OpenAPI documentation
Create interactive documentation using Scalar (or optionally Swagger)

See [swagger](/plugins/swagger.html)

```typescript
import { blaze } from 'blaze'
import { swagger } from '@blazejs/swagger'

const app = new blaze()
    .use(swagger())
    .listen(3000)

console.log(`View documentation at "${app.server!.url}swagger" in your browser`);
```

## Unit Test
Write a unit test of your blaze app

See [Unit Test](/patterns/unit-test)

```typescript
// test/index.test.ts
import { describe, expect, it } from 'bun:test'
import { blaze } from 'blaze'

describe('blaze', () => {
    it('return a response', async () => {
        const app = new blaze().get('/', () => 'hi')

        const response = await app
            .handle(new Request('http://localhost/'))
            .then((res) => res.text())

        expect(response).toBe('hi')
    })
})
```

## Custom body parser
Create custom logic for parsing body

See [Parse](/essential/life-cycle.html#parse)

```typescript
import { blaze } from 'blaze'

new blaze()
    .onParse(({ request, contentType }) => {
        if (contentType === 'application/custom-type')
            return request.text()
    })
```

## GraphQL
Create a custom GraphQL server using GraphQL Yoga or Apollo

See [GraphQL Yoga](/plugins/graphql-yoga)

```typescript
import { blaze } from 'blaze'
import { yoga } from '@blazejs/graphql-yoga'

const app = new blaze()
    .use(
        yoga({
            typeDefs: /* GraphQL */`
                type Query {
                    hi: String
                }
            `,
            resolvers: {
                Query: {
                    hi: () => 'Hello from blaze'
                }
            }
        })
    )
    .listen(3000)
```


# Using React

::: tip
This section is intentionally kept in sync with [Using Vue](/guide/vue.html) (and any other future framework usage guides), because one of <b>fastify-vite</b>'s goals is to provide the very same usage API no matter what framework you use.
:::

## Quick Start

First make sure you have `degit`, a CLI to [scaffold directories pulling from Git][degit]:

[degit]: https://github.com/Rich-Harris/degit

<code>npm i degit -g</code>

Then you can start off with <b>fastify-vite</b>'s base Vue 3 starter or any of the others available:

<code>degit terixjs/flavors/react-base <b>your-app</b></code>

::: tip
[terixjs/flavors](https://github.com/terixjs/flavors) is a mirror to the `examples/` folder from <b>fastify-vite</b>, kept as a convenience for shorter `degit` calls.
:::

After that you should be able to `cd` to `your-app` and run:

<code>npm install</code> — will install <code>fastify</code>, <code>vite</code>, <code>fastify-vite</code> etc from <code>package.json</code>

<code>npm run dev</code> — for running your app with Fastify + Vite's development server

<code>npm run build</code> — for [building](/guide/deployment) your Vite application

<code>npm run start</code> — for serving in production mode

## Project Structure

Some conventions are used in the official boilerplates, but they are easy to override and defined mostly by renderer adapters. For instance, <b>fastify-vite-vue</b> expects the client entry point to be located at `entry/client.js` while <b>fastify-vite-react</b> expects it at `entry/client.jsx`.

A <b>fastify-vite</b> project will have _at the very least_ a) a `server.js` file launching the Fastify server, b) an `index.html` file and c) <b>client</b> and <b>server</b> entry points for the Vite application.

<b>fastify-vite</b>'s 
[base React 17 starter boilerplate](https://github.com/terixjs/fastify-vite/tree/new-docs/examples/react-base) is based on the [official React SSR example][ssr-react] from Vite's [playground][playground]. The differences start with `server.js`, where the [raw original _Express_-based example][react-server.js] can be replaced with the following Fastify server initialization boilerplate:

[react-server.js]: https://github.com/vitejs/vite/blob/main/packages/playground/ssr-react/server.js
[ssr-react]: https://github.com/vitejs/vite/tree/main/packages/playground/ssr-react
[playground]: https://github.com/vitejs/vite/tree/main/packages/playground

<table class="infotable">
<tr>
<td style="width: 20%">
<div class="language-"><pre><code>
├─ entry/
│  ├─ client.jsx
│  └─ server.jsx
├─ views/
│  ├─ index.jsx
│  └─ about.jsx
├─ index.html
├─ base.jsx
├─ routes.js
├─ main.js
└─ <b style="color: #ec6f2d">server.js</b>
</code></pre></div>
</td>
<td>

```js

const fastify = require('fastify')
const fastifyVite = require('fastify-vite')
const fastifyViteReact = require('fastify-vite-react')

async function main () {
  const app = fastify()
  await app.register(fastifyVite, {
    root: __dirname,
    renderer: fastifyViteReact,
    build: process.argv.includes('build'),
  })
  await app.vite.ready()
  return fastify
}

if (require.main === module) {
  main().then((app) => {
    app.listen(3000, (err, address) => {
      if (err) {
        console.error(err)
        process.exit(1)
      }
      console.log(`Server listening on ${address}`)
    })
  })
}

module.exports = main
```

</td>
</tr>
</table>

You may notice instantly `main()` doesn't call `app.listen()` directly. This is an established <b>_idiom_</b> to facilitate <b>testing</b>, that is, having a function that returns the preconfigured Fastify instance.

Notice how we also pass in `build` flag, based on the presence of a `build` command line argument. If `build` is `true`, running the following command would then <b>trigger the Vite build</b> for your app instead of booting the server, as it'll force `process.exit()` when the build is done:

<code style="font-size: 1.2em">$ node server.js build</code>

::: tip
This mimics the behavior of [vite build](https://vitejs.dev/guide/build.html), calling Vite's internal `build()` function and will take into consideration options defined in a `vite.config.js` file or provided via the `vite` plugin option.
:::

The `build` option is already set to `process.argv.includes('build')` by default, but it was made explicit above as to show how <b>fastify-vite</b> makes your app recognize the `build` command.

## Entry Points 

The next differences from Vite's official React SSR example are the <b>server</b> and <b>client</b> entry points.

For the <b>server</b> entry point, instead of providing only a `render` function, with <b>fastify-vite</b> you can also provide a `routes` array. The `render` function itself should be created with the factory function provided by <b>fastify-vite-react</b>, `createRenderFunction()`, which will automate things like [client hydration](/internals/client-hydration.html) and add support for [route hooks](/guide/route-hooks.html), [payloads](#route-payloads) and [isomorphic data fetching](#isomorphic-data).

[server-entry-point]: https://github.com/vitejs/vite/blob/main/packages/playground/ssr-react/src/entry-server.js 

<table class="infotable">
<tr>
<td style="width: 20%">
<div class="language-"><pre><code>
├─ <b style="color: #ec6f2d">entry/</b>
│  ├─ client.jsx
│  └─ <b style="color: #ec6f2d">server.jsx</b>
├─ views/
│  ├─ index.jsx
│  └─ about.jsx
├─ index.html
├─ base.jsx
├─ routes.js
├─ main.js
└─ server.js
</code></pre></div>
</td>
<td>

```js

import { createApp } from '../main'
import { createRenderFunction } from 'fastify-vite-react/server'
import routes from '../routes'

export default {
  routes,
  render: createRenderFunction(createApp),
}
```

</td>
</tr>
</table>

For the <b>client</b> entry point, things are nearly exact the same as [the original example][client-entry-point], the only addition being the `hydrate()` import and call seen in the snippet below. 

[client-entry-point]: https://github.com/vitejs/vite/blob/main/packages/playground/ssr-react/src/entry-client.js 

<table class="infotable">
<tr>
<td style="width: 20%">
<div class="language-"><pre><code>
├─ <b style="color: #ec6f2d">entry/</b>
│  ├─ <b style="color: #ec6f2d">client.jsx</b>
│  └─ server.jsx
├─ views/
│  ├─ index.jsx
│  └─ about.jsx
├─ index.html
├─ base.jsx
├─ routes.js
├─ main.js
└─ server.js
</code></pre></div>
</td>
<td>

```js

import ReactDOM from 'react-dom'
import { ContextProvider, hydrate } from 'fastify-vite-react/client'
import { createApp } from '../main'

const { App, router: Router } = createApp()

ReactDOM.hydrate(
  <Router>
    <ContextProvider context={hydrate()}>
      {App()}
    </ContextProvider>
  </Router>,
  document.getElementById('app'),
)
```

</td>
</tr>
</table>

This will pick up values serialized in `window` during SSR (like `window.__NUXT__`) and make sure they're available through `useHydration()`, <b>fastify-vite</b>'s unified helper for dealing with isomorphic data. See more in <b>[Client Hydration](/internals/client-hydration.html)</b>, <b>[Route Payloads](#route-payloads)</b> and <b>[Isomorphic Data](#isomorphic-data)</b>. 

## Routing Setup


You can set routes directly from JSX view files, just export `path`:

<table class="infotable">
<tr>
<td style="width: 20%">
<div class="language-"><pre><code>
├─ entry/
│  ├─ client.js
│  └─ server.js
├─ views/
│  ├─ <b style="color: #ec6f2d">index.jsx</b>
│  └─ about.jsx
├─ index.html
├─ base.jsx
├─ routes.js
├─ main.js
└─ server.js
</code></pre></div>
</td>
<td>

```jsx

import { Link } from 'react-router-dom'

export const path = '/'

export default function Index () {
  return (
    <>
      <h1>Index Page</h1>
      <p>Go to <Link to="/about">/about</Link></p>
    </>
  )
}
```

</td>
</tr>
</table>

As long as you also use the `loadRoutes()` helper from <b>fastify-vite-vue/app</b>:

<table class="infotable">
<tr>
<td style="width: 20%">
<div class="language-"><pre><code>
├─ entry/
│  ├─ client.js
│  └─ server.js
├─ views/
│  ├─ index.jsx
│  └─ about.jsx
├─ index.html
├─ base.jsx
├─ <b style="color: #ec6f2d">routes.js</b>
├─ main.js
└─ server.js
</code></pre></div>
</td>
<td>

```js

import { loadRoutes } from 'fastify-vite-vue/app'

export default loadRoutes(import.meta.globEager('./views/*.jsx'))
```

</td>
</tr>
</table>

The following snippet is equivalent to the one above:


<table class="infotable">
<tr>
<td style="width: 20%">
<div class="language-"><pre><code>
├─ entry/
│  ├─ client.jsx
│  └─ server.jsx
├─ views/
│  ├─ index.jsx
│  └─ about.jsx
├─ index.html
├─ base.jsx
├─ <b style="color: #ec6f2d">routes.js</b>
├─ main.js
└─ server.js
</code></pre></div>
</td>
<td>

```js

import Index from './views/index.jsx'
import About from './views/about.jsx'

export default [
  {
    path: '/',
    component: Index,
  },
  {
    path: '/about',
    component: About,
  },
]
```

</td>
</tr>
</table>


Similarly to the way `createRenderFunction()` works, providing a `routes` array in your server entry export is what ensures you can have individual Fastify [route hooks](/guide/route-hooks.html), [payloads](#route-payloads) and [isomorphic data](#isomorphic-data) functions for each of your [React Router][react-router] routes. When these are exported directly from your view files, `loadRoutes()` ensures they're collected. 

<b>fastify-vite</b> will use this array to automatically <b>register one individual route</b> for them while applying any hooks and data functions provided.

[vue-router]: https://reactrouter.com/

::: tip
<b>If you don't export</b> `routes`, you have to tell Fastify what routes you want rendering your SSR application:

```js
fastify.vite.get('/*')
```

And in this case, any <b>_hooks_</b> or <b>_data functions_</b> exported directly from your Vue files <b>would be ignored</b>.
:::


## Data Fetching

<b>fastify-vite</b> prepacks two different **_convenience mechanisms_** for fetching data before and after initial page load. Those are in essence just two different <b>data functions</b> you can export from your view files.

One is for when your data function can only run on the server but still needs to be accessible from the client via an HTTP API call, the other is for when your data function needs to be fully isomorphic, that is, run both on server and client exactly the same way.

### Route Payloads

[async-data]: https://nuxtjs.org/examples/data-fetching-async-data
[prepass]: https://github.com/FormidableLabs/react-ssr-prepass
[preHandler]: https://www.fastify.io/docs/latest/Hooks/#prehandler

The first is `getPayload`, which is in a way very similar to `getServerSideProps` [in Next.js](https://nextjs.org/docs/basic-features/data-fetching#getserversideprops-server-side-rendering), with a dash of [react-ssr-prepass][prepass]. Exporting a `getPayload` function from a view will **automatically cause it to be called <b>prior to SSR</b> (still in the Fastify layer) when rendering the route it's associated to**, but will **also automatically register an endpoint where you can call it from the client**. Not only that, coupled with <b>fastify-vite</b>'s [`useHydration`](/functions/use-hydration) isomorphic hook, `getPayload` will stay working seamlessly during client-side navigation ([History API](https://developer.mozilla.org/en-US/docs/Web/API/History_API) via [Vue Router](https://next.router.vuejs.org/), [React Router](https://reactrouter.com/) etc). In that regard it's very similar to `asyncData` Nuxt.js, which will also work seamlessly during SSR and client-side navigation.

In a nutshell — navigating to a route on the client will cause an HTTP request to be triggered automatically for that route's payload data. If the route is being server-side rendered, data is retrieved on the server and just hydrated on the client on first render.

```js
export const path = '/route-payload'

// Will absolutely only run on the server, so it's safe to
// assume req, reply and fastify references are defined 
// in the context object passed as first parameter
export async function getPayload ({ req, reply, fastify }) {
  // Simulate a long running request
  await new Promise((resolve) => setTimeout(resolve, 3000))

  return {
    message: req.query?.message || 'Hello from server',
  }
}
```

On the server, `getPayload` is set up to run automatically in the route's [`preHandler`][preHandler] hook.

Providing this function will also cause a GET JSON endpoint to be set up to run it remotely. The endpoint URL is generated following the <code>/-/payload/:url</code> format. 

If you're exporting a `getPayload` function from the `/foobar` view component, then the automatically registered endpoint will be `/-/payload/foobar`.

So, to recap — because this can be a little confusing at first — <b>here's the rundown</b>:

- During first-render, on the server, `getPayload` is executed via a [`preHandler`][preHandler] hook.
- Its result is stored as <code>req.$payload</code> and then serialized for [client hydration](/internals/client-hydration). 
- <b>Both on the server and on the client, the value is available via `useHydration` as `$payload`</b>.
- <b>For client-side navigation, <code>useHydration</code> will call the HTTP endpoint automatically</b>.


#### Retrieving payload with useHydration

The `useHydration` hook takes a configuration object as first parameter. The purpose of this configuration object is to easily to hold a reference to the data function being used in the view.

```jsx{4-8,12-14}
export const path = '/route-payload'

export async function getPayload ({ req, reply, fastify }) {
  // ...
  // Omitted for brevity
  // ...
}

export default function RoutePayload () {
  const [ctx] = useHydration({ getPayload })
  return (
    <>
      <p>Message: {ctx.$payload.message}</p>
    </>
  )
}
</script>
```

`useHydration` doesn't really do anything with the `getPayload`, **it just uses it to know what to do**.

If it sees a reference to a function named `getPayload`, it will know `getPayload` is defined in that scope and it should try to retrieve a route payload — either live during SSR, hydrated on the client or retrieved via the automatically created HTTP endpoint.

::: tip
As a convention, special properties associated with data fetching functions have the `$` prefix.
:::

`useHydration` also provides the `$payloadPath` function, to get the HTTP endpoint programatically. In the snippet below, a new request to the page's payload endpoint set up to be performed manually.

It also shows the context object returned by `useHydration` used to manage loading status, in the case where it's triggering a request to the payload HTTP endpoint client-side. To update the context object we use the `update` function returned by `useHydration`:

```jsx
export default function RoutePayload () {
  const [ctx, update] = useHydration({ getPayload })
  const [message, setMessage] = useState(null)

  // Example of manually using ctx.$payloadPath()
  // to construct a new request to this page's automatic payload API
  async function refreshPayload () {
    update({ $loading: true })
    const response = await window.fetch(`${
      ctx.$payloadPath()
    }?message=${
      encodeURIComponent('Hello from client')
    }`)
    const json = await response.json()
    setMessage(json.message)
    update({ $loading: false })
  }
  if (ctx.$loading) {
    return (
      <>
        <h2>Automatic route payload endpoint</h2>
        <p>Loading...</p>
      </>
    )
  }
  return (
    <>
      <h2>Automatic route payload endpoint</h2>
      <p>Message: {message || ctx.$payload?.message}</p>
      <button onClick={refreshPayload}>
        Click to refresh payload from server
      </button>
    </>
  )
}
```

Learn more by playing with the [`react-data`](https://github.com/terixjs/flavors/tree/main/react-data) boilerplate flavor:

`degit terixjs/flavors/react-data your-app`


### Isomorphic Data

The second convenience mechanism is `getData`. It's very similar to `getPayload` as <b>fastify-vite</b> will also run it from the route's [preHandler][preHandler] hook. 

But it resembles the classic [`asyncData`][async-data] from Nuxt.js more closely — because the very same `getData` function gets executed both on the server and on the client (during navigation). If it's executed first on the server, the client gets the [hydrated](/internals/client-hydration) value on first render. 

::: tip
When using `getPayload`, the client is able to execute that very same function, but via a HTTP request to an automatically created endpoint that provides access to it on the server.
:::

Below is a minimal example — like in the previous `getPayload` examples, you must pass a reference to the `getData` function used to `useHydration`.

```jsx
import { fetch } from 'fetch-undici'
import { useHydration, isServer } from 'fastify-vite-vue/client'

export const path = '/data-fetching'

export async function getData ({ req }) {
  const response = await fetch('https://httpbin.org/json')
  return {
    message: isServer
      ? req.query?.message ?? 'Hello from server'
      : 'Hello from client',
    result: await response.json(),
  }
}
```

Up until this point, the code is exactly the same as the [Vue example](/guide/vue.html#isomorphic-data), the only difference being that it's now importing from `fastify-vite-react`. The return value for `useHydration` differs slightly from the Vue version though — it returns an array with `[ctx, update]`, where `ctx` is the hydration context and `update` is a function that updates it. Without [Vue-like reactivity](https://v3.vuejs.org/guide/reactivity.html), React requires us to use a function to update the hydration context if we need to.

```jsx
export default function DataFetching (props) {
  const [ctx] = useHydration({ getData })
  if (ctx.$loading) {
    return (
      <>
        <h2>Isomorphic data fetching</h2>
        <p>Loading...</p>
      </>
    )
  }
  return (
    <>
      <h2>Isomorphic data fetching</h2>
      <p>{JSON.stringify(ctx.$data)}</p>
    </>
  )
}
```

Notice how this example uses [fetch-undici](https://www.npmjs.com/package/fetch-undici) to provide isomorphic access to [`fetch`](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch).

To recap:

- During first-render, on the server, `getData` is executed via a [`preHandler`][preHandler] hook.
- Its result is stored as <code>req.$data</code> and then serialized for [client hydration](/internals/client-hydration). 
- <b>Both on the server and on the client, the value is available via `useHydration` as `$data`</b>.
- <b>For client-side navigation, <code>useHydration</code> executes the `getData` function isomorphically</b>.

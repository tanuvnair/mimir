---
title: "SolidStart Documentation"
date: 2026-07-24
tags:
  - solidstart
  - learning
publish: false
---

# [SolidStart Documentation](https://docs.solidjs.com/)

## Overview

- SolidStart is an open source meta-framework designed to unify components that make up a web application. It is built on top of [Solid](https://docs.solidjs.com/) and uses [Vinxi](https://vinxi.vercel.app/), an agnostic Framework Bundler that combines the power of [Vite](https://vitejs.dev) and [Nitro](https://nitro.build/).

## Getting started

### Points to be noted

 - SolidStart uses [Vinxi](https://vinxi.vercel.app/) both for starting a development server with [Vite](https://vitejs.dev/) and for building and starting a production server with [Nitro](https://nitro.build/).
 - When you run your application, you are actually running `vinxi dev` under the hood.
- You can read more about the [Vinxi CLI and how it is configured in the Vinxi documentation](https://vinxi.vercel.app/api/cli.html).

### Project files

The default structure is:

```
public/
src/
├── routes/
│   ├── index.tsx
├── entry-client.tsx
├── entry-server.tsx
├── app.tsx
```

**Note:** Depending on the configuration options you chose when creating your project, your file structure may look slightly different. For example, if you chose to use JavaScript rather than TypeScript, your file extensions will be `.jsx` instead of `.tsx`.

Each directory and file in this structure serves a specific purpose in your SolidStart application:

- `public/` - contains the publicly-accessible assets for your application. This is where images, fonts, and other files that you want to be accessible to the public should be placed.
- `src/` - where your Start application code will live. It is aliased to `~/` for importing in your code.
- `src/routes/` - any files or pages will be located in this directory. You can learn more about the [`routes` folder in the routing section](https://docs.solidjs.com/solid-start/building-your-application/routing).
- [`src/entry-client.tsx`](https://docs.solidjs.com/solid-start/reference/entrypoints/entry-client) - this file is what loads and _hydrates_ the JavaScript for our application on the client side (in browser). In most cases, you will **not** need to modify this file.
- [`src/entry-server.tsx`](https://docs.solidjs.com/solid-start/reference/entrypoints/entry-server) - this file will handle requests on the server. Like `entry-client.tsx`, in most cases you will **not** need to modify this file.
- [`app.tsx`](https://docs.solidjs.com/solid-start/reference/entrypoints/app) - this is the HTML root of your application both for client and server rendering. You can think of this as the shell inside which your application will be rendered.

## Routing

### Creating new routes

SolidStart traverses your `routes` directory, collects all of the routes, and then makes them accessible using the [`<FileRoutes />`](https://docs.solidjs.com/solid-start/reference/routing/file-routes). This component will only include your UI routes, not your API routes. Rather than manually defining each `Route` inside a `Router` component, `<FileRoutes />` will generate the routes for you based on the file system.

Because `<FileRoutes />` returns a routing config object, you can use it with the router of your choice. In this example, we use [`solid-router`](https://docs.solidjs.com/solid-router):

```JSX
import { Suspense } from "solid-js";
import { Router } from "@solidjs/router";
import { FileRoutes } from "@solidjs/start/router";

export default function App() {
  return (
    <Router root={(props) => <Suspense>{props.children}</Suspense>}>
      <FileRoutes />
    </Router>
  );
}
```

`<FileRoutes />` will generate a route for each file in the `routes` directory and its subdirectories. For a route to be rendered as a page, it must default export a component. This component represents the content that will be rendered when users visit the page:

```JSX
export default function Index() {  return <div>Welcome to my site!</div>;}
```

### File based routing

Each file in the `routes` directory is treated as a route. To create a new route or page in your application, simply create a new file in the `routes` directory. The file name will be the URL path for the route:

- `example.com/blog` ➜ `/routes/blog.tsx`
- `example.com/contact` ➜ `/routes/contact.tsx`
- `example.com/directions` ➜ `/routes/directions.tsx`

#### Nested routes

If you need nested routes, you can create a directory with the name of the preceding route segment, and create new files in that directory:

- `example.com/blog/article-1` ➜ `/routes/blog/article-1.tsx`
- `example.com/work/job-1` ➜ `/routes/work/job-1.tsx`

When a file is named `index`, it will be rendered when there are no additional URL route segments being requested for a matching directory:

- `example.com` ➜ `/routes/index.tsx`
- `example.com/socials` ➜ `/routes/socials/index.tsx`

#### Nested layouts

If you want to create nested layouts you can create a file with the same name as a route folder.

```
|-- routes/
    |-- blog.tsx                   // layout file
    |-- blog/
        |-- article-1.tsx         // example.com/blog/article-1
        |-- article-2.tsx        // example.com/blog/article-2
```

In this case, the `blog.tsx` file will act as a layout for the articles in the `blog` folder. You can reference the child's content by using `props.children` in the layout.

```JSX
// routes/blog.tsx
import { RouteSectionProps } from "@solidjs/router";

export default function BlogLayout(props: RouteSectionProps) {
  return <div>{props.children}</div>;
}
```

**Note**: Creating a `blog/index.tsx` or `blog/(blogIndex).tsx` is not the same as it would only be used for the index route.

### Renaming Index

By default, the component that is rendered for a route comes from the default export of the `index.tsx` file in each folder. However, this can make it difficult to find the correct `index.tsx` file when searching, since there will be multiple files with that name.

To avoid this, you can rename the `index.tsx` file to the name of the folder it is in, enclosed in parentheses.

This way, it will be treated as the default export for that route:

```
|-- routes/                       // example.com
    |-- blog/
        |-- article-1.tsx         // example.com/blog/article-1
        |-- article-2.tsx
    |-- work/
        |-- job-1.tsx             // example.com/work/job-1
        |-- job-2.tsx
    |-- socials/
        |-- (socials).tsx           // example.com/socials
```

#### Escaping nested routes

When you have a path that is nested but wish for it to have a separate Layout, you can escape the nested route by applying a name between `( )`. This will allow you to create a new route that is not nested under the previous route:

```
|-- routes/                       // example.com
    |-- users/
        |-- index.tsx            // example.com/users
        |-- projects.tsx         // example.com/users/projects
    |-- users(details)/
        |-- [id].tsx            // example.com/users/1
```

Additionally, you can incorporate nested layouts of their own:

```
|-- routes/
    |-- users.tsx
    |-- users(details).tsx
    |-- users/
        |-- index.tsx
        |-- projects.tsx
    |-- users(details)/
        |-- [id].tsx
```

#### Dynamic routes

Dynamic routes are routes that can match any value for one segment of the route. When your URL path contains a dynamic segment, square brackets (`[]`) are used to define the dynamic segment:

- `example.com/users/:id` ➜ `/routes/users/[id].tsx`
- `example.com/users/:id/:name` ➜ `/routes/users/[id]/[name].tsx`
- `example.com/*missing` ➜ `/routes/[…missing].tsx`

This allows you to create a single route that can match any value for that segment of the URL path. For example, `/users/1` and `/users/2` are both valid routes and rather than defining separate routes for each user, you can use a dynamic route to match any value for the `id` segment.

```
|-- routes/
    |-- users/
        |-- [id].tsx
```

For example, using `solid-router`, you could use the [`useParams`](https://docs.solidjs.com/solid-router/reference/primitives/use-params) primitive to match the dynamic segment:

```JSX
import { useParams } from "@solidjs/router";

export default function UserPage() {
  const params = useParams();
  return <div>User {params.id}</div>;
}
```

#### Optional parameter

If you have optional parameters in your route, you can use the double square brackets (`[[id]]`) to define the dynamic segment. This will match a route with or without a parameter.

```
|-- routes/
    |-- users/
        |-- [[id]].tsx
```

In this case, some pages that could be matched include:

- `/users`
- `/users/1`
- `/users/abc`

#### Catch-all routes

Catch-all routes are a special type of dynamic route that can match any number of segments. They are defined using square brackets with `…` before the label for the route (e.g. `[…post]`).

```
|-- routes/
    |-- blog/
        |-- index.tsx
        |-- [...post].tsx
```

A catch-all route will have one parameter which is a forward-slash delimited string of all the URL segments after the last valid segment. For example, with the route `[…post]` and a URL path of `/post/foo` the `params` object returned from the `useParams` primitive will have a `post` property with the value of `post/foo`. For a URL path of `/post/foo/baz` it will be `post/foo/baz`.

```JSX
import { useParams } from "@solidjs/router";

export default function BlogPage() {
  const params = useParams();
  return <div>Blog {params.post}</div>;
}
```

### Route groups

Using route groups, you can organize your routes in a way that makes sense for your application, without affecting the URL structure. Since file-based routing is based on the file system, it can be difficult to organize your routes in a way that makes sense for your application.

In SolidStart, route groups are defined by using parenthesis (`()`) surrounding the folder name:

```
|-- routes/
    |-- (static)
        |-- about-us                // example.com/about-us
            |-- index.tsx
        |-- contact-us              // example.com/contact-us
            |-- index.tsx
```

### Additional route config

SolidStart offers a way to add additional route configuration outside of the file system. Since SolidStart supports the use of other routers, you can use the `route` export provided by `<FileRoutes />` to define the route configuration for the router of your choice.

```JSX
import type { RouteSectionProps, RouteDefinition } from "@solidjs/router";

export const route = {
  preload() {
    // define preload function
  }
} satisfies RouteDefinition

export default function UsersLayout(props: RouteSectionProps) {
  return (
    <div>
      <h1>Users</h1>
      {props.children}
    </div>
  );
}
```

## API routes

While Server Functions can be a good way to write server-side code for data needed by your UI, sometimes you need to expose API routes. Some reasons for wanting API Routes include:

- There are additional clients that want to share this logic.
- Exposing a GraphQL or tRPC endpoint.
- Exposing a public-facing REST API.
- Writing webhooks or auth callback handlers for OAuth.
- Having URLs not serving HTML, but other kinds of documents like PDFs or images.

For these use cases, SolidStart provides a way to write these routes in a way that is easy to understand and maintain. API routes are just similar to other routes and follow the same filename conventions as [UI Routes](https://docs.solidjs.com/solid-start/building-your-application/routing).

The difference between API routes and UI routes is in what you should export from the file. UI routes export a default Solid component, while API Routes do not. Rather, they export functions that are named after the HTTP method that they handle.

**Note:** API routes are prioritized over UI route alternatives. If you want to have them overlap at the same path remember to use `Accept` headers. Returning without a response in a `GET` route will fallback to UI route handling.

### Writing an API route

To write an API route, you can create a file in a directory. While you can name this directory anything, it is common to name it `api` to indicate that the routes in this directory are for handling API requests:

```jsx
export function GET() {
  // ...
}

export function POST() {
  // ...
}

export function PATCH() {
  // ...
}

export function DELETE() {
  // ...
}
```

API routes get passed an `APIEvent` object as their first argument. This object contains:

- `request`: [`Request`](https://developer.mozilla.org/en-US/docs/Web/API/Request) object representing the request sent by the client.
- `params`: Object that contains the dynamic route parameters. For example, if the route is `/api/users/:id`, and the request is made to `/api/users/123`, then `params` will be `{ id: 123 }`.
- `fetch`: An internal `fetch` function that can be used to make requests to other API routes without worrying about the `origin` of the URL.

An API route is expected to return JSON or a `Response` object. In order to handle all methods, you can define a handler function that binds multiple methods to it:

```ts
async function handler() {
  // ...
}

export const GET = handler;
export const POST = handler;
// ...
```

An example of an API route that returns products from a certain category and brand is shown below:

```ts
import type { APIEvent } from "@solidjs/start/server";
import store from "./store";

export async function GET({ params }: APIEvent) {
  console.log(`Category: ${params.category}, Brand: ${params.brand}`);
  const products = await store.getProducts(params.category, params.brand);
  return products;
}
```

### Session Management

Since HTTP is a stateless protocol, you need to manage the state of the session on the server. For example, if you want to know who the user is, the most secure way of doing this is through the use of HTTP-only cookies. Cookies are a way to store data in the user's browser that persist in the browser between requests.

The user's request is exposed through the `Request` object. Through parsing the [`Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cookie) header, the cookies can be accessed and any helpers from `vinxi/http` can be used to make that a bit easier.

```ts
import type { APIEvent } from "@solidjs/start/server";
import { getCookie } from "vinxi/http";
import store from "./store";

export async function GET(event: APIEvent) {
  const userId = getCookie("userId");
  if (!userId) {
    return new Response("Not logged in", { status: 401 });
  }
  const user = await store.getUser(event.params.userId);
  if (user.id !== userId) {
    return new Response("Not authorized", { status: 403 });
  }
  return user;
}
```

In this example, you can see that the `userId` is read from the cookie and then used to look up the user in the store. For more information on how to use cookies for secure session management, read the [session documentation](https://docs.solidjs.com/solid-start/advanced/session).

## Related

- [[dev-discussions]]
- [[go-fiber-gorm-boilerplate]]

# NextJS
Here is a text summary of the content provided in this video onwards: https://www.youtube.com/watch?v=6-qMfLcPhOw&list=PLC3y8-rFHvwhIEc4I4YsRz5C7GOBnxSJY&index=47

## Rendering in React
- Rendering of NextJS is built on top of Rendering in React, so it gives a good overview of the reason for the development of rendering strategies deployed in NextJS.
- The rendering strategy in React is known as *** Client Side Rendering(CSR) *** 
    - initially only has a HTML file with a `div` with the name of `root`
    - When user visits a site, the JS bundle runs and populates the `root` with the DOM tree

- Problem with this strategy?
    * Search Enginer Optimization (SEO) is hindered
    * Performance issue - the client has to perform way too many tasks based on the JS bundle
        - fetch data
        - build UI
        - make UI interactive etc.

- Evolution
    - Why not make use of the server more instead of overwhelming the client with all these tasks?


## Evolution to *** Server Side Rendering (SSR) *** 
- The process is as following:
    - Request to a server when visiting a site
    - The server generates the HTML (rendering)
    - The server sent the HTML and JS reference to the client
    - The client requests the JS reference to get the required JS files in return
    - The client loads the JS files and `hydrate` the static HTML files with interactive features.

- The process is summarized as shown below:
    <div><img src='/images/server-side-rendering.jpg' /></div>

- These successfully tackle the problem of Rendering in React
    - No more hindering SEO since it always serve at least a static HTML page with relevant content
    - Relieve client's workload as rendering mostly takes place in server, except hydration

- However, it still has its own problem
    - Problem 1: Have to fetch everything before anything can be shown
    - Problem 2: Have to load all the JS files first before hydration can take place
    - Problem 3: Have to wait for hydration before any interaction can happen
    - These problems become an *** all-or-nothing waterfall issue *** where Problem 3 needs to wait for Problem 2 to resolve, Problem 2 needs to wait for Problem 1 to resolve...

- This is highly inefficient since certain part of the components are usually slower and heavier to load, causing all other components can only be displayed or interacted with until those are completely loaded.


## Suspense SSR Architecture
- React therefore introduce `Suspense` architecture, where it can wrap around heavier and slower components
    - As a result, Problem 1 gets resolved
    - Other components no longer need to wait for the heavier ones to complete loading to display themselves
    - Meanwhile, heavier components that are wrapped around by `Suspense` can load on its own and then stream their content eventually when they are ready.
    - This technique is known as `HTML streaming`

- As for problem #2, while downloading the JS bundles, it smartly recognizes what are the most important JS bundles which are prioritized first without waitin for others that are more important.
    - As a result, hydration part can be done first for those more important or lighter component.

- This has effectively solved Problem 1 to 3.

- Specifically, this becomes one of the adopted rendering strategies in NextJS, known as `streaming`, aside from `Static Rendering` and `Dynamic Rendering`
    - It can either be implemented as a `Suspense` Wrapper over the entire `page.tsx` in the form of `loading.tsx`
    - Or it can also be implemented as a `Suspense` Wrapper over a specific component within `page.tsx` and `layout.tsx`
    - READ MORE: https://nextjs.org/learn/dashboard-app/streaming

- There are still a problem though.
    - This architecture still end up downloading the entire codebase, which can eventually become heavy.
    - which raises a question: Do we really need to download eveything from the server or just those that are needed?

## React Server Component (RSC)
- Finally, React introduces *** RSC ***, which composes of
    - *** Client Component *** 
        - just like the typical React component that we know, 
        - it can be rendered on the client side (marked with `'use client'`)
            - however, there are situations where it will also be rendered once on the server
            - you may refer to this video to see the proof: https://www.youtube.com/watch?v=dMCSiA5gzkU&list=PLC3y8-rFHvwhIEc4I4YsRz5C7GOBnxSJY&index=52
        - Advantages:
            - used to handle interactive part
            - have access to HTML API and React's hook
        - Disadvantage:
            - cannot be an async function


    - *** Server Component ***
        - designed exclusively to be used on the server side
        - Disadvantages:
            - hence has no access to client side's HTML API and React's hook
            - cant handle user's interaction
        - Advantages:
            - this code will never be downloaded to the client
            - can be async functions

- Benefits of using server component?
    - Smaller JS bundle: because its code stay on server, therefore reducing the dependencies required to be handled by the client
    - Faster initial page load: static pages are pre-built during *** build time *** 
    - Direct access to server side resources
    - Enhanced security: API key is not shown to the client / sent to the browser
    - Caching: Rendered result can be cached and not require frequent rendering
    - Improved SEO


## Prefetching
- A technique that preloads routes in the background as their link becomes visible




## `generateStaticParams`

* READ MORE: https://nextjs.org/docs/app/api-reference/functions/generate-static-params

1. In the following setup,
    ```text
    /app
        - /products
            - /[id]
                - page.tsx
            - page.tsx
    ```
    - The route `/products/[id]` is not always rendered dynamically
        - If it does not use any dynamic function, it may be dynamically rendered for the first time but since then stored in the client side's route cache.
        - It is important to differentiate the concept of `dynamic route` and `dynamic rendering`

2. `generateStaticParams` allow us to pre-render a few of this variation in advance as following:
    ```ts

    type PrdouctPagePropsType = {
        params: Promise<{
            id: string
        }>
    }

    export async function generateStaticParams(){
        return [
            {id: '1'},
            {id: '2'},
            {id: '3'}
        ]
    }

    export default function ProductPage({ params }:ProductPagePropsType){

        const { id } = await params

        return (
            <h1>This is Product {id}</h1>
        )
    }
    ```
    
    - By doing so, page for Product 1, 2 and 3 are pre-rendered statically.
    - `generateStaticParams` is expected to return an array of object, where each object contains value for the params
    - If needed, API fetch can happen within `generateStaticParams` too

3. In fact, `generateStaticParams` has the potential to
    - make all dynamic route to become static route, given that all the possible values of `params` are known
    - i.e. using the above example, all `[id]` values are known

4. Because of this potential usage, `dynamicParams` is needed to decide that a page that has `generateStaticParams` will do when a dynamic parameter instead of previously defined static parameters are provided:
    ```ts
    // All posts besides the top 10 will be a 404
    export const dynamicParams = false
    
    export async function generateStaticParams() {
    const posts = await fetch('https://.../posts').then((res) => res.json())
    const topPosts = posts.slice(0, 10)
    
    return topPosts.map((post) => ({
        slug: post.slug,
    }))
    }
    ```

    - `export const dynamicParams = false` returns 404 pages when a dynamic parameter is provided

    - `export const dynamicParams = true` proceeds to render the page *** STATICALLY *** [DEFAULT with `generateStaticParams()`]
        - which means now this DYNAMIC ROUTE is RENDERED STATICALLY
        - when requested, it will be rendered ON DEMAND
        - stored in both FULL ROUTE CACHE (server) AND ROUTE CACHE (client side)

5. `generateStaticParams` also can make all dynamic route to be statically rendered
    - With `generateStaticParams` - just returns an empty array
    - Meanwhile, `export const dynamic = 'force-static' ` can also be used.
    - These settings mean that those dynamic route will only be rendered *** ON DEMAND *** and *** STATICALLY *** 
        - rendered on server for the first time when visisted by the user
        - then stored in FULL ROUTE CACHE (server) and ROUTE CACHE (client)

6. The biggest difference between a *** STATIC ROUTE, RENDERED STATICALLY *** vs *** DYNAMIC ROUTE, RENDERED STATICALLY *** are that:
    - Static route are rendered and built during *** build time ***
    - Dynamic route are rendered and built during *** request from the client ***
    - Once rendered / built, they are in the form of static pages, stored in FULL ROUTE CACHE, waiting to be fetched.
    - Refer to this video as a proof: https://youtu.be/oEF3dyNgmcs


7. Multiple Dynamic Segment in a route
    - In case where: `app/products/[category]/[product]/page.tsx` - it can `generateStaticParams` for both `[category]` and `[product]`
    - However, in :  `app/products/[category]/page.tsx` - it can only `generateStaticParams` for `[category]`

    - NextJS provides a specific way that allow the child segment to access the static params generated in the parent:
        ```ts
        // Generate segments for [category] - parent
        export async function generateStaticParams() {
            const products = await fetch('https://.../products').then((res) => res.json())
            
            return products.map((product) => ({
                category: product.category.slug,
            }))
        }
        
        export default function Layout({params}: {params: Promise<{ category: string }>}) {
            // ...
        }
        ```
        
        - Then, the child dynamic segment can a
        ```ts
        // Generate segments for [product] using the `params` passed from
        // the parent segment's `generateStaticParams` function
        export async function generateStaticParams({params: { category }}: {params: { category: string }}) {
            const products = await fetch(
                `https://.../products?category=${category}`
            ).then((res) => res.json())
            
            return products.map((product) => ({
                product: product.id,
            }))
        }
        
        export default function Page({params}: {params: Promise<{ category: string; product: string }>}) {
            // ...
        }
        ```


## `server-only` / `client-only` Node Package
1. With React Server Componenet (RSC) consisting of client component and server component, the boundaries may sometimes blur the line.
    - To ensuring certain functions are only run on server side (for security reason), a package known as `server-only` can be installed.
    - If any client component tries to import anything from a file that has `import "server-only"`, an error will be thrown.
    - Likewise, a package known as `client-only` can be used to exclusively include certain code to be importable in client component only.


## Client Boundary
1. In React Server Component (RSC) Architecture, by default, all components are marked as server component.
    - However, when a component is marked as client component via `'use client'`, all the components imported by this client component will also become 'client component'
        - unless that imported component is executing certain functions that are server-side only, such as `fs` which requires a file system
        - then it will remain as a server component, even invoked by a client component, resulting in error thrown
        - To work around this, it is recommended to pass the server component as `children` prop to the intended client component
    - Therefore, a component can exist as both server and client component, depending on who is importing it.


## Rendering of Static & Dynamic Route
* Assuming no dynamic function is used:
1. Static Route
    - Rendered at build time
    - Stored in Full Route Cache and Client Route Cache

2. Dynamic Route - defined by having [id] or [...slug]
    - If params provided via generateStaticParams
        - rendered at build time
        - stored in Full Route Cache and Client Route Cache

    - If no generateStaticParams (or generrateStaticParams return empty array / force-static)
        - rendered on first request
        - stored in Full Route Cache after first render
        - stored in Client Route Cache

* Assuming using dynamic functions (cookies, headers, searchParams)
3. Regardless of what route
        - rendered on every request
        - NOT stored in Full Route Cache
        - ALSO NOT stored in Client Route Cache 
            - stored for very briefly, 
            - but with `revalidate: 0` by default, indicating that it is stored for 0s, so basically none.

* P/S
    - Dynamically rendered routes are not cached in the Full Route Cache, but can still use the Data Cache for data requests.




## Some key takeaways of how rendering works in NextJS
* Refer to the results from here: https://github.com/wengti/nextjs-playground

1. In `dev` mode
    - There's no full route cache.
    - Even statically rendered route, on request, will be rerendered on the server.
    - So basically, in `dev` mode, it doesnt provide a good actual indicate on how the cache works.

2. `fetch data`
    - Just because a route fetches data, it does not automatically become dynamically rendered route.
    - If the fetched data is cached, it will be statically rendered instead.

3. `Dynamically Rendered Route`
    - When a route consists of dynamic component and does become dynamically rendered, 
        - does not get stored in full route cache
        - get stored in client route cache for x amount of seconds, where x is from `revalidate: x` and x is default to 0
    - Therefore, it can be confirmed that dynamically rendered route does not get stored in any route by default

3. `Layout`
    - Layout does not get remounted when its child segment changes.
    - That is why even if it uses dynamic function and supposed to be dynamically rendered, it does not get triggered.
    - Those dynamic functions only get triggered when its unmounted and remounted once.

4. `Dynamically Rendered Layout`
    - When a layout becomes dynamically rendered, its children automatically becomes a dynamic route too, although it does not use any dynamic functions.
    - This will affect all other child route attached to it.

5. `Dynamically Rendered Parent Route`
    - A parent route can be dynamically rendered, but it doesnt make the child route to also become dynamically rendered.
    - For instance, /rooms/rooms-number
        - /rooms can be dynamically rendered, but does not affect /rooms-number to be dynamically rendered

6. `Modal`
    - Having a dynamically rendered modal placed in the layout makes the layout becomes dynamically rendered.
    - Therefore, the supposedly statically rendered static children props also becomes dynamically rendered.
    - `Modal` in `Layout` does not get remounted unless the navigation goes back to the `children` on the same level as that `modal`.
    - In fact, the `Modal's page.tsx` only got shown when the `children's page.tsx` in the same level is visited
    - If the user directly navigate to the route that has page.tsx but not on the same level as the modal, the `modal's default.tsx` got shown instead.

7. `Dynamic Route`, such as /room-id/[id]
    - In its vanilla form, it does not get stored on full route cache and client route cache.
        - Therefore, it is always rendered on the server when requested just like a typical dynamically rendered route.
    - with `generateStaticParams`, this route becomes a statically rendered route
        - those that are specified in the returned array get statically rendered during build time, stored in full route cache
        - those that are not specified in the returned array get statically rendered on request, then stored in full route cache
        - if it returns an empty array, then all of them will get statically rendered on request, then stored in full route cache

8. `Prefetch`
    - It happens when the user comes across a *link* to a statically rendered route
    - However, if the route consists of dynamic functions, that route will not be prefetched and built in advance. It would only be requested to the server when that link is clicked.
    - Interestingly, if this *link* is to a dynamic route, where the params is not included in the `generateStaticParams`, a request will immediately be sent to the server to build that route.
        - Therefore, if that route has any logged response, it will get presented.
        - This is likely because generateStaticParams basically make the dynamic route to be a static route, with all the route that has the params defined in it being prebuilt during build time
        - However, for params that are not specified, it still should be a static route, but only built on request
        - This request happens during the prefetch
        - Prefetch targeted it because it is treated as a static route
        - Meanwhile, for truely statically rendered route, since its built during build time, you dont see the log happen since it already happened during build time

9. `Full Route Cache`
    - It is stored in the memory of the server.
    - Therefore it will not persist if the server got stopped and restarted.
    - that is why those static route that are built on request due to `generateStaticParams` need to be regenerated when the user first requests them after the server restarted

10. `cacheComponents: true`
    - With this feature turned on in `next.config.ts`, a few things are added, which are very subtle.
        - The build tool becomes more particular about the addition of suspense.
        - Sources say that now a dynamic route can be controlled at a granular level
            - elements that are outside suspense which only involve static content will be generated as a static shell
            - meanwhile, those inside the suspense will only get generated on request
                - *WARNING*: However, based on my testing, with or without turning on this config, the static shell still run when the user requests for that route
        - The static shell is now served from a Content Delivery Network, unlike previously which was served from the server directly, which can be slower.
        - Dynamically Rendered Route are now labelled as Partially Prerender Route, which mostly can be treated as the same
        - enable the use of function such as `use cache`
            - where it stores the computed value of a function, component or route in the cache on server end (maybe on the CDN not sure)
            - which effectively replace `export dynamic = 'force-static`
        - `generateStaticParams`
            - can no longer return empty array
            - must return at least one element in it
            - only that element that are specified are statically rendered during build time
            - all other routes are always dynamically rendered and therefore never stored on route cache.
            - hence forth, the quirky prefetch behaviour that is discussed above also no longer exist.
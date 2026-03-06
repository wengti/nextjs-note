# Next JS

## Caching Mechanism:
- A recap / summary of the key points from here: https://nextjs.org/docs/app/guides/caching#segment-config-options

- There are 4 caching mechanisms in NextJS
    * Request Memoization
    * Data Cache
    * Full Route Cache (On the server)
    * Route Cache (On client side)

### Request Memoization
- ***For each request*** incoming from the users / client, all `GET` request that has the same URL will be called only once.
- To reduce the need of multiple duplicated fetch.

- However, when accessing certain API that doesnt use `fetch` (i.e. supabase), 
    - `cache` can be used for request memoization.
    - READ MORE: https://nextjs.org/docs/app/guides/caching#react-cache-function

### Data Cache
- After a data is fetched, it will be saved in the data cache by default.

### Relationship between Data Cache, Full Route Cache (On Server) and Route Cache (On Client)
<div><img src='/images/static-and-dynamic-routes.avif'/></div>

- Static Route:
    * The page is built during ***Build Time***
    * Stored in Full Route Cache (On Server)
    * On first request from the user, it fetches that page and store on the client's Route Cache for future navigation.

- Dynamic Route:
    * The page is ***NOT*** built during Build Time.
    * On first request from the user, it skips Full Route Cache (On Server) and proceeds to fetch the necessary data and build that page.
    * The built page is then stored on the client's Route Cache for future navigation.

- If a route requires a certain data to be fetched, but that data is not cached in Data Cache after fetched
    - It will invalidate Full Route Cache for this route
    - Hence, this route cannot be a static route based on the mechanism dicussed above. (Because static route requires fetching it from the Full Route Cache on first user's request)
    - Therefore, this makes this route to be a dynamic one.

- However, if we base off of the discussion above. Wouldnt that mean that the user always just fetch from the client side's Route Cache and therefore it will never reflect changes from the server?
    - Here is the thing, ***the client side's ROUTE CACHE IS NOT PERMANENT***
    - It is cleared on:
        - refreshed
        - when it hits the automatic invalidation time
            * By default (`prefetch={null}` from `Link`): Static Routes - 5 minutes / Dyanamic Routes - Not Cached
            * `prefetch = {true}` or `router.prefetch`: Static and Dynamic Routes - 5 minutes
    - But do take note, for a static route, since the page is built during ***build time*** (npm run build), even after client's router cache is cleared, it is still going to fetch the same thing from server's Full Route Cache.
    

- Well, 5 minutes seem like a long time. What if we want the changes from the server to be reflected as soon as possible?
    - Here are some API provided by NextJS, allowing us to do so:
        * `revalidatePath('/path1')`
        * `revalidateTag('user-data')`
        * `router.refresh()`

### Rendering Strategies
- Based on these caching mechancims, 3 rendering strategies are employed in Next JS
    * Static Site Generation (SSG)
    * Incremental Site Regeneration (ISR)
    * Server Side Rendering (SSR)
    * [Extra] Client Side Rendering (CSR) -> pretty much like a typical render component that makes use of hooks - useState....

- These strategies are usually decided by NextJS depending on certain keywords that encounter in the script. However, the user is given the options to override it too based on the following options.
<div><img src='/images/rendering-strategies.jpg'/></div>

- The best way to identify which strategy that the NextJS has decided for each route is to build the page `npm run build` or `next build`


### Development Mode
- In development mode, there's no client side Route Cache. Everything is from the Full Route Cache.
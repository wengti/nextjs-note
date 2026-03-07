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
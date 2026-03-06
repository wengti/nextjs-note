# NextJS

## Routing Mechanism

1. `[...slug] `
    - slug exists in the form of string[], which can be obtained by using `params`
    - for an example, given a url as following `/docs/feature1/concept1`
    - if the file is structured as `/app/docs/[...slug]/page.tsx`
    - in the `page.tsx`, `slug` is in the form of `['feature1', 'concept1']`

2. `[[...slug]]`
    - in above example with only `[...slug]`, when navigate to `/docs/`, it will redirect to not-found page.
    - because in the directories, `/app/docs/page.tsx` does not exist.
    - but with an extra square bracket - `[[...slug]]`, it becomes a catch all routes, which mean it will share the same page as `[...slug]`
    - route-wise: `/docs/feature1`, `/docs/feature1/concep1` and `/docs` all share the same `page.tsx`-
    - Read more: https://nextjs.org/docs/pages/building-your-application/routing/dynamic-routes

3. `not-found.tsx`
    - This page can be called by using `notFound()`
    - An app can have multipe not-found.tsx, when it is triggered, it will find the most specific one in relation to the navigated url.
    - The file name must be exactly *** not-found.tsx ***

4. Private Folder
    - If a folder in `app` is named with the first letter being `_`, then it will be ignored in the routing mechanism
    - Even if it has a page.tsx within it

5. Routes Group

    - With this folder directories, to access login page, the url becomes `/auth/login`
    ```text
    root/
        - app/
            - auth/
                - login
                - register
    ```
    
    - To group mulitple routes into a folder for file organization without affecting the URL, use `()`, i.e.:
    ```text
    root/
        - app/
            - (auth)/
                - login
                - register
    ```
    - With this, the routes become `/login`

6. Multiple Root Layout
    ```text
    root/
        - app/
            -(auth)/
                - login/
                - register/
                - layout.tsx
            -(marketing)/
                - customers/ 
                - revenue/
                - page.tsx
                - layout.tsx
    ```

    - By making use of route groups, `page.tsx` which represents the root `/` URL share the same layout as `customers/` and `revenue/`
    - Meanwhile, `login/` and `register/` has a different layout


## Metadata

1. Routing Metadata

    - These following metadata is always added by default:
        ```html
            <meta charset="utf-8" />
            <meta name="viewport" content="width=device-width, initial-scale=1" />
        ```
    
    - Purpose?
        - For Search Engine Optimization (SEO)
        - The generated or defined metadata will be automatically resolved as the `<head>` part of HTML.

    - Rules:
        * Layout Metadata is applied for the subpages that are within it
        * Page Metadata is only applied specifically to that page.
        * For Metadata that exists in multiple places, they are merged together. If there are colliding keywords, page metadata will overwrite the layout metadata.
    
    - Limitation:
        * Setting metadata using NextJS (either static or dynamic) is only available for *** Server Component ***
        * To bypass this limitation, put the metadata in a server component and have that server component import the client component instead.
    
    
    
    - Type of metadata:
        - Static
            ```tsx
            import type { Metadata } from 'next'

            export const metadata: Metadata = {
                title: 'My Blog',
                description: '...',
            }
            ```

        - Dynamic
            ```tsx
            import type {Metadata} from 'next'

            type PropsType = {
                params: Promise<{
                    id: string
                }>
            }

            export async function generateMetadata({params}: PropsType): Promise<Metadata> {

                const {id} = await params
                const res = await fetch(`http://..../${id}`)
                const data = await res.json()

                return {
                    title: data.product.id
                    description: data.product.description
                }
            }
            ```

        - Read More: https://nextjs.org/docs/app/getting-started/metadata-and-og-images

2. `metadata.title`
    - Read More: https://nextjs.org/docs/app/api-reference/functions/generate-metadata#metadata-fields
    - Can either be a string or an object
    - String is pretty self-explanatory
    - Object, it can take in 3 field : `default`, `template`, `absolute`
        * In Layout.tsx
            ```tsx
            import type {Metadata} from  'next'

            export const metadata: Metadata  = {
                title: {
                    default: 'Pickaboru' // Fallback value
                    template: '%s | Pickaboru' // %s to be replaced by the title of metadata in sub-route
                }
            }
            ```
        
        * In one of the sub-routes under layout.tsx
            ```tsx
            import type {Metadata} from 'next'
            
            export const metadata: Metadata = {
                title: 'About' // Result in 'About | Pickaboru'
            }
            ```
        
        * However, for other routes that fall under the same layout, you may choose to break free from this template by using `absolute`
            ```tsx
            import type {Metadata} from 'next'

            export const metadata: Metadata = {
                title: {
                    absolute: 'Login Page' // Will not follow the template
                }
            }
            ```


## Link
1. How to use Link
    ```tsx
    import Link from 'next/navigation'

    export default function NavLink(){
        return <Link href='/about'>About</Link>
    }
    ```
    * READ MORE: https://nextjs.org/docs/pages/api-reference/components/link

2. Active Link Styling
    - The general idea here is to compare if this link currently matches with the URL path
    - The URL path can be obtained using `usePathname()` - which would require the component to be client-component
    - then the obtained pathname can be checked whether it `startsWith` certain keyword that matches where this link will direct the user to
    - If it does, then add an active style to the link

## Params and Search Params
1. In Server Component
    ```tsx
    export default async function ProductPage({params, searchParams}){
        const { id } = await params
        const { lang } = await searchParams
        return (
            <h1>Page for product ${id} in ${lang}</h1>
        )
    }
    ```

2. In Client Component: No need `async` and `await`, but replace with the `use` hook from react
    ```tsx
    import { use } from 'react'

    export default function ProductPage({params, searchParams}){
        const { id } = use(params)
        const { lang } = use(searchParams)
        return (
            <h1>Page for product ${id} in ${lang}</h1>
        )
    }
    ```

    * READ MORE: https://react.dev/reference/react/use

3.` params` and `searchParams` in `page.tsx` and `layout.tsx`
    - `params` available to both `page.tsx` and `layout.tsx`
    - `searchParams` only available to `page.tsx`


## Programmatic Navigation
1. `useRouter`
    - READ MORE: https://nextjs.org/docs/app/api-reference/functions/use-router
    
    - Very important point to clarify
        - There are 2 different packages in NextJS that has `useRouter`, namely from `next/navigation` and `next/router`
        - `useRouter` from `next/router` is used only in Page Router
        - `useRouter` from `next/navigation` is used in App Router (which is introduced in later version of NextJS)
        - Whether a project is using Page Router or App Router, it was prompted during the project setup.
        - To read more about their differences: https://stackoverflow.com/questions/76285831/whats-the-difference-between-next-router-and-next-navigation
    
    - Caveat of using useRouter()
        - It is a react hook
        - Therefore, it can only be used in a Client Component.
    
    - How to use?
        ```tsx
        'use client'
        import {useRouter} from 'next/navigation'

        export default function OrderPage(){

            const router = useRouter()

            const handleOrder = () => {
                console.log('Your order has been submitted!')
                router.push('/') // Redirect to home page
            }

            return (
                <section>
                    <h1>Click the button to order</h1>
                    <button onClick={handleOrder}>Order Now</button>
                </section>
            )
        }
        ```
    
    - Altenative to `router.push()`
        - `router.replace()` - replacing the route instead of adding one to the browser's history stack
        - `router.refresh()`
        - `router.back()`
        - `router.forward()`

2. `redirect`

    - READ MORE: https://nextjs.org/docs/app/api-reference/functions/redirect

    - How to use ?
        ```tsx
        import {redirect} from 'next navigation'
        export default async function Product({params}){

            const {id} = await params
            if(Number(id) > 10) redirect('/') //if invalid id, return to home page

            return (
                <section>
                    Product Page for Product {id}
                </section>
            )
        }
        ```
    
    - What is the difference between `redirect` and `useRouter` ?
        - `redirect` can be used in both server and client component
        - However, `redirect` component *** doesnt work in event handlers of client component ***. This is where `useRouter` should be used instead.
    
    - Caveat / Things to remember when using `redirect`
        - Internally, `redirect` works by first throwing an error. Therefore, it should not be used within the `try` block of a `try/catch` statement because the internally thrown error may be caught by the `catch` statement.


## `template.tsx`

* READ MORE: https://nextjs.org/docs/app/api-reference/file-conventions/template

1. How to use a `template.tsx` ?
    - It works just like a layout.
    - But it is rendered as the `children` input to the `layout.tsx` if there exists one.
    - and accept `page.tsx` as its children.

2. Difference between `template.tsx` and `layout.tsx`
    - With navigation:
        - `template.tsx` receives a unique key for their own segment level. They *** remount *** when that segment (including its dynamic params) changes.
            - Though, its worth mentioning that navigations within deeper segment do not remount higher-level templates.
            - Search params also do not trigger remount of `template.tsx`
        - `layout.tsx` on the other hand does not remount with navigation at its segment level.

3. Behaviour when a `template.tsx` mounts or remounts
    - Reset react state
    - Effect re-runs
    - DOM reset: ***ALL** DOM elements inside the template are fully recreated.


## `error.tsx`

1. Code template
    ```tsx
    'use client'
    import {useRouter} from 'next/navigation'

    type ErrorPropsType = {
        error: Error
        reset: () => void
    }
    export default function Error({error, reset}: ErrorPropsType){

        const router = useRouter()

        const handleReload = () => {
            startTransition( () => {
                router.refresh()
                reset()
            })
        }

        return (
            <section>
                <p>{error.msg}</p>
                <button onClick={handleReload}>Retry</button>
            </section>
        )

    }
    ```

2. A few caveats in using this `error.tsx`
    - READ MORE: https://nextjs.org/docs/app/getting-started/error-handling#handling-uncaught-exceptions
    - It should be placed at the same level as the `page.tsx`, which will be rendered when there's an error thrown in that page.
    - However, it only covers for error that occurs during *** rendering ***, for errors that happen during `eventHandler`, it should be handled using `try/catch` and `useState` to set errors.
    - Specifically, `error.tsx` is implemented as a `Error Boundary` in the react DOM tree
    - To use `error.tsx` it must be used as a *** client component ***.

3. How to reset errors?
    - `error` and `reset` are 2 built-in props provided for the default returned component in `error.tsx`
    - `reset` is a function that will only re-render the client side.
    - `router.refresh` is needed to ensure that the re-rendering of components happen on the server side too.
    - `startTransition` is a React hook. READ MORE: https://react.dev/reference/react/startTransition
        - In short, it ensures that the action within it are done synchronously (one after another)
        - using here ensures that server side re-renders first (via `router.refresh`)
        - then server side re-renders (via `reset()`) to clear the error state and hence stop ErrorBoundaries from showing up

4. A visualization on why error in the `layout.tsx` and `template.tsx` wont be caught by the `error.tsx`
    - As shown below `layout.tsx` sit above the `Error Boundary`
    <div><img src='https://nextjs.org/_next/image?url=https%3A%2F%2Fh8DxKfmAPhn8O0p3.public.blob.vercel-storage.com%2Fdocs%2Fdark%2Fnested-error-component-hierarchy.png&w=3840&q=75'/></div>


## `global-error.tsx`
* READ MORE: https://nextjs.org/docs/app/getting-started/error-handling#global-errors

1. Used to handle error in the RootLayout component
    - Advisable to add a button to execution `window.location.reload()`
    - must have `html` and `body` tag in the returned component
    - `error boundary` set by this file is only triggered during *** Production *** mode.


## Parallel Routes
* READ MORE: https://nextjs.org/docs/app/api-reference/file-conventions/parallel-routes

1. First let's take a look at how parallel routes are implemented with a code
    ```tsx
    type LayoutPropsType = {
        children: React.ReactNode
        analytics: React.ReactNode
        team: React.ReactNode
    }

    export default function Layout({children, team, analytics}) {
        return (
            <>
                {children}
                {team}
                {analytics}
            </>
        )
    }
    ```

    <div><img src='/images/parallel-routes-file-system.avif' /></div>


2. An important concept to understand from the code is *** `slot` ***
    - slot are pretty much props that can be accepted by a Layout Component.
    - Each `slot` is stored in the file directories in the form of `/@folder/page.tsx`
    - Technically speaking, `page.tsx` at the same level as the `layout.tsx` is also a slot stored implicitly in a folder `/@children`
    - However, slot itself doesnt become a valid route,
        - i.e. in file directories: `/app/@about/page.tsx`, in URL: `/about` or `/@about` is invalid
        - However, route can be further populated within the slot
        - For instance, in file directories: `/app/@about/profile/page.tsx` is equivalent to `/profile`
        - When one of the slot already has a matching route that is nested within it, other slot at the same level becomes unmatched.
        - These unmatched routes will retain their last active state.
        - However, when `/profile` got refrehsed, only `/app/@about/profile/page.tsx` has an active matching route and therefore only that slot is served
        - For all other slot at the same level the `default.tsx` (should placed at the same level as their page.tsx) will get rendered.
        - If those slot dont have a `default.tsx`, it will result in the `not-found` page to be served.
        - `default.tsx` can also be thought of as a fallback component that got displayed when there's no `page.tsx` for that `slot`.
    
    - This concept is best explained from this video: https://youtu.be/697kNwfU-4M
    - Or you may read from the documentation linked above.

3. Advantage 1 of using `parallel route` or `slot`
    - Route handling (Handling Error or Loading state) is done separately for each route as illustrated in the diagram below

4. Advantege 2 of using `parrallel route` or `slot`
    - Part of the layout can be changed without affecting other layout slot / component.

5. Use case of parallel route:
    - Conditional Layout Rendering: check the identity of the user to decide whether to render UI for client or admin in a particular layout slot
        - For example, using the same URL route, we can choose to show `@about` or `@login` component depending on the user's identity.
    
    - Tab Group: In one of the slot, the user can toggle between without affecting other slot's state.


## Intercepting Route
* READ MORE: https://nextjs.org/docs/app/api-reference/file-conventions/intercepting-routes
* Video Explanation: https://youtu.be/FTiwIVxWC00

1. First, lets talk about the general use case of Intercepting Route with the following illustration.
    <div align='center'>
        <img src='/images/intercepting-modal.jpg' width='49%' />
        <img src='/images/intercepting-images.jpg'  width='49%' />
    </div>

    - The idea behind is that when accessing the same URL
        - direct naviagation to that URL (or via reload)
        - and via navigation through `Link` from another route
        - *** CAN RESULT IN 2 DIFFERENT PAGES *** despite sharing the same URL
        - which can be abused to achieve the 2 applications shown above.

2. What are the conventions?
    <div>
        <img src='/images/intercepted-routes-modal-example.avif' />
    </div>

    - `(..)photo` and `photo` folder both share the same URL 
    - However, they will have different page display
    - `(..)photo` will be displayed when that URL is accessed from `/feed`
        - a `@modal slot` is used here, is likely because when the URL is accessed this way, it will be displayed as a component in parallel to the `children` slot.
        - For better demonstration, you may refer to this video which showcases how `parallel intercepted routing` can be achieved: https://youtu.be/U6aRqv7rzQ8
    - `photo` will be displayed when that URL is accessed directly or refresh

3. All the conventions:
    * (.) refers to the current route segment (URL wise)
    * (..) refers to one segment / level above (URL wise)
    * (..)(..) refers to two segment / level above (URL wise)
    * (...) refers to the `app directory`
    * These conventions consider the relation level in term of URL not the file directories hierarchy (which is why `@modal` is ignored in the above illustration)
# NextJS

## Route Handler / `GET`
1. Create an REST API endpoint
    - Create a file: `/app/user/route.ts`
    - `route.ts` is just like `page.ts` in page router

    ```ts
    export async function GET(){
        return new Response('user data')
    }
    ```

    - By accessing URL `/user`, will return `'user data'`
    - Important to note that
        - it is a `.ts` file - given not returning React Component
        - not returning a default function
        - The function is async

2. Return in `json` format
    ```ts
    import {comments} from '@/app/comments'
    export async function GET(){
        return new Response.json(comments)
    }
    ```

    - By default `return new Response.json()` returns a status code of `200`

## POST
1. Code
    ```ts
    import {comments} from '@/app/comments'
    export async function POST(request: Request){
        const comment = await request.json()
        const newComment = {
            id: comments.length + 1,
            text: comment.text
        }
        comments.push(newComment)
        return new Response(
            JSON.stringify(newComment),
            {
                headers: {'Content-Type': 'application/json'},
                status: 201
            }
        )
    }
    ```

2. In demonstrating, we are using a simple `array of object` to represent the database for the sake of simplicity.
    - Status 201 is more appropriate in a successful POST request
    - Alternatively, `Response.json()` can also be used (But must remember to change the returned status code)
    - READ MORE about `Response`: https://developer.mozilla.org/en-US/docs/Web/API/Response


## Dynamic Route Handler with `params` / `PATCH` & `DELETE`
1. Create the directories as following
    ```
    - app
        - comments
            - [id]
                - route.ts
    ```

2. Code
    ```ts
    import {comments} from '@/app/comments'

    type ContextType = {
        params: Promise<{
            id: string
        }>
    }
        

    export async function GET(_request: Request, { params }: ContextType){
        const { id } = await params
        const comment = comments.find(comment => comment.id === parseInt(id))
        return Response.json(comment)
    }
    ```

3. Similarly to in Page Router, `params` is in the `props` object
    - in route handler, `params` is in the `context` object

4. Apply Dynamic Route Handlers to `PATCH` request and `DELETE` request
    ```ts
    import {comments} from '@/app/comments'

    type ContextType = {
        params: Promise<{
            id: string
        }>
    }

    // handle PATCH
    export async function PATCH(request: Request, {params}: ContextType){
        const { text } = await request.json() // assuming incoming request has a body of {'text': 'new comment'}
        const { id } = await params

        const targetComment = comments.find(comment => comment.id === parseInt(id))
        targetComment.text = text
        return Response.json(targetComment)
    }

    // handle DELETE
    export async function PATCH(_request: Request, {params}: ContextType){
        const { id } = await params

        const targetIdx= comments.findIndex(comment => comment.id === parseInt(id))
        const deletedComment = comments.splice(targetIdx, 1)
        return Response.json(deletedComment)
    }
    ```

5. For `PATCH` and `DELETE` request, can consider returning status of
    - `200` - ok
    - `204` - intentionally no content


## Dynamic Route Handler - `searchParams`
1. Thus far, we have type request parameter as `Request`, however it can also be typed with `NextRequest` which has more functionalities.
    - READ MORE: https://nextjs.org/docs/app/api-reference/functions/next-request
    - For demonstration, this showcase how a `NextRequest` type can access the `searchParams`
        - The documentation on this part can be confusing.
        - `request.nextUrl.searhParams` returns a `URLSearchParams`, for reference: https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams


2. Code
    ```ts
    import { comments } from '@/app/comments/data'
    import { type NextRequest } from 'next/server'

    // Assuming that incoming request is at '/app/comments?query=first'
    export async function GET(request: NextRequest) {
        const searchParams = request.nextUrl.searchParams
        const query = searchParams.get('query')
        const filteredComment = query ?
            comments.filter(comment => comment.text.includes(query)) :
            comments

        return Response.json(filteredComment)
    }
    ```

## Headers

* READ MORE: https://nextjs.org/docs/app/api-reference/functions/headers

1. code
    ```ts
    import { headers } from 'next/headers'
 
    export default async function Page() {
        const headersList = await headers()
        const userAgent = headersList.get('user-agent')
    }
    ```


## Cookies

1. Conventionally, `cookies` can be set as following (*** on the server ***)
    - Where the cookie is set using `Set-Cookie` in the `header`
    - Therefore, it technically does not stop the user from sending a request with the same setup in the header to change the cookie value
    - READ MORE: https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Set-Cookie

    ```ts
    export async function GET(){
        return new Response(
            JSON.stringify({'data': 'random_data'}),
            {
                status: 200,
                headers: {
                    'Content-Type': 'application/json',
                    'Set-Cookie': 'theme=dark' // the syntax
                }
            }
        )
    }
    ```

2. A caveat with `cookie` usage
    - It will be sent to the server on every request, therefore it should be kept lightweight and rarely used outside authentication purpose.

3. With NextJS, the `cookie` is set as following:

    * The following example only show how to set them. Refer to the docs for further details on how to read them.

    * Option 1: using `NextRequest`
        - READ MORE: https://nextjs.org/docs/app/api-reference/functions/next-request

        ```ts
        export async function GET(request: NextRequest){

            request.cookies.set('show-banner', 'false')

            return Response.json('some random data')
        }
        ```
    
    * Option 2: using `cookies`
        - READ MORE: https://nextjs.org/docs/app/api-reference/functions/cookies

        ```ts
        import { cookies } from 'next/headers'

        export async function GET() {
            const cookieStore = await cookies()
            const theme = cookieStore.set('show-banner', 'false')
            return '...'
        }
        ```
    
    * What is the difference between these 2 options?
        - `NextRequest` is only limited to the route handlers.
        - `Cookie` can be used in both route handles (in sending out response) and page router (for sending out request)
            - `Cookie` also provide the access to set more advanced option in cookie such as `same-site`, `secure` etc...


## `NextRequest` & `NextResponse`
1. Both of these api extends the basic Javascript API of `request` and `response`
    - Therefore, while using API, we can still refer to their fundamental version and identify if there's any other functionalities that can be used.
    - Since `NextRequest` has been extensively discussed in previous session, here we will focus on discussing `NextResponse`
    - READ MORE (`NextRequest`): https://nextjs.org/docs/app/api-reference/functions/next-request
    - READ MORE (`NextResponse`): https://nextjs.org/docs/app/api-reference/functions/next-response

2. `NextResponse.cookies`
    - just like `NextRequest`, it also has an `cookies` api to `set`, `get`...

3. `NextResponse.redirect()` and `NextResponse.rewrite()`

    * `Redirect`

        ```ts
        import {type NextRequest, NextResponse } from 'next/server'
        export async function proxy(request: NextRequest){
            return NextResponse.redirect(new URL('/new', request.url))
        }
        ```
        - We will discuss `proxy` in a later section
        - `NextResponse.redirect()` expects a `URL constructor` (READ MORE: https://developer.mozilla.org/en-US/docs/Web/API/URL/URL)
        - `URL constructor` took in 2 parameters
            - url(`'/new'`) to add onto the base url (`request.url`) 
    
    * `Rewrite`
        ```ts
        import {type NextRequest, NextResponse } from 'next/server'
        export async function proxy(request: NextRequest){
            return NextResponse.rewrite(new URL('/new', request.url))
        }
        ```

        - `Rewrite` is similar to `Redirect` except that it doesnt change the displayed URL
        - useful when trying to redirect user from an outdated API endpoint to an updated API endpoint

4. `NextResponse.next()`
    - allows returning of a response but not to the client, instead allowing for continue navigation to find the matching route handler
    - pretty much like `next()` in node.js
    - usually used in `proxy` (which was previously named as `middleware`)


## `Proxy` / `Middleware` 

1. `Middleware` is renamed to `Proxy`
    - You may read more to find out why: https://nextjs.org/docs/messages/middleware-to-proxy

2. How to setup a `Proxy`?
    - There should only ever be 1 `proxy` for entire project, which exists in the form of `proxy.ts` at the root level of this file.
    - Inside the file, it should export a function named `proxy`
    - It functions similar to the `middleware` of `express.js`, but a lot more restrictive.
    - Think of it as doing something before passing on the request to the matching handler.
    - READ MORE: https://nextjs.org/docs/app/api-reference/file-conventions/proxy

    - However, it is not always active. It can be set to active by using 1 of the following 2 methods:
        - matcher
            ```ts
            import { NextResponse, NextRequest } from 'next/server'

            // This function can be marked `async` if using `await` inside
            export default function proxy(request: NextRequest) {
                return NextResponse.redirect(new URL('/home', request.url))
            }
                
            export const config = {
                matcher: '/about/:path*',
            }
            ```
            * READ MORE: https://nextjs.org/docs/app/api-reference/file-conventions/proxy#matcher

        - conditional statement by checking the pathname
            ```ts
            import {NextRequest, NextResponse} from 'next/server'

            export default function proxy(request: NextRequest){

                if(request.nextUrl.pathname === '/profile') {
                    return NextResponse.redirect(new URL('/', request.url))
                }
            }
            ```

3. Making use of `NextResponse.next()`
    ```ts
    import {NextRequest, NextResponse} from 'next/server'

    export default function proxy(request: NextRequest){

        const response = NextResponse.next()

        if(request.nextUrl.pathname === '/profile') {
            /* 
                Make use of the response api to set header or cookies etc...
            */
           return response // This doesnt send back to the client, but instead it continues to look for the matching route handler
        }

    }
    ```




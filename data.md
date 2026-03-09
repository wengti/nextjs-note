# NextJS

## Request Memoization
1. In server component, the result of `GET` fetch request will get cached and therefore you can call multiple of the same fetch command, but it will only be executed for once and share with the rest of other call.
2. Refer to `cache.md`

## Sequential and Parallel Data Fetching
1. When you have to call multiple API fetching and they are not dependent on one another, it is a good practice / consideration to use parallel fetching
    - READ MORE on parallel data fetching: https://nextjs.org/docs/app/getting-started/fetching-data#parallel-data-fetching

2. Here are the example of *** Sequential Data Fetching ***
    ```ts
    import { getArtist, getAlbums } from '@/app/lib/data'
 
    export default async function Page({ params }) {
        // These requests will be sequential
        const { username } = await params
        const artist = await getArtist(username)
        const albums = await getAlbums(username)
        return <div>{artist.name}</div>
    }
    ```

    - `getAlbums` is executed after `getAlbums`, which can be slow.

3. Here is the example of Parallel Data Fetching - achieved via `Promise.all` - READ MORE: 
    ```ts
    import Albums from './albums'
 
    async function getArtist(username: string) {
    const res = await fetch(`https://api.example.com/artist/${username}`)
        return res.json()
    }
    
    async function getAlbums(username: string) {
    const res = await fetch(`https://api.example.com/artist/${username}/albums`)
        return res.json()
    }
    
    export default async function Page({params}: {params: Promise<{ username: string }>}) {
        const { username } = await params
        
        // Initiate requests
        const artistData = getArtist(username)
        const albumsData = getAlbums(username)
        
        const [artist, albums] = await Promise.all([artistData, albumsData]) //making use of Promise.all
        
        return (
            <>
                <h1>{artist.name}</h1>
                <Albums list={albums} />
            </>
        )
    }
    ```

## Data Updating

* READ MORE: https://nextjs.org/docs/app/getting-started/updating-data

1. *** Server Action ***
    - *** Server Action *** is a type of Server Function , where Server Function is an asynchronous function that runs only on the server.
        - In the context of data mutation / updating, they are known as *** Server Action ***
    
    - Behind the scene, Server Action uses `POST` method.
        - However, it doesnt mean that it is only strictly limited to just adding new resources to the database.
        - It is just that `POST` method is the HTTP communication method.
        - Within the function, the users are free to define functions that will add, edit or delete a resource from the database.
    
    - Without Server Action, when a user wants to mutate the data, the implementation flow goes like this:
        - setup an API endpoint that receives `POST` / `PATCH` / `DELETE` request and handle the corresponding task
        - on the client side, when triggered by the user, send a request to that defined endpoint
        - API endpoint returns the response to the client side
        - WATCH: https://youtu.be/F8DB4LM1dME to see how cumbersome it can be and how server action massively simplify this flow.

2. How to Create a *** Server Function *** with `use server`
    - It can be marked as a server function by writing `use server` on a separate file
        - which will mark all the exported function as a Server Function
    
    - It can also be marked at the top of the scope of a function, i.e.
        ```ts
        export async function deletePost(){
            'use server'
            //
            // what to do in this function...
        }
        ```
    
    - If in a *** Server Component ***, it can also be defined directly inline, i.e.
        ```ts
        export async function AboutPage(){

            async function deletePost(){
                'use server'
                //
                // what to do in this function
            }
        }
        ```
    
    - Meanwhile, in a *** Client Component ***, the server action cant be defined directly. 
        - However, it can be imported.
            ```ts
            'use client'

            import { createPost } from '@/app/actions' //createPost is a server action defined in a separate file

            export function Button() {
                return <button formAction={createPost}>Create</button>
            }
            ```
        
        - It can even be passed in as a props.
            ```ts
            // ClientComponent expecting a server action to be passed in as a props
            'use client'

            export default function ClientComponent({updateItemAction}: {updateItemAction: (formData: FormData) => void}) {
                return <form action={updateItemAction}>{/* ... */}</form>
            }
            ```

3. Invoking a *** Server Action ***

    - A server action can be invoked as following:
        - Server Component: Form
        - Client Component: Form, Event Handler, useEffect
    
    - *** Forms ***
        - Pass the server action into form as its action

        - Code:
        ```ts
        import { createPost } from '@/app/actions'

        export function Form() {
            return (
                <form action={createPost}> //createPost is a server action which get access to the formData
                    <input type="text" name="title" />
                    <input type="text" name="content" />
                    <button type="submit">Create</button>
                </form>
            )
        }
        ```

        - There are times when the action is better passed into the button within the form instead of to the form directly
            - i.e., to make only that button a client component instead of the entire form
            - To do this, make use of the `formAction` attribute of the button
    
    - *** Event Handler ***

        - Code
            ```ts
            'use client'

            import { incrementLike } from './actions'
            import { useState } from 'react'

            export default function LikeButton({ initialLikes }: { initialLikes: number }) {
                const [likes, setLikes] = useState(initialLikes)
                
                return (
                    <>
                        <p>Total Likes: {likes}</p>
                        <button
                            onClick={async () => {
                            const updatedLikes = await incrementLike() //server action to update the like count in the database
                            setLikes(updatedLikes) // set it to update the like count on UI
                            }}
                        >
                            Like
                        </button>
                    </>
                )
            }
            ```

## UI/UX enhancement with DATA UPDATING
1. Showing a Pending State
    - make use of the `useActionState` hook in react which provide a `pending` and `error` state 
    - but would require the explicit declaration of that component to be Client Component

    ```ts
    'use client'
 
    import { useActionState, startTransition } from 'react'
    import { createPost } from '@/app/actions'
    import { LoadingSpinner } from '@/app/ui/loading-spinner'
    
    export function Button() {
        const [state, action, pending] = useActionState(createPost, false)
        
        return (
            <button onClick={() => startTransition(action)}> // Refer to the explanation below for why startTransition is needed
            {pending ? <LoadingSpinner /> : 'Create Post'}
            </button>
        )
    }
    ```

    - Why is `startTransition` is needed?
        - This is because the action defined in `useActionState` is meant to be used as the action for a `form` element.
        - Within `form`, `startTransition` is handle automatically under the hood.
        - In the above implementation, as it can be seen the action is not passed into a form, but into a button instead.
        - `startTransition` effectively tells React that this state update is a low-priority transition
            - so as it is updated, there's no need to block the current UI from being interactive
            - which is really the primary use case of when you would use `startTransition`
        - More importantly, without being wrapped in a `startTransition`, `pending` will stay false all the time.

2. Refreshing
    - code
        ```ts
        'use server'
        import { refresh } from 'next/cache'
        export async function updatePost(formData: FormData) {
            // Update data
            // ...
            
            refresh()
        }
        ```
    - But this only refresh the client router cache
    - Therefore, to ensure rerender on the server side, `revalidation` may be needed as discussed below.

3. Revalidating
    - code
        ```ts
        import { revalidatePath } from 'next/cache'
        
        export async function createPost(formData: FormData) {
            'use server'
            // Update data
            // ...
            
            revalidatePath('/posts')
        }
        ```

4. Redirecting
    - Code
        ```ts
        'use server'

        import { revalidatePath } from 'next/cache'
        import { redirect } from 'next/navigation'
        
        export async function createPost(formData: FormData) {
            // Update data
            // ...
            
            revalidatePath('/posts')
            redirect('/posts')
        }
        ```
    
    - because `redirect` is executed by throwing errors where any code after it wont run
    - `revalidate`, if needed, needs to be executed in advance


## OPTIMISTIC UPDATE WITH `useOptimistic`
* READ MORE: https://react.dev/reference/react/useOptimistic

1. Use Case
    - WATCH: https://youtu.be/ipmfUw8I2qc
    - This video suggests that `Optimistic Update` can be used in the case of deleting an object.
    - While the `server action` of deleting it from the database will take a while to be completed
    - Instead of waiting it to fetch back the updated database after the deletion action
    - We can define a state that will showcase the expected output in advance while that action is happening

2. Code
    ```ts
    const [optimisticProducts, setOptimisticProducts] = useOptimistic(
        products, //default value of the state
        (currentProducts, productId) => {
            return currentProducts.filter((product) => product.id !== productId);
        } //pass in extra arguments to decide how the optimistic state is set
    );

    const removeProductById = async (productId: number) => {
        setOptimisticProducts(productId); // set the expected optimistic state
        await removeProduct(productId); // let the action happens on the background
    };
    ```

3. `setOptimisticProducts` can be used just like any other `stateSetter` function in `useState`
    - in that it can either
        - directly set a value
        - use a callback function that takes in previous value of the state to set a value
    
4. Meanwhile, the second argument of `useOptimistic` also let you define another function that replace what the `setOptimisticProducts` will do
    - that take in the following arugments
        - First argument: current state (but no need defined when called)
        - Second or more argument: additional argument which can be used


## Form
- READ MORE: https://nextjs.org/docs/app/api-reference/components/form
- A progressive enhancement over the conventional `<form>` element in HTML

1. `action` field input of `<Form>`
    - when it is a `string`, it changes the URL and append the field input as query parameters to it
        - when it is used this way, the following features are granted:
            - Prefetches the path indicated in the `string` (including the `layout.tsx` and `loading.tsx`)
    - using a `string` can also be used in the `formAction` in the form `button`, but it does not support prefetching.

    ```ts
    import Form from 'next/form'
 
    export default function Page() {
        return (
            <Form action="/search">
                {/* On submission, the input value will be appended to
                    the URL, e.g. /search?query=abc */}
                <input name="query" />
                <button type="submit">Submit</button>
                </Form>
        )
    }
    ```

    - when it is a function, it becomes the typical React form that we have used thus far

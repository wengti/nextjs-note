# NextJS

## Request Memoization
1. In server component, the result of `GET` fetch request will get cached and therefore you can call multiple of the same fetch command, but it will only be executed for once and share with the rest of other call.
2. Refer to `cache.md`

## Parallel Data Fetching
1. When you have to call multiple API fetching and they are not dependent on one another, it is a good practice / consideration to use parallel fetching
    - READ MORE on parallel data fetching: https://nextjs.org/docs/app/getting-started/fetching-data#parallel-data-fetching

2. Here are the example of Sequential Data Fetching
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
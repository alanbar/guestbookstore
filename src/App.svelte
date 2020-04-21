<script>
    import firebase from "firebase/app" 
    import "firebase/auth"
    import config from './firebase/config'
    import Guestbook from  "./firebase/Guestbook.svelte"
    import Profile from  "./firebase/Profile.svelte"
    
    import { store } from "./firebase/store.js" 
    

    if(!firebase.apps.length) {
        firebase.initializeApp(config) 
    }

    let user;
    
    async function login() {
        const result = await firebase.auth().signInWithPopup(new firebase.auth.GoogleAuthProvider())
        user = result.user
        store.set(user.displayName)
    }

    async function logout() {
        await firebase.auth().signOut()
        user = null
        store.set(null)
    }
</script>

{#if $store !== null}
    {$store}
    <Profile displayName={user.displayName} photoURL={user.photoURL} />
    <button on:click={logout}>Logout</button>
    <p></p>
    <Guestbook />
{:else}
    <button on:click={login}>Login</button>
    {$store}
{/if}


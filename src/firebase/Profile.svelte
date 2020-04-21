<script>
    //import Guestbook from  "./Guestbook.svelte"
    export let displayName
    export let photoURL
    
</script>

<div>
    <p>{displayName}</p>
    <img class="profile" src={photoURL} alt="user photo" width="50" height="50" />
</div>

<style>
    .profile {
        height: 50px;
        width: 50px;
        display: block
    }
</style> 
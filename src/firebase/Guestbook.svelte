<script>
    import firebase from "firebase/app" 
    import "firebase/firestore"
    import "firebase/storage"
    import { onMount } from "svelte"  // added to test initialisation
    
    // test add remote
    let emptyRecord = {title: "", file: null}
    let record = { ...emptyRecord}
    let records = []
    
    // guestbook collection s/cut
    const collection = firebase.firestore().collection("guestbook")
    const storageRef = firebase.storage().ref();
    
    onMount( () => {
        collection.onSnapshot(snap => (records = snap.docs.map(d => [d.id, d.data()])))
        //collection.onSnapshot(snap => (records = snap.docs))
        console.log('records: ' + records)
    })

    function submit(e) {
        console.log('record: ' +JSON.stringify(records))
         collection.add(record)
        
        record = { ...emptyRecord} // spreads objects onto clean record
        e.target.reset()
    }

    async function remove(id, file) {
        // interesting problem...storageRef picture may be used by another id ??? -ignore
        if (id) {await collection.doc(id).delete()}
        if (file) {await storageRef.child(file).delete()}
    }

    async function upload(e) {
        if(e.target.files && e.target.files.length > 0) {
            const file = e.target.files[0]
              
            if (window.BrowserImageResizer) {
                const blob = await window.BrowserImageResizer.readAndCompressImage(file, {  
                 quality: 0.8,
                 maxWidth: 200,
                 maxHeight: 200
            })
            await storageRef.child(file.name).put(blob) // put overwrites previous data
            } else {
                await storageRef.child(file.name).put(file)
            }
            record.file = file.name
              
        }
    }
</script>

<s-head>
    <script src="https://cdn.jsdelivr.net/gh/ericnograles/browser-image-resizer@2.0.1/dist/index.js"></script>
</s-head>


<form on:submit|preventDefault={submit} >
    <input bind:value={record.title} placeholder="Say something.." />
    <br/>
    <input type="file" accept="image/*" on:change={upload}/>
    <br/>
    <button type="submit">Submit</button>
</form>

<ul>
    {#each records as [id, rec] (id)} <!-- as returned by Snapshot line 13 & new adds line 19-->
    
    <li title={id} class="record">
        <button class="delete" on:click={() => remove(id, rec.file)}>delete</button>
        <p>{rec.title}</p>
        {#if rec.file}
            {#await storageRef.child(rec.file).getDownloadURL()}
                <p>loading......</p>
            {:then url}
                <img src={url} alt={rec.file} />
            {/await}
        {/if}
    </li>
    {/each}
</ul>


<style>
    .record {
        position: relative
    }

    .record .delete {
        position: absolute;
        top: 0px;
        right: 0px;
    }

</style>

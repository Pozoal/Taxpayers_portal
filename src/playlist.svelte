<script>
    import { createEventDispatcher } from "svelte";
    export let videos = [];
    let dispatch = createEventDispatcher();
    const date = new Date();
    function onVideoClick(video) {
        dispatch("play", video);
        
    }
    
</script>

<style>
</style>

<div class="playlist-container">
    {#each videos as video}
        <div class="border-bottom border-muted p-1" style="cursor: pointer">
            <div
                style="width:100%;"
                on:click={() => onVideoClick(video)}
                class="outer-frame">
                <p style="margin:1rem; font-weight:500" class="text-dark">
                    {video.title}
                </p>
            </div>
        </div>
    {/each}
</div>

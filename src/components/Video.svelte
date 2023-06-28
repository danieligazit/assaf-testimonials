<script>
    import { onMount } from "svelte";

  
    export let src; // Source URL of the video    
    let playbackRate = 1;
    let paused;
    let video;
    export let visible = false;

    $: if (visible) {
        playVideo();
    }

    const playVideo = () => {
        let isPlaying = video.currentTime > 0 && !video.paused && !video.ended 
            && video.readyState > video.HAVE_CURRENT_DATA;
        
        if (!isPlaying){
            video.play();
        }
    }
    var checkScrollSpeed = (function(settings){
        settings = settings || {};
    
        var lastPos, newPos, timer, delta, 
            delay = settings.delay || 50; // in "ms" (higher means lower fidelity )
    
        function clear() {
            lastPos = null;
            delta = 0;
        }
    
        clear();
        
        return function(){
        newPos = window.scrollY;
        if ( lastPos != null ){ // && newPos < maxScroll 
            delta = newPos -  lastPos;
        }
        lastPos = newPos;
        clearTimeout(timer);
        timer = setTimeout(clear, delay);
        return -delta;
    };

})();

let timer = null;

const handleScroll = () => {
    let rawPlaybackRate = Math.abs(checkScrollSpeed()/12);
    if (visible){
        playVideo();
    }

    let normalizedPlaybackRate = rawPlaybackRate > 10 ? 10 : rawPlaybackRate;
    playbackRate = normalizedPlaybackRate < 0.07 ? 0.07 : normalizedPlaybackRate;


    if(timer !== null) {
        clearTimeout(timer);        
    }
    timer = setTimeout(function() {
        video.pause();
    }, 150);

    
}



</script>

<svelte:window on:scroll={handleScroll} />

<div class="video-container">
<video
    bind:this={video}  
    bind:playbackRate={playbackRate}
    src={src}
    preload="auto"
    muted
>
    <track kind="captions">

</video>
</div>

<style>
.video-container {
    position: relative;
    overflow: auto;
    max-width: 100%;
    max-height: 100vh;
}
.video-container video {
  /* Make video to at least 100% wide and tall */
  min-width: 100%; 
  min-height: 100%; 

  /* Setting width & height to auto prevents the browser from stretching or squishing the video */
  width: auto;
  height: auto;

  /* Center the video */
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%,-50%);
}
</style>

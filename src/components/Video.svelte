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
    }, 250);

    
}



</script>

<svelte:window on:scroll={handleScroll} />

<div class="video-container">
<video
    bind:this={video}  
    bind:playbackRate={playbackRate}
    
    preload="auto"
    muted
    
>
    <track kind="captions">
    <source src={src} type="video/mp4"/>

</video>
</div>

<style>
.video-container {
    height: 100vh;
	overflow: hidden;
	position: relative;
}
.video-container video {
  /* Make video to at least 100% wide and tall */
  /* max-width: 100%;  */
  /* max-height: 100%;  */
  min-width: 100%; 
  min-height: 100%; 
  
  width: 100%;
  /* height: auto; */
  object-fit: cover;
  /* position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%,-50%); */
}
</style>

<script>
    import { onMount } from "svelte";
  
    export let src; // Source URL of the video
    export let pos; // Index of the frame to display
    let currentTime, duration;
  
    onMount(() => {
    //   context = canvas.getContext("2d");
      // Draw the frame to the canvas when video is in the correct position
    //   video.addEventListener("seeked", () => {
    //     context.drawImage(video, 0, 0, canvas.width, canvas.height);
    //   });
    });
    
    const setTime = (time) => {
        currentTime = time;
    }

    $: {
        if (pos !== undefined) {
            console.log(pos, duration);
            setTime( pos * duration);
        }
    }
  </script>
  

<div class="container"  >
    <video
        bind:currentTime={currentTime}
        bind:duration={duration}
        preload="preload"
        
        playsInline
    >
    <source
              type="video/mp4"
              src={src}
            />
    <track kind="captions" />
    </video>
</div>
  <!-- <canvas bind:this={canvas} width="640" height="360"></canvas> -->





<style>

	.container {
		/* width: 100vw; */
		height: 100vh;
		overflow: hidden;
		position: relative;
	}


</style>
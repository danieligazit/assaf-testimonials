<script>
    import { fade } from 'svelte/transition';
	import Timeline from './Timeline.svelte'
	import Slideshow from './Slideshow.svelte';
	let text;
	let newText;
	let firstLoad = true;
	let crossfading = false;
	export let currentText;
    export let duration = 600;
	export let timeline;
	export let timelineIndex;
	export let imageUrl;
	let textToId = {

	}

	let textIdCounter = 0

	const  sleep = (ms) => new Promise(r => setTimeout(r, ms));

	async function handleFetch(t) {
		let id = textToId[t] || (() => {
            textToId[t] = textIdCounter;
            let returnValue = textIdCounter;
            textIdCounter++;
            return returnValue;
        })();
		const f = {id, t : t}
		if (!text) { // init
			text = f;
			newText = f;
			return;
		}

		newText = f;
	}
	async function onLoad(){
		if (newText.id === text.id) {
			firstLoad = true;
			return;
		}	
		crossfading = true;
		await sleep(duration);
		text = newText;
		crossfading = false;
	}

	$: {
		handleFetch(currentText);
        onLoad();
	};
</script>

<div class="all" style="position: relative;">
<div style="position: absolute; z-index: -1;">
	<Slideshow imageUrl={imageUrl}/>
</div>

<div class="wrapper" style="z-Index: 1;">
	<div class="container" class:firstLoad >
		{#if text}
			{#if !crossfading}
			<figure class="orig">
				<p out:fade={{duration: duration}}>{text.t}</p>
			</figure>
			{/if}
			{#if newText.id !== text.id}
				<figure class="new" class:crossfading >
					<p in:fade={{duration: duration}}>{newText.t}</p>     
				</figure>
			{/if}
		{/if}

	</div>
	<div class="timeline-footer" style="z-index: 1;">
		<Timeline timeline={timeline} timelineIndex={timelineIndex}></Timeline>
	</div>	

</div>
</div>


<style>
	.wrapper {
		display: flex;
		flex-direction: column;
		height: 65vh;
		/* background-color: rgba(0, 0, 0, 0); */
		background-color: transparent;
		border: 1px solid #CCCCCC; /* a subtle border */
		border-radius: 5px; /* rounded corners for a softer look */
		padding: 20px; /* some spacing inside the wrapper */
	}
	
	.timeline-footer {
		width: 100%;
		height: 100px;
		position: relative;
		background-color: transparent;
		border-top: 1px solid #CCCCCC; /* a subtle border */
		padding: 10px; /* some spacing inside the timeline-footer */
	}
	
	.container {
		height: 100vh;
		overflow: hidden;
		position: relative;
		padding: 10px; /* some spacing inside the container */
	}
	
	figure {
		position: absolute;
		width: 100%;
		height: 100%;
		display: flex; /* set display to flex */
		align-items: center; /* vertically center the content */
		justify-content: center; /* horizontally center the content */
		margin: 0; /* remove any default margins */
	}
	
	p {
		color: #333333; /* a dark grey color for text */
		transition: all 1s ease;
		object-fit: cover;
		width: 100%;
		height: 100%;
		font-size: 16px; /* a readable font size */
	}
	
	.firstLoad figure.orig p {
		opacity: rgba(0, 0, 0, 1);
	}
	
	figure.new {
		z-index: 1;
	}
	
	figure.new.crossfading p {
		color: #333333;
	}
	</style>
	
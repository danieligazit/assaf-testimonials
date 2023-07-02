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
<div style="position: absolute; display: block; width: 100%; max-width:100%">
	<div style="position: relative; z-index: 1; display: block">
		<Slideshow imageUrl={imageUrl}/>
	</div>
</div>

<div class="wrapper" style="position: relative; z-index: 2; display: block">
	<div class="container" class:firstLoad >
		<div class="box">
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
	</div>
	<div class="timeline-footer" style="z-index: 2;">
		<Timeline timeline={timeline} timelineIndex={timelineIndex}></Timeline>
	</div>	

</div>
</div>


<style>
	.wrapper {
		display: flex;
		flex-direction: column;
		height: 80vh;
		/* background-color: rgba(0, 0, 0, 0); */
		background-color: transparent;
		border-radius: 5px; /* rounded corners for a softer look */
	}
	
	.timeline-footer {
		width: 100%;
		height: 100%;

		background-color: rgba(40, 40, 40, 0.7); 
		border-top: 1px solid #a0a0a0; /* a subtle border */
	}
	
	.box {
		vertical-align: middle;
		display: flex;
    	
    	justify-content: center;
		text-align: center;
		background-color: rgba(40, 40, 40, 0.6);
		padding: 1em; /* add space around the content */
		min-height: 7%;
		min-width: 60%;
		width: auto; /* allow the box to expand horizontally */
		height: auto; /* allow the box to expand vertically */
		margin-top: 60vh;
	}
	.container {
		height: 100%;
		overflow: hidden;
		position: relative;
		margin-left: auto;
		margin-right: auto;
		display: flex;
		justify-content: center; /* horizontal centering */
		align-items: center;
		text-align: center;
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
		color: #ffffff; /* a dark grey color for text */
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
		color: #ffffff;
	}
	</style>
	
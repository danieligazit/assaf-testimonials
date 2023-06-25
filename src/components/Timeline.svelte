<script>
	import { themes } from '../config.js';
	import { getContext } from 'svelte';

	export let theme = getContext('theme');

	import { onMount } from 'svelte'
	import { quadInOut } from 'svelte/easing'
	import { tweened } from 'svelte/motion'
	
	let timeline = [
		{ year: 1995, label: "DHTML" },
		{ year: 2006, label: "jQuery" },
		{ year: 2010, label: "Angular 1" },
		{ year: 2013, label: "React" },
		{ year: 2019, label: "Svelte3" },
	]
	const width = 900
	
	let duration = 1000
	let index = tweened(0, {duration, easing: quadInOut})
	let dotSize = tweened(3, {duration: 200, easing: quadInOut})
	
	export let timelineIndex;

    $: {
        
        $index=timelineIndex;
        
    }

    
</script>

<div class="timeline-container">

<svg viewBox="{-60+$index*200} 0 {width/timeline.length} 120">
	<line x1=20 y1=95 x2={30+($index*200)} y2=95 stroke=black/>
	
	{#each timeline as event, i}
		<g>
			<text class="label" x={25+i*40*timeline.length} y=20 text-anchor="middle">{event.label}</text>
			<text class="year" x={25+i*40*timeline.length} y=60 text-anchor="middle">{event.year}</text>
			<circle cx={25 + i*40*timeline.length} cy=95 r={$dotSize}/>
		</g>
	{/each}
	
</svg>
</div>

<style>
    .timeline-container {
        height: 50vh;
        width: 100%; /* You can adjust this as needed */
        margin: 0;
        position: absolute;
        top: 50%;
        -ms-transform: translateY(50%);
        transform: translateY(50%);
    } 
	/* :global(body) {
        width: 100%;
        height: auto;
		display: flex;
	} */
	svg {
		overflow: visible;
        height: 100%;
        width: auto; /* This will maintain the aspect ratio of the SVG */
	}
	text {
		fill: #555;
	}
	.year {
		font-size: 2rem;
		font-weight: 400;
	}
	.label {
		fill: #666;
	}
	line {
		stroke-width: 5px;
		stroke: #ccc;
		stroke-dasharray: 2 2;
	}
	circle {
		fill: orange;
	}
</style>
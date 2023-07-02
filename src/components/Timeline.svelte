<script>
	import { themes } from '../config.js';
	import { getContext } from 'svelte';

	export let theme = getContext('theme');

	import { onMount } from 'svelte'
	import { quadInOut } from 'svelte/easing'
	import { tweened } from 'svelte/motion'
	
	let timeline = [
		{ year: 2004, label: "Nov. 3rd" },
		{ year: 2004, label: "Dec. 23rd" },
		{ year: 2005, label: "Apr. 8th" },
		{ year: 2006, label: "March 22nd" },
		{ year: 2006, label: "April 10th" },
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
	<line x1=20 y1=50 x2={30+($index*200)} y2=50 stroke=black/>
	
	{#each timeline as event, i}
		<g>
			<text class={$index === i ? "selected-label" : "label"} x={25+i*40*timeline.length} y=0 text-anchor="middle">{event.label}</text>
			<text class={$index === i ? "selected-year" : "year"} x={25+i*40*timeline.length} y=35 text-anchor="middle">{event.year}</text>
			<circle cx={25 + i*40*timeline.length} cy=50 r={$dotSize}/>
		</g>
	{/each}
	
</svg>
</div>

<style>
    .timeline-container {
        height: 27vh;
        width: 100%; /* You can adjust this as needed */
        margin: 0;
        position: absolute;
        top: 90%;
        -ms-transform: translateY(50%);
        transform: translateY(50%);
    } 

	svg {
		overflow: visible;
        height: 100%;
        width: auto; /* This will maintain the aspect ratio of the SVG */
	}
	text {
		fill: #a2a2a2;
	}
	.year {
		font-size: 2rem;
		font-weight: 400;
	}
	.label {
		fill: #808080;
	}
	.selected-year {
		fill: #ffffff;
		font-size: 2rem;
		font-weight: 400;
		transition: 0.5s;
	}
	.selected-label {
		fill: #ffffff;
		transition: 0.5s;
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
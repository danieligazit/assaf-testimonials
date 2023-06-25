<script>
	import { onMount } from 'svelte';	

	let photo;
	let newPhoto;
	let firstLoad = true;
	let crossfading = false;
	export let imageUrl;
	export let imageId;

	let imageUrlToId = {

	}

	let imageIdCounter = 0

	const  sleep = (ms) => new Promise(r => setTimeout(r, ms));

	async function handleFetch(url, id) {
		id = id ? id : (() => {
			return imageUrlToId[url] || (() => {
				imageUrlToId[url] = imageIdCounter;
				let returnValue = imageIdCounter;
				imageIdCounter++;
				return returnValue;
			})();
		})();

		const f = {id, url: url}
		console.log('f', f)
		if (!photo) { // init
			photo = f;
			newPhoto = f;
			return;
		}
		newPhoto = f;
	}
	async function onLoad(){
		console.log('onLoad')
		if (newPhoto.id === photo.id) {
			firstLoad = true;
			return;
		}	
		crossfading = true;
		await sleep(1000);
		photo = newPhoto;
		crossfading = false;
	}

	$: {
		handleFetch(imageUrl, imageId);
	};
</script>



<div class="container" class:firstLoad >
	{#if photo}
		<figure class="orig">
			<img src={photo.url} alt="" on:load={onLoad} >
		</figure>
		{#if newPhoto.id !== photo.id}
		<figure class="new" class:crossfading >
				<img src={newPhoto.url} alt="" on:load={onLoad}>
			</figure>
		{/if}
	{/if}
</div>
<style>
	body, html {
		margin: 0;
		padding: 0;
		overflow-x: hidden; /* Hide horizontal scrollbar */
	}

	.container {
		/* width: 100vw; */
		height: 100vh;
		overflow: hidden;
		position: relative;
	}
	figure {
		position: absolute;
		width: 100%;
		height: 100%;
	}
	img {
		opacity: 0;
		transition: all 1s ease;
		object-fit: cover;
		width: 100%;
		height: 100%;
	}
	.firstLoad figure.orig img {
		opacity: 1;
	}
	figure.new {
		z-index: 1;
	}
	figure.new.crossfading img {
		opacity: 1;
	}

</style>
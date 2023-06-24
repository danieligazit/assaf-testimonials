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

	let images = [
    	'https://cdn.abcotvs.com/dip/images/1238804_030916-mba-baby-otter-img.jpg',
    	'https://www.travelandleisure.com/thmb/FUsS1wKYFV6LQC8A1c5TAqEvAFg=/750x0/filters:no_upscale():max_bytes(150000):strip_icc():format(webp)/header-otter-pup-aqaurium-of-the-pacific-NAMEOTTER0822-1ca395679e384efead83c6f88eb4484d.jpg',
    	'https://cdn.vox-cdn.com/thumbor/oTE7ZTjbIW0grAUz6GYk6h6nRkI=/0x0:1502x1001/920x613/filters:focal(786x389:1026x629):format(webp)/cdn.vox-cdn.com/uploads/chorus_image/image/71045160/sn_2.0.jpeg'
	];
	
	const  sleep = (ms) => new Promise(r => setTimeout(r, ms));

	async function handleFetch(url, id) {
		id = id ? id : (() => {
			(url in imageUrlToId) ? imageUrlToId[url] : (() => {
				imageUrlToId[imageIdCounter];
				imageIdCounter++;
			})();
		})();

		const f = {id, url: url}
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
	.container {
		height: 10rem;
		width: 100%; 
  		height: 200px;
  		overflow: hidden;
	}
	figure {
		position: absolute;
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
	img {
		opacity: 0;
		transition: all 1s ease;
	}
</style>
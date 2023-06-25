<!-- App.svelte -->
<script>
	import { getMotion } from "./utils.js";
	import { themes } from "./config.js";
	import { setContext, onMount } from "svelte";
	import { getData, setColors, getTopo, getBreaks, getColor } from "./utils.js";
	import { colors, units } from "./config.js";
	import Scroller from "./components/Scroller.svelte";
	import Em from "./components/Em.svelte";
	import { Router, Route, Link } from 'svelte-routing';
	import { Map, MapSource, MapLayer, MapTooltip } from "@onsvisual/svelte-maps";
	import bbox from "@turf/bbox";
	import { scaleLinear } from "d3-scale";
    import Slideshow from "./components/Slideshow.svelte";
    import MapPin from "./components/MapPin.svelte";
	import Divider from "./components/Divider.svelte";
    import Timeline from "./components/Timeline.svelte";
    import Section from "./components/Section.svelte";
    import Panorama from "./components/Panorama.svelte";
	let data = {district: {}, region: {}};
	let metadata = {district: {}, region: {}};
	let imageUrl = 'images/south-sudan-refugee-camp-3.jpg';
	let regionsGeo;
	let geojson;
	
	let theme = "dark";
	setContext("theme", theme);

	const colorScale = scaleLinear()
      .domain([0, 1])
      .range(["#feedde", "#fd8d3c"]);

	const threshold = 0.65;
	let animation = getMotion(); // Set animation preference depending on browser preference
	let id = {}; // Object to hold visible section IDs of Scroller components
	let idPrev = {}; // Object to keep track of previous IDs, to compare for changes
	onMount(() => {
		idPrev = {...id};
	});

	const datasets = ["region"];
	
	const mapstyle = "https://bothness.github.io/ons-basemaps/data/style-omt.json";
	const mapbounds = {
		uk: [[ -9, 49 ], [ 2, 61 ]],
		eritrea: [[ 36.5, 12.4 ], [ 43.3, 18.1 ]],
		asmaraToSudan: [[ 36.0, 13.0 ], [ 36.4, 17.7 ]],
	};

	let displayJourney = false;
	
	const journey = {
		"type": "FeatureCollection",
		"features": [
			{
				"type": "Feature",
				"properties": {
					"name": "Journey from Asmara to Al Qadarif",
					"journey": true
				},
				id: "123",
				"geometry": {
                "type": "LineString",
					"coordinates": [
						[38.925051, 15.333333],  // Asmara
						[38.616667, 15.116667],  // Keren, Eritrea
						[37.59278, 15.54917],     // Agordat, Eritrea
						[37.215, 15.78556],       // Barentu, Eritrea
						[36.86667, 16.18333],     // Teseney, Eritrea
						[36.4, 17.7]              // Al Qadarif
					]
				}
			}
		]
	}

	let map = null; // Bound to mapbox 'map' instance once initialised

	// State
	let hovered; // Hovered district (chart or map)
	let selected; // Selected district (chart or map)
	let mapHighlighted = []; // Highlighted district (map only)
	let mapKey = "militaryPresence"; // Key for data to be displayed on map
	let explore = false; // Allows chart/map interactivity to be toggled on/off
	let mapPins = false;

	let timelineIndex = 0;
	let timeline = [
		{ year: 1995, label: "DHTML" },
		{ year: 2006, label: "jQuery" },
		{ year: 2010, label: "Angular 1" },
		{ year: 2013, label: "React" },
		{ year: 2019, label: "Svelte3" },
	]

	// Functions for chart and map on:select and on:hover events
	function doSelect(e) {
		selected = e.detail.id;
		if (e.detail.feature) fitById(selected); // Fit map if select event comes from map
	}
	function doHover(e) {
		hovered = e.detail.id;
	}

	// Functions for map component
	function fitBounds(bounds) {
		if (map) {
			map.fitBounds(bounds, {animate: animation, padding: 30});
		}
	}
	function fitById(id) {
		if (geojson && id) {
			let feature = geojson.features.find(d => d.properties.name == id);
			let bounds = bbox(feature.geometry);
			fitBounds(bounds);
		}
	}

	// Actions for Scroller components
	const actions = {
		timeline: {
			timeline01: () => {
				timelineIndex = 0;
			},
			timeline02: () => {
				timelineIndex = 1;
			},
			timeline03: () => {
				timelineIndex = 2;
			},
			timeline04: () => {
				timelineIndex = 3;
			},
			timeline05: () => {
				timelineIndex = 4;
			},
		},
		map: { // Actions for <Scroller/> with id="map"
			map01: () => { // Action for <section/> with data-id="map01"
				fitBounds(mapbounds.eritrea);
				mapKey = "militaryPresence";
				mapHighlighted = [];
				mapPins = false;
				displayJourney = false;
				explore = false;
			},
			map02: () => {
				fitBounds(mapbounds.eritrea);
				mapKey = "militaryPresence";
				mapHighlighted = [];
				explore = false;
				mapPins = true;
				displayJourney = false;

			},
			map03: () => {
				fitBounds(mapbounds.asmaraToSudan);				
				mapKey = "militaryPresence";
				mapHighlighted = [];
				mapPins = false;
				displayJourney = true;
				explore = false;
			},
		},

		slideshow: {
			slideshow01: () => {
				imageUrl = 'images/south-sudan-refugee-camp-3.jpg';
			},
			slideshow02: () => {
				imageUrl = 'images/south-sudan-refugee-camp-2.jpg';
			},
			slideshow03: () => {
				imageUrl = 'images/south-sudan-refugee-camp-5.jpg';
			},
			
		}
	};

		// Code to run Scroller actions when new caption IDs come into view
	function runActions(codes = []) {
		codes.forEach(code => {
			if (id[code] != idPrev[code]) {
				if (actions[code][id[code]]) {
					actions[code][id[code]]();
				}
				idPrev[code] = id[code];
			}
		});
	}
	$: id && runActions(Object.keys(actions)); // Run above code when 'id' object changes

	// INITIALISATION CODE
	datasets.forEach(geo => {
		getData(`./data/data_${geo}.csv`)
		.then(arr => {
			let meta = arr.map(d => ({
				code: d.name,
				name: d.name,
			}));
			let lookup = {};
			meta.forEach(d => {
				lookup[d.code] = d;
			});
			metadata[geo].array = meta;
			metadata[geo].lookup = lookup;

			let indicators = arr.map((d, i) => ({
				...meta[i],
				militaryPresence: d.militaryPresence,

			}));
			
			['militaryPresence'].forEach(key => {
				indicators.forEach((d, i) => indicators[i][key + '_color'] = colorScale(indicators[i][key]));
			});
		
			data[geo].indicators = indicators;
			
		});
	});

	getTopo("./data/eritrea-with-regions_.geojson")
		.then(geo => {
			geo.features.sort((a, b) => a.properties.name.localeCompare(b.properties.name));
			regionsGeo = geo;
			geojson = regionsGeo;
		});

</script>

<main>


<Scroller {threshold} bind:id={id['panorama']} index={0}>
	<!-- <div slot="background">
				<Panorama
	src="https://upload.wikimedia.org/wikipedia/commons/thumb/0/0a/Veste_Oberhaus_%28Passau%2C_full_spherical_panoramic_image%2C_equirectangular_projection%29.jpg/800px-Veste_Oberhaus_%28Passau%2C_full_spherical_panoramic_image%2C_equirectangular_projection%29.jpg?20180919194115" 
	alt="Pretty Sky" 
/>
		
	</div>
	<div slot="foreground">
		<section data-id="panorama01">
			<div class="col-medium">
				<p>
					baba
				</p>
			</div>
		</section>
		
	</div>
</Scroller> -->


	<div>
	<Scroller {threshold} bind:id={id['timeline']} index={0} splitscreen={true}>
		<div slot="background" style="position: relative;">
		    <!-- <div>
				<Slideshow imageUrl={imageUrl}/>
			</div> -->
			<div style="position: absolute;  top: 0; left: 0;">
				<Timeline timeline={timeline} timelineIndex={timelineIndex}></Timeline>
			</div>

		</div>
		<div slot="foreground">
			<section data-id="timeline01">
				<div class="col-medium">
					<p>
						"A fellow nurse at the hospital where I worked disappeared. Rumor had it she helped an army deserter, and the authorities took her away."
					</p>
				</div>
			</section>
			<section data-id="timeline02">
				<div class="col-medium">
					<p>
						"Government officials visited our hospital, scrutinizing our records and questioning our loyalties. A sense of fear started to permeate."
					</p>
				</div>
			</section>
			<section data-id="timeline03">
				<div class="col-medium">
					<p>
						"The authorities introduced new regulations, requiring us to report any suspicious activity among patients or staff. The climate of fear deepened."
					</p>
				</div>
			</section>
			<section data-id="timeline04">
				<div class="col-medium">
					<p>
						"I noticed unknown men observing the hospital from a distance. My colleagues whispered about 'eyes watching us'."
					</p>
				</div>
			</section>
			<section data-id="timeline05">
				<div class="col-medium">
					<p>
						"A trusted friend warned me, 'They're asking about you. It's not safe.' That night, I made the decision to leave with Aman."
					</p>
				</div>
			</section>
			
		</div>
	</Scroller>
    </div>
	
	{#if geojson && data.region.indicators}
	<Scroller {threshold} bind:id={id['map']}>
		<div slot="background">
			<figure>
				<div class="col-full height-full">
					<Map style={mapstyle} bind:map interactive={false} location={{bounds: mapbounds.eritrea}}>
						<MapSource
						  	id="lad"
						  	type="geojson"
						  	data={geojson}
						  	promoteId="name"
						  	maxzoom={13}>
						  	<MapLayer
							  	id="lad-fill"
							  	idKey="code"
							  	colorKey={mapKey + "_color"}
							  	data={data.region.indicators}
							  	type="fill"
								select {selected} on:select={doSelect} clickIgnore={!explore}
								hover {hovered} on:hover={doHover}
								highlight highlighted={mapHighlighted}
							  	paint={{
									'fill-color': [
										'case',
										['!=', ['feature-state', 'color'], null], ['feature-state', 'color'],
									  	'black'
								  	],
								  	'fill-opacity': [
										'case',
										['==', ['feature-state', 'type'], 'LineString'], 0,
									  	0.7
								  	]
							  	}}>
									<MapTooltip content={
										hovered && metadata.region.lookup[hovered] ? `${metadata.region.lookup[hovered].name}<br/><strong>${data.region.indicators.find(d => d.code == hovered)[mapKey].toLocaleString()} ${units[mapKey]}</strong>` : ''
									}/>
									{#if mapPins}
									<MapPin 
										content={
											`<img src=${'images/ertirea-military-1.jpg'}/>`
										}
										lngLat={[38.9251, 15.3229]}
										/>
									{/if}
									}
						   </MapLayer>
							<MapLayer
								id="lad-line"
							  	type="line"
							  	paint={{
									'line-color': [
										'case',
									  	['==', ['feature-state', 'hovered'], true], 'orange',
									  	['==', ['feature-state', 'selected'], true], 'black',
									  	['==', ['feature-state', 'highlighted'], true], 'black',
									  	'rgba(255,255,255,0)'
								  	],
								  'line-width': 2
							  }}
							/>
					  </MapSource>
					
					  {#if displayJourney}
					  <MapSource
						  	id="journey"
						  	type="geojson"
						  	data={journey}
						  	promoteId="namee"
						  	maxzoom={13}>

							<MapLayer
								id="journey-line"
							  	type="line"
							  	paint={{
									'line-color': 'black',
								  'line-width': 3
							  }}
							/>
						
					  </MapSource>
					  {/if}
					</Map>
				</div>
			</figure>
		</div>
	
		<div slot="foreground">
			<section data-id="map01">
				<div class="col-medium">
					<p>
						At the heart of Eritrea lies Asmara - the captical - Lemlem's and Aman's home.
					</p>
				</div>
			</section>
			<section data-id="map02">
				<div class="col-medium">
					<p>
						Swarmed with goverment facilities and military bases, ertirea is not easy to flee.
					</p>
				</div>
			</section>
			<section data-id="map03">
				<div class="col-medium">
					<p>
						Lemlem's and Aman's journey began with a covert nighttime exit from Asmara, a journey through the Eritrean highlands to the Sudanese border.
					</p>
				</div>
			</section>
		</div>
	</Scroller>
	{/if}
	
	<Scroller {threshold} bind:id={id['slideshow']} index={0}>
		<div slot="background">
			<Slideshow imageUrl={imageUrl}>

			</Slideshow>
		</div>
		<div slot="foreground">
			<section data-id="slideshow01">
				<div class="col-medium">
					<p>
						The arduous journey from Eritrea led Lemlem and Aman to the refugee camps of South Sudan. Life in the camp was marked by scarcity and uncertainty, a world away from the home they once knew.
					</p>
				</div>
			</section>
			<section data-id="slideshow02">
				<div class="col-medium">
					<p>
						"The camp was a sea of makeshift tents," Lemlem recalls. "The days were scorching hot, and the nights were frigid. We were always either in line for food, water, or medical assistance. But it was a sanctuary compared to what we had escaped from."
					</p>
				</div>
			</section>
			<section data-id="slideshow03">
				<div class="col-medium">
					<p>
						Despite the hardships, the refugee camp was also a place of resilience and hope. "We were all survivors, all clinging onto hope. There was a sense of community, and we looked out for each other," Lemlem said. She found work helping distribute aid supplies, while Aman started attending a makeshift school run by aid workers.
					</p>
				</div>
			</section>
			
		</div>
	</Scroller>



	
</main>
  

<style>
	:global(svelte-scroller-foreground) {
		pointer-events: none !important;
	}
	:global(svelte-scroller-foreground section div) {
		pointer-events: all !important;
	}
	select {
		max-width: 350px;
	}

	/* The properties below make the media DIVs grey, for visual purposes in demo */
	.media {
		background-color: #f0f0f0;
		display: -webkit-box;
		display: -ms-flexbox;
		display: flex;
		-webkit-box-orient: vertical;
		-webkit-box-direction: normal;
		-ms-flex-flow: column;
		flex-flow: column;
		-webkit-box-pack: center;
		-ms-flex-pack: center;
		justify-content: center;
		text-align: center;
		color: #aaa;
	}
</style>
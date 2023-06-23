<script>
    import { onMount } from 'svelte';
    import { Canvas, Layer } from 'svelte-canvas';
    import { feature } from 'topojson-client';
    import { geoOrthographic, geoAlbers, geoPath, geoGraticule10 } from 'd3-geo';
    // import countriesData from './ne_110m_admin_0_countries.geojson';
    const INITIAL_SCALE = 500;
    let map, width, dragging = false, λ = 0, φ = -10, scale = INITIAL_SCALE, position = [0, 0];
    const projection = geoOrthographic(),
            elevatedProjection = geoAlbers().scale(260), // Slightly larger scale
            path = geoPath(projection),
            elevatedPath = geoPath(elevatedProjection);

    const dragSensitivity = 0.1 * scale / INITIAL_SCALE;
    const zoomSensitivity = 0.35;
    
    onMount(async () => {
        const data = await fetch('https://cdn.jsdelivr.net/npm/world-atlas@2/land-110m.json').then(res => res.json());
        map = feature(data, 'land');
    
        const container = document.querySelector('#globe-container');
        let startλ, startφ, startX, startY;
        const [containerWidth, containerHeight] = [container.clientWidth, container.clientWidth];
        position = [containerWidth / 2, containerHeight / 2];
        projection.fitSize([containerWidth, containerHeight], { type: 'Sphere' });
        
    
        container.addEventListener('mousedown', event => {
            dragging = true;
            startX = event.clientX;
            startY = event.clientY;
            startλ = λ;
            startφ = φ;
        });
    
        window.addEventListener('mousemove', event => {
            if (dragging) {
                λ = startλ + (event.clientX - startX) * dragSensitivity;
                φ = startφ - (event.clientY - startY) * dragSensitivity;
            }
        });
    
        window.addEventListener('mouseup', () => dragging = false);
    
        container.addEventListener('wheel', event => {
            event.preventDefault();
            scale = Math.max(scale - event.deltaY * zoomSensitivity, 50);
        
        }, { passive: false });
    });
    
    $: graticule = ({ context, width, height }) => {
        projection
    
            .rotate([λ, φ])
            .scale(scale)
            .translate(position);
        context.strokeStyle = '#ccc';
        context.beginPath();
        path(geoGraticule10());
        context.stroke();
    };
    
    $: globe = ({ context }) => {
        context.fillStyle = 'tomato';
        context.beginPath();
        path(map);
        context.fill();
    };

    // $: countries = ({ context }) => {
    //     elevatedProjection.rotate([λ, φ]);
    //     context.fillStyle = 'rgba(0, 128, 0, 0.5)';
    //     context.beginPath();
    //     // elevatedPath(countriesData.features);
    //     context.fill();
    // };
</script>
    
<div id="globe-container" bind:clientWidth={width}>
    <Canvas {width} height={width}>
        <Layer setup={({ context }) => path.context(context)} render={graticule} />
        <Layer render={globe} />
    </Canvas>
</div>

<style>
    #globe-container {
        margin: 0 auto;
        /* max-width: 500px; */
        /* max-height: 500px; */
        border: 1px solid #ccc;
    }
</style>
    
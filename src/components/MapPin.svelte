<script>
    import { getContext, onDestroy } from 'svelte';
    import maplibre from 'maplibre-gl';
    import { onMount } from 'svelte';
    
    export let content;
    export let lngLat;

    const tooltip = new maplibre.Popup({
          closeButton: false,
          closeOnClick: false
      });

    const { getMap } = getContext('map');
      const map = getMap();

    onMount(() => {
        tooltip
            .setLngLat(lngLat)
            .setHTML(content)
            .addTo(map);

        
    });

    onDestroy(() => tooltip.remove());
  </script>
  
<style>
:global(.maplibregl-popup-content) {
        padding: 5px 10px !important;
}
</style>
<script>
	export let name;

	import 'src/components/wrldjs-map.svelte';
import MbMap from './components/mapbox-gl-map.svelte';
import WrldjsMap from './components/wrldjs-map.svelte';
import DarkmodeButton from './components/button-darkmode.svelte'

</script>

<main>
	<h1>Hello {name}!</h1>
	<p><a href="https://svelte.dev/tutorial">Svelte tutorial</a> to learn how to build Svelte apps.</p>
</main>

<MbMap />
<WrldjsMap />

<DarkmodeButton>Toggle Darkmode</DarkmodeButton>

<style>
	:global(body) {
		background-color: #f2eee2;
		color: #0084f6;
		transition: background-color 0.3s
	}
	:global(body.dark-mode) {
		background-color: #1d3040;
		color: #bfc2c7;
	}

	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>
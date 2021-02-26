<script lang="ts">
	import Tailwindcss from './Tailwindcss.svelte';
	import Ticker from './Ticker.svelte';
</script>

<Tailwindcss />
<main class="w-4/5 mx-auto my-10 flex flex-col items-stretch justify-center">
	<h1 class="text-5xl mx-auto my-10 w-1/4 text-center">Stock ticker</h1>
	<Ticker />
</main>


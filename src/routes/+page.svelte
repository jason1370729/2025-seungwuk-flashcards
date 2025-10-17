<script>
	import FLASHCARD from '../lib/flash-card.svelte';
	import { CARDS } from '../utils/cards-data.js';
	import { onMount } from 'svelte';

	let cards = $state(CARDS);

	onMount(() => {
		let c = localStorage.getItem('cards');
		if (c !== null) {
			cards = JSON.parse(c);
		}
	});
</script>

<div class="m-2 badge badge-warning">upcoming work</div>
<div
	class="flex min-h-80 w-full flex-grow flex-col flex-wrap space-y-5 rounded-3xl bg-green-900 p-5"
>
	{#each cards.filter((c) => c.score < 100) as card}
		<FLASHCARD {card} />
	{/each}
</div>
<div class="m-2 badge badge-success">completed</div>
<div
	class="flex min-h-80 w-full flex-grow flex-col flex-wrap space-y-5 rounded-3xl bg-green-900 p-5"
>
	{#each cards.filter((c) => c.score === 100) as card}
		<FLASHCARD {card} />
	{/each}
</div>

<script>
	import { FAKE_DATA } from '../../../data.js';
	import { CARDS } from '../../../utils/cards-data.js';
	import { onMount } from 'svelte';

	let index = $state(0);
	let flipped = $state(false);
	let score = $state(0);
	let results = [];
	let pastResults = $state([]);

	onMount(() => {
		let r = localStorage.getItem('cards');

		if (r !== null) {
			r = JSON.parse(r);
			console.log(123, r[1]);
			pastResults = r[1].results;
		}
	});

	function flipCard() {
		flipped = !flipped;
	}

	function nextCard(correct) {
		flipCard();

		results.push(correct);

		if (FAKE_DATA.length > index) {
			index += 1;
			if (correct) {
				score++;
			}
		}

		if (index >= FAKE_DATA.length) {
			let updated = CARDS.map((c) =>
				c.id === 2 ? { ...c, score: (score / FAKE_DATA.length) * 100, results } : c
			);

			localStorage.setItem('cards', JSON.stringify(updated));
		}
	}

	function restart() {
		score = 0;
		index = 0;
	}
</script>

<div class="flex flex-grow flex-col items-center justify-center space-y-10">
	<!-- score -->
	<div class="text-2xl">{score}</div>

	<!-- progress -->
	<div class="join">
		{#each FAKE_DATA as q, i}
			<div
				class={`relative flex flex-col items-center ${i < FAKE_DATA.length - 1 && 'border-r border-accent'}`}
			>
				{#if i < index && results[i]}
					<button class="btn join-item btn-success"
						>O
						{#if pastResults[i]}
							<div class="h-2 w-2 bg-success"></div>
						{/if}
					</button>
				{:else if i < index && !results[i]}
					<button class="btn join-item btn-error">X</button>
				{:else if i == index}
					<button class="btn btn-active join-item">{i + 1}</button>
				{:else}
					<button class="btn join-item">{i + 1}</button>
				{/if}
				{#if pastResults[i]}
					<div class="absolute h-2 w-2 rounded-full bg-success"></div>
				{/if}
			</div>
		{/each}
	</div>

	<!-- SCORE SCREEN -->
	<div class={`${FAKE_DATA.length > index && 'hidden'}`}>
		<div class="flex h-70 w-50 items-center justify-center rounded-3xl bg-base-300 text-7xl">
			{score}/{index}
			<a href="/">
				<button class="btn absolute top-5 right-5 flex btn-circle"
					><i class="fa-solid fa-xmark"></i></button
				>
			</a>
		</div>
	</div>

	<!-- FLASH CARD SCREEN -->
	<div class={`${FAKE_DATA.length <= index && 'hidden'} select-none`}>
		<div
			onclick={() => flipCard()}
			class="flex h-50 w-100 items-center rounded-3xl bg-base-300 p-4 text-left text-2xl break-normal"
		>
			{flipped ? FAKE_DATA[index]?.answer : FAKE_DATA[index]?.question}
		</div>

		<div class={`mt-5 flex space-x-2 ${!flipped && 'hidden'}`}>
			<button onclick={() => nextCard(true)} class="btn bg-green-400 btn-soft"
				><i class="fa-solid fa-check text-white"></i></button
			>
			<button onclick={() => nextCard(false)} class="btn bg-red-400 btn-soft"
				><i class="fa-solid fa-xmark"></i></button
			>
		</div>
	</div>
</div>

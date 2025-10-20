<script>
	import FLASHCARD from '../lib/flash-card.svelte';
	import { onMount } from 'svelte';

	let flashcardSets = $state([]);

	onMount(() => {
		// Load flashcard sets from localStorage
		let sets = localStorage.getItem('flashcardSets');
		if (sets !== null) {
			flashcardSets = JSON.parse(sets);
		}
	});
</script>

<!-- Flashcard Sets Section -->
{#if flashcardSets.length > 0}
	<div class="m-2 badge badge-info">Flashcard Sets</div>
	<div class="flex w-full flex-grow flex-col space-y-3 rounded-3xl bg-base-300 p-5">
		{#each flashcardSets as set}
			<div class="card bg-base-100 shadow-xl">
				<div class="card-body">
					<h2 class="card-title">{set.name}</h2>
					<div class="text-sm text-base-content/70">
						<p>
							{set.cards.length} cards • Created: {new Date(set.createdAt).toLocaleDateString()}
						</p>
						{#if set.progress}
							<p class="mt-1">
								Progress:
								<span class="text-success">{set.progress.correct} correct</span>,
								<span class="text-error">{set.progress.incorrect} incorrect</span>
								{#if set.progress.completed}
									<span class="ml-2 badge badge-sm badge-success">Completed</span>
								{/if}
							</p>
							{#if set.progress.lastAttempt}
								<p class="mt-1">
									Last attempt: {new Date(set.progress.lastAttempt).toLocaleString()}
								</p>
							{/if}
						{/if}
					</div>
					<div class="card-actions justify-end">
						<a href="/quiz/{set.id}" class="btn btn-sm btn-primary">
							{set.progress?.completed ? 'Retry Quiz' : 'Start Quiz'}
						</a>
					</div>
				</div>
			</div>
		{/each}
	</div>
{/if}

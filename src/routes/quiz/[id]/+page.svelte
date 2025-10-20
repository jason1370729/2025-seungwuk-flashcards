<script>
	import { onMount } from 'svelte';
	import { page } from '$app/stores';

	let flashcardSet = $state(null);
	let currentCardIndex = $state(0);
	let showAnswer = $state(false);
	let quizProgress = $state([]);
	let quizCompleted = $state(false);
	let setId = $state(null);

	$effect(() => {
		// Get the ID from the URL
		const id = $page.params.id;
		if (id && !setId) {
			setId = parseInt(id);
			loadQuiz();
		}
	});

	function loadQuiz() {
		// Load flashcard set from localStorage
		const sets = localStorage.getItem('flashcardSets');
		if (sets) {
			const allSets = JSON.parse(sets);
			flashcardSet = allSets.find((set) => set.id === setId);

			if (flashcardSet) {
				// Initialize progress array
				quizProgress = flashcardSet.cards.map(() => ({
					answered: false,
					correct: null
				}));

				// Load existing progress if available
				loadProgress();

				// Skip to first unanswered question
				skipToNextUnanswered();
			}
		}
	}

	function loadProgress() {
		const progressKey = 'quiz_progress_' + setId;
		const savedProgress = localStorage.getItem(progressKey);
		if (savedProgress) {
			const progress = JSON.parse(savedProgress);
			quizProgress = progress;
		}
	}

	function saveProgress() {
		const progressKey = 'quiz_progress_' + setId;
		localStorage.setItem(progressKey, JSON.stringify(quizProgress));

		// Also update the flashcardSets with the latest progress
		updateFlashcardSetProgress();
	}

	function updateFlashcardSetProgress() {
		const sets = localStorage.getItem('flashcardSets');
		if (sets) {
			const allSets = JSON.parse(sets);
			const setIndex = allSets.findIndex((set) => set.id === setId);

			if (setIndex !== -1) {
				// Calculate progress statistics
				const correct = quizProgress.filter((p) => p.correct).length;
				const incorrect = quizProgress.filter((p) => p.answered && !p.correct).length;
				const total = flashcardSet.cards.length;
				const allCompleted = quizProgress.every((p) => p.answered);

				// Update the set with progress information
				allSets[setIndex].progress = {
					correct,
					incorrect,
					total,
					completed: allCompleted,
					lastAttempt: new Date().toISOString()
				};

				// Save back to localStorage
				localStorage.setItem('flashcardSets', JSON.stringify(allSets));
			}
		}
	}

	function skipToNextUnanswered() {
		// Find the first unanswered question
		for (let i = 0; i < quizProgress.length; i++) {
			if (!quizProgress[i].answered || !quizProgress[i].correct) {
				currentCardIndex = i;
				return;
			}
		}
		// If all questions are answered correctly, stay at current position
	}

	function checkAnswer() {
		showAnswer = true;
	}

	function markCorrect() {
		quizProgress[currentCardIndex] = {
			answered: true,
			correct: true
		};
		saveProgress();
		moveToNext();
	}

	function markIncorrect() {
		quizProgress[currentCardIndex] = {
			answered: true,
			correct: false
		};
		saveProgress();
		moveToNext();
	}

	function moveToNext() {
		showAnswer = false;

		// Check if quiz is completed
		const allAnswered = quizProgress.every((p) => p.answered);
		if (allAnswered) {
			quizCompleted = true;
			return;
		}

		// Move to next card
		if (currentCardIndex < flashcardSet.cards.length - 1) {
			currentCardIndex++;
			// Skip already correct answers
			while (
				currentCardIndex < flashcardSet.cards.length &&
				quizProgress[currentCardIndex].correct
			) {
				currentCardIndex++;
			}
		}
	}

	function goToQuestion(index) {
		currentCardIndex = index;
		showAnswer = false;
	}

	function restartQuiz() {
		quizProgress = flashcardSet.cards.map(() => ({
			answered: false,
			correct: null
		}));
		currentCardIndex = 0;
		showAnswer = false;
		quizCompleted = false;
		saveProgress();
	}

	let correctCount = $derived(quizProgress.filter((p) => p.correct).length);
	let incorrectCount = $derived(quizProgress.filter((p) => p.answered && !p.correct).length);
</script>

{#if !flashcardSet}
	<div class="flex h-screen items-center justify-center">
		<p class="text-xl">Loading quiz...</p>
	</div>
{:else if quizCompleted}
	<div class="flex h-screen flex-col items-center justify-center space-y-6 p-6">
		<h1 class="text-4xl font-bold">Quiz Completed! 🎉</h1>
		<div class="stats shadow">
			<div class="stat">
				<div class="stat-title">Correct</div>
				<div class="stat-value text-success">{correctCount}</div>
			</div>
			<div class="stat">
				<div class="stat-title">Incorrect</div>
				<div class="stat-value text-error">{incorrectCount}</div>
			</div>
			<div class="stat">
				<div class="stat-title">Total</div>
				<div class="stat-value">{flashcardSet.cards.length}</div>
			</div>
		</div>
		<div class="flex space-x-4">
			<a href="/" class="btn btn-primary">Go Home</a>
			<button onclick={restartQuiz} class="btn btn-secondary">Restart Quiz</button>
		</div>
	</div>
{:else}
	<div class="container mx-auto p-6">
		<!-- Header with quiz name -->
		<div class="mb-6">
			<h1 class="text-3xl font-bold">{flashcardSet.name}</h1>
			<p class="text-sm text-base-content/70">
				Progress: {correctCount} correct, {incorrectCount} incorrect
			</p>
		</div>

		<!-- Problem number indicators -->
		<div class="mb-8 flex flex-wrap gap-2">
			{#each flashcardSet.cards as card, index}
				<button
					onclick={() => goToQuestion(index)}
					class="btn btn-sm {currentCardIndex === index
						? 'btn-primary'
						: quizProgress[index].correct
							? 'btn-success'
							: quizProgress[index].answered && !quizProgress[index].correct
								? 'btn-error'
								: 'btn-ghost'}"
				>
					{index + 1}
				</button>
			{/each}
		</div>

		<!-- Current question card -->
		<div class="card bg-base-100 shadow-xl">
			<div class="card-body">
				<h2 class="mb-4 card-title">
					Question {currentCardIndex + 1} of {flashcardSet.cards.length}
				</h2>

				<!-- Question -->
				<div class="mb-6 rounded-lg bg-info p-6 text-info-content">
					<p class="text-xl font-semibold">
						{flashcardSet.cards[currentCardIndex].question}
					</p>
				</div>

				<!-- Skip indicator if already correct -->
				{#if quizProgress[currentCardIndex].correct}
					<div class="mb-4 alert alert-success">
						<span>✓ Previously answered correctly - Skipped</span>
					</div>
				{/if}

				<!-- Show answer button or answer with self-evaluation -->
				{#if !showAnswer && !quizProgress[currentCardIndex].correct}
					<div class="mb-6 rounded-lg bg-base-200 p-6 text-center">
						<p class="mb-4 text-base-content/70">
							Think of your answer, then click below to reveal the correct answer
						</p>
						<button onclick={checkAnswer} class="btn btn-lg btn-primary"> Show Answer </button>
					</div>
				{/if}

				<!-- Show correct answer and self-evaluation buttons -->
				{#if showAnswer}
					<div class="mb-6 rounded-lg bg-success p-6 text-success-content">
						<p class="mb-2 font-semibold">Answer:</p>
						<p class="text-xl">{flashcardSet.cards[currentCardIndex].answer}</p>
					</div>

					<div class="mb-4 text-center">
						<p class="mb-4 text-lg font-semibold">Did you get it right?</p>
					</div>

					<div class="card-actions justify-center space-x-4">
						<button onclick={markCorrect} class="btn btn-lg btn-success">✓ Correct</button>
						<button onclick={markIncorrect} class="btn btn-lg btn-error">✗ Incorrect</button>
					</div>
				{/if}
			</div>
		</div>

		<!-- Navigation -->
		<div class="mt-6 flex justify-between">
			<a href="/" class="btn btn-ghost">← Back to Home</a>
		</div>
	</div>
{/if}

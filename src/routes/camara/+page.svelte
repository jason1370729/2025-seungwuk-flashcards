<script>
	import './loading.css';
	import { PUBLIC_GEMINI_API } from '$env/static/public';
	import { GoogleGenAI } from '@google/genai';
	let category = $state('');
	let categories = $state([]);
	let loading = $state(true);

	$effect(() => {
		if (localStorage.getItem('categories') !== null) {
			categories = JSON.parse(localStorage.getItem('categories'));
		}
	});

	async function createCards() {
		loading = true;
		my_modal_1.showModal();
		const API_KEY = 'AIzaSyAbJflQp2pXKxp0CGDNV9xAYI9dcVYVY4Y';
		const ai = new GoogleGenAI({ apiKey: PUBLIC_GEMINI_API });
		const response = await ai.models.generateContent({
			model: 'gemini-2.5-flash',
			contents: `create 3 random u.s. history questions in a JSON format - an array of objects which contains fields: question and answer`
		});
		console.log(response.candidates[0].content.parts[0].text);
		loading = false;
	}
</script>

<div class=" flex flex-grow items-end justify-end p-3">
	<button onclick={() => my_modal_3.showModal()} class="btn text-green-400 btn-soft"
		>+ Flashcard</button
	>
</div>

<dialog id="my_modal_3" class="modal">
	<div class="modal-box">
		<form method="dialog">
			<button class="btn absolute top-2 right-2 btn-circle btn-ghost btn-sm">✕</button>
		</form>
		<div># of cards</div>
		<input type="number" class="input m-2" />

		<div>Choose the image</div>
		<input type="file" class="file-input" />

		<div class="flex w-full justify-end">
			<button onclick={createCards} class="btn m-2 btn-soft">Create card</button>
		</div>
	</div>
</dialog>

<dialog id="my_modal_1" class="modal">
	<div class="modal-box">
		{#if loading}
			<p class="py-4">Creating Flashcards <i class="fa-solid fa-spinner animate-spin"></i></p>
		{/if}
		{#if !loading}
			<p>Done</p>
			<div class="modal-action">
				<form method="dialog">
					<a href="/">
						<button class="btn">Go to homepage</button>
					</a>
				</form>
			</div>
		{/if}
	</div>
</dialog>

<!-- Open the modal using ID.showModal() method -->

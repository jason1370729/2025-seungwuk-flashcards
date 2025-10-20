<script>
	import './loading.css';
	import { PUBLIC_GEMINI_API } from '$env/static/public';
	import { GoogleGenAI } from '@google/genai';
	let loading = $state(true);
	let text = $state('...');
	let selectedFile = $state(null);
	let numberOfCards = $state(5);
	let cardSetName = $state('');

	$effect(() => {
		if (localStorage.getItem('categories') !== null) {
			categories = JSON.parse(localStorage.getItem('categories'));
		}
	});

	function parseGeminiResponse(responseText) {
		// Remove the ```json and ``` wrapper lines
		const lines = responseText.trim().split('\n');
		let jsonStart = 0;
		let jsonEnd = lines.length;

		// Find the start of JSON (skip ```json line)
		for (let i = 0; i < lines.length; i++) {
			if (lines[i].includes('```json') || lines[i].includes('```')) {
				jsonStart = i + 1;
				break;
			}
		}

		// Find the end of JSON (skip ``` line)
		for (let i = lines.length - 1; i >= 0; i--) {
			if (lines[i].includes('```')) {
				jsonEnd = i;
				break;
			}
		}

		// Extract JSON content
		const jsonContent = lines.slice(jsonStart, jsonEnd).join('\n').trim();
		return JSON.parse(jsonContent);
	}

	function saveCardsToLocalStorage(cards, name) {
		// Get existing flashcard sets
		let flashcardSets = [];
		if (localStorage.getItem('flashcardSets') !== null) {
			flashcardSets = JSON.parse(localStorage.getItem('flashcardSets'));
		}

		// Create new set
		const newSet = {
			id: Date.now(), // Simple ID generation
			name: name || `Flashcard Set ${flashcardSets.length + 1}`,
			cards: cards,
			createdAt: new Date().toISOString()
		};

		// Add to existing sets
		flashcardSets.push(newSet);

		// Save back to localStorage
		localStorage.setItem('flashcardSets', JSON.stringify(flashcardSets));

		return newSet;
	}

	function handleFileChange(event) {
		const file = event.target.files[0];
		if (file) {
			selectedFile = file;
		}
	}

	async function fileToBase64(file) {
		return new Promise((resolve, reject) => {
			const reader = new FileReader();
			reader.readAsDataURL(file);
			reader.onload = () => resolve(reader.result.split(',')[1]);
			reader.onerror = (error) => reject(error);
		});
	}

	async function createCards() {
		if (!selectedFile) {
			alert('Please select a file first');
			return;
		}

		if (!cardSetName.trim()) {
			alert('Please enter a name for your flashcard set');
			return;
		}

		loading = true;
		my_modal_3.close();
		my_modal_1.showModal();

		try {
			const ai = new GoogleGenAI({ apiKey: PUBLIC_GEMINI_API });
			let response;

			// Convert file to base64 for image files
			if (selectedFile.type.startsWith('image/')) {
				const fileContent = await fileToBase64(selectedFile);

				response = await ai.models.generateContent({
					model: 'gemini-2.5-flash',
					contents: [
						{
							text: `give me a parsable JSON content only. use the image to generate ${numberOfCards} flash cards. Use an array of objects each object should contain a question field and an answer field, both of which are strings`
						},
						{
							inlineData: {
								data: fileContent,
								mimeType: selectedFile.type
							}
						}
					]
				});
			} else if (selectedFile.type === 'text/plain' || selectedFile.name.endsWith('.txt')) {
				// Handle text files
				const text = await selectedFile.text();

				response = await ai.models.generateContent({
					model: 'gemini-2.5-flash',
					contents: `give me a parsable JSON content wrapped in \`\`\`json\`\`\` markers. Based on the following text content, generate ${numberOfCards} flash cards. Use an array of objects each object should contain a question field and an answer field, both of which are strings.\n\nText content:\n${text}`
				});
			} else {
				alert('Please select an image file or text file');
				loading = false;
				return;
			}

			console.log('Raw response:', response.text);

			// Parse the response and extract JSON
			const cards = parseGeminiResponse(response.text);
			console.log('Parsed cards:', cards);

			// Save to localStorage
			const savedSet = saveCardsToLocalStorage(cards, cardSetName.trim());
			console.log('Saved flashcard set:', savedSet);

			// Reset form
			cardSetName = '';
			selectedFile = null;
			numberOfCards = 5;
		} catch (error) {
			console.error('Error creating cards:', error);
			alert('Error creating cards. Please try again.');
		}

		loading = false;
	}
</script>

<div class=" flex flex-grow items-center justify-center p-30">
	<button onclick={() => my_modal_3.showModal()} class="btn text-green-400 btn-soft"
		>+ Flashcard</button
	>
</div>

<dialog id="my_modal_3" class="modal">
	<div class="modal-box">
		<form method="dialog">
			<button class="btn absolute top-2 right-2 btn-circle btn-ghost btn-sm">✕</button>
		</form>

		<div class="mb-4">
			<label for="cardSetName" class="mb-2 block text-sm font-medium">Flashcard Set Name</label>
			<input
				id="cardSetName"
				type="text"
				class="input-bordered input w-full"
				bind:value={cardSetName}
				placeholder="Enter a name for your flashcard set"
			/>
		</div>

		<div class="mb-4">
			<label for="numberOfCards" class="mb-2 block text-sm font-medium"># of cards</label>
			<input
				id="numberOfCards"
				type="number"
				class="input-bordered input w-full"
				bind:value={numberOfCards}
				min="1"
				max="20"
			/>
		</div>

		<div class="mb-4">
			<label for="fileInput" class="mb-2 block text-sm font-medium"
				>Choose the image or text file</label
			>
			<input
				id="fileInput"
				type="file"
				class="file-input-bordered file-input w-full"
				accept="image/*,.txt"
				onchange={handleFileChange}
			/>
		</div>

		{#if selectedFile}
			<div class="mt-2 mb-4 text-sm text-gray-600">
				Selected: {selectedFile.name}
			</div>
		{/if}

		<div class="flex w-full justify-end">
			<button onclick={createCards} class="btn btn-primary">Create Cards</button>
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

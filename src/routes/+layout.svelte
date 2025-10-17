<script>
	import '../app.css';

	/** @type {{children: import('svelte').Snippet}} */
	let { children } = $props();
</script>

<div class="flex h-screen flex-col">
	<div class="flex-1 overflow-y-auto">
		{@render children()}
	</div>
	<div class="flex h-20 items-center justify-evenly bg-base-300">
		<a href="/">
			<button class="btn btn-square"><i class="fa-regular fa-folder"></i></button>
		</a>
		<a href="/camara">
			<button class="btn btn-square"><i class="fa-regular fa-camera"></i></button>
		</a>
	</div>
</div>

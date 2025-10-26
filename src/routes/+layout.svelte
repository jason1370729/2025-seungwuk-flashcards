<script>
	import '../app.css';

	/** @type {{children: import('svelte').Snippet}} */
	let { children } = $props();
</script>

<div class="flex h-svh flex-col">
	<div class="flex-1 overflow-y-auto p-4">
		{@render children()}
	</div>
	<div class="flex h-20 items-center justify-evenly bg-base-300">
		<a href="/" class="btn btn-square" aria-label="home">
			<i class="fa-regular fa-folder"></i>
		</a>
		<a href="/camara" class="btn btn-square" aria-label="camera">
			<i class="fa-regular fa-camera"></i>
		</a>
	</div>
</div>

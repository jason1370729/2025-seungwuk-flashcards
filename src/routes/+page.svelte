<script>
  import FLASHCARD from "../lib/flash-card.svelte";
  import {CARDS } from "../utils/cards-data.js";
  import { onMount } from 'svelte';

  let cards = $state(CARDS);

  onMount(() => {
    let c = localStorage.getItem("cards");
    if(c !== null) {
      cards = JSON.parse(c);
    }
  })
</script>

<div class="badge badge-warning m-2">upcoming work</div>
<div class="w-full min-h-80 rounded-3xl flex flex-col flex-wrap p-5 space-y-5 flex-grow bg-green-900">

{#each cards.filter((c) => c.score < 100) as card}
  <FLASHCARD card={card} />
{/each}


</div>
<div class="badge badge-success m-2">completed</div>
<div class="w-full min-h-80 rounded-3xl flex flex-col flex-wrap p-5 space-y-5 flex-grow bg-green-900">

{#each cards.filter((c) => c.score === 100) as card}
  <FLASHCARD card={card} />
{/each}

</div>

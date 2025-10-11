<script>
  import { FAKE_DATA } from '../../../data.js';
  import {CARDS } from "../../../utils/cards-data.js";
  import { onMount } from 'svelte';

  let index = $state(0);
  let flipped = $state(false);
  let score = 0;
  let results = [];

  function flipCard() {
    flipped = !flipped;
  }

  function nextCard(correct) {
    flipCard();

    results.push(correct);
    
    if(FAKE_DATA.length > index) {
      index+=1;
      if(correct) {
        score++;
      }
    }

    if(index >= FAKE_DATA.length) {
      let updated = CARDS.map((c) => c.id === 2 ? {...c, "score": score / FAKE_DATA.length * 100} : c);

      localStorage.setItem("cards", JSON.stringify(updated));
  
    }
  }

  function restart() {
    score = 0;
    index = 0;
  }



</script>

<div class="flex-grow flex flex-col justify-center items-center space-y-10">
  <!-- score -->
  <div class="text-2xl">{score}</div>

  <!-- progress -->
  <div class="join">
    {#each FAKE_DATA as q, i}
      {#if i < index && results[i]}
        <button class="join-item btn btn-success">O</button>
      {:else if i < index && !results[i]}
        <button class="join-item btn btn-error">X</button>
      {:else if i == index}
        <button class="join-item btn btn-active">{i+1}</button>
      {:else}
        <button class="join-item btn">{i+1}</button>
      {/if}
    {/each}
  </div>
  


  <!-- SCORE SCREEN -->
  <div class={`${FAKE_DATA.length > index && "hidden"}`}>
    <div class="h-70 w-50 bg-base-300 rounded-3xl text-7xl flex items-center justify-center">{score}/{index} 
      <a href="/">
        <button class="btn btn-circle flex absolute right-5 top-5"><i class="fa-solid fa-xmark"></i></button>
      </a>
    </div>
  </div>

  <!-- FLASH CARD SCREEN -->
  <div class={`${FAKE_DATA.length <= index && "hidden"} select-none`}>
    <div onclick={() => flipCard()} class="h-50 w-100 bg-base-300 rounded-3xl text-left p-4 text-2xl flex items-center break-normal">{flipped ? FAKE_DATA[index]?.answer : FAKE_DATA[index]?.question}</div>

    <div class={`mt-5 space-x-2 flex ${!flipped && "hidden"}`}>
      <button onclick={() => nextCard(true)} class="btn btn-soft bg-green-400"><i class="fa-solid fa-check text-white"></i></button>
      <button onclick={() => nextCard(false)} class="btn btn-soft bg-red-400"><i class="fa-solid fa-xmark"></i></button>
    </div>
  </div>
</div>
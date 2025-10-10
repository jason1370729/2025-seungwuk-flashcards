<script>
  import "./loading.css"
  let category = $state("");
  let categories = $state([]);
  let loading = $state(true);

  $effect(() => {
    if(localStorage.getItem("categories") !== null) {
      categories = JSON.parse(localStorage.getItem("categories"));
    }
  })
  
  function createCards() {

    

  }
</script>

<div class=" flex flex-grow justify-end items-end p-3">

  <button onclick={() => my_modal_3.showModal()} class="btn btn-soft text-green-400">+ Flashcard</button>

</div>

<dialog id="my_modal_3" class="modal">
  <div class="modal-box">
    <form method="dialog">
      <button class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2">✕</button>
    </form>
    <div># of cards</div>
    <input type="number" class="input m-2"/>

    <div>Choose the image</div>
    <input type="file" class="file-input" />

    <div class="flex w-full justify-end">
      <button onclick={() => {
        loading = true;
        my_modal_1.showModal();
        setTimeout(() => {loading = false}, (Math.random() * 3 + 2) * 1000);      

      }} class="btn btn-soft m-2">Create card</button>
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


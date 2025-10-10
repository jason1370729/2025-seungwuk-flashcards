<script>
  export let card;
</script>

<div class="card w-96 card-xs bg-base-300 shadow-sm p-3">
  <div class="card-body">
    <h2 class="card-title">{card.title}
      <div class="badge badge-xs badge-primary">{card.subject}</div>
    </h2>
    <p>Start your study</p>
    <div class="justify-end card-actions">
      <a href="/quiz/{card.id}">
        <button class="btn btn-primary">Start</button>
      </a>
    </div>
    <div class="w-full">
      <progress class="progress progress-primary w-full" value={card.score} max={100}></progress>
    </div>
  </div>
</div>
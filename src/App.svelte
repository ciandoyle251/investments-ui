<script>
  import svelteLogo from './assets/svelte.svg'
  import viteLogo from '/vite.svg'
  // import Counter from './lib/Counter.svelte'
</script>

<main>
  <div>
    <a href="https://vite.dev" target="_blank" rel="noreferrer">
      <img src={viteLogo} class="logo" alt="Vite Logo" />
    </a>
    <a href="https://svelte.dev" target="_blank" rel="noreferrer">
      <img src={svelteLogo} class="logo svelte" alt="Svelte Logo" />
    </a>
  </div>
  <h1>Vite + Svelte</h1>

  <div class="card">
    <!-- Placeholder for future content -->
  </div>

  <p>
    Check out <a href="https://github.com/sveltejs/kit#readme" target="_blank" rel="noreferrer">SvelteKit</a>, the official Svelte app framework powered by Vite!
  </p>

  <p class="read-the-docs">
    Click on the Vite and Svelte logos to learn more
  </p>
</main>

<style>
  table {
    width: 100%;
    border-collapse: collapse;
  }

  th,
  td {
    padding: 0.75rem;
    border: 1px solid #ccc;
  }

  th {
    background-color: #f0f0f0 !important;
    color: #000 !important;
  }

  @media (prefers-color-scheme: dark) {
    th {
      background-color: #1a1a1a !important;
      color: #ffffff !important;
    }

    table {
      background-color: #1e1e1e;
      color: #e0e0e0;
    }

    td {
      border-color: #333;
    }
  }
</style>

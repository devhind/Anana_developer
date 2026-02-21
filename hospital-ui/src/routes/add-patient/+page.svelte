<script lang="ts">
  import AddPatient from './AddPatient.svelte';
  
</script>

<div class="container">
  <AddPatient />
</div>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    min-height: 100vh;
    background: linear-gradient(135deg, #5f676f 0%, #e2e8f0 100%);
    display: flex;
    align-items: center;
    justify-content: center;
    font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  }

  .container {
    width: 100%;
    max-width: 1100px;
    display: flex;
    flex-direction: column;
    gap: 24px;
    padding: 24px;
  }

  .switch {
    position: fixed;
    top: 20px;
    left: 20px;
    display: flex;
    gap: 10px;
  }

  button {
    padding: 8px 14px;
    border-radius: 6px;
    border: none;
    cursor: pointer;
    background: #2563eb;
    color: white;
    font-weight: 500;
    transition: background 0.2s ease;
  }

  button:hover {
    background: #1e40af;
  }
</style>

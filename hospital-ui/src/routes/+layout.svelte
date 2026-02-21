<script lang="ts">
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';
  import { page } from '$app/stores';
  import Header from '$lib/components/Header.svelte';
  import Sidebar from '$lib/components/Sidebar.svelte';
  import Footer from '$lib/components/Footer.svelte';

  let sidebarOpen = true;

  function toggleSidebar() {
    sidebarOpen = !sidebarOpen;
  }
  // Detect login/register pages
$: isAuthPage =
  $page.url.pathname === '/login' ||
  $page.url.pathname === '/register';

// Redirect if not logged in
onMount(() => {
  const doctor = localStorage.getItem("doctor");

  if (!doctor && !isAuthPage) {
    goto('/login');
  }
});
</script>

{#if !isAuthPage}
<div class="app">

  <Header on:toggleSidebar={toggleSidebar} />

  <div class="body">

    <div class="sidebar-container" class:open={sidebarOpen}>
      <Sidebar bind:open={sidebarOpen} />
    </div>

    <main class="content">
      <slot />
    </main>

  </div>

  <Footer />

</div>
<!-- Overlay for mobile -->

{/if}
{#if isAuthPage}
  <slot />
{/if}
<style>
.app {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

/* HEADER */
header {
  flex-shrink: 0;
}

/* FOOTER */
footer {
  flex-shrink: 0;
}

/* BODY (sidebar + content) */
.body {
  display: flex;
  flex: 1;
  overflow: hidden;
}

/* SIDEBAR */
.sidebar-container {
  width: 220px;
  transition: width 0.3s ease;
}

/* COLLAPSED */
.sidebar-container:not(.open) {
  width: 0;
  overflow: hidden;
}

/* CONTENT */
.content {
  flex: 1;
  overflow-y: auto;
  padding: 24px;
}
</style>
<!-- Overlay for mobile - FIXED: Added show class -->
<!-- {#if sidebarOpen}
  <div class="overlay show" on:click={toggleSidebar} />
{/if} -->

<script context="module">
  // You can also add this to your global styles for better performance
  // Add to your global.css or app.css
  /*
  * {
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-rendering: optimizeLegibility;
  }

  body {
    overscroll-behavior-y: none;
    overflow: hidden;
  }
  */
</script>
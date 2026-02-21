<script lang="ts">
  import { page } from '$app/stores';
  export let open = true;
</script>

<aside class="sidebar {open ? 'open' : 'closed'}">
  <div class="logo">
    <span class="logo-icon">✚</span>
    <span class="logo-text" class:hidden={!open}>Anana</span>
  </div>

  <nav>
  <a href="/patient" class:active={$page.url.pathname.startsWith('/patient')}>
    Patients
  </a>

  <a href="/add-patient" class:active={$page.url.pathname === '/add-patient'}>
    Add Patient
  </a>

  <!-- 🔥 NEW OPD LINK -->
  <a href="/opd-patient" class:active={$page.url.pathname.startsWith('/opd-patient')}>
    OPD Patients
  </a>

  <!-- <a href="/custom-field" class:active={$page.url.pathname.startsWith('/custom-field')}>
    Custom Fields
  </a> -->
  <a href="/custom-field"
   class="disabled"
   on:click|preventDefault>
  Custom Fields
</a>
</nav>

</aside>

<style>
.disabled {
  pointer-events: none;
  opacity: 0.5;
}
.sidebar {
  height: 100vh;
  background: #f8fafc; /* very light gray-blue */
  border-right: 1px solid #e2e8f0;
  padding: 22px 14px;
  transition: width 0.25s ease;
  overflow: hidden;
}

/* OPEN / CLOSE */
.sidebar.open {
  width: 220px;
}

.sidebar.closed {
  width: 220px;
}

/* LOGO */
.logo {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 28px;
}

.logo-icon {
  font-size: 20px;
  color: #2563eb; /* primary blue */
}

.logo-text {
  font-size: 16px;
  font-weight: 700;
  color: #1e293b; /* slate */
}

.hidden {
  display: none;
}

/* NAV */
nav {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

nav a {
  padding: 11px 14px;
  border-radius: 10px;
  text-decoration: none;
  font-size: 14px;
  color: #475569; /* muted text */
  white-space: nowrap;
  transition: all 0.2s ease;
}

/* Hover */
nav a:hover {
  background: #eff6ff; /* light blue hover */
  color: #1d4ed8;
}

/* Active */
nav a.active {
  background: #2563eb; /* primary blue */
  color: white;
  font-weight: 600;
  box-shadow: 0 4px 12px rgba(37, 99, 235, 0.25);
}

</style>

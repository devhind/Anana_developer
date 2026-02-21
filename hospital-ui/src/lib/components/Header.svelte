<script lang="ts">
  import { createEventDispatcher } from 'svelte';
  import { goto } from '$app/navigation';
  import { page } from '$app/stores';
  import { onMount } from 'svelte';

  const dispatch = createEventDispatcher();
  let inputValue = '';
  let isSearchFocused = false;

  let doctorName = "Doctor";
  let dropdownOpen = false;

  // Sync input with URL
  $: {
    const urlSearch = $page.url.searchParams.get('search') ?? '';
    inputValue = urlSearch;
  }

  onMount(() => {
    const doctor = localStorage.getItem("doctor");
    if (doctor) {
      const parsed = JSON.parse(doctor);
      doctorName = parsed.name;
    }

    document.addEventListener("click", handleOutsideClick);
  });

  function handleOutsideClick(e: MouseEvent) {
    const target = e.target as HTMLElement;
    if (!target.closest('.user-section')) {
      dropdownOpen = false;
    }
  }

  function toggleDropdown() {
    dropdownOpen = !dropdownOpen;
  }

  function logout() {
  localStorage.removeItem("doctor");
  window.location.replace('/login');
}

  function onInput(e: Event) {
    inputValue = (e.target as HTMLInputElement).value;
  }

  function handleSearch() {
    if (inputValue.trim()) {
      goto(`/patient?search=${encodeURIComponent(inputValue.trim())}`);
    } else {
      goto('/patient');
    }
  }

  function handleKeyPress(e: KeyboardEvent) {
    if (e.key === 'Enter') {
      handleSearch();
    }
  }

  function clearSearch() {
    inputValue = '';
    goto('/patient');
  }
</script>

<header class="header">
  <div class="header-inner">
    <!-- LEFT -->
    <div class="left-section">
      <button class="menu-btn" on:click={() => dispatch('toggleSidebar')}>
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M3 12h18M3 6h18M3 18h18"/>
        </svg>
      </button>

      <div class="title-container">
        <div class="hospital-icon">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path d="M12 2L2 7v10l10 5 10-5V7L12 2z"/>
            <path d="M12 2v20"/>
          </svg>
        </div>
        <h1>Hospital System</h1>
      </div>
    </div>

    <!-- CENTER -->
    <div class="center-group">
      <div class="search-container" class:focused={isSearchFocused}>
        <svg class="search-icon-left" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
            d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"/>
        </svg>

        <input
          class="search-input"
          placeholder="Search patients..."
          value={inputValue}
          on:input={onInput}
          on:keypress={handleKeyPress}
          on:focus={() => isSearchFocused = true}
          on:blur={() => isSearchFocused = false}
        />

        {#if inputValue}
          <button class="clear-btn" on:click={clearSearch}>
            ✕
          </button>
        {:else}
          <div class="search-hint">⌘K</div>
        {/if}

        <button class="search-action-btn" on:click={handleSearch}>
          →
        </button>
      </div>

      <a href="/add-patient" class="add-patient-btn">
        <span class="add-text">Add Patient</span>
      </a>
    </div>

    <!-- RIGHT -->
    <div class="right-section">
      <div class="user-section" style="position: relative;">
        <div class="user-badge">
          <span class="user-role">{doctorName}</span>
        </div>

        <div class="avatar" on:click={toggleDropdown} style="cursor:pointer;">
          👤
        </div>

        {#if dropdownOpen}
          <div class="dropdown">
            <div class="dropdown-name">{doctorName}</div>
            <button class="logout-btn" on:click={logout}>
              Logout
            </button>
          </div>
        {/if}
      </div>
    </div>
  </div>
</header>

<style>
/* ADD ONLY THIS NEW CSS */

.dropdown {
  position: absolute;
  top: 50px;
  right: 0;
  background: white;
  border-radius: 10px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.15);
  padding: 12px;
  width: 180px;
  animation: fadeIn 0.2s ease;
  z-index: 999;
}

.dropdown-name {
  font-weight: 600;
  margin-bottom: 10px;
}

.logout-btn {
  width: 100%;
  padding: 8px;
  border: none;
  border-radius: 6px;
  background: #ef4444;
  color: white;
  font-weight: 600;
  cursor: pointer;
}

.logout-btn:hover {
  background: #dc2626;
}

  /* HEADER BAR - Dark blue gradient */
  .header {
    height: 56px;
    background: linear-gradient(135deg, #1e3a8a 0%, #1e40af 100%);
    display: flex;
    align-items: center;
    position: sticky;
    top: 0;
    z-index: 100;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
    border-bottom: none;
  }

  .header-inner {
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 20px;
    gap: 16px;
  }

  /* LEFT SECTION */
  .left-section {
    display: flex;
    align-items: center;
    gap: 16px;
    flex-shrink: 0;
  }

  .menu-btn {
    background: rgba(255, 255, 255, 0.15);
    border: 1px solid rgba(255, 255, 255, 0.2);
    width: 36px;
    height: 36px;
    border-radius: 8px;
    cursor: pointer;
    color: #ffffff;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all 0.2s ease;
    flex-shrink: 0;
  }

  .menu-btn:hover {
    background: rgba(255, 255, 255, 0.25);
    border-color: rgba(255, 255, 255, 0.3);
    transform: translateY(-1px);
  }

  .title-container {
    display: flex;
    align-items: center;
    gap: 12px;
    flex-shrink: 0;
  }

  .hospital-icon {
    color: #93c5fd;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  h1 {
    font-size: 18px;
    font-weight: 700;
    margin: 0;
    color: #ffffff;
    letter-spacing: -0.3px;
    white-space: nowrap;
  }

  /* CENTER GROUP – SEARCH + ADD BUTTON */
  .center-group {
    display: flex;
    align-items: center;
    gap: 12px;
    flex: 1;
    justify-content: center;
    max-width: 600px;
    margin: 0 auto;
  }

  /* COMPACT SEARCH BAR */
  .search-container {
    flex: 1;
    max-width: 300px;
    position: relative;
    display: flex;
    align-items: center;
    background: white;
    border-radius: 8px;
    padding: 4px 8px;
    transition: all 0.2s ease;
    border: 2px solid #e2e8f0;
    height: 38px;
  }

  .search-container.focused {
    box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.2);
    border-color: #3b82f6;
  }

  .search-icon-left {
    position: absolute;
    left: 12px;
    color: #64748b;
    z-index: 1;
    pointer-events: none;
  }

  .search-input {
    width: 100%;
    padding: 0 36px 0 32px;
    border: none;
    outline: none;
    background: transparent;
    color: #1e293b;
    font-size: 14px;
    font-weight: 500;
    border-radius: 6px;
    height: 100%;
  }

  .search-input::placeholder {
    color: #94a3b8;
    font-weight: 400;
    font-size: 14px;
  }

  .clear-btn {
    position: absolute;
    right: 42px;
    background: none;
    border: none;
    color: #94a3b8;
    cursor: pointer;
    padding: 4px;
    border-radius: 4px;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all 0.2s;
    width: 18px;
    height: 18px;
  }

  .clear-btn:hover {
    background: #f1f5f9;
    color: #64748b;
    transform: scale(1.1);
  }

  .search-hint {
    position: absolute;
    right: 42px;
    color: #94a3b8;
    font-size: 11px;
    font-weight: 500;
    padding: 2px 6px;
    background: #f8fafc;
    border-radius: 4px;
    border: 1px solid #e2e8f0;
    pointer-events: none;
  }

  .search-action-btn {
    position: absolute;
    right: 8px;
    background: #3b82f6;
    border: none;
    width: 24px;
    height: 24px;
    border-radius: 6px;
    cursor: pointer;
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all 0.2s ease;
  }

  .search-action-btn:hover {
    background: #2563eb;
    transform: scale(1.05);
  }

  /* ADD PATIENT BUTTON – green, next to search */
  .add-patient-btn {
    display: flex;
    align-items: center;
    justify-content: center;
    background: #10b981;
    color: white;
    text-decoration: none;
    padding: 8px 16px;
    border-radius: 8px;
    font-size: 13px;
    font-weight: 600;
    transition: all 0.3s ease;
    border: none;
    cursor: pointer;
    min-width: 36px;
    height: 38px;
    position: relative;
    overflow: hidden;
    box-shadow: 0 2px 4px rgba(16, 185, 129, 0.2);
    flex-shrink: 0;
  }

  .add-patient-btn:hover {
    background: #059669;
    transform: translateY(-1px);
    box-shadow: 0 4px 8px rgba(16, 185, 129, 0.3);
  }

  .add-patient-btn:active {
    transform: translateY(0);
  }

  .add-icon {
    flex-shrink: 0;
  }

  .add-text {
    display: none;
    white-space: nowrap;
    margin-left: 8px;
  }

  /* RIGHT SECTION – user only */
  .right-section {
    display: flex;
    align-items: center;
    gap: 12px;
    flex-shrink: 0;
  }

  .user-section {
    display: flex;
    align-items: center;
    gap: 10px;
  }

  .user-badge {
    display: flex;
    flex-direction: column;
    align-items: flex-end;
  }

  .user-role {
    font-size: 11px;
    color: #dbeafe;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 0.5px;
    background: rgba(255, 255, 255, 0.15);
    padding: 3px 8px;
    border-radius: 4px;
  }

  .avatar {
    width: 36px;
    height: 36px;
    border-radius: 8px;
    background: linear-gradient(135deg, #8b5cf6, #7c3aed);
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.2);
    transition: all 0.3s ease;
    border: 2px solid rgba(255, 255, 255, 0.2);
  }

  .avatar:hover {
    transform: translateY(-1px) scale(1.05);
    box-shadow: 0 4px 12px rgba(139, 92, 246, 0.3);
  }

  /* RESPONSIVE */
  @media (min-width: 1024px) {
    .add-text {
      display: inline;
    }
    
    .add-patient-btn {
      padding: 8px 16px;
      min-width: auto;
    }
    
    .search-container {
      max-width: 300px;
    }
  }

  @media (max-width: 1024px) {
    .header-inner {
      padding: 0 16px;
      gap: 14px;
    }
    
    h1 {
      font-size: 17px;
    }
    
    .center-group {
      max-width: 500px;
    }
  }

  @media (max-width: 768px) {
    .header {
      height: 54px;
    }
    
    .header-inner {
      padding: 0 14px;
      gap: 12px;
    }
    
    h1 {
      font-size: 16px;
    }
    
    .hospital-icon {
      display: none;
    }
    
    .center-group {
      max-width: 400px;
      gap: 8px;
    }
    
    .search-container {
      max-width: 220px;
      height: 36px;
    }
    
    .search-input {
      font-size: 13px;
      padding: 0 34px 0 30px;
    }
    
    .search-icon-left {
      left: 10px;
      width: 13px;
      height: 13px;
    }
    
    .clear-btn {
      right: 38px;
    }
    
    .search-hint {
      display: none;
    }
    
    .search-action-btn {
      width: 22px;
      height: 22px;
      right: 7px;
    }
    
    .add-patient-btn {
      min-width: 34px;
      height: 34px;
      padding: 7px;
    }
    
    .add-icon {
      width: 15px;
      height: 15px;
    }
    
    .user-role {
      display: none;
    }
    
    .avatar {
      width: 34px;
      height: 34px;
    }
  }

  @media (max-width: 640px) {
    h1 {
      display: none;
    }
    
    .center-group {
      max-width: 320px;
    }
    
    .search-container {
      max-width: 180px;
      height: 34px;
    }
    
    .search-input {
      font-size: 12px;
      padding: 0 32px 0 28px;
    }
    
    .add-patient-btn {
      min-width: 32px;
      height: 32px;
    }
    
    .avatar {
      width: 32px;
      height: 32px;
    }
  }

  @media (max-width: 480px) {
    .header-inner {
      padding: 0 12px;
      gap: 10px;
    }
    
    .left-section {
      gap: 10px;
    }
    
    .menu-btn {
      width: 32px;
      height: 32px;
    }
    
    .center-group {
      max-width: 260px;
    }
    
    .search-container {
      max-width: 150px;
      height: 32px;
    }
    
    .add-patient-btn {
      min-width: 30px;
      height: 30px;
      padding: 6px;
    }
    
    .add-icon {
      width: 14px;
      height: 14px;
    }
    
    .avatar {
      width: 30px;
      height: 30px;
    }
  }

  @media (max-width: 360px) {
    .center-group {
      max-width: 220px;
    }
    
    .search-container {
      max-width: 130px;
      height: 32px;
    }
    
    .add-patient-btn {
      min-width: 28px;
      height: 28px;
      padding: 5px;
    }
    
    .avatar {
      width: 28px;
      height: 28px;
    }
  }

  /* Animation */
  @keyframes slideIn {
    from {
      opacity: 0;
      transform: translateY(-10px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  .header {
    animation: slideIn 0.3s ease-out;
  }
</style>
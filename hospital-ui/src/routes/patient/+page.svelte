<script lang="ts">
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';
  import { browser } from '$app/environment';
  import { page } from '$app/stores';
  import Swal from 'sweetalert2';



  $: search = $page.url.searchParams.get('search') ?? '';
  let data: any[] = [];
  let loading = true;
  let error = '';
  let showOptionsMenu = false;
  let showColumnModal = false;
  let showExportMenu = false;
  let isExporting = false;
  let isConverted = false;

  
  // Font size settings
  let fontSize = 13;
  const minFontSize = 10;
  const maxFontSize = 18;

  let columns = {
    id: { label: 'ID', visible: true, locked: true },
    name: { label: 'Name', visible: true },
    phone: { label: 'Phone', visible: true },
    gender: { label: 'Gender', visible: true },
    age: { label: 'Age', visible: true },
    city: { label: 'City', visible: true },
    email: { label: 'Email', visible: false },
    bloodGroup: { label: 'Blood Group', visible: false },
    doctor: { label: 'Doctor', visible: false },
    occupation: { label: 'Occupation', visible: false },
    maritalStatus: { label: 'Marital Status', visible: false }
  };

  let tempColumns = structuredClone(columns);
  let columnOrder = [
    'id',
    'name',
    'phone',
    'gender',
    'age',
    'city',
    'email',
    'bloodGroup',
    'doctor',
    'occupation',
    'maritalStatus'
  ];
  let draggedIndex: number | null = null;

  // Patient detail modal
  let showPatientDetail = false;
  let selectedPatient: any = null;

  function onDragStart(index: number, key: string) {
    if (columns[key].locked) return;
    draggedIndex = index;
  }

  function onDrop(index: number) {
    if (draggedIndex === null || draggedIndex === index) return;
    const updated = [...columnOrder];
    const [moved] = updated.splice(draggedIndex, 1);
    updated.splice(index, 0, moved);
    columnOrder = updated;
    draggedIndex = null;
  }

  function openColumnMenu() {
    tempColumns = structuredClone(columns);
    showColumnModal = true;
  }

  function closeExportMenu() {
    showExportMenu = false;
  }

  function applyColumns() {
    columns = structuredClone(tempColumns);
    showColumnModal = false;
  }

  function cancelColumns() {
    showColumnModal = false;
  }

  function openPatientDetail(patientData: any) {
    selectedPatient = patientData;
    showPatientDetail = true;
  }

  function closePatientDetail() {
    showPatientDetail = false;
    selectedPatient = null;
  }
 async function convertToOpd() {
  if (!selectedPatient?.patient?.id) return;

  const patientId = selectedPatient.patient.id;

  // 🔹 Toast config (small popup)
  const Toast = Swal.mixin({
    toast: true,
    position: 'top-end',
    showConfirmButton: false,
    timer: 2000,
    timerProgressBar: true
  });

  try {
    const res = await fetch(
      `http://localhost:8081/api/opd/convert/${patientId}`,
      { method: 'POST' }
    );

    // 🟡 If already ACTIVE
    if (res.status === 409) {
      const result = await Swal.fire({
        toast: true,
        position: 'top-end',
        icon: 'warning',
        title: 'Patient already in OPD',
        showConfirmButton: true,
        confirmButtonText: 'Recreate',
        showCancelButton: true,
        cancelButtonText: 'Cancel',
        timer: 4000
      });

      if (!result.isConfirmed) return;

      const recreateRes = await fetch(
        `http://localhost:8081/api/opd/convert/${patientId}?forceCreate=true`,
        { method: 'POST' }
      );

      if (!recreateRes.ok) {
        const err = await recreateRes.json();
        throw new Error(err.message);
      }

      await Toast.fire({
        icon: 'success',
        title: 'OPD recreated successfully'
      });

      closePatientDetail();
      goto('/opd-patient');
      return;
    }

    if (!res.ok) {
      const err = await res.json();
      throw new Error(err.message);
    }

    // 🟢 Success Toast
    await Toast.fire({
      icon: 'success',
      title: 'Converted to OPD'
    });

    closePatientDetail();
    goto('/opd-patient');

  } catch (err: any) {
    Toast.fire({
      icon: 'error',
      title: err.message || 'Server error'
    });
  }
}

  function increaseFontSize() {
    if (fontSize < maxFontSize) {
      fontSize += 1;
      saveFontSize();
    }
  }

  function decreaseFontSize() {
    if (fontSize > minFontSize) {
      fontSize -= 1;
      saveFontSize();
    }
  }

  function resetFontSize() {
    fontSize = 13;
    saveFontSize();
  }

  function saveFontSize() {
    if (browser) {
      try {
        localStorage.setItem('patientTableFontSize', fontSize.toString());
      } catch (e) {
        console.warn('Could not save font size:', e);
      }
    }
  }

  // Get data in export format
  function getExportData() {
    const visibleColumns = columnOrder.filter(key => columns[key].visible);
    const headers = visibleColumns.map(key => columns[key].label);
    
    const rows = filtered.map(row => {
      return visibleColumns.map(key => {
        const p = row.patient || {};
        const d = row.additionalDetails || {};
        
        switch (key) {
          case 'id':
            return `#${p.id}`;
          case 'name':
            return p.title ? `${p.title} ${p.patientName}`.trim() : p.patientName;
          case 'gender':
            return p.gender === 'M' ? 'Male' : p.gender === 'F' ? 'Female' : p.gender || '';
          case 'age':
            return p.age || '';
          case 'city':
            return p.city || '';
          case 'phone':
            return p.phoneNumber || '';
          case 'email':
            return p.email || '';
          case 'bloodGroup':
            return d.bloodGroup || '';
          case 'doctor':
            return d.referringDoctor || '';
          case 'occupation':
            return d.occupation || '';
          case 'maritalStatus':
            return d.maritalStatus || '';
          default:
            return '';
        }
      });
    });
    
    return { headers, rows, visibleColumns };
  }

  // Export to Excel (XLSX)
  async function exportToExcel() {
    if (isExporting) return;
    isExporting = true;
    
    try {
      const { headers, rows } = getExportData();
      
      // Create worksheet
      const worksheetData = [headers, ...rows];
      const worksheet = XLSX.utils.aoa_to_sheet(worksheetData);
      
      // Auto-size columns
      const colWidths = headers.map((_, colIndex) => {
        const maxLength = Math.max(
          ...worksheetData.map(row => (row[colIndex] || '').toString().length),
          headers[colIndex].length
        );
        return { wch: Math.min(maxLength + 2, 50) };
      });
      worksheet['!cols'] = colWidths;
      
      // Create workbook
      const workbook = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(workbook, worksheet, 'Patients');
      
      // Generate file
      const fileName = `Patients_Export_${new Date().toISOString().split('T')[0]}.xlsx`;
      XLSX.writeFile(workbook, fileName);
      
      closeExportMenu();
    } catch (error) {
      alert('Error exporting to Excel: ' + error.message);
      console.error('Excel export error:', error);
    } finally {
      isExporting = false;
    }
  }

  // Export to CSV
  function exportToCSV() {
    if (isExporting) return;
    isExporting = true;
    
    try {
      const { headers, rows } = getExportData();
      
      // Convert to CSV with proper escaping
      const csvRows = [];
      
      // Add headers
      csvRows.push(headers.map(header => escapeCSV(header)).join(','));
      
      // Add data rows
      rows.forEach(row => {
        csvRows.push(row.map(cell => escapeCSV(cell.toString())).join(','));
      });
      
      // Create and download file
      const csvContent = csvRows.join('\n');
      const blob = new Blob(['\uFEFF' + csvContent], { type: 'text/csv;charset=utf-8;' });
      const url = URL.createObjectURL(blob);
      const link = document.createElement('a');
      link.href = url;
      link.download = `Patients_Export_${new Date().toISOString().split('T')[0]}.csv`;
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
      URL.revokeObjectURL(url);
      
      closeExportMenu();
    } catch (error) {
      alert('Error exporting to CSV: ' + error.message);
      console.error('CSV export error:', error);
    } finally {
      isExporting = false;
    }
  }

  // Helper functions
  function escapeCSV(str: string): string {
    if (str.includes(',') || str.includes('"') || str.includes('\n') || str.includes('\r')) {
      return '"' + str.replace(/"/g, '""') + '"';
    }
    return str;
  }

  function escapeHTML(str: string): string {
    const div = document.createElement('div');
    div.textContent = str;
    return div.innerHTML;
  }

  // Enhanced search function
  function searchPatient(row: any, query: string): boolean {
    if (!query.trim()) return true;
    
    const p = row.patient || {};
    const d = row.additionalDetails || {};
    const q = query.toLowerCase().trim();
    
    // Define all searchable fields with their values
    const searchFields = [
      // Patient fields
      p.id?.toString(),
      p.patientName,
      p.title,
      `${p.title || ''} ${p.patientName || ''}`.trim(), // Full name with title
      p.gender,
      p.gender === 'M' ? 'male' : p.gender === 'F' ? 'female' : '',
      p.age?.toString(),
      p.city,
      p.phoneNumber,
      p.email,
      p.address,
      p.state,
      p.country,
      p.pincode,
      p.emergencyContact,
      p.alternatePhone,
      
      // Additional details fields
      d.bloodGroup,
      d.referringDoctor,
      d.occupation,
      d.maritalStatus,
      d.insuranceProvider,
      d.insuranceNumber,
      d.allergies,
      d.medicalHistory,
      d.familyHistory,
      d.notes
    ];
    
    // Special handling for ID search (search with or without #)
    if (q.startsWith('#') && p.id?.toString()?.includes(q.substring(1))) {
      return true;
    }
    
    // Search through all fields
    return searchFields.some(field => {
      if (!field) return false;
      return field.toString().toLowerCase().includes(q);
    });
  }

  async function loadPatients() {
    try {
      const res = await fetch('http://localhost:8081/api/patients/with-details');
      if (!res.ok) throw new Error('Failed to fetch patients');
      data = await res.json();
    } catch {
      error = 'Unable to load patients';
    } finally {
      loading = false;
    }
  }

  onMount(() => {
    loadPatients();
    
    // Load saved font size
    if (browser) {
      try {
        const savedFontSize = localStorage.getItem('patientTableFontSize');
        if (savedFontSize) {
          const parsed = parseInt(savedFontSize);
          if (!isNaN(parsed) && parsed >= minFontSize && parsed <= maxFontSize) {
            fontSize = parsed;
          }
        }
      } catch (e) {
        console.warn('Could not load font size:', e);
      }
    }

    // Load SheetJS library for Excel export
    if (browser) {
      const script = document.createElement('script');
      script.src = 'https://cdn.sheetjs.com/xlsx-0.20.0/package/dist/xlsx.full.min.js';
      script.onload = () => console.log('SheetJS loaded successfully');
      script.onerror = () => console.error('Failed to load SheetJS');
      document.head.appendChild(script);
    }

    // Close menus when clicking outside
    function handleClickOutside(event: MouseEvent) {
      const target = event.target as HTMLElement;
      if (showExportMenu && !target.closest('.three-line-wrapper') && !target.closest('.export-menu')) {
        closeExportMenu();
      }
      if (showPatientDetail && !target.closest('.patient-modal-content') && !target.closest('.patient-name')) {
        closePatientDetail();
      }
      if (showOptionsMenu && !target.closest('.three-line-button') && !target.closest('.options-menu')) {
        showOptionsMenu = false;
      }
    }
    
    if (browser) {
      document.addEventListener('click', handleClickOutside);
      return () => document.removeEventListener('click', handleClickOutside);
    }
  });

  // Filter data based on search
  $: filtered = search.trim() === '' 
    ? data 
    : data.filter(row => searchPatient(row, search));
</script>

<div class="container">
  <!-- HEADER -->
  <div class="header">
    <div>
      <h1 class="page-title">Registered Patients</h1>
      {#if !loading && !error}
        <p class="patient-count">{filtered.length} {filtered.length === 1 ? 'patient' : 'patients'}</p>
        {#if search.trim()}
          <p class="search-result-info">Showing results for "{search}"</p>
        {/if}
      {/if}
    </div>
    
    <div class="header-actions">    
      <!-- Font Size Tool -->
      <div class="font-size-tool">
        <button 
          class="font-size-btn {fontSize === minFontSize ? 'disabled' : ''}" 
          on:click={decreaseFontSize}
          title="Decrease font size"
          disabled={fontSize === minFontSize}
        >
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20 12H4"/>
          </svg>
        </button>
        
        <div class="font-size-display">
          <span class="font-size-value">{fontSize}px</span>
          <span class="font-size-label">Text Size</span>
        </div>
        
        <button 
          class="font-size-btn {fontSize === maxFontSize ? 'disabled' : ''}" 
          on:click={increaseFontSize}
          title="Increase font size"
          disabled={fontSize === maxFontSize}
        >
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4"/>
          </svg>
        </button>
        
        <button 
          class="font-size-reset" 
          on:click={resetFontSize}
          title="Reset to default size"
        >
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"/>
          </svg>
        </button>
      </div>
      
      <!-- Three-line Options Button -->
      <div class="three-line-wrapper">
        <button 
          class="three-line-button {showOptionsMenu ? 'active' : ''}" 
          on:click={() => showOptionsMenu = !showOptionsMenu}
          title="More options"
        >
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <circle cx="12" cy="6" r="1"></circle>
            <circle cx="12" cy="12" r="1"></circle>
            <circle cx="12" cy="18" r="1"></circle>
          </svg>
        </button>

        <!-- Options Menu Dropdown -->
        {#if showOptionsMenu}
          <div class="options-menu" on:click|stopPropagation>
            <div class="menu-header">
              <h4>Table Options</h4>
              <p>Manage your view and data</p>
            </div>
            
            <div class="menu-items">
              <button 
                class="menu-item"
                on:click={() => {
                  showOptionsMenu = false;
                  openColumnMenu();
                }}
              >
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path stroke-linecap="round" stroke-linejoin="round" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2"/>
                </svg>
                <span>Manage Columns</span>
              </button>
              
              <button 
                class="menu-item"
                on:click={() => {
                  showOptionsMenu = false;
                  showExportMenu = true;
                }}
              >
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path stroke-linecap="round" stroke-linejoin="round" d="M12 10v6m0 0l-3-3m3 3l3-3m2 8H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"/>
                </svg>
                <span>Export Data</span>
              </button>
            </div>
          </div>
        {/if}

        <!-- Export Menu (opens from options menu) -->
        {#if showExportMenu}
          <div class="export-menu" on:click|stopPropagation>
            <div class="export-header">
              <h4>Export Patient Data</h4>
              <p>Export {filtered.length} patient{filtered.length !== 1 ? 's' : ''}</p>
            </div>

            <div class="export-options">
              <button class="export-option" on:click={exportToExcel} disabled={isExporting}>
                <div class="export-icon">
                  <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M9 17v-2m3 2v-4m3 4v-6m2 10H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"/>
                  </svg>
                </div>
                <div class="export-info">
                  <span class="export-type">Excel (.xlsx)</span>
                  <span class="export-desc">Spreadsheet with formatting</span>
                </div>
              </button>

              <button class="export-option" on:click={exportToCSV} disabled={isExporting}>
                <div class="export-icon">
                  <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"/>
                  </svg>
                </div>
                <div class="export-info">
                  <span class="export-type">CSV (.csv)</span>
                  <span class="export-desc">Simple text format</span>
                </div>
              </button>
            </div>

            <div class="export-footer">
              <span class="export-note">
                {columnOrder.filter(key => columns[key].visible).length} columns • {filtered.length} records
              </span>
              <button class="btn btn-outline btn-sm" on:click={closeExportMenu}>
                Cancel
              </button>
            </div>
          </div>
        {/if}
      </div>
    </div>
  </div>

  <!-- COLUMN MENU MODAL -->
  {#if showColumnModal}
    <div class="column-modal">
      <div class="modal-content">
        <div class="modal-header">
          <h3>Manage Columns</h3>
          <p>Select which columns to display and drag to reorder</p>
        </div>
        
        <div class="columns-list">
          {#each columnOrder as key, index}
            {#if tempColumns[key]}
              <div class="column-item {tempColumns[key].locked ? 'locked' : ''}"
                   draggable={!tempColumns[key].locked}
                   on:dragstart={() => onDragStart(index, key)}
                   on:dragover|preventDefault
                   on:drop={() => onDrop(index)}>
                <div class="column-info">
                  <label class="custom-checkbox">
                    <input type="checkbox" bind:checked={tempColumns[key].visible} disabled={tempColumns[key].locked} />
                    <span class="checkmark"></span>
                  </label>
                  <span class="column-label">{tempColumns[key].label}</span>
                </div>
                
                <div class="column-status">
                  {#if tempColumns[key].locked}
                    <span class="badge badge-locked">Locked</span>
                  {:else}
                    <span class="drag-handle">⋮⋮</span>
                  {/if}
                </div>
              </div>
            {/if}
          {/each}
        </div>
        
        <div class="modal-footer">
          <button class="btn btn-outline" on:click={cancelColumns}>Cancel</button>
          <button class="btn btn-primary" on:click={applyColumns}>Save Changes</button>
        </div>
      </div>
    </div>
  {/if}

  <!-- PATIENT DETAIL MODAL - COMPACT SQUARE HEADER DESIGN -->
  {#if showPatientDetail && selectedPatient}
    <div class="patient-modal">
      <div class="patient-modal-content compact-design" on:click|stopPropagation>
        <!-- Compact Square Header -->
        <div class="compact-header">
          <div class="compact-header-content">
            <div class="compact-avatar">
              <div class="avatar-initials">
                {#if selectedPatient.patient?.patientName}
                  {selectedPatient.patient.patientName.split(' ').map(n => n[0]).join('').substring(0, 2).toUpperCase()}
                {:else}
                  PP
                {/if}
              </div>
            </div>
            <div class="compact-patient-info">
              <div class="compact-name">
                {#if selectedPatient.patient?.title}
                  {selectedPatient.patient.title} {selectedPatient.patient.patientName}
                {:else}
                  {selectedPatient.patient.patientName}
                {/if}
              </div>
              <div class="compact-details">
                <span class="compact-id">ID: #{selectedPatient.patient.id}</span>
                <span class="compact-gender">
                  {selectedPatient.patient.gender === 'M' ? 'Male' : selectedPatient.patient.gender === 'F' ? 'Female' : selectedPatient.patient.gender}
                </span>
                <span class="compact-age">{selectedPatient.patient.age} years</span>
              </div>
            </div>
            <button class="compact-close" on:click={closePatientDetail}>
              <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/>
              </svg>
            </button>
          </div>
        </div>
        
        <!-- Modal Body -->
        <div class="compact-body">
          <div class="info-sections">
            <!-- Contact Info -->
            <div class="info-section">
              <h4 class="section-title">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 5a2 2 0 012-2h3.28a1 1 0 01.948.684l1.498 4.493a1 1 0 01-.502 1.21l-2.257 1.13a11.042 11.042 0 005.516 5.516l1.13-2.257a1 1 0 011.21-.502l4.493 1.498a1 1 0 01.684.949V19a2 2 0 01-2 2h-1C9.716 21 3 14.284 3 6V5z"/>
                </svg>
                Contact Information
              </h4>
              <div class="info-grid">
                <div class="info-item">
                  <span class="info-label">Phone</span>
                  <span class="info-value">{selectedPatient.patient.phoneNumber}</span>
                </div>
                {#if selectedPatient.patient.email}
                <div class="info-item">
                  <span class="info-label">Email</span>
                  <span class="info-value">{selectedPatient.patient.email}</span>
                </div>
                {/if}
                {#if selectedPatient.patient.emergencyContact}
                <div class="info-item">
                  <span class="info-label">Emergency Contact</span>
                  <span class="info-value">{selectedPatient.patient.emergencyContact}</span>
                </div>
                {/if}
              </div>
            </div>
            
            <!-- Address Info -->
            <div class="info-section">
              <h4 class="section-title">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z"/>
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 11a3 3 0 11-6 0 3 3 0 016 0z"/>
                </svg>
                Address Information
              </h4>
              <div class="info-grid">
                <div class="info-item full-width">
                  <span class="info-label">Address</span>
                  <span class="info-value">{selectedPatient.patient.address || 'Not specified'}</span>
                </div>
                <div class="info-item">
                  <span class="info-label">City</span>
                  <span class="info-value">{selectedPatient.patient.city || 'Not specified'}</span>
                </div>
                <div class="info-item">
                  <span class="info-label">State</span>
                  <span class="info-value">{selectedPatient.patient.state || 'Not specified'}</span>
                </div>
                {#if selectedPatient.patient.pincode}
                <div class="info-item">
                  <span class="info-label">Pincode</span>
                  <span class="info-value">{selectedPatient.patient.pincode}</span>
                </div>
                {/if}
              </div>
            </div>
            
            <!-- Medical Info -->
            <div class="info-section">
              <h4 class="section-title">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19.428 15.428a2 2 0 00-1.022-.547l-2.387-.477a6 6 0 00-3.86.517l-.318.158a6 6 0 01-3.86.517L6.05 15.21a2 2 0 00-1.806.547M8 4h8l-1 1v5.172a2 2 0 00.586 1.414l5 5c1.26 1.26.367 3.414-1.415 3.414H4.828c-1.782 0-2.674-2.154-1.414-3.414l5-5A2 2 0 009 10.172V5L8 4z"/>
                </svg>
                Medical Information
              </h4>
              <div class="info-grid">
                {#if selectedPatient.additionalDetails?.bloodGroup}
                <div class="info-item">
                  <span class="info-label">Blood Group</span>
                  <span class="info-value blood-group">{selectedPatient.additionalDetails.bloodGroup}</span>
                </div>
                {/if}
                {#if selectedPatient.additionalDetails?.referringDoctor}
                <div class="info-item">
                  <span class="info-label">Doctor</span>
                  <span class="info-value">{selectedPatient.additionalDetails.referringDoctor}</span>
                </div>
                {/if}
                {#if selectedPatient.additionalDetails?.occupation}
                <div class="info-item">
                  <span class="info-label">Occupation</span>
                  <span class="info-value">{selectedPatient.additionalDetails.occupation}</span>
                </div>
                {/if}
                {#if selectedPatient.additionalDetails?.maritalStatus}
                <div class="info-item">
                  <span class="info-label">Marital Status</span>
                  <span class="info-value">{selectedPatient.additionalDetails.maritalStatus}</span>
                </div>
                {/if}
                {#if selectedPatient.additionalDetails?.insuranceProvider}
                <div class="info-item">
                  <span class="info-label">Insurance Provider</span>
                  <span class="info-value">{selectedPatient.additionalDetails.insuranceProvider}</span>
                </div>
                {/if}
                {#if selectedPatient.additionalDetails?.allergies}
                <div class="info-item full-width">
                  <span class="info-label">Allergies</span>
                  <span class="info-value">{selectedPatient.additionalDetails.allergies}</span>
                </div>
                {/if}
                {#if selectedPatient.additionalDetails?.medicalHistory}
                <div class="info-item full-width">
                  <span class="info-label">Medical History</span>
                  <span class="info-value">{selectedPatient.additionalDetails.medicalHistory}</span>
                </div>
                {/if}
              </div>
            </div>
          </div>
        </div>
        
                         <!-- Modal Footer -->
<div class="compact-footer">
  <button class="btn btn-outline" on:click={closePatientDetail}>
    Close
  </button>

  <!-- 🔥 CONVERT TO OPD BUTTON -->
 <button 
  class="btn"
  style="background:#f59e0b;color:white;"
  disabled={isConverted}
  on:click={convertToOpd}>

  {#if isConverted}
    Already in OPD
  {:else}
    Convert to OPD
  {/if}
</button>


  <button class="btn btn-primary" on:click={() => {
    goto(`/patient/edit/${selectedPatient.patient.id}`);
    closePatientDetail();
  }}>
    Edit Patient
  </button>
</div>
      </div>
    </div>
  {/if}

  <!-- MAIN CONTENT -->
  <div class="content-card">
    {#if loading}
      <div class="loading-state">
        <div class="loader"></div>
        <p>Loading patient data...</p>
      </div>
    {:else if error}
      <div class="error-state">
        <svg width="40" height="40" viewBox="0 0 24 24" fill="none" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/>
        </svg>
        <p>{error}</p>
        <button class="btn btn-outline" on:click={loadPatients}>Retry</button>
      </div>
    {:else if filtered.length === 0}
      <div class="empty-state">
        <svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M9.172 16.172a4 4 0 015.656 0M9 10h.01M15 10h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/>
        </svg>
        <h3>No patients found</h3>
        <p>{search ? `No results for "${search}"` : 'Get started by adding your first patient'}</p>
        {#if search}
          <button class="btn btn-text" on:click={() => goto('/patient')}>Clear search</button>
        {:else}
          <button class="btn btn-primary" on:click={() => goto('/add-patient')}>Add Patient</button>
        {/if}
      </div>
    {:else}
      <div class="table-scroll-container" style="--table-font-size: {fontSize}px;">
        <div class="table-wrapper">
          <table>
            <thead>
              <tr>
                {#each columnOrder as key, index}
                  {#if columns[key].visible}
                    <th draggable={!columns[key].locked}
                        class:locked={columns[key].locked}
                        on:dragstart={() => onDragStart(index, key)}
                        on:dragover|preventDefault
                        on:drop={() => onDrop(index)}>
                      <div class="th-content">
                        {columns[key].label}
                        {#if !columns[key].locked}
                          <span class="drag-indicator">⋮</span>
                        {/if}
                      </div>
                    </th>
                  {/if}
                {/each}
              </tr>
            </thead>
            
            <tbody>
              {#each filtered as row}
                <tr>
                  {#each columnOrder as key}
                    {#if columns[key].visible}
                      <td class="cell-{key}">
                        {#if key === 'id'}
                          <span class="patient-id">#{row.patient.id}</span>
                        {:else if key === 'name'}
                          <div class="patient-name clickable" on:click|stopPropagation={() => openPatientDetail(row)}>
                            <strong>{row.patient.patientName}</strong>
                            {#if row.patient.title}
                              <span class="title">{row.patient.title}</span>
                            {/if}
                            <svg class="name-arrow" width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor">
                              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"/>
                            </svg>
                          </div>
                        {:else if key === 'gender'}
                          <span class="gender-badge {row.patient.gender?.toLowerCase()}">
                            {row.patient.gender === 'M' ? 'Male' : row.patient.gender === 'F' ? 'Female' : row.patient.gender}
                          </span>
                        {:else if key === 'age'}
                          <span class="age-cell">{row.patient.age ?? '—'}</span>
                        {:else if key === 'city'}
                          <span class="city-cell">{row.patient.city ?? '—'}</span>
                        {:else if key === 'phone'}
                          <span class="phone-number">{row.patient.phoneNumber}</span>
                        {:else if key === 'email'}
                          <span class="email">{row.patient.email ?? '—'}</span>
                        {:else if key === 'bloodGroup'}
                          <span class="blood-group">{row.additionalDetails?.bloodGroup ?? '—'}</span>
                        {:else if key === 'doctor'}
                          <span class="doctor-cell">{row.additionalDetails?.referringDoctor ?? '—'}</span>
                        {:else if key === 'occupation'}
                          <span class="occupation-cell">{row.additionalDetails?.occupation ?? '—'}</span>
                        {:else if key === 'maritalStatus'}
                          <span class="marital-cell">{row.additionalDetails?.maritalStatus ?? '—'}</span>
                        {/if}
                      </td>
                    {/if}
                  {/each}
                </tr>
              {/each}
            </tbody>
          </table>
        </div>
        <div class="scroll-indicator">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 14l-7 7m0 0l-7-7m7 7V3"/>
          </svg>
          <span>Scroll to see more</span>
        </div>
      </div>
    {/if}
  </div>
</div>

<style>
  :global(*) {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  .container {
    padding: 24px;
    max-width: 1400px;
    margin: 0 auto;
    background: #f8fafc;
    min-height: 100vh;
  }

  /* HEADER */
  .header {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    margin-bottom: 24px;
  }

  .page-title {
    font-size: 24px;
    font-weight: 600;
    color: #1e293b;
    margin-bottom: 6px;
  }

  .patient-count {
    color: #64748b;
    font-size: 13px;
    font-weight: 500;
  }

  .search-result-info {
    color: #f97316;
    font-size: 13px;
    font-weight: 500;
    margin-top: 4px;
    background: #fff7ed;
    padding: 4px 8px;
    border-radius: 4px;
    display: inline-block;
  }

  .header-actions {
    display: flex;
    gap: 8px;
    align-items: center;
    position: relative;
  }

  /* FONT SIZE TOOL */
  .font-size-tool {
    display: flex;
    align-items: center;
    background: white;
    border: 1px solid #e2e8f0;
    border-radius: 8px;
    padding: 4px;
    height: 40px;
  }

  .font-size-btn {
    width: 32px;
    height: 32px;
    display: flex;
    align-items: center;
    justify-content: center;
    background: white;
    border: 1px solid #e2e8f0;
    border-radius: 6px;
    cursor: pointer;
    transition: all 0.2s;
    color: #475569;
  }

  .font-size-btn:hover:not(.disabled) {
    background: #f8fafc;
    border-color: #cbd5e1;
  }

  .font-size-btn.disabled {
    opacity: 0.4;
    cursor: not-allowed;
  }

  .font-size-display {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-width: 70px;
    padding: 0 8px;
    font-family: 'SF Mono', Monaco, 'Cascadia Mono', monospace;
  }

  .font-size-value {
    font-size: 13px;
    font-weight: 600;
    color: #1e293b;
    line-height: 1.2;
  }

  .font-size-label {
    font-size: 10px;
    color: #64748b;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    line-height: 1.2;
  }

  .font-size-reset {
    width: 32px;
    height: 32px;
    display: flex;
    align-items: center;
    justify-content: center;
    background: #f8fafc;
    border: 1px solid #e2e8f0;
    border-radius: 6px;
    margin-left: 4px;
    cursor: pointer;
    transition: all 0.2s;
    color: #475569;
  }

  .font-size-reset:hover {
    background: #e2e8f0;
    border-color: #cbd5e1;
  }

  /* THREE-LINE BUTTON */
  .three-line-wrapper {
    position: relative;
  }

  .three-line-button {
    width: 40px;
    height: 40px;
    display: flex;
    align-items: center;
    justify-content: center;
    background: white;
    border: 1px solid #e2e8f0;
    border-radius: 8px;
    cursor: pointer;
    transition: all 0.2s ease;
    color: #475569;
  }

  .three-line-button:hover {
    background: #f8fafc;
    border-color: #cbd5e1;
  }

  .three-line-button.active {
    background: #3b82f6;
    border-color: #3b82f6;
    color: white;
  }

  .three-line-button svg {
    transition: transform 0.2s ease;
  }

  .three-line-button.active svg {
    transform: rotate(90deg);
  }

  /* OPTIONS MENU */
  .options-menu {
    position: absolute;
    top: 100%;
    right: 0;
    margin-top: 8px;
    background: white;
    border: 1px solid #e2e8f0;
    border-radius: 12px;
    box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1);
    width: 240px;
    z-index: 1000;
    animation: slideDown 0.2s ease;
    overflow: hidden;
  }

  @keyframes slideDown {
    from {
      opacity: 0;
      transform: translateY(-10px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  .menu-header {
    padding: 16px 20px;
    border-bottom: 1px solid #f1f5f9;
    background: #f8fafc;
  }

  .menu-header h4 {
    margin: 0 0 4px 0;
    font-size: 14px;
    color: #1e293b;
    font-weight: 600;
  }

  .menu-header p {
    margin: 0;
    font-size: 12px;
    color: #64748b;
  }

  .menu-items {
    padding: 8px 0;
  }

  .menu-item {
    display: flex;
    align-items: center;
    gap: 12px;
    width: 100%;
    padding: 12px 20px;
    border: none;
    background: white;
    cursor: pointer;
    transition: all 0.2s;
    text-align: left;
    font-size: 13px;
    color: #475569;
  }

  .menu-item:hover {
    background: #f8fafc;
  }

  .menu-item svg {
    color: #64748b;
    flex-shrink: 0;
  }

  .menu-item span {
    flex: 1;
  }

  /* EXPORT MENU */
  .export-menu {
    position: absolute;
    top: 100%;
    right: 0;
    margin-top: 8px;
    background: white;
    border: 1px solid #e2e8f0;
    border-radius: 12px;
    box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.1);
    width: 300px;
    z-index: 1001;
    animation: slideDown 0.2s ease;
  }

  .export-header {
    padding: 20px;
    border-bottom: 1px solid #f1f5f9;
  }

  .export-header h4 {
    margin: 0 0 4px 0;
    font-size: 16px;
    color: #1e293b;
    font-weight: 600;
  }

  .export-header p {
    margin: 0;
    font-size: 13px;
    color: #64748b;
  }

  .export-options {
    padding: 12px;
  }

  .export-option {
    display: flex;
    align-items: center;
    gap: 12px;
    width: 100%;
    padding: 12px;
    border: none;
    background: white;
    border-radius: 8px;
    cursor: pointer;
    transition: all 0.2s;
    text-align: left;
  }

  .export-option:hover:not(:disabled) {
    background: #f8fafc;
    transform: translateY(-1px);
  }

  .export-option:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }

  .export-icon {
    width: 40px;
    height: 40px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 8px;
    background: #eff6ff;
    color: #3b82f6;
  }

  .export-info {
    flex: 1;
    display: flex;
    flex-direction: column;
    align-items: flex-start;
  }

  .export-type {
    font-size: 14px;
    font-weight: 500;
    color: #1e293b;
  }

  .export-desc {
    font-size: 12px;
    color: #64748b;
  }

  .export-footer {
    padding: 12px 20px;
    border-top: 1px solid #f1f5f9;
    background: #f8fafc;
    border-radius: 0 0 12px 12px;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .export-note {
    font-size: 12px;
    color: #64748b;
  }

  .btn-sm {
    padding: 6px 12px;
    font-size: 12px;
  }

  /* BUTTONS */
  .btn {
    display: inline-flex;
    align-items: center;
    gap: 6px;
    padding: 8px 16px;
    border-radius: 6px;
    font-size: 13px;
    font-weight: 500;
    border: none;
    cursor: pointer;
    transition: all 0.2s ease;
    white-space: nowrap;
  }

  .btn-primary {
    background: #3b82f6;
    color: white;
  }

  .btn-primary:hover {
    background: #2563eb;
  }

  .btn-outline {
    background: transparent;
    color: #475569;
    border: 1px solid #cbd5e1;
  }

  .btn-outline:hover {
    background: #f8fafc;
  }

  .btn-text {
    background: transparent;
    color: #3b82f6;
    padding: 6px 12px;
    font-size: 12px;
  }

  .btn-text:hover {
    background: #eff6ff;
  }

  /* COLUMN MODAL */
  .column-modal, .patient-modal {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 1000;
    animation: fadeIn 0.2s ease;
    padding: 20px;
  }

  .modal-content {
    background: white;
    border-radius: 12px;
    width: 90%;
    max-width: 460px;
    max-height: 70vh;
    display: flex;
    flex-direction: column;
    box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1);
  }

  /* COMPACT PATIENT MODAL - SMALL SQUARE HEADER */
  .patient-modal-content.compact-design {
    background: white;
    border-radius: 12px;
    width: 90%;
    max-width: 800px;
    max-height: 85vh;
    display: flex;
    flex-direction: column;
    box-shadow: 0 10px 25px rgba(0, 0, 0, 0.15);
    border: 1px solid #e2e8f0;
    animation: modalSlideUp 0.3s ease;
    overflow: hidden;
  }

  @keyframes modalSlideUp {
    from {
      opacity: 0;
      transform: translateY(20px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  /* Compact Square Header - Small, Light Blue */
  .compact-header {
    width: 100%;
    height: 100px;
    background: linear-gradient(135deg, #e6f2ff 0%, #d6e8ff 100%);
    display: flex;
    align-items: center;
    padding: 0 24px;
    border-bottom: 1px solid #c5d9ff;
    position: relative;
  }

  .compact-header-content {
    display: flex;
    align-items: center;
    gap: 16px;
    width: 100%;
  }

  .compact-avatar {
    width: 64px;
    height: 64px;
    background: white;
    border-radius: 8px;
    display: flex;
    align-items: center;
    justify-content: center;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    border: 2px solid #b8d6ff;
    flex-shrink: 0;
  }

  .avatar-initials {
    width: 56px;
    height: 56px;
    border-radius: 6px;
    background: linear-gradient(135deg, #4a90e2 0%, #3a7bc8 100%);
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    font-size: 18px;
    font-weight: 600;
  }

  .compact-patient-info {
    flex: 1;
    min-width: 0;
  }

  .compact-name {
    font-size: 18px;
    font-weight: 600;
    color: #2c3e50;
    margin-bottom: 4px;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .compact-details {
    display: flex;
    align-items: center;
    gap: 12px;
    flex-wrap: wrap;
  }

  .compact-id {
    font-size: 13px;
    color: #4a90e2;
    font-weight: 500;
    background: rgba(74, 144, 226, 0.1);
    padding: 4px 10px;
    border-radius: 4px;
    border: 1px solid rgba(74, 144, 226, 0.2);
  }

  .compact-gender, .compact-age {
    font-size: 13px;
    color: #64748b;
    font-weight: 500;
    padding: 4px 0;
  }

  .compact-close {
    background: white;
    border: 1px solid #e2e8f0;
    width: 36px;
    height: 36px;
    border-radius: 6px;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    color: #64748b;
    transition: all 0.2s ease;
    flex-shrink: 0;
  }

  .compact-close:hover {
    background: #f1f5f9;
    border-color: #cbd5e1;
  }

  /* Compact Body */
  .compact-body {
    padding: 24px;
    overflow-y: auto;
    flex: 1;
    background: white;
  }

  .info-sections {
    display: flex;
    flex-direction: column;
    gap: 24px;
  }

  .info-section {
    background: #f8fafc;
    border-radius: 8px;
    padding: 20px;
    border: 1px solid #e2e8f0;
  }

  .section-title {
    font-size: 14px;
    font-weight: 600;
    color: #475569;
    margin: 0 0 16px 0;
    padding-bottom: 12px;
    border-bottom: 1px solid #e2e8f0;
    display: flex;
    align-items: center;
    gap: 8px;
  }

  .info-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 16px;
  }

  .info-item {
    display: flex;
    flex-direction: column;
    gap: 4px;
  }

  .info-item.full-width {
    grid-column: 1 / -1;
  }

  .info-label {
    font-size: 12px;
    color: #64748b;
    font-weight: 500;
    text-transform: uppercase;
    letter-spacing: 0.05em;
  }

  .info-value {
    font-size: 14px;
    color: #1e293b;
    font-weight: 500;
    line-height: 1.5;
  }

  .blood-group {
    background: #fee2e2;
    color: #dc2626;
    padding: 4px 12px;
    border-radius: 4px;
    font-weight: 600;
    display: inline-block;
    width: fit-content;
  }

  /* Compact Footer */
  .compact-footer {
    padding: 20px 24px;
    border-top: 1px solid #e2e8f0;
    display: flex;
    justify-content: flex-end;
    gap: 12px;
    background: #f8fafc;
  }

  .modal-header {
    padding: 20px;
    border-bottom: 1px solid #f1f5f9;
  }

  .modal-header h3 {
    font-size: 16px;
    color: #1e293b;
    margin-bottom: 4px;
  }

  .modal-header p {
    font-size: 13px;
    color: #64748b;
  }

  .columns-list {
    padding: 16px 20px;
    overflow-y: auto;
    flex: 1;
  }

  .column-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 10px 0;
    border-bottom: 1px solid #f8fafc;
    cursor: move;
  }

  .column-item:hover {
    background: #f8fafc;
    margin: 0 -20px;
    padding: 10px 20px;
  }

  .column-item.locked {
    cursor: not-allowed;
    opacity: 0.7;
  }

  .column-info {
    display: flex;
    align-items: center;
    gap: 10px;
  }

  .custom-checkbox {
    position: relative;
    display: inline-block;
  }

  .custom-checkbox input {
    opacity: 0;
    position: absolute;
  }

  .checkmark {
    width: 16px;
    height: 16px;
    border: 2px solid #cbd5e1;
    border-radius: 4px;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all 0.2s;
  }

  .custom-checkbox input:checked + .checkmark {
    background: #3b82f6;
    border-color: #3b82f6;
  }

  .checkmark:after {
    content: '✓';
    color: white;
    font-size: 11px;
    opacity: 0;
    transition: opacity 0.2s;
  }

  .custom-checkbox input:checked + .checkmark:after {
    opacity: 1;
  }

  .column-label {
    font-size: 13px;
    color: #334155;
  }

  .column-status {
    display: flex;
    align-items: center;
  }

  .badge {
    padding: 3px 8px;
    border-radius: 10px;
    font-size: 11px;
    font-weight: 500;
  }

  .badge-locked {
    background: #fef2f2;
    color: #dc2626;
  }

  .drag-handle {
    color: #94a3b8;
    font-size: 14px;
    cursor: grab;
    user-select: none;
  }

  .modal-footer {
    padding: 16px 20px;
    border-top: 1px solid #f1f5f9;
    display: flex;
    justify-content: flex-end;
    gap: 10px;
    background: #f8fafc;
    border-radius: 0 0 12px 12px;
  }

 .content-card {
  background: white;
  border-radius: 10px;
  border: 1px solid #e2e8f0;
  max-height: 600px;
  overflow-y: auto;
  overflow: hidden;
}


  /* TABLE SCROLL CONTAINER */
  .table-scroll-container {
    position: relative;
    --table-font-size: 13px;
  }

 .table-wrapper {
  height: 500px;          /* vertical scroll */
  overflow-y: auto;       /* vertical */
  overflow-x: auto;       /* horizontal */
  white-space: nowrap;    /* prevents column wrapping */
  scrollbar-width: thin;
}

  .table-wrapper::-webkit-scrollbar {
  height: 8px;   /* horizontal scrollbar height */
  width: 8px;    /* vertical scrollbar width */
}

.table-wrapper::-webkit-scrollbar-track {
  background: #f1f5f9;
  border-radius: 4px;
}

.table-wrapper::-webkit-scrollbar-thumb {
  background: #cbd5e1;
  border-radius: 4px;
}

.table-wrapper::-webkit-scrollbar-thumb:hover {
  background: #94a3b8;
}

  /* DYNAMIC FONT SIZE TABLE */
  table {
  min-width: 1400px;   /* increase width */
  border-collapse: collapse;
  font-size: var(--table-font-size, 13px);
}


  thead {
    position: sticky;
    top: 0;
    background: white;
    z-index: 10;
  }

  th {
    padding: calc(var(--table-font-size, 13px) * 0.92) calc(var(--table-font-size, 13px) * 1.23);
    text-align: left;
    font-weight: 600;
    color: #475569;
    border-bottom: 2px solid #e2e8f0;
    user-select: none;
    position: relative;
    white-space: nowrap;
    font-size: calc(var(--table-font-size, 13px) * 0.92);
    text-transform: uppercase;
    letter-spacing: 0.05em;
  }

  th:not(.locked):hover {
    background: #f8fafc;
  }

  .th-content {
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  .drag-indicator {
    color: #94a3b8;
    font-size: calc(var(--table-font-size, 13px) * 1.23);
    cursor: grab;
    opacity: 0.5;
    transition: opacity 0.2s;
  }

  th:not(.locked):hover .drag-indicator {
    opacity: 1;
  }

  /* TABLE ROWS WITH DYNAMIC FONT SIZE */
  td {
    padding: calc(var(--table-font-size, 13px) * 0.92) calc(var(--table-font-size, 13px) * 1.23);
    border-bottom: 1px solid #f1f5f9;
    color: #475569;
    vertical-align: top;
    line-height: 1.4;
  }

  tbody tr {
    transition: background 0.2s ease;
  }

  tbody tr:hover {
    background: #f8fafc;
  }

  tbody tr:last-child td {
    border-bottom: none;
  }

  /* CELL STYLES - SCALING WITH FONT SIZE */
  .patient-id {
    background: #eff6ff;
    color: #1d4ed8;
    padding: calc(var(--table-font-size, 13px) * 0.23) calc(var(--table-font-size, 13px) * 0.62);
    border-radius: 12px;
    font-size: calc(var(--table-font-size, 13px) * 0.85);
    font-weight: 500;
    display: inline-block;
  }

  .patient-name {
    display: flex;
    flex-direction: column;
    gap: 2px;
    min-width: 150px;
    position: relative;
    cursor: default;
  }

  .patient-name.clickable {
    cursor: pointer;
  }

  .patient-name.clickable:hover strong {
    color: #3b82f6;
    text-decoration: underline;
  }

  .patient-name.clickable:hover .name-arrow {
    opacity: 1;
    transform: translateX(3px);
  }

  .name-arrow {
    position: absolute;
    right: 0;
    top: 0;
    opacity: 0;
    transition: all 0.2s ease;
    color: #3b82f6;
  }

  .patient-name strong {
    color: #1e293b;
    font-weight: 500;
    font-size: var(--table-font-size, 13px);
    line-height: 1.3;
    transition: color 0.2s ease;
    padding-right: 16px;
  }

  .patient-name .title {
    font-size: calc(var(--table-font-size, 13px) * 0.85);
    color: #94a3b8;
    font-weight: normal;
  }

  .gender-badge {
    display: inline-block;
    padding: calc(var(--table-font-size, 13px) * 0.23) calc(var(--table-font-size, 13px) * 0.62);
    border-radius: 12px;
    font-size: calc(var(--table-font-size, 13px) * 0.85);
    font-weight: 500;
  }

  .gender-badge.male {
    background: #dbeafe;
    color: #1e40af;
  }

  .gender-badge.female {
    background: #fce7f3;
    color: #9d174d;
  }

  .phone-number {
    font-family: 'SF Mono', Monaco, 'Cascadia Mono', monospace;
    font-size: calc(var(--table-font-size, 13px) * 0.92);
    color: #475569;
  }

  .email {
    color: #3b82f6;
    font-size: calc(var(--table-font-size, 13px) * 0.92);
    overflow: hidden;
    text-overflow: ellipsis;
    max-width: 180px;
    display: inline-block;
  }

  .age-cell, .city-cell, .doctor-cell, .occupation-cell, .marital-cell {
    font-size: calc(var(--table-font-size, 13px) * 0.92);
  }

  .blood-group {
    font-weight: 500;
    color: #dc2626;
    font-size: calc(var(--table-font-size, 13px) * 0.92);
  }

  /* SCROLL INDICATOR */
  .scroll-indicator {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    padding: calc(var(--table-font-size, 13px) * 0.77);
    background: #f8fafc;
    border-top: 1px solid #e2e8f0;
    color: #64748b;
    font-size: calc(var(--table-font-size, 13px) * 0.92);
  }

  .scroll-indicator svg {
    color: #94a3b8;
    animation: bounce 2s infinite;
  }

  /* STATES */
  .loading-state, .error-state, .empty-state {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 60px 40px;
    text-align: center;
  }

  .loader {
    width: 32px;
    height: 32px;
    border: 3px solid #e2e8f0;
    border-top-color: #3b82f6;
    border-radius: 50%;
    animation: spin 1s linear infinite;
    margin-bottom: 16px;
  }

  .error-state svg, .empty-state svg {
    color: #94a3b8;
    margin-bottom: 16px;
  }

  .error-state h3, .empty-state h3 {
    color: #1e293b;
    margin-bottom: 8px;
    font-size: 16px;
  }

  .error-state p, .empty-state p {
    color: #64748b;
    margin-bottom: 16px;
    font-size: 13px;
  }

  /* ANIMATIONS */
  @keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
  }

  @keyframes bounce {
    0%, 20%, 50%, 80%, 100% {transform: translateY(0);}
    40% {transform: translateY(-5px);}
    60% {transform: translateY(-3px);}
  }

  /* RESPONSIVE */
  @media (max-width: 1024px) {
    .container {
      padding: 20px;
    }
    
    .header {
      flex-direction: column;
      gap: 16px;
    }
    
    .header-actions {
      width: 100%;
      flex-wrap: wrap;
      justify-content: space-between;
    }
    
    .font-size-tool {
      order: 1;
      width: auto;
      flex: 1;
      justify-content: center;
      margin-bottom: 0;
    }
    
    .three-line-wrapper {
      order: 2;
      width: auto;
    }
    
    .options-menu, .export-menu {
      left: 50%;
      right: auto;
      transform: translateX(-50%);
    }
    
    .patient-modal-content.compact-design {
      width: 95%;
      max-height: 90vh;
    }
    
    .compact-header {
      height: 90px;
      padding: 0 16px;
    }
    
    .compact-avatar {
      width: 56px;
      height: 56px;
    }
    
    .avatar-initials {
      width: 48px;
      height: 48px;
      font-size: 16px;
    }
    
    .compact-name {
      font-size: 16px;
    }
    
    .compact-body {
      padding: 16px;
    }
    
    .info-section {
      padding: 16px;
    }
    
    .info-grid {
      grid-template-columns: 1fr;
    }
  }

  @media (max-width: 768px) {
    .modal-content {
      width: 95%;
      margin: 20px;
    }
    
    .header-actions {
      flex-direction: row;
      align-items: center;
    }
    
    .table-wrapper {
      max-height: 400px;
    }
    
    .font-size-tool {
      justify-content: space-between;
    }
    
    .options-menu {
      width: 200px;
    }
    
    .export-menu {
      width: 280px;
    }
    
    .compact-header {
      flex-direction: column;
      height: auto;
      padding: 16px;
    }
    
    .compact-header-content {
      flex-direction: column;
      text-align: center;
      gap: 12px;
    }
    
    .compact-details {
      justify-content: center;
    }
    
    .compact-close {
      position: absolute;
      top: 12px;
      right: 12px;
    }
    
    .compact-footer {
      flex-direction: column;
      gap: 12px;
      align-items: stretch;
      padding: 16px;
    }
    
    .btn {
      width: 100%;
      justify-content: center;
    }
  }

  @media (max-width: 480px) {
    .header-actions {
      flex-direction: column;
      align-items: stretch;
    }
    
    .font-size-tool, .three-line-wrapper {
      width: 100%;
      justify-content: center;
    }
    
    .three-line-button {
      width: 100%;
      justify-content: center;
    }
    
    .patient-modal-content.compact-design {
      width: 100%;
      margin: 0;
      border-radius: 0;
      max-height: 100vh;
    }
    
    .compact-header {
      padding: 12px;
    }
    
    .compact-avatar {
      width: 48px;
      height: 48px;
    }
    
    .avatar-initials {
      width: 40px;
      height: 40px;
      font-size: 14px;
    }
    
    .compact-name {
      font-size: 14px;
    }
    
    .compact-details {
      flex-direction: column;
      align-items: center;
      gap: 6px;
    }
    
    .compact-id, .compact-gender, .compact-age {
      font-size: 12px;
    }
  }
</style>
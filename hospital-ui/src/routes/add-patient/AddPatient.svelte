<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  
  // Reactive state
  let patientName = '';
  let phoneNumber = '';
  let gender: 'M' | 'F' | 'Other' = 'M';
  let age = '';
  let ageUnit = 'years';
  let dob = '';
  let city = '';
  let language = 'English';
  let pinCode = '';
  let address = '';
  let title = 'Mr';
  
  // Additional details
  let maritalStatus = 'Marital Status';
  let anniversaryDate = '';
  let bloodGroup = 'Blood Group';
  let spouseName = '';
  let spouseBloodGroup = 'Spouse Blood Group';
  let referringDoctor = '';
  let speciality = 'Speciality';
  let patientId = '';
  let email = '';
  let channel = '';
  let c_o = '';
  let occupation = '';
  let tag = '';
  let alternateMobile = '';
  let aadharNumber = '';
  
  // UI state
  let showExtraSection = false;
  let cameraActive = false;
  let showVideo = false;
  let showPreview = false;
  let previewImage = '';
  let notification = {
    show: false,
    message: '',
    type: 'success'
  };
  let formStatus = 1; // 1: Basic, 2: Additional, 3: Review
  
  // Camera variables
  let stream: MediaStream | null = null;
  let videoElement: HTMLVideoElement;
  let previewElement: HTMLImageElement;
  let fileInput: HTMLInputElement;
  
  // Titles for dropdown
  const titles = ['Mr', 'Mrs', 'Ms', 'Dr', 'Baby', 'Master'];
  const languages = ['English', 'Hindi', 'Gujarati', 'Marathi', 'Other'];
  const maritalStatuses = ['Marital Status', 'Single', 'Married', 'Divorced', 'Widowed'];
  const bloodGroups = [
    'Blood Group', 'A +ve', 'B +ve', 'O +ve', 'AB +ve',
    'A -ve', 'B -ve', 'O -ve', 'AB -ve'
  ];
  const specialities = ['Speciality', 'Cardiologist', 'Dermatologist', 'Gynecologist', 'Pediatrician', 'Orthopedic'];
  
  // Gender options
  const genderOptions = [
    { value: 'M', icon: 'mars', text: 'Male', color: '#3b82f6' },
    { value: 'F', icon: 'venus', text: 'Female', color: '#ec4899' },
    { value: 'Other', icon: 'transgender-alt', text: 'Other', color: '#8b5cf6' }
  ];
  
  // Age units
  const ageUnits = ['years', 'months', 'weeks', 'days'];
  
  // Handle gender selection
  function selectGender(value: 'M' | 'F' | 'Other') {
    gender = value;
  }
  
  // Toggle extra section
  function toggleExtraSection() {
    showExtraSection = !showExtraSection;
    if (showExtraSection) {
      formStatus = 2;
    }
  }
  
  // Handle age/DOB sync
  function handleDobChange() {
    if (dob) {
      const birthDate = new Date(dob);
      const today = new Date();
      let calculatedAge = today.getFullYear() - birthDate.getFullYear();
      const monthDiff = today.getMonth() - birthDate.getMonth();
      
      if (monthDiff < 0 || (monthDiff === 0 && today.getDate() < birthDate.getDate())) {
        calculatedAge--;
      }
      
      if (calculatedAge > 0) {
        age = calculatedAge.toString();
        ageUnit = 'years';
      }
    }
  }
  
  function handleAgeChange() {
    if (age && ageUnit === 'years') {
      const today = new Date();
      const birthYear = today.getFullYear() - parseInt(age);
      const calculatedDob = new Date(birthYear, today.getMonth(), today.getDate());
      dob = calculatedDob.toISOString().split('T')[0];
    }
  }
  
  // Camera functions
  async function startCamera() {
    try {
      if (stream) {
        stopCamera();
      }
      
      stream = await navigator.mediaDevices.getUserMedia({
        video: {
          facingMode: "user",
          width: { ideal: 1280 },
          height: { ideal: 720 }
        }
      });
      
      cameraActive = true;
      showVideo = true;
      showPreview = false;
      
      if (videoElement) {
        videoElement.srcObject = stream;
      }
      
      showNotification("Camera started", "success");
    } catch (error) {
      console.error("Error accessing camera:", error);
      showNotification("Unable to access camera", "error");
    }
  }
  
  function capturePhoto() {
    if (!stream || !videoElement) {
      showNotification("Please start camera first", "warning");
      return;
    }
    
    const canvas = document.createElement("canvas");
    canvas.width = videoElement.videoWidth;
    canvas.height = videoElement.videoHeight;
    const ctx = canvas.getContext("2d");
    
    if (ctx) {
      ctx.drawImage(videoElement, 0, 0);
      previewImage = canvas.toDataURL("image/jpeg", 0.9);
      showPreview = true;
      showVideo = false;
      
      stopCamera();
      showNotification("Photo captured", "success");
    }
  }
  
  function stopCamera() {
    if (stream) {
      stream.getTracks().forEach(track => track.stop());
      stream = null;
    }
    cameraActive = false;
  }
  
  function handleFileUpload(event: Event) {
    const target = event.target as HTMLInputElement;
    if (target.files && target.files[0]) {
      const reader = new FileReader();
      reader.onload = function(e) {
        previewImage = e.target?.result as string;
        showPreview = true;
        showVideo = false;
        stopCamera();
        showNotification("Photo uploaded", "success");
      };
      reader.readAsDataURL(target.files[0]);
    }
  }
  
  async function handleAction(type: 'Rx' | 'Bill' | 'Appointment') {

  // 1️⃣ Validate required fields
  if (!patientName || !phoneNumber || (!age && !dob)) {
    showNotification("Please fill all required fields", "error");
    return;
  }

  // 2️⃣ Prepare payload
const payload = {
  patient: {
    title,
    patientName,
    phoneNumber,
    gender,
    age: age ? Number(age) : null,
    dob,
    city,
    language,
    pinCode,
    address,
    email
  },

  additionalDetails: {
  maritalStatus: maritalStatus !== 'Marital Status' ? maritalStatus : null,
  anniversaryDate: anniversaryDate ? anniversaryDate : null,
  bloodGroup: bloodGroup !== 'Blood Group' ? bloodGroup : null,
  spouseName: spouseName || null,
  spouseBloodGroup:
    spouseBloodGroup !== 'Spouse Blood Group' ? spouseBloodGroup : null,
  referringDoctor: referringDoctor || null,
  speciality: speciality !== 'Speciality' ? speciality : null,
  channel: channel || null,
  co: c_o || null,
  occupation: occupation || null,
  tag: tag || null,
  alternateMobile: alternateMobile || null,
  aadharNumber: aadharNumber || null
}

};


  try {
    const res = await fetch('http://localhost:8081/api/patients', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload)
    });

    if (!res.ok) {
      throw new Error('Failed to save patient');
    }

    formStatus = 3;
    showNotification(`Patient saved successfully`, "success");

    setTimeout(() => {
      resetForm();
      showNotification(`${type} created for new patient`, "info");
    }, 1500);

  } catch (error) {
    console.error(error);
    showNotification("Backend error while saving patient", "error");
  }
}

  
  function closeForm() {
    if (confirm('Are you sure you want to close? Any unsaved changes will be lost.')) {
      resetForm();
      showNotification('Form closed', 'info');
    }
  }
  
  function resetForm() {
    patientName = '';
    phoneNumber = '';
    gender = 'M';
    age = '';
    ageUnit = 'years';
    dob = new Date().toISOString().split('T')[0];
    city = '';
    language = 'English';
    pinCode = '';
    address = '';
    title = 'Mr';
    
    maritalStatus = 'Marital Status';
    anniversaryDate = '';
    bloodGroup = 'Blood Group';
    spouseName = '';
    spouseBloodGroup = 'Spouse Blood Group';
    referringDoctor = '';
    speciality = 'Speciality';
    patientId = '';
    email = '';
    channel = '';
    c_o = '';
    occupation = '';
    tag = '';
    alternateMobile = '';
    aadharNumber = '';
    
    showExtraSection = false;
    showVideo = false;
    showPreview = false;
    previewImage = '';
    cameraActive = false;
    formStatus = 1;
    
    stopCamera();
  }
  
  function showNotification(message: string, type: 'success' | 'error' | 'info' | 'warning') {
    notification = { show: true, message, type };
    setTimeout(() => { notification.show = false; }, 3000);
  }
  
  onMount(() => {
    dob = new Date().toISOString().split('T')[0];
  });
  
  onDestroy(() => {
    stopCamera();
  });
</script>

<div class="modal">
  <!-- ================= HEADER ================= -->
  <div class="modal-header">
    <div class="header-left">
      <div class="header-icon"><i class="fas fa-user-plus"></i></div>
      <div>
        <h2>Add New Patient</h2>
        <div class="header-subtitle">Patient Registration Form</div>
      </div>
    </div>
    <div class="header-actions">
      <div class="header-badge"><i class="fas fa-circle"></i> New Registration</div>
      <button class="close-btn" title="Close" on:click={closeForm}><i class="fas fa-times"></i></button>
    </div>
  </div>

  <!-- ================= FORM ================= -->
  <div class="modal-content">
    <div class="form-status">
      <div class="status-item {formStatus >= 1 ? 'active' : ''}"><span class="status-circle"></span>Basic Information</div>
      <div class="status-item {formStatus >= 2 ? 'active' : ''}"><span class="status-circle"></span>Additional Details</div>
      <div class="status-item {formStatus >= 3 ? 'active' : ''}"><span class="status-circle"></span>Review & Submit</div>
    </div>

    <form id="patientForm">
      <div class="form-grid">
        <!-- Patient Name -->
        <div class="form-group">
          <label class="required">Patient Name</label>
          <div class="name-group">
            <select bind:value={title}>
              {#each titles as t}<option>{t}</option>{/each}
            </select>
            <input placeholder="Enter Name" required bind:value={patientName}>
          </div>
        </div>

        <!-- Phone Number -->
        <div class="form-group">
          <label class="required">Phone Number</label>
          <input type="tel" placeholder="Enter Number" required bind:value={phoneNumber}>
        </div>

        <!-- Gender -->
        <div class="form-group">
           <div> 
            <span class="required">Gender</span>
            <i class="fas fa-info-circle info-icon" title="Select patient's gender"></i>
          </div>
          <div class="gender-container">
            {#each genderOptions as option}
              <button 
                type="button" 
                class="gender-option {gender === option.value ? 'active' : ''}" 
                on:click={() => selectGender(option.value)}
              >
                <i class="fas fa-{option.icon} gender-icon" style="color: {option.color};"></i>
                <span class="gender-text">{option.text}</span>
              </button>
            {/each}
          </div>
          <input type="hidden" id="gender" name="gender" value={gender} required>
        </div>

        <!-- Age/DOB -->
        <div class="form-group">
          <div class="age-label">
            <span class="required">Age or DOB</span>
            <i class="fas fa-info-circle info-icon" title="Enter age or select date of birth"></i>
          </div>
          <div class="age-dob-container">
            <div class="age-input-group">
              <input 
                type="number" 
                class="age-number" 
                placeholder="Age" 
                min="0" max="120" 
                bind:value={age}
                on:input={handleAgeChange}
              >
              <select class="age-unit-select" bind:value={ageUnit} on:change={() => { age = ''; dob = ''; }}>
                {#each ageUnits as unit}
                  <option value={unit}>{unit.charAt(0).toUpperCase() + unit.slice(1)}</option>
                {/each}
              </select>
            </div>
            <div style="color: #9ca3af; font-size: 13px; font-weight: 500;">OR</div>
            <input 
              type="date" 
              class="dob-input" 
              bind:value={dob}
              on:change={handleDobChange}
            >
          </div>
        </div>

        <!-- City -->
        <div class="form-group">
          <label>City</label>
          <input placeholder="Enter City" bind:value={city}>
        </div>

        <!-- Language & Pin Code side by side (with proportional widths) -->
        <div class="form-group language-pin-group">
          <div class="language-field">
            <label>Preferred Language</label>
            <select bind:value={language}>
              {#each languages as lang}<option>{lang}</option>{/each}
            </select>
          </div>
          <div class="pin-field">
            <label>Pin Code</label>
            <input type="text" placeholder="Enter Pin" bind:value={pinCode}>
          </div>
        </div>

        <!-- Address (full width) -->
        <div class="form-group" style="grid-column: span 2;">
          <label>Address</label>
          <textarea rows="2" placeholder="Enter Address" bind:value={address}></textarea>
        </div>
      </div>

      <!-- Expand link -->
      <div class="more">
        <a on:click={toggleExtraSection}>
          <i class="fas fa-chevron-{showExtraSection ? 'up' : 'down'}"></i>
          {showExtraSection ? 'Hide additional details' : 'Add more details (Optional)'}
        </a>
      </div>

      <!-- Expanded section (unchanged) -->
      {#if showExtraSection}
        <div class="extra" id="extraSection">
          <h3><i class="fas fa-info-circle"></i> Additional Information</h3>
          <div class="grid">
            <select id="marital" bind:value={maritalStatus}>
              {#each maritalStatuses as status}<option>{status}</option>{/each}
            </select>
            <input type="date" placeholder="Date of Anniversary" bind:value={anniversaryDate}>
            <select bind:value={bloodGroup}>
              {#each bloodGroups as bg}<option>{bg}</option>{/each}
            </select>
            <input 
              id="spouseName" 
              class="{maritalStatus !== 'Married' ? 'hidden' : ''}" 
              placeholder="Spouse Name" 
              bind:value={spouseName}
            >
            <select 
              id="spouseBlood" 
              class="{maritalStatus !== 'Married' ? 'hidden' : ''}" 
              bind:value={spouseBloodGroup}
            >
              <option>Spouse Blood Group</option>
              <option>A +ve</option><option>B +ve</option><option>O +ve</option><option>AB +ve</option>
            </select>
            <input placeholder="Referring Doctor" bind:value={referringDoctor}>
            <select bind:value={speciality}>
              {#each specialities as spec}<option>{spec}</option>{/each}
            </select>
            <input placeholder="Patient ID" bind:value={patientId}>
            <input type="email" placeholder="Email Address" bind:value={email}>
            <input placeholder="Channel" bind:value={channel}>
            <input placeholder="C/O" bind:value={c_o}>
            <input placeholder="Occupation" bind:value={occupation}>
            <input placeholder="Tag" bind:value={tag}>
            <input type="tel" placeholder="Alternate Mobile" bind:value={alternateMobile}>
            <input type="text" placeholder="Aadhar Number" bind:value={aadharNumber}>
          </div>

          <!-- Camera -->
          <div class="camera-section">
            <h4 style="margin-bottom: 12px; color: #4b5563; display: flex; align-items: center; gap: 8px; font-size: 14px;">
              <i class="fas fa-camera"></i> Patient Photo (Optional)
            </h4>
            <div class="camera-box">
              <video 
                id="video" 
                class="{showVideo ? '' : 'hidden'}" 
                autoplay 
                bind:this={videoElement}
              ></video>
              <img 
                id="preview" 
                class="{showPreview ? '' : 'hidden'}" 
                alt="Patient Photo Preview" 
                src={previewImage}
                bind:this={previewElement}
              >
              <div style="color: #6b7280; margin-bottom: 12px; font-size: 13px;">
                <i class="fas fa-info-circle"></i> Take a photo or upload from device
              </div>
              <div class="camera-controls">
                <button class="btn" on:click={startCamera}><i class="fas fa-camera"></i> Open Camera</button>
                <button class="btn" on:click={capturePhoto}><i class="fas fa-circle"></i> Capture</button>
                <button class="btn" on:click={() => fileInput?.click()}><i class="fas fa-upload"></i> Upload</button>
                <input type="file" id="upload" accept="image/*" class="hidden" bind:this={fileInput} on:change={handleFileUpload}>
              </div>
            </div>
          </div>
        </div>
      {/if}

      <!-- Actions -->
      <div class="actions">
        <button type="button" class="primary-btn" on:click={() => handleAction('Rx')}>Add & Create Rx</button>
        <div class="secondary">
          <button type="button" on:click={() => handleAction('Bill')}>Add & Create Bill</button>
          <button type="button" on:click={() => handleAction('Appointment')}>Add & Create Appointment</button>
          <button type="button" class="view-btn" on:click={() => window.location.href = '/patient'}>View Patients</button>
        </div>
      </div>
    </form>
  </div>
</div>

{#if notification.show}
  <div class="notification notification-{notification.type}">
    <i class="fas fa-{notification.type === 'success' ? 'check-circle' : notification.type === 'error' ? 'exclamation-circle' : 'info-circle'}"></i>
    <span>{notification.message}</span>
  </div>
{/if}

<style>
  /* ===== GLOBAL & LAYOUT ===== */
  * {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
    font-family: 'Poppins', sans-serif;
  }

  body {
    background: linear-gradient(135deg, #f8fafc 0%, #e2e8f0 100%);
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 20px;
  }

  /* ===== MODAL ===== */
  .modal {
    background: #fff;
    width: 920px;
    max-width: 100%;
    margin: auto;
    border-radius: 12px;
    box-shadow: 0 10px 40px rgba(0, 0, 0, 0.08);
    overflow: hidden;
    border: 1px solid #e2e8f0;
  }

  /* ===== HEADER ===== */
  .modal-header {
    background: #fff;
    padding: 18px 28px;
    border-bottom: 1px solid #e5e7eb;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .header-left {
    display: flex;
    align-items: center;
    gap: 12px;
  }

  .header-icon {
    width: 36px;
    height: 36px;
    background: linear-gradient(135deg, #3b82f6 0%, #2563eb 100%);
    border-radius: 8px;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    font-size: 16px;
  }

  .modal-header h2 {
    margin: 0;
    font-size: 18px;
    font-weight: 600;
    color: #1f2937;
    letter-spacing: -0.3px;
  }

  .header-subtitle {
    color: #6b7280;
    font-size: 12px;
    font-weight: 400;
    margin-top: 2px;
  }

  .header-actions {
    display: flex;
    gap: 10px;
    align-items: center;
  }

  .header-badge {
    background: #f3f4f6;
    padding: 6px 12px;
    border-radius: 6px;
    font-size: 12px;
    font-weight: 500;
    color: #4b5563;
    display: flex;
    align-items: center;
    gap: 6px;
  }

  .header-badge i {
    color: #3b82f6;
    font-size: 11px;
  }

  .close-btn {
    background: none;
    border: none;
    color: #9ca3af;
    cursor: pointer;
    font-size: 14px;
    padding: 6px;
    border-radius: 6px;
    transition: all 0.2s;
  }

  .close-btn:hover {
    background: #f3f4f6;
    color: #6b7280;
  }

  /* ===== MODAL CONTENT ===== */
  .modal-content {
    padding: 24px 28px 28px;
  }

  /* ===== FORM GRID ===== */
  .form-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 20px;
    margin-top: 8px;
  }

  .form-group {
    margin-bottom: 0;
  }

  label {
    font-size: 13px;
    font-weight: 500;
    margin-bottom: 6px;
    display: block;
    color: #374151;
    letter-spacing: 0.2px;
  }

  .required:after {
    content: " *";
    color: #ef4444;
  }

  input, select, textarea {
    width: 100%;
    padding: 10px 14px;
    border: 1.5px solid #e5e7eb;
    border-radius: 8px;
    font-size: 14px;
    transition: all 0.2s ease;
    background: white;
    color: #1f2937;
  }

  input:focus, select:focus, textarea:focus {
    outline: none;
    border-color: #3b82f6;
    box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
  }

  select {
    appearance: none;
    background-image: url("data:image/svg+xml;charset=UTF-8,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='%236b7280' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3e%3cpolyline points='6 9 12 15 18 9'%3e%3c/polyline%3e%3c/svg%3e");
    background-repeat: no-repeat;
    background-position: right 12px center;
    background-size: 14px;
    padding-right: 36px;
  }

  textarea {
    resize: vertical;
    min-height: 76px;
  }

  /* ===== NAME GROUP ===== */
  .name-group {
    display: flex;
    gap: 10px;
  }

  .name-group select {
    flex: 0 0 90px;
    font-size: 13px;
  }

  .name-group input {
    flex: 1;
  }

  /* ===== GENDER GROUP ===== */
  .gender-label {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 6px;
  }

  .gender-container {
    display: flex;
    border: 1.5px solid #e5e7eb;
    border-radius: 8px;
    overflow: hidden;
    background: white;
  }

  .gender-option {
    flex: 1;
    padding: 12px 10px;
    border: none;
    background: white;
    cursor: pointer;
    transition: all 0.2s ease;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 5px;
  }

  .gender-option:not(:last-child) {
    border-right: 1.5px solid #e5e7eb;
  }

  .gender-option:hover {
    background: #f8fafc;
  }

  .gender-option.active {
    background: #eff6ff;
    color: #2563eb;
  }

  .gender-icon {
    font-size: 18px;
    transition: all 0.2s ease;
  }

  .gender-text {
    font-size: 13px;
    font-weight:normal;
  }

  /* ===== AGE/DOB GROUP ===== */
  .age-label {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 6px;
  }

  .age-dob-container {
    display: flex;
    gap: 8px;
    align-items: center;
  }

  .age-input-group {
    display: flex;
    flex: 2;
    border: 1.5px solid #e5e7eb;
    border-radius: 8px;
    overflow: hidden;
  }

  .age-number {
    flex: 1;
    border: none;
    padding: 10px 14px;
    font-size: 14px;
    min-width: 60px;
  }

  .age-number:focus {
    outline: none;
  }

  .age-unit-select {
    flex: 1;
    border: none;
    border-left: 1.5px solid #e5e7eb;
    padding: 10px 14px;
    font-size: 14px;
    min-width: 100px;
    background: white;
    cursor: pointer;
    appearance: none;
    background-image: url("data:image/svg+xml;charset=UTF-8,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='%236b7280' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3e%3cpolyline points='6 9 12 15 18 9'%3e%3c/polyline%3e%3c/svg%3e");
    background-repeat: no-repeat;
    background-position: right 10px center;
    background-size: 12px;
    padding-right: 30px;
  }

  .dob-input {
    flex: 1;
    padding: 10px 14px;
    border: 1.5px solid #e5e7eb;
    border-radius: 8px;
    font-size: 14px;
    min-width: 140px;
  }

  /* ===== LANGUAGE & PIN CODE (side by side with proportional widths) ===== */
  .language-pin-group {
    display: flex;
    gap: 20px;
    align-items: flex-end; /* Aligns inputs to bottom */
  }

  .language-field {
    flex: 1.5;  /* Wider, like age-input-group */
  }

  .pin-field {
    flex: 0.9;  /* Narrower, like dob-input */
  }

  .language-field select,
  .pin-field input {
    width: 100%;
  }

  /* ===== MORE DETAILS LINK ===== */
  .more {
    text-align: center;
    margin-top: 28px;
    padding-top: 20px;
    border-top: 1px solid #f0f0f0;
  }

  .more a {
    color: #3b82f6;
    text-decoration: none;
    font-weight: 500;
    cursor: pointer;
    display: inline-flex;
    align-items: center;
    gap: 6px;
    padding: 8px 16px;
    border-radius: 6px;
    transition: all 0.2s ease;
    background: #f8fafc;
    font-size: 13px;
  }

  .more a:hover {
    background: #eff6ff;
  }

  /* ===== EXPANDED SECTION ===== */
  .extra {
    margin-top: 24px;
    padding: 20px;
    border-radius: 8px;
    background: #f9fafb;
    border: 1px solid #e5e7eb;
    animation: fadeIn 0.3s ease;
  }

  @keyframes fadeIn {
    from { opacity: 0; transform: translateY(8px); }
    to { opacity: 1; transform: translateY(0); }
  }

  .extra h3 {
    color: #4b5563;
    margin-bottom: 16px;
    font-size: 15px;
    display: flex;
    align-items: center;
    gap: 8px;
    font-weight: 600;
  }

  .grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 16px;
  }

  .hidden {
    display: none !important;
  }

  /* ===== CAMERA ===== */
  .camera-section {
    margin-top: 24px;
    padding-top: 20px;
    border-top: 1px solid #e5e7eb;
  }

  .camera-box {
    border: 1.5px dashed #d1d5db;
    padding: 20px;
    border-radius: 8px;
    max-width: 500px;
    text-align: center;
    background: white;
    margin: 0 auto;
  }

  .camera-box video,
  .camera-box img {
    width: 100%;
    max-height: 280px;
    border-radius: 6px;
    margin-bottom: 12px;
    object-fit: cover;
    background: #f8fafc;
    border: 1px solid #e5e7eb;
  }

  .camera-controls {
    display: flex;
    justify-content: center;
    gap: 10px;
    flex-wrap: wrap;
    margin-top: 12px;
  }

  .btn {
    background: #f1f5f9;
    border: 1.5px solid #cbd5e1;
    color: #475569;
    padding: 8px 16px;
    border-radius: 6px;
    cursor: pointer;
    font-weight: 500;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: 6px;
    transition: all 0.2s ease;
    min-width: 110px;
    font-size: 13px;
  }

  .btn:hover {
    background: #e2e8f0;
    border-color: #94a3b8;
  }

  /* ===== ACTION BUTTONS ===== */
  .actions {
    text-align: center;
    margin-top: 32px;
    padding-top: 24px;
    border-top: 1px solid #e5e7eb;
  }

  .primary-btn {
    background: linear-gradient(135deg, #3b82f6 0%, #2563eb 100%);
    color: white;
    border: none;
    padding: 12px 28px;
    border-radius: 8px;
    cursor: pointer;
    font-size: 14px;
    font-weight: 600;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: 8px;
    transition: all 0.2s ease;
    box-shadow: 0 4px 12px rgba(37, 99, 235, 0.15);
    min-width: 200px;
  }

  .primary-btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 16px rgba(37, 99, 235, 0.2);
  }

  .secondary {
    margin-top: 16px;
    display: flex;
    justify-content: center;
    gap: 12px;
    flex-wrap: wrap;
  }

  .secondary button {
    background: white;
    border: 1.5px solid #3b82f6;
    color: #3b82f6;
    padding: 8px 20px;
    border-radius: 6px;
    cursor: pointer;
    font-weight: 500;
    transition: all 0.2s ease;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 6px;
    font-size: 13px;
  }

  .secondary button:hover {
    background: #eff6ff;
  }

  .view-btn {
    background: white;
    border: 1.5px solid #3b82f6;
    color: #3b82f6;
    padding: 8px 20px;
    border-radius: 6px;
    cursor: pointer;
    font-weight: 500;
    transition: all 0.2s ease;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 6px;
    font-size: 13px;
  }

  .view-btn:hover {
    background: #eff6ff;
  }

  /* ===== FORM STATUS ===== */
  .form-status {
    display: flex;
    gap: 8px;
    margin-bottom: 20px;
    padding-bottom: 16px;
    border-bottom: 1px solid #e5e7eb;
  }

  .status-item {
    display: flex;
    align-items: center;
    gap: 6px;
    font-size: 12px;
    color: #6b7280;
  }

  .status-item.active {
    color: #3b82f6;
    font-weight: 500;
  }

  .status-circle {
    width: 6px;
    height: 6px;
    border-radius: 50%;
    background: #d1d5db;
  }

  .status-item.active .status-circle {
    background: #3b82f6;
  }

  /* ===== NOTIFICATION ===== */
  .notification {
    position: fixed;
    top: 20px;
    right: 20px;
    padding: 12px 16px;
    border-radius: 8px;
    color: white;
    font-weight: 500;
    display: flex;
    align-items: center;
    gap: 8px;
    z-index: 1000;
    box-shadow: 0 4px 12px rgba(0,0,0,0.1);
    animation: slideIn 0.3s ease;
    font-size: 14px;
  }

  .notification-success { background: #10b981; }
  .notification-error { background: #ef4444; }
  .notification-info { background: #3b82f6; }
  .notification-warning { background: #f59e0b; }

  @keyframes slideIn {
    from { transform: translateX(100%); opacity: 0; }
    to { transform: translateX(0); opacity: 1; }
  }

  /* ===== INFO ICON ===== */
  .info-icon {
    color: #9ca3af;
    font-size: 13px;
    cursor: help;
  }

  /* ===== RESPONSIVE ===== */
  @media (max-width: 768px) {
    .modal { width: 95%; }
    .form-grid { grid-template-columns: 1fr; gap: 16px; }
    .language-pin-group { flex-direction: column; gap: 16px; }
    .age-dob-container { flex-direction: column; }
    .age-input-group { width: 100%; }
    .dob-input { width: 100%; }
    .grid { grid-template-columns: 1fr; }
    .camera-controls { flex-direction: column; align-items: center; }
    .btn { width: 100%; max-width: 240px; }
    .secondary { flex-direction: column; align-items: center; }
    .secondary button, .view-btn { width: 100%; max-width: 240px; }
  }
</style>
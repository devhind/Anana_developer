<script>
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';

    function goToPatientPage() {
    goto('/patient');
}


    // Reactive state
    let formData = {
        labelName: '',
        dataType: '',
        helpText: '',
        externalField: '',
        externalService: '',
        attachmentImage: false,
        attachmentPdf: false,
        attachmentDocument: false,
        attachmentAll: false,
        lookupModule: '',
        autoPrefix: '',
        autoStart: '',
        autoSuffix: '',
        autoApplyToExisting: false,
        colorPlacement: 'inline',
        pii: false,
        ephi: false,
        preventDuplicate: 'no',
        inputFormat: '',
        defaultValue: '',
        inheritFrom: '',
        mandatory: 'no',
        showInPdf: 'no'
    };

    // UI state variables
    let previewLabel = 'Label';
    let externalLookupSection = false;
    let attachmentSection = false;
    let lookupSection = false;
    let autoNumberSection = false;
    let dropdownSection = false;
    let dataPrivacySection = true;
    let preventDuplicateSection = true;
    let inputFormatSection = true;
    let defaultContainer = true;
    let previewContainer = true;
    let inheritSection = false;
    let inheritBtnVisible = false;
    
    // Preview elements
    let previewTextareaVisible = false;
    let previewInputVisible = true;
    let previewInputType = 'text';
    let previewInputDisabled = false;
    let previewCheckboxVisible = false;
    let previewCheckboxChecked = false;
    let previewSelectVisible = false;
    let previewSelectDisabled = false;
    let previewSelectOptions = [];
    let previewUploadVisible = false;
    let previewUploadDisabled = true;
    
    // Default elements
    let defaultInputType = 'text';
    let defaultInputVisible = true;
    let defaultCheckboxVisible = false;
    let defaultCheckboxChecked = false;
    let defaultSelectVisible = false;
    let defaultSelectOptions = [];
    let defaultUploadVisible = false;
    
    // Icons
    let defaultIcon = '';
    let previewIcon = '';
    let defaultCurrencyPrefix = '';
    let previewCurrencyPrefix = '';
    let defaultPercentSuffix = false;
    let previewPercentSuffix = false;
    
    // Dropdown
    let dropdownOptions = [];
    let optionCount = 0;
    let colorToggle = true;
    let defaultColors = ["#f44336", "#2196f3", "#ff9800", "#4caf50", "#9c27b0"];
    
    // File upload
    let uploadedFileName = '';
    let uploadedFile = null;
    let uploadedFileUrl = '';

    // External Field Modal
    let showExternalFieldModal = false;
    let selectedField = '';
    let selectedService = '';
    let searchQuery = '';
    
    // Sample data for external field modal
    let services = [
        { id: 'zoho_desk', name: 'Zoho Desk' },
        { id: 'zoho_crm', name: 'Zoho CRM' },
        { id: 'zoho_books', name: 'Zoho Books' },
        { id: 'zoho_projects', name: 'Zoho Projects' }
    ];
    
    let externalFields = {
        zoho_desk: [
            { id: 'ticket_id', name: 'Ticket ID' },
            { id: 'subject', name: 'Subject' },
            { id: 'status', name: 'Status' },
            { id: 'priority', name: 'Priority' },
            { id: 'department', name: 'Department' }
        ],
        zoho_crm: [
            { id: 'contact_id', name: 'Contact ID' },
            { id: 'account_name', name: 'Account Name' },
            { id: 'email', name: 'Email' },
            { id: 'phone', name: 'Phone' },
            { id: 'industry', name: 'Industry' }
        ],
        zoho_books: [
            { id: 'invoice_id', name: 'Invoice ID' },
            { id: 'customer_name', name: 'Customer Name' },
            { id: 'amount', name: 'Amount' },
            { id: 'status', name: 'Status' },
            { id: 'due_date', name: 'Due Date' }
        ],
        zoho_projects: [
            { id: 'project_id', name: 'Project ID' },
            { id: 'project_name', name: 'Project Name' },
            { id: 'owner', name: 'Owner' },
            { id: 'status', name: 'Status' },
            { id: 'start_date', name: 'Start Date' }
        ]
    };
    
    // Filtered fields based on search
    let filteredFields = [];

    // Drag and drop variables
    let draggedOption = null;
    let dragOverOption = null;

    // Initialize
    onMount(() => {
        updateAutoPreview();
    });

    // Functions
   function updateLabel(e) {
    const value = e.target.value;

    // 🔥 THIS LINE FIXES THE BACKEND ERROR
    formData.labelName = value;

    // Preview text
    previewLabel = value.trim() !== '' ? value : 'Label';
}


    function toggleInherit() {
        inheritSection = !inheritSection;
    }

    function enableInherit() {
        inheritBtnVisible = true;
    }

    function handleDataTypeChange(e) {
        const type = e.target.value;
        formData.dataType = type;
        
        // Reset all UI states
        resetUIStates();
        
        // Set default states
        previewContainer = true;
        externalLookupSection = false;
        attachmentSection = false;
        defaultContainer = true;
        inheritBtnVisible = false;
        inheritSection = false;
        previewTextareaVisible = false;
        previewInputVisible = true;
        lookupSection = false;
        dataPrivacySection = true;
        preventDuplicateSection = true;
        inputFormatSection = true;
        autoNumberSection = false;
        defaultInputVisible = true;
        previewInputDisabled = false;
        dropdownSection = false;
        defaultCheckboxVisible = false;
        defaultUploadVisible = false;
        previewUploadVisible = false;
        
        // Remove all icons and prefixes
        defaultIcon = '';
        previewIcon = '';
        defaultCurrencyPrefix = '';
        previewCurrencyPrefix = '';
        defaultPercentSuffix = false;
        previewPercentSuffix = false;
        
        // Reset uploaded file
        uploadedFileName = '';
        uploadedFile = null;
        uploadedFileUrl = '';
        
        // Handle each data type
        switch(type) {
            case 'email':
                enableInherit();
                defaultIcon = '✉';
                previewIcon = '✉';
                defaultInputType = 'email';
                previewInputType = 'email';
                break;
                
            case 'url':
                enableInherit();
                defaultIcon = '🌐';
                previewIcon = '🌐';
                defaultInputType = 'url';
                previewInputType = 'url';
                break;
                
            case 'phone':
                defaultIcon = '📞';
                previewIcon = '📞';
                defaultInputType = 'tel';
                previewInputType = 'tel';
                break;
                
            case 'decimal':
                dataPrivacySection = false;
                preventDuplicateSection = false;
                break;
                
            case 'amount':
                enableInherit();
                dataPrivacySection = false;
                preventDuplicateSection = false;
                defaultCurrencyPrefix = 'INR';
                previewCurrencyPrefix = 'INR';
                previewInputDisabled = true;
                break;
                
            case 'percent':
                dataPrivacySection = false;
                preventDuplicateSection = false;
                defaultPercentSuffix = true;
                previewPercentSuffix = true;
                previewInputDisabled = true;
                break;
                
            case 'data':
                enableInherit();
                preventDuplicateSection = false;
                defaultInputType = 'date';
                previewInputType = 'date';
                previewInputDisabled = true;
                break;
                
            case 'data and time':
                enableInherit();
                preventDuplicateSection = false;
                defaultInputType = 'datetime-local';
                previewInputType = 'datetime-local';
                previewInputDisabled = true;
                break;
                
            case 'check box':
                dataPrivacySection = false;
                preventDuplicateSection = false;
                inputFormatSection = false;
                defaultInputVisible = false;
                previewInputVisible = false;
                previewCheckboxVisible = true;
                defaultCheckboxVisible = true;
                break;
                
            case 'auto-generate number':
                autoNumberSection = true;
                dataPrivacySection = false;
                preventDuplicateSection = false;
                inputFormatSection = false;
                defaultInputVisible = false;
                previewInputVisible = true;
                previewInputDisabled = true;
                updateAutoPreview();
                break;
                
            case 'dropdown':
            case 'multi-select':
                dropdownSection = true;
                dataPrivacySection = false;
                preventDuplicateSection = false;
                defaultInputVisible = false;
                previewInputVisible = false;
                defaultSelectVisible = true;
                previewSelectVisible = true;
                // Initialize with default options if empty
                if(dropdownOptions.length === 0) {
                    addOption('Red');
                    addOption('Blue');
                    addOption('Green');
                    addOption('Yellow');
                }
                updateDropdownPreview();
                break;
                
            case 'lookup':
                dataPrivacySection = false;
                preventDuplicateSection = false;
                inputFormatSection = false;
                autoNumberSection = false;
                dropdownSection = false;
                defaultInputVisible = false;
                previewInputVisible = false;
                lookupSection = true;
                previewSelectVisible = true;
                previewSelectDisabled = true;
                previewSelectOptions = [{value: '', text: 'Select Record'}];
                break;
                
            case 'textarea':
                dataPrivacySection = false;
                preventDuplicateSection = false;
                inputFormatSection = true;
                defaultContainer = false;
                inheritSection = false;
                previewInputVisible = false;
                previewTextareaVisible = true;
                break;
                
            case 'attachment':
                dataPrivacySection = false;
                preventDuplicateSection = false;
                inputFormatSection = false;
                autoNumberSection = false;
                dropdownSection = false;
                attachmentSection = true;
                defaultInputVisible = false;
                previewInputVisible = false;
                defaultUploadVisible = true;
                previewUploadVisible = true;
                break;
                
            case 'external lookup':
                dataPrivacySection = false;
                preventDuplicateSection = false;
                inputFormatSection = false;
                autoNumberSection = false;
                dropdownSection = false;
                lookupSection = false;
                previewContainer = false;
                defaultInputVisible = false;
                previewInputVisible = false;
                // Show external field section
                externalLookupSection = true;
                break;
                
            case 'image':
                dataPrivacySection = false;
                preventDuplicateSection = false;
                inputFormatSection = false;
                inheritBtnVisible = false;
                inheritSection = false;
                autoNumberSection = false;
                dropdownSection = false;
                lookupSection = false;
                previewContainer = true;
                defaultInputVisible = false;
                defaultUploadVisible = true;
                previewInputVisible = false;
                previewUploadVisible = true;
                break;
        }
    }

    function resetUIStates() {
        previewTextareaVisible = false;
        previewInputVisible = false;
        previewCheckboxVisible = false;
        previewSelectVisible = false;
        previewUploadVisible = false;
        defaultInputVisible = false;
        defaultCheckboxVisible = false;
        defaultSelectVisible = false;
        defaultUploadVisible = false;
        defaultIcon = '';
        previewIcon = '';
        defaultCurrencyPrefix = '';
        previewCurrencyPrefix = '';
        defaultPercentSuffix = false;
        previewPercentSuffix = false;
    }

    function updateAutoPreview() {
        const prefix = formData.autoPrefix || '';
        const start = formData.autoStart || '';
        const suffix = formData.autoSuffix || '';
        formData.defaultValue = prefix + start + suffix;
    }

    function addOption(value = '', index = null) {
        const color = defaultColors[dropdownOptions.length % defaultColors.length];
        const newOption = {
            id: Date.now() + Math.random(),
            value: value,
            color: color,
            checked: false
        };
        
        if (index !== null && index >= 0 && index <= dropdownOptions.length) {
            dropdownOptions.splice(index, 0, newOption);
        } else {
            dropdownOptions.push(newOption);
        }
        
        optionCount = dropdownOptions.length;
        updateDropdownPreview();
    }

    function addOptionAtIndex(index) {
        addOption('', index);
    }

    function deleteOption(id) {
        dropdownOptions = dropdownOptions.filter(option => option.id !== id);
        optionCount = dropdownOptions.length;
        updateDropdownPreview();
    }

    function updateDropdownPreview() {
        const values = dropdownOptions
            .map(opt => opt.value)
            .filter(v => v.trim() !== '');
        
        defaultSelectOptions = values.map(v => ({value: v, text: v}));
        previewSelectOptions = [...defaultSelectOptions];
        
        // Add empty option for preview
        if (previewSelectOptions.length === 0 || previewSelectOptions[0].value !== '') {
            previewSelectOptions.unshift({value: '', text: 'Select'});
        }
    }

    // Drag and drop functions
    function handleDragStart(e, optionId) {
        draggedOption = optionId;
        e.dataTransfer.effectAllowed = 'move';
        e.dataTransfer.setData('text/plain', optionId);
    }

    function handleDragOver(e, optionId) {
        e.preventDefault();
        dragOverOption = optionId;
    }

    function handleDragLeave(e) {
        e.preventDefault();
        if (!e.currentTarget.contains(e.relatedTarget)) {
            dragOverOption = null;
        }
    }

    function handleDrop(e, optionId) {
        e.preventDefault();
        
        if (draggedOption && draggedOption !== optionId) {
            const fromIndex = dropdownOptions.findIndex(opt => opt.id === draggedOption);
            const toIndex = dropdownOptions.findIndex(opt => opt.id === optionId);
            
            if (fromIndex !== -1 && toIndex !== -1) {
                // Remove from old position
                const [movedOption] = dropdownOptions.splice(fromIndex, 1);
                // Insert at new position
                dropdownOptions.splice(toIndex, 0, movedOption);
                
                // Update UI
                dropdownOptions = [...dropdownOptions];
                updateDropdownPreview();
            }
        }
        
        draggedOption = null;
        dragOverOption = null;
    }

    function handleDragEnd(e) {
        draggedOption = null;
        dragOverOption = null;
    }

    function handleColorPlacementChange(value) {
        formData.colorPlacement = value;
    }

    function handleDefaultSelectChange(e) {
        formData.defaultValue = e.target.value;
    }

    function handlePreviewSelectDblClick() {
        const newOption = prompt("Add new option");
        if(!newOption || newOption.trim() === '') return;
        addOption(newOption);
        // Set as selected
        formData.defaultValue = newOption;
    }

    // File upload functions
    function handleFileUpload(e) {
        const file = e.target.files[0];
        if (file) {
            // Validate file size for image (10MB limit)
            if (formData.dataType === 'image' && file.size > 10 * 1024 * 1024) {
                alert('File size must be 10MB or less');
                e.target.value = '';
                return;
            }
            
            // Validate file types for attachment
            if (formData.dataType === 'attachment') {
                const fileType = file.type;
                const fileName = file.name.toLowerCase();
                
                // Check if All Files is selected
                if (formData.attachmentAll) {
                    // Accept all files
                } else {
                    // Check specific file types
                    let isValid = false;
                    
                    if (formData.attachmentImage && fileType.startsWith('image/')) {
                        isValid = true;
                    }
                    
                    if (formData.attachmentPdf && fileName.endsWith('.pdf')) {
                        isValid = true;
                    }
                    
                    if (formData.attachmentDocument && 
                        (fileName.endsWith('.doc') || fileName.endsWith('.docx') || 
                         fileName.endsWith('.txt') || fileName.endsWith('.rtf'))) {
                        isValid = true;
                    }
                    
                    // If no file type is selected, show error
                    if (!formData.attachmentImage && !formData.attachmentPdf && !formData.attachmentDocument && !formData.attachmentAll) {
                        alert('Please select at least one file type');
                        e.target.value = '';
                        return;
                    }
                    
                    // If specific types are selected but file doesn't match
                    if (!isValid) {
                        let allowedTypes = [];
                        if (formData.attachmentImage) allowedTypes.push('Images');
                        if (formData.attachmentPdf) allowedTypes.push('PDF');
                        if (formData.attachmentDocument) allowedTypes.push('Documents');
                        alert(`Please upload a ${allowedTypes.join(' or ')} file`);
                        e.target.value = '';
                        return;
                    }
                }
            }
            
            uploadedFileName = file.name;
            uploadedFile = file;
            
            // Create object URL for preview (for images)
            if (file.type.startsWith('image/')) {
                if (uploadedFileUrl) {
                    URL.revokeObjectURL(uploadedFileUrl);
                }
                uploadedFileUrl = URL.createObjectURL(file);
            }
            
            console.log('File uploaded:', file.name);
        }
    }

    function triggerFileUpload() {
        document.getElementById('fileUpload').click();
    }

    function removeUploadedFile() {
        uploadedFileName = '';
        uploadedFile = null;
        if (uploadedFileUrl) {
            URL.revokeObjectURL(uploadedFileUrl);
            uploadedFileUrl = '';
        }
        document.getElementById('fileUpload').value = '';
    }

    // External Field Modal Functions
    function openExternalFieldModal() {
        showExternalFieldModal = true;
        selectedService = '';
        selectedField = '';
        searchQuery = '';
        filteredFields = [];
    }

    function closeExternalFieldModal() {
        showExternalFieldModal = false;
        selectedService = '';
        selectedField = '';
        searchQuery = '';
        filteredFields = [];
    }

    function selectService(serviceId) {
        selectedService = serviceId;
        selectedField = '';
        searchQuery = '';
        filteredFields = externalFields[serviceId] || [];
    }

    function selectField(fieldId) {
        selectedField = fieldId;
        const service = services.find(s => s.id === selectedService);
        const field = filteredFields.find(f => f.id === fieldId);
        
        if (service && field) {
            formData.externalField = `${field.name} (${service.name})`;
            formData.externalService = service.id;
            closeExternalFieldModal();
        }
    }

    function handleSearch(e) {
        searchQuery = e.target.value.toLowerCase();
        if (selectedService && externalFields[selectedService]) {
            filteredFields = externalFields[selectedService].filter(field => 
                field.name.toLowerCase().includes(searchQuery)
            );
        }
    }

    function clearSelection() {
        formData.externalField = '';
        formData.externalService = '';
    }
    async function handleSave() {

    // 🔍 ADD THIS
    console.log('Sending to backend:', formData);

    // 🔐 Safety check
    if (!formData.labelName || formData.labelName.trim() === '') {
        alert('Label Name is required');
        return;
    }

    try {
        const res = await fetch('http://localhost:8081/api/custom-fields', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(formData)
        });

        if (!res.ok) throw new Error('Failed');

        alert('Custom Field Created Successfully ✅');
        handleCancel();
    } catch (e) {
        console.error(e);
        alert('Error saving custom field ❌');
    }
}
function handleCancel() {
    // 🔄 Reset form data
    formData = {
        labelName: '',
        dataType: '',
        helpText: '',
        externalField: '',
        externalService: '',
        attachmentImage: false,
        attachmentPdf: false,
        attachmentDocument: false,
        attachmentAll: false,
        lookupModule: '',
        autoPrefix: '',
        autoStart: '',
        autoSuffix: '',
        autoApplyToExisting: false,
        colorPlacement: 'inline',
        pii: false,
        ephi: false,
        preventDuplicate: 'no',
        inputFormat: '',
        defaultValue: '',
        inheritFrom: '',
        mandatory: 'no',
        showInPdf: 'no'
    };

    // 🧹 Reset preview
    previewLabel = 'Label';

    // 🧹 Reset UI sections
    externalLookupSection = false;
    attachmentSection = false;
    lookupSection = false;
    autoNumberSection = false;
    dropdownSection = false;
    inheritSection = false;
    inheritBtnVisible = false;

    // 🧹 Reset visibility & defaults
    previewTextareaVisible = false;
    previewInputVisible = true;
    previewCheckboxVisible = false;
    previewSelectVisible = false;
    previewUploadVisible = false;

    defaultInputVisible = true;
    defaultCheckboxVisible = false;
    defaultSelectVisible = false;
    defaultUploadVisible = false;

    previewInputType = 'text';
    previewInputDisabled = false;
    defaultInputType = 'text';

    // 🧹 Reset dropdowns
    dropdownOptions = [];
    optionCount = 0;

    // 🧹 Reset uploaded file
    uploadedFileName = '';
    uploadedFile = null;
    if (uploadedFileUrl) {
        URL.revokeObjectURL(uploadedFileUrl);
        uploadedFileUrl = '';
    }

    // 🧹 Reset icons & prefixes
    defaultIcon = '';
    previewIcon = '';
    defaultCurrencyPrefix = '';
    previewCurrencyPrefix = '';
    defaultPercentSuffix = false;
    previewPercentSuffix = false;

    // 🧹 Reset sections visibility
    dataPrivacySection = true;
    preventDuplicateSection = true;
    inputFormatSection = true;
    defaultContainer = true;
    previewContainer = true;

    // 🔁 Reset helper states
    resetUIStates();
}  
</script>

<!-- Hidden file input -->
<input type="file" id="fileUpload" style="display: none;" on:change={handleFileUpload}>

<!-- Main Content -->
<h1>New Custom Field - Invoice</h1>

<div class="main-container">
    <!-- Label Name -->
    <div class="Label1">
        <label class="Label-Name">Label Name<span>*</span></label>
        <input type="text" id="labelName" bind:value={formData.labelName} on:input={updateLabel} placeholder="Enter Label Name">
    </div>

    <!-- Data Type -->
    <div class="Label2">
        <label class="Label-Name">Data Type<span>*</span></label>
        <select id="DataType" bind:value={formData.dataType} on:change={handleDataTypeChange}>
            <option value="">Choose Data Type</option>
            <option value="text">Text Box(Single Line)</option>
            <option value="email">Email</option>
            <option value="url">URL</option>
            <option value="phone">Phone</option>
            <option value="number">Number</option>
            <option value="decimal">Decimal</option>
            <option value="amount">Amount</option>
            <option value="percent">Percent</option>
            <option value="data">Data</option>
            <option value="data and time">Data And Time</option>
            <option value="check box">Check Box</option>
            <option value="auto-generate number">Auto-Generate Number</option>
            <option value="dropdown">Dropdown</option>
            <option value="multi-select">Multi-Select</option>
            <option value="lookup">Lookup</option>
            <option value="textarea">Text Box (Multi-Line)</option>
            <option value="attachment">Attachment</option>
            <option value="external lookup">External Lookup</option>
            <option value="image">Image</option>
        </select>
        <div class="remaining-fields">Remaining Fields: 5</div>
    </div>

    <!-- Help Text -->
    <div class="Label3">
        <label class="Label-Name">Help Text</label>
        <input type="text" id="helpText" bind:value={formData.helpText} placeholder="Enter some text to help users understand the purpose of this custom field">
    </div>

    <!-- External Field -->
    {#if formData.dataType === 'external lookup'}
        <div class="LabelExternal">
            <label class="Label-Name">External Field<span>*</span></label>
            <div class="external-field-container">
                <input type="text" id="externalFieldInput" bind:value={formData.externalField} placeholder="Click to select Field" readonly on:click={openExternalFieldModal}>
                {#if formData.externalField}
                    <button type="button" class="clear-btn" on:click={clearSelection}>×</button>
                {/if}
            </div>
        </div>
    {/if}

    <!-- Attachment Section -->
    {#if attachmentSection}
        <div class="LabelAttachment" id="attachmentSection">
            <label class="Label-Name">File Type<span>*</span></label>
            <label><input type="checkbox" bind:checked={formData.attachmentImage}> Image</label>
            <label><input type="checkbox" bind:checked={formData.attachmentPdf}> PDF</label>
            <label><input type="checkbox" bind:checked={formData.attachmentDocument}> Document</label>
            <label><input type="checkbox" bind:checked={formData.attachmentAll}> All Files</label>
        </div>
    {/if}

    <!-- Lookup Section -->
    {#if lookupSection}
        <div class="LabelLookup" id="lookupSection">
            <label class="Label-Name">Module<span>*</span></label>
            <select id="lookupModule" bind:value={formData.lookupModule}>
                <option value="">Select Module</option>
                <option value="Estimate">Estimate</option>
                <option value="Invoice">Invoice</option>
                <option value="Projects">Projects</option>
                <option value="Sales Order">Sales Order</option>
                <option value="Purchase Order">Purchase Order</option>
                <option value="Customers">Customers</option>
                <option value="Items">Items</option>
            </select>
        </div>
    {/if}

    <!-- Auto Number Section -->
    {#if autoNumberSection}
        <div id="autoNumberSection">
            <div class="LabelAuto">
                <label class="Label-Name">Prefix</label>
                <input type="text" id="autoPrefix" bind:value={formData.autoPrefix} on:input={updateAutoPreview}>
            </div>
            <div class="LabelAuto">
                <label class="Label-Name">Starting Number<span>*</span></label>
                <input type="number" id="autoStart" bind:value={formData.autoStart} on:input={updateAutoPreview} placeholder="1">
            </div>
            <div class="LabelAuto">
                <label class="Label-Name">Suffix</label>
                <input type="text" id="autoSuffix" bind:value={formData.autoSuffix} on:input={updateAutoPreview}>
            </div>
            <div class="auto-info">
                <label>
                    <input type="checkbox" bind:checked={formData.autoApplyToExisting}>
                    Add this custom field to all the existing invoices and auto-generate the number in all of them.
                </label>
                <p>⚠ This is a one-time setup and you cannot edit this setting later.</p>
            </div>
        </div>
    {/if}

    <!-- Dropdown Section -->
    {#if dropdownSection}
        <div id="dropdownSection">
            <div style="margin-bottom:12px;">
                <label class="Label-Name">Color Placement</label>
                <label><input type="radio" name="colorPlacement" value="inline" checked={formData.colorPlacement === 'inline'} on:change={() => handleColorPlacementChange('inline')}> Next to Option</label>
                <label style="margin-left:15px;">
                    <input type="radio" name="colorPlacement" value="wrap" checked={formData.colorPlacement === 'wrap'} on:change={() => handleColorPlacementChange('wrap')}> Wrap Option
                </label>
            </div>
            <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:12px;">
                <strong>Options Count : <span id="optionCount">{optionCount}</span></strong>
                <div style="display:flex; align-items:center; gap:15px;">
                    <label style="font-size:14px;">
                        <input type="checkbox" id="colorToggle" bind:checked={colorToggle}> Add color to options
                    </label>
                    <button type="button" id="addOptionBtn" class="save" on:click={() => addOption('')}>+ Add Options</button>
                </div>
            </div>
            <div id="dropdownOptions">
                {#each dropdownOptions as option, index (option.id)}
                    <div class="dropdown-row" 
                         class:wrap={formData.colorPlacement === 'wrap'}
                         class:drag-over={dragOverOption === option.id}
                         style={formData.colorPlacement === 'wrap' && colorToggle ? `background: ${option.color}22` : ''}
                         draggable="true"
                         on:dragstart={(e) => handleDragStart(e, option.id)}
                         on:dragover={(e) => handleDragOver(e, option.id)}
                         on:dragleave={handleDragLeave}
                         on:drop={(e) => handleDrop(e, option.id)}
                         on:dragend={handleDragEnd}>
                        <span class="drag" title="Drag to reorder">⋮⋮</span>
                        <input type="checkbox" class="option-check" bind:checked={option.checked}>
                        {#if colorToggle && formData.colorPlacement === 'inline'}
                            <span class="color-dot" style="background:{option.color};"></span>
                        {/if}
                        {#if colorToggle}
                            <input type="color" class="color-select" bind:value={option.color} on:input={updateDropdownPreview}>
                        {/if}
                        <input type="text" class="dropdown-input" placeholder="Option" bind:value={option.value} on:input={updateDropdownPreview}>
                        <div class="option-actions">
                            <span class="add" title="Add option below" on:click={() => addOptionAtIndex(index + 1)}>+</span>
                            <span class="delete" title="Delete option" on:click={() => deleteOption(option.id)}>×</span>
                        </div>
                    </div>
                {:else}
                    <div class="no-options">No options added yet. Click "Add Options" to create options.</div>
                {/each}
            </div>
        </div>
    {/if}

    <!-- Data Privacy -->
    {#if dataPrivacySection}
        <div class="Label4">
            <label class="Label-Name">Data Privacy</label>
            <input type="checkbox" bind:checked={formData.pii}><label>PII</label>
            <input type="checkbox" bind:checked={formData.ephi}><label>ePHI</label>
            <div class="data-privacy-note">Data will be stored without encryption and will be visible to all users.</div>
        </div>
    {/if}

    <!-- Prevent Duplicate -->
    {#if preventDuplicateSection}
        <div class="Label5">
            <label class="Label-Name">Prevent Duplicate Values</label>
            <label><input type="radio" name="preventDuplicate" value="yes" bind:group={formData.preventDuplicate}> Yes</label>
            <label><input type="radio" name="preventDuplicate" value="no" bind:group={formData.preventDuplicate}> No</label>
        </div>
    {/if}

    <!-- Input Format -->
    {#if inputFormatSection}
        <div class="Label6">
            <label class="Label-Name">Input Format</label>
            <select name="inputFormat" bind:value={formData.inputFormat}>
                <option value="">Select format</option>
                <option value="Numbers">Numbers</option>
                <option value="AlphanumericWithoutSpaces">Alphanumeric Characters Without Spaces</option>
                <option value="AlphanumericWithSpaces">Alphanumeric Characters With Spaces</option>
                <option value="AlphanumericWithHyphenUnderscore">Alphanumeric Characters With Hyphens and Underscores</option>
                <option value="AlphabetsWithoutSpaces">Alphabets Without Spaces</option>
            </select>
        </div>
    {/if}

    <!-- Default Value -->
    {#if defaultContainer}
        <div class="Label7">
            <label class="Label-Name">Default Value</label>
            {#if defaultInputVisible}
                <div class="input-with-icon" style="position:relative; flex:1;">
                    {#if defaultCurrencyPrefix}
                        <span style="position:absolute; left:10px; top:50%; transform:translateY(-50%); color:#6c757d; font-size:13px; pointer-events:none;">{defaultCurrencyPrefix}</span>
                    {/if}
                    {#if defaultIcon}
                        <span style="position:absolute; left:{defaultCurrencyPrefix ? '45px' : '10px'}; top:50%; transform:translateY(-50%); color:#6c757d; pointer-events:none; font-size:14px;">{defaultIcon}</span>
                    {/if}
                    <input type={defaultInputType} id="defaultValue" bind:value={formData.defaultValue} style={`padding-left: ${defaultIcon ? (defaultCurrencyPrefix ? '60px' : '32px') : (defaultCurrencyPrefix ? '45px' : '10px')}; padding-right: ${defaultPercentSuffix ? '35px' : '10px'}`}>
                    {#if defaultPercentSuffix}
                        <span style="position:absolute; right:10px; top:50%; transform:translateY(-50%); color:#6c757d; font-size:13px; pointer-events:none;">%</span>
                    {/if}
                </div>
            {/if}
            {#if defaultCheckboxVisible}
                <div>
                    <input type="checkbox" id="defaultCheckbox" bind:checked={defaultCheckboxChecked} on:change={() => previewCheckboxChecked = defaultCheckboxChecked}>
                    <label style="margin-left:6px;">Ticked by default</label>
                </div>
            {/if}
            {#if defaultSelectVisible}
                <select bind:value={formData.defaultValue} on:change={handleDefaultSelectChange}>
                    <option value="">Select</option>
                    {#each defaultSelectOptions as option}
                        <option value={option.value}>{option.text}</option>
                    {/each}
                </select>
            {/if}
            {#if defaultUploadVisible}
                <div class="upload-container">
                    <div class="upload-btn" on:click={triggerFileUpload}>
                        ⬆ Upload File
                    </div>
                    {#if uploadedFileName}
                        <div class="uploaded-file">
                            <span>📄 {uploadedFileName}</span>
                            <span class="remove-file" on:click={removeUploadedFile}>×</span>
                        </div>
                    {/if}
                    {#if formData.dataType === 'image'}
                        <div class="upload-note">You can upload a file that is 10MB or lesser</div>
                    {:else if formData.dataType === 'attachment'}
                        <div class="upload-note">Select file types above before uploading</div>
                    {/if}
                </div>
            {/if}
            {#if inheritBtnVisible}
                <a href="javascript:void(0)" class="inherit-values" on:click={toggleInherit}>
                    {inheritSection ? 'Remove Inherit' : 'Inherit Values'}
                </a>
            {/if}
        </div>
    {/if}

    <!-- Inherit Section -->
    {#if inheritSection}
        <div class="Label10" id="inheritSection">
            <label class="Label-Name">Inherit Value From<span>*</span></label>
            <select name="inheritFrom" bind:value={formData.inheritFrom}>
                <option value="">Select Source</option>
                <option value="Customer">Customer</option>
                <option value="Estimate">Estimate</option>
                <option value="Sales Order">Sales Order</option>
                <option value="Delivery Challan">Delivery Challan</option>
            </select>
        </div>
    {/if}

    <!-- Mandatory -->
    <div class="Label8">
        <label class="Label-Name">Is Mandatory</label>
        <label><input type="radio" name="mandatory" value="yes" bind:group={formData.mandatory}> Yes</label>
        <label><input type="radio" name="mandatory" value="no" bind:group={formData.mandatory}> No</label>
    </div>

    <!-- PDF -->
    <div class="Label9">
        <label class="Label-Name">Show in All PDF's</label>
        <label><input type="radio" name="showInPdf" value="yes" bind:group={formData.showInPdf}> Yes</label>
        <label><input type="radio" name="showInPdf" value="no" bind:group={formData.showInPdf}> No</label>
    </div>

    <!-- Preview -->
    {#if previewContainer}
        <div class="preview">
            <h3>Preview</h3>
            <label id="previewLabel">{previewLabel}</label>
            
            {#if previewInputVisible}
                <div class="input-with-icon" style="position:relative;">
                    {#if formData.dataType === 'auto-generate number'}
                        <!-- For auto-generate number, show the combined prefix + start + suffix -->
                        <input type={previewInputType} id="previewInput" bind:value={formData.defaultValue} disabled={previewInputDisabled} style="padding-left: 10px; padding-right: 10px; background:#e9ecef; pointer-events:auto">
                    {:else}
                        {#if previewCurrencyPrefix}
                            <span style="position:absolute; left:10px; top:50%; transform:translateY(-50%); color:#6c757d; font-size:13px; pointer-events:none;">{previewCurrencyPrefix}</span>
                        {/if}
                        {#if previewIcon}
                            <span style="position:absolute; left:{previewCurrencyPrefix ? '45px' : '10px'}; top:50%; transform:translateY(-50%); color:#6c757d; pointer-events:none; font-size:14px;">{previewIcon}</span>
                        {/if}
                        <input type={previewInputType} id="previewInput" bind:value={formData.defaultValue} disabled={previewInputDisabled} style={`padding-left: ${previewIcon ? (previewCurrencyPrefix ? '60px' : '32px') : (previewCurrencyPrefix ? '45px' : '10px')}; padding-right: ${previewPercentSuffix ? '35px' : '10px'}; background:#e9ecef; pointer-events:auto`}>
                        {#if previewPercentSuffix}
                            <span style="position:absolute; right:10px; top:50%; transform:translateY(-50%); color:#6c757d; font-size:13px; pointer-events:none;">%</span>
                        {/if}
                    {/if}
                </div>
            {/if}
            
            {#if previewTextareaVisible}
                <textarea id="previewTextarea" rows="4" disabled>
You can enter a maximum of 36000 characters
                </textarea>
            {/if}
            
            {#if previewCheckboxVisible}
                <input type="checkbox" disabled bind:checked={previewCheckboxChecked}>
            {/if}
            
            {#if previewSelectVisible}
                <select disabled={previewSelectDisabled} on:dblclick={handlePreviewSelectDblClick}>
                    {#each previewSelectOptions as option}
                        <option value={option.value}>{option.text}</option>
                    {/each}
                </select>
            {/if}
            
            {#if previewUploadVisible}
                <div class="preview-upload-container">
                    {#if uploadedFileUrl && uploadedFile?.type?.startsWith('image/')}
                        <div class="image-preview">
                            <img src={uploadedFileUrl} alt="Uploaded Image" />
                        </div>
                    {:else if uploadedFileName}
                        <div class="uploaded-file-preview">
                            <span>📄 {uploadedFileName}</span>
                        </div>
                    {:else}
                        <div class="upload-btn disabled" on:click={triggerFileUpload}>
                            ⬆ Upload File
                        </div>
                    {/if}
                    {#if formData.dataType === 'image'}
                        <div class="upload-note">You can upload a file that is 10MB or lesser</div>
                    {:else if formData.dataType === 'attachment'}
                        <div class="upload-note">Select file types above before uploading</div>
                    {/if}
                </div>
            {/if}
        </div>
    {/if}

    <!-- Actions -->
    <div class="actions">
    <button
        type="button"
        class="back"
        on:click={goToPatientPage}
    >
        ← Back to Patients
    </button>

    <button
        type="button"
        id="saveBtn"
        class="save"
        on:click={handleSave}
    >
        Save
    </button>

    <button
        type="button"
        class="cancel"
        on:click={handleCancel}
    >
        Cancel
    </button>
</div>

</div>

<!-- External Field Modal -->
{#if showExternalFieldModal}
    <div class="modal-overlay" on:click={closeExternalFieldModal}>
        <div class="modal-content" on:click|stopPropagation>
            <div class="modal-header">
                <h3>Select External Field</h3>
                <button class="modal-close" on:click={closeExternalFieldModal}>×</button>
            </div>
            <div class="modal-body">
                <div class="field-source-section">
                    <h4>Field Source</h4>
                    <div class="service-options">
                        {#each services as service}
                            <div class="service-option {selectedService === service.id ? 'selected' : ''}" on:click={() => selectService(service.id)}>
                                <div class="service-radio">
                                    <input type="radio" name="externalService" value={service.id} bind:group={selectedService} style="display:none;">
                                    <div class="radio-circle"></div>
                                </div>
                                <span class="service-name">{service.name}</span>
                            </div>
                        {/each}
                    </div>
                </div>
                
                {#if selectedService}
                    <div class="field-search-section">
                        <h4>Search</h4>
                        <input type="text" placeholder="Search fields..." bind:value={searchQuery} on:input={handleSearch} class="search-input">
                        <div class="field-list">
                            {#each filteredFields as field}
                                <div class="field-option {selectedField === field.id ? 'selected' : ''}" on:click={() => selectField(field.id)}>
                                    <div class="field-radio">
                                        <input type="radio" name="externalField" value={field.id} bind:group={selectedField} style="display:none;">
                                        <div class="radio-circle"></div>
                                    </div>
                                    <span class="field-name">{field.name}</span>
                                </div>
                            {:else}
                                <div class="no-results">No fields found</div>
                            {/each}
                        </div>
                    </div>
                {/if}
            </div>
        </div>
    </div>
{/if}

<style>
    * {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
        font-family: "Segoe UI", "Inter", -apple-system, BlinkMacSystemFont, Arial, sans-serif;
    }
    
    h1 {
        margin-bottom: 16px;
        color: #0f172a;
        font-size: 20px;
        font-weight: 700;
        padding-bottom: 8px;
        padding-left: 20px;
    }
    
    .main-container {
        background: #ffffff;
        padding: 20px;
        border-radius: 8px;
        max-width: 800px;
        box-shadow: 
            0 2px 8px rgba(0, 0, 0, 0.06),
            0 1px 2px rgba(0, 0, 0, 0.03);
        border: 1px solid #e2e8f0;
        margin: 0 auto;
    }
    
    /* FORM ROWS - COMPACT SPACING */
    .main-container > div {
        margin-bottom: 16px;
    }

    /* LABELS */
    .Label-Name {
        display: block;
        font-weight: 600;
        margin-bottom: 4px;
        color: #334155;
        font-size: 12px;
    }

    .Label-Name span {
        color: #ef4444;
        margin-left: 2px;
    }

    /* INPUTS & SELECTS */
    input[type="text"],
    input[type="number"],
    input[type="date"],
    input[type="datetime-local"],
    input[type="email"],
    input[type="url"],
    input[type="tel"],
    select,
    textarea {
        width: 100%;
        padding: 8px 10px;
        border-radius: 4px;
        border: 1px solid #d1d9e6;
        font-size: 12px;
        transition: all 0.15s ease;
        color: #1e293b;
        background: #ffffff;
        height: 32px;
    }

    /* FOCUS STATES */
    input[type="text"]:focus,
    input[type="number"]:focus,
    input[type="date"]:focus,
    input[type="datetime-local"]:focus,
    input[type="email"]:focus,
    input[type="url"]:focus,
    input[type="tel"]:focus,
    select:focus,
    textarea:focus {
        outline: none;
        border-color: #3b82f6;
        box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.1);
    }

    /* CHECKBOXES & RADIOS - COMPACT */
    input[type="checkbox"],
    input[type="radio"] {
        width: 14px;
        height: 14px;
        margin-right: 6px;
        vertical-align: middle;
        accent-color: #3b82f6;
    }

    /* LABELS FOR CHECKBOXES/RADIOS */
    input[type="checkbox"] + label,
    input[type="radio"] + label {
        font-size: 12px;
        color: #475569;
        vertical-align: middle;
        margin-right: 20px;
        cursor: pointer;
    }

    /* COMPACT INLINE GROUPS */
    .Label4,
    .Label5,
    .Label8,
    .Label9 {
        display: flex;
        align-items: center;
        flex-wrap: wrap;
        gap: 20px;
        padding: 8px 0;
    }

    .Label4 label:first-of-type,
    .Label5 label:first-of-type,
    .Label8 label:first-of-type,
    .Label9 label:first-of-type {
        font-weight: 600;
        color: #334155;
        min-width: 120px;
        font-size: 12px;
    }

    /* PREVIEW BOX - COMPACT */
    .preview {
        margin-top: 20px;
        padding: 16px;
        border-radius: 6px;
        background: #f8fafc;
        border: 1px solid #e2e8f0;
        position: relative;
    }

    .preview h3 {
        margin-bottom: 8px;
        color: #475569;
        font-size: 13px;
        font-weight: 600;
    }

    #previewLabel {
        display: block;
        font-weight: 600;
        margin-bottom: 4px;
        color: #334155;
        font-size: 12px;
    }

    #previewInput {
        width: 100%;
        padding: 8px 10px;
        background: #ffffff;
        border: 1px solid #e2e8f0;
        border-radius: 4px;
        color: #64748b;
        pointer-events: none;
        height: 32px;
        font-size: 12px;
    }

    /* ACTION BUTTONS - COMPACT */
    .actions {
        margin-top: 20px;
        display: flex;
        gap: 10px;
        padding-top: 16px;
        border-top: 1px solid #e2e8f0;
        justify-content: flex-end;
    }

    button {
        padding: 8px 20px;
        border-radius: 4px;
        border: none;
        cursor: pointer;
        font-size: 12px;
        font-weight: 600;
        transition: all 0.15s ease;
        min-width: 80px;
        height: 32px;
    }

    .save {
        background: #3b82f6;
        color: #ffffff;
    }

    .save:hover {
        background: #2563eb;
    }

    .cancel {
        background: #ffffff;
        color: #64748b;
        border: 1px solid #d1d9e6;
    }

    .cancel:hover {
        background: #f8fafc;
        color: #475569;
        border-color: #cbd5e1;
    }

    /* INHERIT VALUES LINK - COMPACT */
    .inherit-values {
        margin-left: 8px;
        color: #3b82f6;
        cursor: pointer;
        font-size: 11px;
        text-decoration: none;
        font-weight: 500;
    }

    .inherit-values:hover {
        text-decoration: underline;
    }

    /* AUTO NUMBER SECTION - COMPACT */
    #autoNumberSection {
        margin-top: 12px;
        background: #f8fafc;
        padding: 12px;
        border-radius: 6px;
        border: 1px solid #e2e8f0;
    }

    .LabelAuto {
        margin-bottom: 12px;
    }

    .auto-info {
        background: #fffbeb;
        padding: 10px;
        border-radius: 4px;
        border: 1px solid #fde68a;
        font-size: 11px;
        margin-top: 12px;
    }

    .auto-info p {
        margin-top: 4px;
        color: #92400e;
        font-size: 10px;
    }

    /* DROPDOWN OPTION ROW - COMPACT */
    .dropdown-row {
        display: flex;
        align-items: center;
        gap: 8px;
        margin-bottom: 8px;
        padding: 6px 8px;
        background: #ffffff;
        border: 1px solid #e2e8f0;
        border-radius: 4px;
        transition: all 0.2s ease;
        cursor: move;
    }

    .dropdown-row:hover {
        background: #f8fafc;
        border-color: #cbd5e1;
    }

    .dropdown-row.drag-over {
        border: 2px dashed #3b82f6;
        background: #e0f2fe;
        transform: scale(1.02);
    }

    .dropdown-row.wrap {
        background: #f8fafc;
    }

    .drag {
        cursor: grab;
        font-size: 14px;
        color: #94a3b8;
        padding: 2px;
        user-select: none;
    }

    .drag:hover {
        color: #3b82f6;
    }

    .dropdown-row:active .drag {
        cursor: grabbing;
    }

    .color-select {
        width: 28px;
        height: 28px;
        border-radius: 4px;
        border: 1px solid #e2e8f0;
        cursor: pointer;
        padding: 0;
    }

    .dropdown-input {
        flex: 1;
        padding: 6px 8px;
        border: 1px solid #e2e8f0;
        border-radius: 3px;
        font-size: 12px;
        height: 28px;
        min-width: 0;
    }

    .option-actions {
        display: flex;
        gap: 8px;
        font-size: 16px;
        color: #64748b;
    }

    .option-actions span {
        cursor: pointer;
        padding: 2px 4px;
        border-radius: 3px;
        transition: all 0.2s;
        display: flex;
        align-items: center;
        justify-content: center;
        width: 24px;
        height: 24px;
    }

    .option-actions .add:hover {
        background: #d1fae5;
        color: #059669;
    }

    .option-actions .delete:hover {
        background: #fee2e2;
        color: #dc2626;
    }

    /* NO OPTIONS MESSAGE */
    .no-options {
        padding: 12px;
        text-align: center;
        color: #64748b;
        font-size: 12px;
        font-style: italic;
        background: #f8fafc;
        border: 1px dashed #e2e8f0;
        border-radius: 4px;
        margin-top: 8px;
    }

    /* UPLOAD BUTTONS - COMPACT */
    .upload-container {
        display: flex;
        flex-direction: column;
        gap: 8px;
    }

    .upload-btn {
        display: inline-flex;
        align-items: center;
        gap: 6px;
        padding: 8px 12px;
        border: 1px solid #e2e8f0;
        background: #ffffff;
        border-radius: 4px;
        cursor: pointer;
        font-size: 12px;
        font-weight: 500;
        color: #475569;
        transition: all 0.15s ease;
        min-width: 100px;
        height: 32px;
        justify-content: center;
    }

    .upload-btn:hover {
        background: #f8fafc;
        border-color: #cbd5e1;
    }

    .upload-btn.disabled {
        background: #f1f5f9;
        cursor: pointer;
        color: #94a3b8;
    }

    .upload-btn.disabled:hover {
        background: #e2e8f0;
    }

    .uploaded-file {
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding: 6px 10px;
        background: #f0f9ff;
        border: 1px solid #bae6fd;
        border-radius: 4px;
        font-size: 12px;
        color: #0369a1;
    }

    .uploaded-file-preview {
        display: flex;
        align-items: center;
        padding: 6px 10px;
        background: #f0f9ff;
        border: 1px solid #bae6fd;
        border-radius: 4px;
        font-size: 12px;
        color: #0369a1;
    }

    .remove-file {
        cursor: pointer;
        font-size: 16px;
        color: #64748b;
        padding: 0 4px;
    }

    .remove-file:hover {
        color: #ef4444;
    }

    .upload-note {
        font-size: 10px;
        color: #64748b;
        margin-top: 2px;
        font-style: italic;
    }

    /* IMAGE PREVIEW */
    .preview-upload-container {
        display: flex;
        flex-direction: column;
        gap: 8px;
    }

    .image-preview {
        width: 100%;
        max-height: 200px;
        overflow: hidden;
        border-radius: 4px;
        border: 1px solid #e2e8f0;
    }

    .image-preview img {
        width: 100%;
        height: auto;
        object-fit: contain;
        max-height: 200px;
    }

    /* EXTERNAL FIELD INPUT - COMPACT */
    .external-field-container {
        position: relative;
    }

    #externalFieldInput {
        background: #ffffff;
        cursor: pointer;
        border: 1px solid #e2e8f0;
        transition: all 0.15s ease;
        padding-right: 30px;
    }

    #externalFieldInput:hover {
        border-color: #3b82f6;
        background: #f8fafc;
    }

    .clear-btn {
        position: absolute;
        right: 5px;
        top: 50%;
        transform: translateY(-50%);
        background: none;
        border: none;
        color: #64748b;
        font-size: 16px;
        cursor: pointer;
        padding: 2px 6px;
        min-width: auto;
        height: auto;
    }

    .clear-btn:hover {
        color: #ef4444;
    }

    /* SECTION HEADERS */
    h4 {
        color: #1e293b;
        font-size: 13px;
        font-weight: 600;
        margin-bottom: 10px;
    }

    /* TEXTAREA - COMPACT */
    textarea {
        resize: vertical;
        min-height: 60px;
        line-height: 1.4;
        padding: 8px 10px;
        font-size: 12px;
        height: auto;
    }

    /* SELECT STYLING */
    select {
        appearance: none;
        background-image: url("data:image/svg+xml;charset=UTF-8,%3csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='%2364748b' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3e%3cpolyline points='6 9 12 15 18 9'%3e%3c/polyline%3e%3c/svg%3e");
        background-repeat: no-repeat;
        background-position: right 8px center;
        background-size: 14px;
        padding-right: 30px;
    }

    /* COLOR DOT - COMPACT */
    .color-dot {
        width: 14px;
        height: 14px;
        border-radius: 50%;
        display: inline-block;
        border: 1px solid #ffffff;
    }

    /* COMPACT DATA PRIVACY NOTE */
    .data-privacy-note {
        font-size: 11px;
        color: #64748b;
        margin-top: 4px;
        font-style: italic;
        line-height: 1.3;
        width: 100%;
    }

    /* REMAINING FIELDS NOTE */
    .remaining-fields {
        font-size: 11px;
        color: #64748b;
        margin-top: 4px;
        font-style: italic;
    }

    /* HELP TEXT - COMPACT */
    .Label3 input {
        background: #f8fafc;
    }

    /* DEFAULT VALUE SECTION - COMPACT */
    .Label7 {
        display: flex;
        align-items: center;
        gap: 10px;
    }

    .Label7 input {
        flex: 1;
    }

    /* DROPDOWN SECTION - COMPACT */
    #dropdownSection {
        background: #f8fafc;
        padding: 12px;
        border-radius: 6px;
        border: 1px solid #e2e8f0;
        margin-top: 12px;
    }

    #dropdownSection > div:first-child {
        margin-bottom: 8px;
    }

    #optionCount {
        font-size: 12px;
        color: #3b82f6;
        font-weight: 600;
    }

    #addOptionBtn {
        padding: 4px 8px;
        font-size: 11px;
        height: 26px;
        min-width: auto;
    }

    /* ATTACHMENT SECTION - COMPACT */
    .LabelAttachment {
        background: #f8fafc;
        padding: 10px;
        border-radius: 6px;
        border: 1px solid #e2e8f0;
    }

    .LabelAttachment label {
        display: inline-block;
        margin-right: 16px;
        font-size: 12px;
    }

    /* LOOKUP SECTION - COMPACT */
    .LabelLookup {
        background: #f8fafc;
        padding: 10px;
        border-radius: 6px;
        border: 1px solid #e2e8f0;
        margin-top: 8px;
    }

    /* EXTERNAL LOOKUP SECTION - COMPACT */
    #externalLookupSection {
        background: #f8fafc;
        padding: 12px;
        border-radius: 6px;
        border: 1px solid #e2e8f0;
        margin-top: 12px;
    }

    /* INHERIT SECTION - COMPACT */
    #inheritSection {
        background: #f8fafc;
        padding: 10px;
        border-radius: 6px;
        border: 1px solid #e2e8f0;
        margin-top: 8px;
    }

    /* PREVIEW TEXTAREA - COMPACT */
    #previewTextarea {
        min-height: 60px;
        max-height: 80px;
        padding: 8px 10px;
        font-size: 12px;
        line-height: 1.4;
    }

    /* MODAL STYLES */
    .modal-overlay {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background: rgba(0, 0, 0, 0.5);
        display: flex;
        align-items: center;
        justify-content: center;
        z-index: 1000;
    }

    .modal-content {
        background: white;
        border-radius: 8px;
        width: 90%;
        max-width: 600px;
        max-height: 80vh;
        overflow: hidden;
        box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
        display: flex;
        flex-direction: column;
    }

    .modal-header {
        padding: 16px 20px;
        border-bottom: 1px solid #e2e8f0;
        display: flex;
        align-items: center;
        justify-content: space-between;
        background: #f8fafc;
    }

    .modal-header h3 {
        margin: 0;
        font-size: 16px;
        color: #1e293b;
        font-weight: 600;
    }

    .modal-close {
        background: none;
        border: none;
        font-size: 20px;
        color: #64748b;
        cursor: pointer;
        padding: 4px 8px;
        min-width: auto;
        height: auto;
    }

    .modal-close:hover {
        color: #ef4444;
    }

    .modal-body {
        padding: 20px;
        overflow-y: auto;
        flex: 1;
    }

    .field-source-section,
    .field-search-section {
        margin-bottom: 24px;
    }

    .field-source-section h4,
    .field-search-section h4 {
        margin-bottom: 12px;
        font-size: 14px;
        color: #334155;
    }

    .service-options {
        display: grid;
        grid-template-columns: repeat(2, 1fr);
        gap: 10px;
        margin-bottom: 20px;
    }

    .service-option {
        display: flex;
        align-items: center;
        padding: 12px;
        border: 1px solid #e2e8f0;
        border-radius: 6px;
        cursor: pointer;
        transition: all 0.2s;
        background: white;
    }

    .service-option:hover {
        border-color: #3b82f6;
        background: #f0f9ff;
    }

    .service-option.selected {
        border-color: #3b82f6;
        background: #e0f2fe;
    }

    .service-radio {
        margin-right: 10px;
        display: flex;
        align-items: center;
        justify-content: center;
    }

    .radio-circle {
        width: 16px;
        height: 16px;
        border: 2px solid #cbd5e1;
        border-radius: 50%;
        position: relative;
    }

    .service-option.selected .radio-circle {
        border-color: #3b82f6;
    }

    .service-option.selected .radio-circle::after {
        content: '';
        position: absolute;
        top: 3px;
        left: 3px;
        width: 10px;
        height: 10px;
        background: #3b82f6;
        border-radius: 50%;
    }

    .service-name {
        font-size: 13px;
        color: #334155;
        font-weight: 500;
    }

    .search-input {
        width: 100%;
        padding: 10px 12px;
        border: 1px solid #e2e8f0;
        border-radius: 6px;
        font-size: 13px;
        margin-bottom: 15px;
    }

    .search-input:focus {
        outline: none;
        border-color: #3b82f6;
        box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
    }

    .field-list {
        max-height: 250px;
        overflow-y: auto;
        border: 1px solid #e2e8f0;
        border-radius: 6px;
    }

    .field-option {
        display: flex;
        align-items: center;
        padding: 12px;
        border-bottom: 1px solid #f1f5f9;
        cursor: pointer;
        transition: all 0.2s;
        background: white;
    }

    .field-option:last-child {
        border-bottom: none;
    }

    .field-option:hover {
        background: #f8fafc;
    }

    .field-option.selected {
        background: #e0f2fe;
    }

    .field-radio {
        margin-right: 10px;
    }

    .field-name {
        font-size: 13px;
        color: #334155;
    }

    .no-results {
        padding: 20px;
        text-align: center;
        color: #64748b;
        font-size: 13px;
        font-style: italic;
    }

    /* RESPONSIVE DESIGN - MOBILE */
    @media (max-width: 640px) {
        body {
            padding: 12px;
        }
        
        .main-container {
            padding: 16px;
        }
        
        .Label4,
        .Label5,
        .Label8,
        .Label9 {
            flex-direction: column;
            align-items: flex-start;
            gap: 8px;
            padding: 8px 0;
        }
        
        .actions {
            flex-direction: row;
            justify-content: center;
        }
        
        button {
            flex: 1;
        }
        
        .dropdown-row {
            flex-wrap: wrap;
        }
        
        .service-options {
            grid-template-columns: 1fr;
        }
        
        .modal-content {
            width: 95%;
            margin: 10px;
        }
    }

    /* UTILITY CLASSES */
    .hidden {
        display: none !important;
    }

    .visible {
        display: block !important;
    }

    /* FOCUS VISIBLE FOR ACCESSIBILITY */
    *:focus-visible {
        outline: 2px solid #3b82f6;
        outline-offset: 1px;
    }

    /* PLACEHOLDER STYLING */
    ::placeholder {
        color: #94a3b8;
        opacity: 1;
        font-size: 12px;
    }

    /* DISABLED STATE STYLING */
    input:disabled,
    select:disabled,
    textarea:disabled {
        background: #f8fafc;
        color: #94a3b8;
        cursor: not-allowed;
        border-color: #e2e8f0;
    }

    /* CUSTOM SCROLLBAR */
    ::-webkit-scrollbar {
        width: 6px;
        height: 6px;
    }

    ::-webkit-scrollbar-track {
        background: #f1f5f9;
        border-radius: 3px;
    }

    ::-webkit-scrollbar-thumb {
        background: #cbd5e1;
        border-radius: 3px;
    }

    ::-webkit-scrollbar-thumb:hover {
        background: #94a3b8;
    }
   .back {
    background: #f1f5f9;
    color: #334155;
    border: 1px solid #d1d9e6;
}

.back:hover {
    background: #e2e8f0;
    color: #1e293b;
}
 
    
</style>
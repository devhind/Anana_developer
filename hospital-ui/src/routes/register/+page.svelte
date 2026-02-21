<script>
    import { goto } from '$app/navigation';

    let form = {
        hospitalName: '',
        address: '',
        centerContact: '',
        doctorName: '',
        licenseNo: '',
        degree: '',
        specialization: '',
        doctorContact: '',
        adminName: '',
        adminContact: '',
        email: '',
        password: ''
    };

    let message = '';
    let error = '';

    async function registerDoctor() {
        message = '';
        error = '';

        const res = await fetch('http://localhost:8081/api/doctors/register', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(form)
        });

        const data = await res.text();

        if (res.ok) {
            message = data;
            setTimeout(() => goto('/login'), 1500);
        } else {
            error = data;
        }
    }
</script>

<div class="register-page">
    <div class="register-card">
        <h1>Doctor Registration</h1>
        <p class="subtitle">Please fill in all details to create your account</p>

        <!-- Added autocomplete="off" to the form -->
        <form on:submit|preventDefault={registerDoctor} autocomplete="off">
            <div class="form-grid">
                <!-- Hospital Name -->
                <div class="form-group">
                    <label for="hospitalName">Hospital Name</label>
                    <input type="text" id="hospitalName" placeholder="e.g., City Medical Center" bind:value={form.hospitalName} required />
                </div>
                <!-- Address -->
                <div class="form-group">
                    <label for="address">Address</label>
                    <input type="text" id="address" placeholder="123 Main St, City" bind:value={form.address} required />
                </div>
                <!-- Center Contact -->
                <div class="form-group">
                    <label for="centerContact">Center Contact</label>
                    <input type="tel" id="centerContact" placeholder="+1 234 567 890" bind:value={form.centerContact} required />
                </div>
                <!-- Doctor Name -->
                <div class="form-group">
                    <label for="doctorName">Doctor Name</label>
                    <input type="text" id="doctorName" placeholder="Dr. John Doe" bind:value={form.doctorName} required />
                </div>
                <!-- License No -->
                <div class="form-group">
                    <label for="licenseNo">License Number</label>
                    <input type="text" id="licenseNo" placeholder="LIC-12345" bind:value={form.licenseNo} required />
                </div>
                <!-- Degree -->
                <div class="form-group">
                    <label for="degree">Degree</label>
                    <input type="text" id="degree" placeholder="M.D., Ph.D." bind:value={form.degree} required />
                </div>
                <!-- Specialization -->
                <div class="form-group">
                    <label for="specialization">Specialization</label>
                    <input type="text" id="specialization" placeholder="Cardiology" bind:value={form.specialization} required />
                </div>
                <!-- Doctor Contact -->
                <div class="form-group">
                    <label for="doctorContact">Doctor Contact</label>
                    <input type="tel" id="doctorContact" placeholder="+1 234 567 890" bind:value={form.doctorContact} required />
                </div>
                <!-- Admin Name -->
                <div class="form-group">
                    <label for="adminName">Admin Name</label>
                    <input type="text" id="adminName" placeholder="Jane Smith" bind:value={form.adminName} required />
                </div>
                <!-- Admin Contact -->
                <div class="form-group">
                    <label for="adminContact">Admin Contact</label>
                    <input type="tel" id="adminContact" placeholder="+1 234 567 890" bind:value={form.adminContact} required />
                </div>
                <!-- Email (autocomplete off) -->
                <div class="form-group">
                    <label for="email">Email</label>
                    <input type="email" id="email" placeholder="doctor@example.com" bind:value={form.email} required autocomplete="off" />
                </div>
                <!-- Password (autocomplete new-password) -->
                <div class="form-group">
                    <label for="password">Password</label>
                    <input type="password" id="password" placeholder="••••••••" bind:value={form.password} required autocomplete="new-password" />
                </div>
            </div>

            <button type="submit" class="register-button">Register</button>
        </form>

        {#if message}
            <div class="message success">{message}</div>
        {/if}
        {#if error}
            <div class="message error">{error}</div>
        {/if}

        <p class="login-link">
            Already have an account? <a href="/login">Login here</a>
        </p>
    </div>
</div>

<style>
    .register-page {
        display: flex;
        align-items: center;
        justify-content: center;
        min-height: 100vh;
        background: #f8fafc;
        font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
        padding: 1.5rem;
    }

    .register-card {
        background: white;
        max-width: 900px;
        width: 100%;
        padding: 2.5rem;
        border-radius: 2rem;
        box-shadow: 0 20px 35px -8px rgba(0, 0, 0, 0.1), 0 5px 10px -4px rgba(0, 0, 0, 0.05);
        transition: transform 0.2s ease, box-shadow 0.2s ease;
        animation: cardFadeIn 0.3s ease-out;
    }

    .register-card:hover {
        box-shadow: 0 25px 45px -10px rgba(0, 0, 0, 0.15);
    }

    h1 {
        font-size: 2rem;
        font-weight: 600;
        color: #0f172a;
        margin: 0 0 0.25rem 0;
        letter-spacing: -0.02em;
    }

    .subtitle {
        color: #475569;
        margin-top: 0;
        margin-bottom: 2rem;
        font-size: 1rem;
    }

    .form-grid {
        display: grid;
        grid-template-columns: 1fr 1fr;
        gap: 1.5rem;
        margin-bottom: 2rem;
    }

    .form-group {
        display: flex;
        flex-direction: column;
    }

    label {
        font-size: 0.9rem;
        font-weight: 500;
        color: #334155;
        margin-bottom: 0.4rem;
    }

    input {
        width: 100%;
        padding: 0.75rem 1rem;
        font-size: 1rem;
        border: 1px solid #e2e8f0;
        border-radius: 0.75rem;
        background-color: #f9f9fc;
        transition: border 0.2s ease, box-shadow 0.2s ease, background-color 0.2s ease;
        box-sizing: border-box;
    }

    input:focus {
        outline: none;
        border-color: #4f46e5;
        background-color: #ffffff;
        box-shadow: 0 0 0 4px rgba(79, 70, 229, 0.1);
    }

    input::placeholder {
        color: #94a3b8;
        font-weight: 300;
        font-size: 0.95rem;
    }

    .register-button {
        width: 100%;
        padding: 0.85rem 1rem;
        font-size: 1.1rem;
        font-weight: 600;
        color: white;
        background: linear-gradient(135deg, #4f46e5 0%, #6366f1 100%);
        border: none;
        border-radius: 0.75rem;
        cursor: pointer;
        transition: all 0.2s ease;
        box-shadow: 0 4px 10px rgba(79, 70, 229, 0.3);
        margin-bottom: 1.5rem;
    }

    .register-button:hover {
        background: linear-gradient(135deg, #4338ca 0%, #4f46e5 100%);
        transform: translateY(-2px);
        box-shadow: 0 8px 20px rgba(79, 70, 229, 0.4);
    }

    .register-button:active {
        transform: translateY(0);
        box-shadow: 0 4px 12px rgba(79, 70, 229, 0.4);
    }

    .message {
        padding: 1rem;
        border-radius: 0.75rem;
        font-weight: 500;
        text-align: center;
        margin-bottom: 1rem;
    }

    .message.success {
        background-color: #f0fdf4;
        color: #166534;
        border: 1px solid #86efac;
    }

    .message.error {
        background-color: #fef2f2;
        color: #991b1b;
        border: 1px solid #fecaca;
    }

    .login-link {
        text-align: center;
        font-size: 0.95rem;
        color: #475569;
        margin: 0;
    }

    .login-link a {
        color: #4f46e5;
        text-decoration: none;
        font-weight: 500;
        transition: color 0.2s ease;
    }

    .login-link a:hover {
        color: #4338ca;
        text-decoration: underline;
    }

    @keyframes cardFadeIn {
        from {
            opacity: 0;
            transform: translateY(10px);
        }
        to {
            opacity: 1;
            transform: translateY(0);
        }
    }

    /* Responsive: single column on small screens */
    @media (max-width: 640px) {
        .register-card {
            padding: 1.5rem;
        }
        .form-grid {
            grid-template-columns: 1fr;
            gap: 1rem;
        }
        h1 {
            font-size: 1.75rem;
        }
    }
</style>
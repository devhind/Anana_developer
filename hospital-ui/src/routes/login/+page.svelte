<script>
  import { goto } from '$app/navigation';

  let email = '';
  let password = '';

  async function login() {
    const res = await fetch("http://localhost:8081/api/doctors/login", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email, password })
    });

    const data = await res.text();

    if (res.ok) {
      localStorage.setItem("doctor", JSON.stringify({
        name: email   // you can later replace with doctorName from backend
      }));
      goto('/patient');
    } else {
      alert(data);
    }
  }
</script>

<div class="login-page">
  <div class="login-card">
    <h1>Doctor Login</h1>
    <!-- Added autocomplete="off" to prevent browser autofill -->
    <form on:submit|preventDefault={login} autocomplete="off">
      <div class="form-group">
        <label for="email">Email</label>
        <input
          type="email"
          id="email"
          placeholder="doctor@example.com"
          bind:value={email}
          required
          autocomplete="off"              
        />
      </div>
      <div class="form-group">
        <label for="password">Password</label>
        <input
          type="password"
          id="password"
          placeholder="••••••••"
          bind:value={password}
          required
          autocomplete="new-password"     
        />
      </div>
      <button type="submit" class="login-button">Log in</button>
    </form>
    <p class="register-link">
      Don't have an account? <a href="/register">Register Here</a>
    </p>
  </div>
</div>

<style>
  .login-page {
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
    background: #f8fafc;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
    margin: 0;
    padding: 1rem;
  }

  .login-card {
    background: white;
    max-width: 400px;
    width: 100%;
    padding: 2.5rem 2rem;
    border-radius: 1.5rem;
    box-shadow: 0 20px 35px -8px rgba(0, 0, 0, 0.1), 0 5px 10px -4px rgba(0, 0, 0, 0.05);
    transition: transform 0.2s ease, box-shadow 0.2s ease;
    animation: cardFadeIn 0.3s ease-out;
  }

  .login-card:hover {
    box-shadow: 0 25px 45px -10px rgba(0, 0, 0, 0.15);
  }

  h1 {
    font-size: 2rem;
    font-weight: 600;
    color: #0f172a;
    margin-top: 0;
    margin-bottom: 2rem;
    letter-spacing: -0.02em;
    text-align: center;
  }

  .form-group {
    margin-bottom: 1.5rem;
  }

  label {
    display: block;
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
  }

  .login-button {
    width: 100%;
    padding: 0.85rem 1rem;
    font-size: 1rem;
    font-weight: 600;
    color: white;
    background: linear-gradient(135deg, #4f46e5 0%, #6366f1 100%);
    border: none;
    border-radius: 0.75rem;
    cursor: pointer;
    transition: all 0.2s ease;
    box-shadow: 0 4px 10px rgba(79, 70, 229, 0.3);
    margin-top: 0.5rem;
  }

  .login-button:hover {
    background: linear-gradient(135deg, #4338ca 0%, #4f46e5 100%);
    transform: translateY(-2px);
    box-shadow: 0 8px 20px rgba(79, 70, 229, 0.4);
  }

  .login-button:active {
    transform: translateY(0);
    box-shadow: 0 4px 12px rgba(79, 70, 229, 0.4);
  }

  .register-link {
    text-align: center;
    margin-top: 2rem;
    font-size: 0.95rem;
    color: #475569;
  }

  .register-link a {
    color: #4f46e5;
    text-decoration: none;
    font-weight: 500;
    transition: color 0.2s ease;
  }

  .register-link a:hover {
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
</style>
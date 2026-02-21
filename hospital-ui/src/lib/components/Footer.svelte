<footer class="footer">
  <div class="footer-content">
    <div class="footer-left">
      <span>© 2026 Hospital Management System</span>
      <span class="footer-divider">•</span>
      <span class="footer-version">v2.1.4</span>
    </div>
    
    <div class="footer-right">
      <span class="footer-info">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor">
          <circle cx="12" cy="12" r="10"/>
          <path d="M12 16v-4M12 8h.01"/>
        </svg>
        Secure & HIPAA Compliant
      </span>
      <span class="footer-divider">•</span>
      <a href="/" class="footer-link">Privacy Policy</a>
      <span class="footer-divider">•</span>
      <a href="/" class="footer-link">Terms of Service</a>
    </div>
  </div>
</footer>

<style>
.footer {
  height: 52px;
  background: linear-gradient(180deg, #f8fafc 0%, #f1f5f9 100%);
  border-top: 1px solid #e2e8f0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  color: #475569;
  position: relative;
  overflow: hidden;
  box-shadow: 0 -2px 10px rgba(30, 41, 59, 0.04);
}

.footer::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 2px;
  background: linear-gradient(90deg, #3b82f6, #1d4ed8);
  opacity: 0.8;
}

.footer-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
  max-width: 1200px;
  padding: 0 24px;
  gap: 16px;
}

.footer-left, .footer-right {
  display: flex;
  align-items: center;
  gap: 8px;
}

.footer-divider {
  color: #cbd5e1;
  font-weight: 300;
}

.footer-version {
  font-family: 'SF Mono', 'Monaco', 'Inconsolata', monospace;
  background: rgba(30, 41, 59, 0.08);
  padding: 2px 8px;
  border-radius: 4px;
  color: #475569;
  font-weight: 500;
  font-size: 11px;
  border: 1px solid #e2e8f0;
}

.footer-info {
  display: flex;
  align-items: center;
  gap: 6px;
  color: #3b82f6;
  font-weight: 500;
  background: rgba(59, 130, 246, 0.08);
  padding: 4px 10px;
  border-radius: 6px;
  border: 1px solid rgba(59, 130, 246, 0.2);
}

.footer-info svg {
  color: #3b82f6;
  flex-shrink: 0;
}

.footer-link {
  color: #475569;
  text-decoration: none;
  padding: 4px 8px;
  border-radius: 4px;
  transition: all 0.2s ease;
  position: relative;
}

.footer-link:hover {
  color: #1d4ed8;
  background: rgba(59, 130, 246, 0.1);
}

.footer-link::after {
  content: '';
  position: absolute;
  bottom: -2px;
  left: 8px;
  right: 8px;
  height: 1px;
  background: linear-gradient(90deg, #3b82f6, transparent);
  opacity: 0;
  transition: opacity 0.2s ease;
}

.footer-link:hover::after {
  opacity: 1;
}

/* Responsive Design */
@media (max-width: 768px) {
  .footer {
    height: auto;
    padding: 12px 0;
  }
  
  .footer-content {
    flex-direction: column;
    gap: 12px;
    text-align: center;
    padding: 0 16px;
  }
  
  .footer-left, .footer-right {
    flex-wrap: wrap;
    justify-content: center;
  }
  
  .footer-divider {
    display: none;
  }
  
  .footer-link {
    padding: 4px 6px;
  }
}

@media (max-width: 480px) {
  .footer {
    font-size: 11px;
  }
  
  .footer-content {
    padding: 0 12px;
  }
  
  .footer-info {
    padding: 3px 8px;
    font-size: 11px;
  }
  
  .footer-version {
    font-size: 10px;
    padding: 2px 6px;
  }
}

/* Animation */
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.footer {
  animation: fadeInUp 0.4s ease-out 0.1s both;
}

/* Optional: Add a subtle pulse animation to the security badge */
@keyframes pulseBlue {
  0%, 100% {
    border-color: rgba(59, 130, 246, 0.2);
    background: rgba(59, 130, 246, 0.08);
  }
  50% {
    border-color: rgba(59, 130, 246, 0.4);
    background: rgba(59, 130, 246, 0.12);
  }
}

.footer-info {
  animation: pulseBlue 4s infinite;
}
</style>
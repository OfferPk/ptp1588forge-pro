:root {  
  \--bg-primary: \#020617;  
  \--bg-secondary: \#0a1128;  
  \--bg-surface: \#0f1c3f;  
  \--border-color: \#1e3a75;  
  \--border-focus: \#38bdf8;  
  \--text-main: \#f8fafc;  
  \--text-muted: \#94a3b8;  
  \--accent-cyan: \#06b6d4;  
  \--accent-sky: \#38bdf8;  
  \--accent-emerald: \#10b981;  
  \--accent-teal: \#14b8a6;  
  \--accent-blue: \#3b82f6;  
  \--accent-amber: \#f59e0b;  
  \--accent-red: \#ef4444;  
  \--font-sans: \-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;  
  \--font-mono: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", monospace;  
}

\* {  
  box-sizing: border-box;  
  margin: 0;  
  padding: 0;  
}

body {  
  background-color: var(--bg-primary);  
  color: var(--text-main);  
  font-family: var(--font-sans);  
  line-height: 1.5;  
  min-height: 100vh;  
  display: flex;  
  flex-direction: column;  
}

header {  
  background-color: var(--bg-secondary);  
  border-bottom: 1.5px solid var(--border-color);  
  padding: 1rem 2rem;  
  display: flex;  
  align-items: center;  
  justify-content: space-between;  
  flex-wrap: wrap;  
  gap: 1rem;  
}

.brand-group {  
  display: flex;  
  align-items: center;  
  gap: 0.85rem;  
}

.brand-badge {  
  background: linear-gradient(135deg, var(--accent-sky), var(--accent-blue));  
  color: \#020617;  
  font-weight: 900;  
  font-size: 0.75rem;  
  padding: 0.35rem 0.65rem;  
  border-radius: 6px;  
  letter-spacing: 0.08em;  
}

.brand-title {  
  font-size: 1.25rem;  
  font-weight: 800;  
  color: var(--text-main);  
}

.brand-subtitle {  
  font-size: 0.85rem;  
  color: var(--accent-sky);  
  font-weight: 500;  
}

.header-badges {  
  display: flex;  
  gap: 0.5rem;  
  align-items: center;  
}

.badge {  
  padding: 0.25rem 0.6rem;  
  border-radius: 12px;  
  font-size: 0.75rem;  
  font-weight: 600;  
  text-transform: uppercase;  
}

.badge-ieee {  
  background: rgba(56, 189, 248, 0.15);  
  color: var(--accent-sky);  
  border: 1px solid rgba(56, 189, 248, 0.4);  
}

.badge-score {  
  background: rgba(16, 185, 129, 0.15);  
  color: var(--accent-emerald);  
  border: 1px solid rgba(16, 185, 129, 0.4);  
}

.preset-container {  
  display: flex;  
  align-items: center;  
  gap: 0.75rem;  
}

select, input, textarea {  
  background: \#02040a;  
  border: 1px solid var(--border-color);  
  color: var(--text-main);  
  padding: 0.45rem 0.75rem;  
  border-radius: 6px;  
  font-family: inherit;  
  font-size: 0.875rem;  
}

select:focus, input:focus, textarea:focus {  
  outline: none;  
  border-color: var(--border-focus);  
  box-shadow: 0 0 0 2px rgba(56, 189, 248, 0.25);  
}

.main-content {  
  flex: 1;  
  display: grid;  
  grid-template-columns: 1fr 1fr;  
  gap: 1.5rem;  
  padding: 1.5rem 2rem;  
}

@media (max-width: 1024px) {  
  .main-content {  
    grid-template-columns: 1fr;  
  }  
}

.panel {  
  background: var(--bg-secondary);  
  border: 1.5px solid var(--border-color);  
  border-radius: 12px;  
  padding: 1.25rem;  
  display: flex;  
  flex-direction: column;  
  gap: 1.25rem;  
}

.panel-title {  
  font-size: 1.05rem;  
  font-weight: 700;  
  color: var(--text-main);  
  border-bottom: 1px solid var(--border-color);  
  padding-bottom: 0.5rem;  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
}

.form-grid {  
  display: grid;  
  grid-template-columns: 1fr 1fr;  
  gap: 0.85rem;  
}

.form-group {  
  display: flex;  
  flex-direction: column;  
  gap: 0.3rem;  
}

.form-group.full-width {  
  grid-column: span 2;  
}

label {  
  font-size: 0.75rem;  
  font-weight: 600;  
  text-transform: uppercase;  
  color: var(--text-muted);  
  letter-spacing: 0.05em;  
}

.checkbox-group {  
  display: flex;  
  align-items: center;  
  gap: 0.5rem;  
}

.checkbox-group input\[type="checkbox"\] {  
  width: 1rem;  
  height: 1rem;  
  accent-color: var(--accent-sky);  
}

.scorecard-banner {  
  display: flex;  
  align-items: center;  
  gap: 1rem;  
  padding: 1rem;  
  border-radius: 8px;  
  background: rgba(16, 185, 129, 0.1);  
  border: 1px solid rgba(16, 185, 129, 0.3);  
}

.score-badge {  
  width: 54px;  
  height: 54px;  
  border-radius: 8px;  
  display: flex;  
  align-items: center;  
  justify-content: center;  
  font-size: 1.5rem;  
  font-weight: 800;  
  font-family: var(--font-mono);  
  background: rgba(16, 185, 129, 0.2);  
  color: var(--accent-emerald);  
  border: 1px solid var(--accent-emerald);  
}

.score-details h3 {  
  font-size: 1.05rem;  
  font-weight: 700;  
}

.score-details p {  
  font-size: 0.8rem;  
  color: var(--text-muted);  
}

.audit-list {  
  display: flex;  
  flex-direction: column;  
  gap: 0.5rem;  
  max-height: 250px;  
  overflow-y: auto;  
}

.audit-item {  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  padding: 0.5rem 0.75rem;  
  border-radius: 6px;  
  background: \#020617;  
  border: 1px solid var(--border-color);  
  font-size: 0.8rem;  
}

.audit-item.PASS {  
  border-left: 3px solid var(--accent-emerald);  
}

.audit-item.WARNING {  
  border-left: 3px solid var(--accent-amber);  
}

.audit-item.FAIL {  
  border-left: 3px solid var(--accent-red);  
}

.hex-box {  
  background: \#01030a;  
  border: 1px solid var(--border-color);  
  border-radius: 6px;  
  padding: 0.75rem;  
  font-family: var(--font-mono);  
  font-size: 0.8rem;  
  color: \#38bdf8;  
  white-space: pre;  
  overflow-x: auto;  
  max-height: 180px;  
}

.tabs {  
  display: flex;  
  gap: 0.35rem;  
  border-bottom: 1px solid var(--border-color);  
  padding-bottom: 0.35rem;  
  flex-wrap: wrap;  
}

.tab-btn {  
  background: transparent;  
  border: 1px solid transparent;  
  color: var(--text-muted);  
  padding: 0.35rem 0.75rem;  
  border-radius: 4px;  
  cursor: pointer;  
  font-size: 0.8rem;  
  font-weight: 600;  
}

.tab-btn.active {  
  background: var(--bg-surface);  
  border-color: var(--border-color);  
  color: var(--accent-sky);  
}

.code-output {  
  background: \#01030a;  
  border: 1px solid var(--border-color);  
  border-radius: 6px;  
  padding: 0.75rem;  
  font-family: var(--font-mono);  
  font-size: 0.8rem;  
  color: \#bae6fd;  
  white-space: pre-wrap;  
  height: 260px;  
  overflow-y: auto;  
}

.btn-group {  
  display: flex;  
  gap: 0.5rem;  
}

.btn {  
  background: var(--accent-sky);  
  color: \#020617;  
  border: none;  
  padding: 0.45rem 0.85rem;  
  border-radius: 6px;  
  cursor: pointer;  
  font-size: 0.8rem;  
  font-weight: 700;  
  display: inline-flex;  
  align-items: center;  
  gap: 0.35rem;  
}

.btn:hover {  
  filter: brightness(1.1);  
}

.btn-secondary {  
  background: var(--bg-surface);  
  border: 1px solid var(--border-color);  
  color: var(--text-main);  
}  

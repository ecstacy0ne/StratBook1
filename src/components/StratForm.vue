<template>
  <div class="form-overlay" @click.self="$emit('close')">
    <div class="protocol-card">
      <header class="protocol-header">
        <div class="h-text">
          <span class="protocol-id">TACTICAL INITIALIZATION</span>
          <h2>SECTOR: <span class="accent">{{ activeMap }}</span></h2>
        </div>
        <button 
          @click="$emit('close')" 
          class="btn-protocol-close" 
          title="Close form"
        >
          ✕
        </button>
      </header>

      <div class="protocol-body">
        <div class="protocol-grid">
          <!-- Левая колонка -->
          <div class="grid-left">
            <div class="input-field">
              <label>
                STRATEGY DESIGNATION 
                <span class="required">*</span>
              </label>
              <input 
                v-model="form.name" 
                class="hud-input-dark" 
                placeholder="e.g. MIRAGE FAST A-SITE SMOKES"
                maxlength="100"
                @input="validateName"
              >
              <span v-if="nameError" class="field-error">{{ nameError }}</span>
              <span v-else-if="form.name.length > 80" class="field-hint">
                {{ form.name.length }}/100 characters
              </span>
            </div>
            
            <div class="input-field">
              <label>SEQUENTIAL INSTRUCTIONS</label>
              <textarea 
                v-model="form.description" 
                class="hud-input-dark" 
                rows="8" 
                placeholder="Enter tactical steps...&#10;&#10;Step 1: Smoke CT spawn&#10;Step 2: Flash A site&#10;Step 3: ..."
                maxlength="2000"
              ></textarea>
              <span v-if="form.description.length > 1500" class="field-hint">
                {{ form.description.length }}/2000 characters
              </span>
            </div>

            <div class="input-field">
              <label>DIGITAL FOOTAGE (URL)</label>
              <input 
                v-model="form.videoUrl" 
                class="hud-input-dark" 
                placeholder="https://youtube.com/..."
                type="url"
                @blur="validateUrl"
              >
              <span v-if="urlError" class="field-error">{{ urlError }}</span>
              <span v-else-if="form.videoUrl && !urlError" class="field-success">
                ✓ Valid URL format
              </span>
            </div>
          </div>

          <!-- Правая колонка -->
          <div class="grid-right">
            <div class="input-field">
              <label>TEAM SIDE <span class="required">*</span></label>
              <div class="switch-group">
                <button 
                  @click="form.side = 't'" 
                  :class="{ active: form.side === 't' }" 
                  class="btn-side t"
                  type="button"
                >
                  <span class="side-icon">⚔️</span>
                  T SIDE
                </button>
                <button 
                  @click="form.side = 'ct'" 
                  :class="{ active: form.side === 'ct' }" 
                  class="btn-side ct"
                  type="button"
                >
                  <span class="side-icon">🛡️</span>
                  CT SIDE
                </button>
              </div>
            </div>

            <div class="input-field">
              <label>TARGET SECTOR</label>
              <div class="site-switch">
                <button 
                  v-for="s in sites" 
                  :key="s.value" 
                  @click="form.site = s.value" 
                  :class="{ active: form.site === s.value }"
                  type="button"
                  :title="s.description"
                >
                  {{ s.label }}
                </button>
              </div>
            </div>

            <div class="input-field">
              <label>UTILITY ALLOCATION</label>
              <div class="utility-setup">
                <div class="u-row">
                  <span class="u-icon">☁️</span>
                  <div class="u-control">
                    <button 
                      @click="decrementUtility('smokes')" 
                      class="u-btn"
                      :disabled="form.utility.smokes <= 0"
                      type="button"
                    >−</button>
                    <input 
                      type="number" 
                      v-model.number="form.utility.smokes" 
                      min="0" 
                      max="5"
                      class="u-input"
                    >
                    <button 
                      @click="incrementUtility('smokes')" 
                      class="u-btn"
                      :disabled="form.utility.smokes >= 5"
                      type="button"
                    >+</button>
                  </div>
                  <span class="u-label">Smokes</span>
                </div>
                
                <div class="u-row">
                  <span class="u-icon">🔥</span>
                  <div class="u-control">
                    <button 
                      @click="decrementUtility('mollys')" 
                      class="u-btn"
                      :disabled="form.utility.mollys <= 0"
                      type="button"
                    >−</button>
                    <input 
                      type="number" 
                      v-model.number="form.utility.mollys" 
                      min="0" 
                      max="5"
                      class="u-input"
                    >
                    <button 
                      @click="incrementUtility('mollys')" 
                      class="u-btn"
                      :disabled="form.utility.mollys >= 5"
                      type="button"
                    >+</button>
                  </div>
                  <span class="u-label">Mollys</span>
                </div>
                
                <div class="u-row">
                  <span class="u-icon">✨</span>
                  <div class="u-control">
                    <button 
                      @click="decrementUtility('flashes')" 
                      class="u-btn"
                      :disabled="form.utility.flashes <= 0"
                      type="button"
                    >−</button>
                    <input 
                      type="number" 
                      v-model.number="form.utility.flashes" 
                      min="0" 
                      max="5"
                      class="u-input"
                    >
                    <button 
                      @click="incrementUtility('flashes')" 
                      class="u-btn"
                      :disabled="form.utility.flashes >= 5"
                      type="button"
                    >+</button>
                  </div>
                  <span class="u-label">Flashes</span>
                </div>
              </div>
              
              <div class="utility-total" v-if="totalUtility > 0">
                Total utility: {{ totalUtility }} grenades
              </div>
            </div>
          </div>
        </div>
      </div>

      <footer class="protocol-footer">
        <div class="footer-actions">
          <button 
            @click="$emit('close')" 
            class="btn-protocol-cancel"
            type="button"
          >
            ABORT MISSION
          </button>
          <button 
            @click="handleDeploy" 
            class="btn-protocol-submit"
            :disabled="!isFormValid"
            type="button"
          >
            {{ isFormValid ? 'DEPLOY DATA TO REGISTRY' : 'FILL REQUIRED FIELDS' }}
          </button>
        </div>
        <div v-if="!isFormValid" class="form-error-summary">
          Please fill in all required fields marked with *
        </div>
      </footer>
    </div>
  </div>
</template>

<script>
export default {
  name: 'StratForm',
  
  props: { 
    activeMap: { 
      type: String, 
      required: true 
    } 
  },
  
  emits: ['submit', 'close'],
  
  data() {
    return {
      form: { 
        name: '', 
        description: '', 
        videoUrl: '', 
        side: 't', 
        site: 'A', 
        utility: { 
          smokes: 0, 
          mollys: 0, 
          flashes: 0 
        } 
      },
      nameError: '',
      urlError: '',
      sites: [
        { value: 'A', label: 'A', description: 'A Bomb Site' },
        { value: 'B', label: 'B', description: 'B Bomb Site' },
        { value: 'Mid', label: 'MID', description: 'Middle Area' },
        { value: 'Any', label: 'ANY', description: 'Any Site' }
      ]
    }
  },
  
  computed: {
    isFormValid() {
      return this.form.name.trim().length >= 3 && !this.nameError && !this.urlError;
    },
    
    totalUtility() {
      const { smokes, mollys, flashes } = this.form.utility;
      return smokes + mollys + flashes;
    }
  },
  
  methods: {
    validateName() {
      const name = this.form.name.trim();
      
      if (name.length === 0) {
        this.nameError = '';
      } else if (name.length < 3) {
        this.nameError = 'Designation too short (minimum 3 characters)';
      } else if (name.length > 100) {
        this.nameError = 'Designation too long (maximum 100 characters)';
      } else if (!/^[A-Za-z0-9\s\-_.,!?()]+$/.test(name)) {
        this.nameError = 'Contains invalid characters';
      } else {
        this.nameError = '';
      }
    },
    
    validateUrl() {
      const url = this.form.videoUrl.trim();
      
      if (url.length === 0) {
        this.urlError = '';
        return;
      }
      
      try {
        const urlObj = new URL(url);
        if (!['http:', 'https:'].includes(urlObj.protocol)) {
          this.urlError = 'URL must start with http:// or https://';
        } else {
          this.urlError = '';
        }
      } catch {
        if (url.includes('.') && !url.includes(' ') && url.length > 5) {
          this.urlError = 'Add https:// prefix (e.g. https://' + url + ')';
        } else {
          this.urlError = 'Invalid URL format';
        }
      }
    },
    
    incrementUtility(type) {
      if (this.form.utility[type] < 5) {
        this.form.utility[type]++;
      }
    },
    
    decrementUtility(type) {
      if (this.form.utility[type] > 0) {
        this.form.utility[type]--;
      }
    },
    
    handleDeploy() {
      this.validateName();
      if (this.form.videoUrl) {
        this.validateUrl();
      }
      
      if (!this.isFormValid) {
        if (!this.form.name.trim()) {
          alert('SYSTEM ERROR: STRATEGY DESIGNATION REQUIRED');
        }
        return;
      }
      
      // Создаем копию объекта для отправки
      const payload = {
        name: this.form.name.trim(),
        description: this.form.description.trim(),
        videoUrl: this.form.videoUrl.trim(),
        side: this.form.side,
        site: this.form.site,
        utility: { ...this.form.utility }
      };
      
      this.$emit('submit', payload);
      
      // Сброс формы после успешной отправки
      this.resetForm();
    },
    
    resetForm() {
      this.form = {
        name: '',
        description: '',
        videoUrl: '',
        side: 't',
        site: 'A',
        utility: {
          smokes: 0,
          mollys: 0,
          flashes: 0
        }
      };
      this.nameError = '';
      this.urlError = '';
    }
  },
  
  mounted() {
    // Автофокус на поле имени при открытии формы
    this.$nextTick(() => {
      const nameInput = this.$el.querySelector('input[type="text"]');
      if (nameInput) {
        nameInput.focus();
      }
    });
  }
}
</script>

<style scoped>
.form-overlay { 
  position: fixed; 
  inset: 0; 
  background: var(--overlay); 
  backdrop-filter: blur(12px); 
  display: flex; 
  align-items: center; 
  justify-content: center; 
  z-index: 1000; 
  padding: 25px; 
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.protocol-card { 
  background: var(--surface); 
  border: 1px solid var(--border); 
  border-radius: 32px; 
  width: 100%; 
  max-width: 900px; 
  max-height: 90vh;
  overflow-y: auto;
  box-shadow: 0 40px 100px rgba(0,0,0,0.6); 
  animation: slideUp 0.3s ease;
}

@keyframes slideUp {
  from { transform: translateY(30px); opacity: 0; }
  to { transform: translateY(0); opacity: 1; }
}

.protocol-card::-webkit-scrollbar {
  width: 8px;
}

.protocol-card::-webkit-scrollbar-track {
  background: transparent;
}

.protocol-card::-webkit-scrollbar-thumb {
  background: var(--border);
  border-radius: 4px;
}

.protocol-header { 
  display: flex; 
  justify-content: space-between; 
  align-items: flex-start;
  padding: 40px 50px 30px; 
  border-bottom: 1px solid var(--border); 
  position: sticky;
  top: 0;
  background: var(--surface);
  z-index: 10;
  border-radius: 32px 32px 0 0;
}

.protocol-id { 
  font-size: 10px; 
  font-weight: 900; 
  color: var(--muted); 
  letter-spacing: 2.5px; 
  display: block; 
  margin-bottom: 8px; 
}

.accent { 
  color: var(--accent); 
}

.btn-protocol-close { 
  background: none; 
  border: none; 
  color: var(--text); 
  font-size: 28px; 
  cursor: pointer; 
  opacity: 0.4; 
  transition: all 0.2s ease; 
  padding: 5px 10px;
  border-radius: 8px;
}

.btn-protocol-close:hover { 
  opacity: 1; 
  transform: scale(1.1);
  background: var(--panel);
}

.protocol-body { 
  padding: 45px 50px; 
}

.protocol-grid { 
  display: grid; 
  grid-template-columns: 1.2fr 0.8fr; 
  gap: 50px; 
}

.input-field {
  margin-bottom: 35px;
}

.input-field:last-child {
  margin-bottom: 0;
}

label { 
  display: block; 
  font-size: 11px; 
  font-weight: 900; 
  color: var(--muted); 
  margin-bottom: 15px; 
  letter-spacing: 1px; 
}

.required {
  color: #ef4444;
  margin-left: 2px;
}

.hud-input-dark { 
  background: var(--input); 
  border: 1px solid var(--border); 
  color: var(--text); 
  padding: 18px; 
  border-radius: 16px; 
  width: 100%; 
  outline: none; 
  transition: all 0.2s ease; 
  font-size: 15px; 
  font-family: inherit;
  resize: vertical;
}

.hud-input-dark:focus { 
  border-color: var(--accent); 
  box-shadow: 0 0 15px rgba(0, 209, 255, 0.1); 
}

.hud-input-dark::placeholder {
  color: var(--muted);
  opacity: 0.5;
}

.field-error {
  display: block;
  color: #ef4444;
  font-size: 11px;
  font-weight: 600;
  margin-top: 8px;
  padding-left: 5px;
}

.field-hint {
  display: block;
  color: var(--muted);
  font-size: 11px;
  font-weight: 600;
  margin-top: 8px;
  padding-left: 5px;
}

.field-success {
  display: block;
  color: #10b981;
  font-size: 11px;
  font-weight: 600;
  margin-top: 8px;
  padding-left: 5px;
}

/* SWITCH GROUP */
.switch-group { 
  display: flex; 
  gap: 12px; 
}

.btn-side { 
  flex: 1; 
  padding: 16px; 
  background: var(--panel); 
  border: 2px solid var(--border); 
  color: var(--muted); 
  border-radius: 14px; 
  cursor: pointer; 
  font-size: 12px; 
  font-weight: 900; 
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.side-icon {
  font-size: 16px;
}

.btn-side:hover:not(.active) {
  border-color: var(--text);
  color: var(--text);
}

.btn-side.t.active { 
  border-color: var(--t-color); 
  color: var(--t-color); 
  background: rgba(255, 160, 0, 0.1);
  box-shadow: 0 0 20px rgba(255, 160, 0, 0.2);
}

.btn-side.ct.active { 
  border-color: var(--ct-color); 
  color: var(--ct-color); 
  background: rgba(0, 162, 255, 0.1);
  box-shadow: 0 0 20px rgba(0, 162, 255, 0.2);
}

/* SITE SWITCH */
.site-switch { 
  display: grid; 
  grid-template-columns: 1fr 1fr; 
  gap: 12px; 
}

.site-switch button { 
  padding: 16px; 
  background: var(--panel); 
  border: 2px solid var(--border); 
  color: var(--muted); 
  border-radius: 14px; 
  cursor: pointer; 
  font-size: 13px; 
  font-weight: 900; 
  transition: all 0.2s ease;
}

.site-switch button:hover:not(.active) {
  border-color: var(--text);
  color: var(--text);
}

.site-switch button.active { 
  background: var(--accent); 
  color: #fff; 
  border-color: var(--accent);
  box-shadow: 0 0 20px rgba(0, 209, 255, 0.3); 
}

/* UTILITY SETUP */
.utility-setup { 
  display: flex; 
  flex-direction: column;
  gap: 15px;
}

.u-row { 
  display: flex; 
  align-items: center; 
  gap: 12px;
  padding: 12px;
  background: var(--panel);
  border-radius: 12px;
  border: 1px solid var(--border);
}

.u-icon {
  font-size: 20px;
  width: 30px;
  text-align: center;
}

.u-control {
  display: flex;
  align-items: center;
  gap: 8px;
  flex: 1;
}

.u-btn {
  width: 30px;
  height: 30px;
  background: var(--surface);
  border: 1px solid var(--border);
  color: var(--text);
  border-radius: 8px;
  cursor: pointer;
  font-size: 16px;
  font-weight: 900;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
}

.u-btn:hover:not(:disabled) {
  background: var(--accent);
  color: #fff;
  border-color: var(--accent);
}

.u-btn:disabled {
  opacity: 0.3;
  cursor: not-allowed;
}

.u-input {
  width: 50px;
  background: var(--surface);
  border: 1px solid var(--border);
  color: var(--text);
  padding: 8px;
  border-radius: 8px;
  text-align: center;
  font-weight: 900;
  font-family: monospace;
  font-size: 16px;
  outline: none;
}

.u-input:focus {
  border-color: var(--accent);
}

.u-label {
  font-size: 11px;
  font-weight: 700;
  color: var(--muted);
  min-width: 60px;
}

.utility-total {
  margin-top: 15px;
  text-align: right;
  font-size: 12px;
  font-weight: 800;
  color: var(--accent);
  padding: 8px 12px;
  background: var(--panel);
  border-radius: 8px;
  border: 1px solid var(--border);
}

/* FOOTER */
.protocol-footer { 
  padding: 30px 50px 40px; 
  border-top: 1px solid var(--border);
  position: sticky;
  bottom: 0;
  background: var(--surface);
  border-radius: 0 0 32px 32px;
}

.footer-actions {
  display: flex;
  gap: 15px;
}

.btn-protocol-cancel {
  flex: 0.3;
  padding: 18px;
  background: var(--panel);
  color: var(--muted);
  border: 1px solid var(--border);
  border-radius: 20px;
  font-weight: 900;
  cursor: pointer;
  font-size: 12px;
  letter-spacing: 1px;
  transition: all 0.2s ease;
}

.btn-protocol-cancel:hover {
  color: #ef4444;
  border-color: #ef4444;
  background: rgba(239, 68, 68, 0.1);
}

.btn-protocol-submit { 
  flex: 1;
  padding: 18px 24px; 
  background: var(--accent); 
  color: #fff; 
  border: none; 
  border-radius: 20px; 
  font-weight: 900; 
  cursor: pointer; 
  font-size: 15px; 
  letter-spacing: 1.5px; 
  box-shadow: 0 10px 30px rgba(0, 209, 255, 0.3);
  transition: all 0.2s ease;
}

.btn-protocol-submit:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 15px 40px rgba(0, 209, 255, 0.4);
}

.btn-protocol-submit:disabled {
  opacity: 0.4;
  cursor: not-allowed;
  filter: grayscale(0.5);
}

.form-error-summary {
  margin-top: 15px;
  text-align: center;
  color: #ef4444;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.5px;
}

/* RESPONSIVE */
@media (max-width: 768px) {
  .form-overlay {
    padding: 10px;
  }
  
  .protocol-header,
  .protocol-body,
  .protocol-footer {
    padding-left: 25px;
    padding-right: 25px;
  }
  
  .protocol-grid {
    grid-template-columns: 1fr;
    gap: 0;
  }
  
  .footer-actions {
    flex-direction: column;
  }
  
  .btn-protocol-cancel {
    order: 2;
  }
  
  .btn-protocol-submit {
    order: 1;
  }
}
</style>
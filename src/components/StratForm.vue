<template>
  <form class="strat-creator" @submit.prevent="submitForm">
    <h2>➕ Новая стратегия</h2>

    <input
      v-model="localStrat.name"
      placeholder="Название стратегии"
      class="input-field"
      maxlength="60"
    >

    <textarea
      v-model="localStrat.description"
      placeholder="Описание стратегии..."
      rows="3"
      class="input-field"
      maxlength="300"
    ></textarea>

    <div class="round-type">
      <p class="round-label">Тип раунда:</p>
      <div class="round-options">
        <label :class="{ active: localStrat.roundType === 'pistol' }">
          <input type="radio" value="pistol" v-model="localStrat.roundType">
          🔫 Пистолетный раунд
        </label>

        <label :class="{ active: localStrat.roundType === 'full' }">
          <input type="radio" value="full" v-model="localStrat.roundType">
          💰 Фулл бай
        </label>

        <label :class="{ active: localStrat.roundType === 'force' }">
          <input type="radio" value="force" v-model="localStrat.roundType">
          ⚡ Форс бай
        </label>

        <label :class="{ active: localStrat.roundType === 'eco' }">
          <input type="radio" value="eco" v-model="localStrat.roundType">
          🐭 Эко раунд
        </label>
      </div>
    </div>

    <p v-if="errorMessage" class="form-error">{{ errorMessage }}</p>

    <button type="submit" class="btn-primary">➕ Добавить стратегию</button>
  </form>
</template>

<script>
export default {
  name: 'StratForm',
  data() {
    return {
      localStrat: {
        name: '',
        description: '',
        roundType: 'full'
      },
      errorMessage: ''
    }
  },
  methods: {
    submitForm() {
      const name = this.localStrat.name.trim()
      const description = this.localStrat.description.trim()

      if (!name || !description) {
        this.errorMessage = 'Заполните название и описание стратегии!'
        return
      }

      this.errorMessage = ''

      this.$emit('add-strat', {
        name,
        description,
        roundType: this.localStrat.roundType
      })

      this.resetForm()
    },

    resetForm() {
      this.localStrat = {
        name: '',
        description: '',
        roundType: 'full'
      }
    }
  }
}
</script>

<style scoped>
.strat-creator {
  padding: 28px;
  background: 
    linear-gradient(180deg, rgba(27, 33, 48, 0.95), rgba(21, 25, 35, 0.98)),
    rgba(255, 255, 255, 0.02);
  border-bottom: 1px solid var(--line);
  backdrop-filter: blur(12px);
  border-radius: 0 0 24px 24px;
}

.strat-creator h2 {
  color: var(--text);
  margin-bottom: 20px;
  font-size: 1.4rem;
  font-weight: 800;
  letter-spacing: 0.5px;
}

.input-field {
  width: 100%;
  padding: 16px 18px;
  margin: 12px 0;
  border: 2px solid rgba(255, 255, 255, 0.08);
  border-radius: 16px;
  background: rgba(255, 255, 255, 0.025);
  color: var(--text);
  font-family: inherit;
  font-size: 16px;
  transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.input-field::placeholder {
  color: var(--muted);
}

.input-field:focus {
  outline: none;
  border-color: var(--accent);
  background: rgba(255, 255, 255, 0.06);
  box-shadow: 
    0 0 0 4px rgba(109, 124, 255, 0.2),
    0 8px 24px rgba(0, 0, 0, 0.25);
  transform: translateY(-1px);
}

.input-field.error {
  border-color: var(--danger);
  box-shadow: 0 0 0 4px rgba(239, 68, 68, 0.15);
}

.round-type {
  margin: 24px 0;
}

.round-label {
  font-weight: 700;
  margin-bottom: 16px;
  color: var(--text);
  font-size: 0.95rem;
  letter-spacing: 0.3px;
}

.round-options {
  display: flex;
  flex-wrap: wrap;
  gap: 14px;
}

.round-options label {
  display: flex;
  align-items: center;
  gap: 10px;
  cursor: pointer;
  padding: 14px 22px;
  background: rgba(255, 255, 255, 0.04);
  border: 2px solid rgba(255, 255, 255, 0.08);
  border-radius: 999px;
  color: var(--text);
  font-weight: 600;
  font-size: 15px;
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  position: relative;
  overflow: hidden;
}

.round-options label::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.1), transparent);
  transition: left 0.5s;
}

.round-options label:hover::before {
  left: 100%;
}

.round-options label:hover {
  transform: translateY(-3px) scale(1.02);
  border-color: rgba(109, 124, 255, 0.4);
  background: rgba(109, 124, 255, 0.08);
  box-shadow: 0 12px 28px rgba(109, 124, 255, 0.15);
}

.round-options label.active {
  background: linear-gradient(135deg, var(--accent) 0%, var(--accent-2) 100%);
  border-color: var(--accent);
  color: white;
  box-shadow: 
    0 16px 32px rgba(109, 124, 255, 0.3),
    0 0 0 1px rgba(109, 124, 255, 0.4);
  transform: translateY(-2px) scale(1.03);
}

.round-options input {
  accent-color: var(--accent);
  width: 18px;
  height: 18px;
  margin: 0;
}

.btn-primary {
  background: 
    linear-gradient(135deg, var(--accent) 0%, var(--accent-2) 100%),
    linear-gradient(135deg, rgba(255, 255, 255, 0.2), rgba(255, 255, 255, 0.1));
  color: white;
  border: none;
  padding: 16px 36px;
  border-radius: 16px;
  cursor: pointer;
  font-size: 16px;
  font-weight: 700;
  margin-top: 20px;
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  box-shadow: 
    0 12px 28px rgba(109, 124, 255, 0.3),
    0 4px 12px rgba(0, 0, 0, 0.2),
    inset 0 1px 0 rgba(255, 255, 255, 0.3);
  position: relative;
  overflow: hidden;
  min-height: 56px;
}

.btn-primary::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.3), transparent);
  transition: left 0.6s;
}

.btn-primary:hover::before {
  left: 100%;
}

.btn-primary:hover {
  transform: translateY(-4px);
  box-shadow: 
    0 20px 40px rgba(109, 124, 255, 0.4),
    0 8px 20px rgba(0, 0, 0, 0.3);
}

.btn-primary:active {
  transform: translateY(-2px);
}

.form-error {
  color: var(--danger);
  margin-top: 12px;
  font-size: 14px;
  padding: 14px 16px;
  background: rgba(239, 68, 68, 0.12);
  border: 1px solid rgba(239, 68, 68, 0.25);
  border-radius: 12px;
  border-left: 4px solid var(--danger);
  animation: shake 0.5s ease-in-out;
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-4px); }
  75% { transform: translateX(4px); }
}

@media (max-width: 640px) {
  .strat-creator {
    padding: 20px;
  }

  .round-options {
    flex-direction: column;
    gap: 10px;
  }

  .round-options label {
    padding: 16px 20px;
    justify-content: center;
  }
}
</style>
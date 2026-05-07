<template>
  <div class="strat-card" :class="roundTypeClass">
    <div class="strat-header">
      <h3>{{ strat.name }}</h3>
      <span class="strat-date">{{ formatDate(strat.date) }}</span>
    </div>

    <div class="round-badge" :class="roundTypeClass">
      <span class="round-icon">{{ roundIcon }}</span>
      <span class="round-text">{{ roundText }}</span>
    </div>

    <p class="strat-description">{{ strat.description }}</p>

    <div class="strat-footer">
      <div class="strat-actions">
        <button type="button" @click="openEditModal" class="btn-edit" title="Редактировать">✏️</button>
        <button type="button" @click="$emit('delete')" class="btn-delete" title="Удалить">🗑️</button>
      </div>
    </div>

    <div v-if="showModal" class="modal" @click.self="closeModal">
      <div class="modal-content">
        <div class="modal-header">
          <h3>Редактировать стратегию</h3>
          <button type="button" @click="closeModal" class="modal-close">✕</button>
        </div>

        <form class="modal-body" @submit.prevent="saveChanges">
          <label>Название:</label>
          <input v-model="editName" class="edit-input" placeholder="Название стратегии" maxlength="60">

          <label>Описание:</label>
          <textarea
            v-model="editDescription"
            rows="4"
            class="edit-input"
            placeholder="Описание стратегии"
            maxlength="300"
          ></textarea>

          <label>Тип раунда:</label>
          <div class="modal-round-options">
            <label :class="{ active: editRoundType === 'pistol' }">
              <input type="radio" value="pistol" v-model="editRoundType"> 🔫 Пистолетный
            </label>

            <label :class="{ active: editRoundType === 'full' }">
              <input type="radio" value="full" v-model="editRoundType"> 💰 Фулл бай
            </label>

            <label :class="{ active: editRoundType === 'force' }">
              <input type="radio" value="force" v-model="editRoundType"> ⚡ Форс бай
            </label>

            <label :class="{ active: editRoundType === 'eco' }">
              <input type="radio" value="eco" v-model="editRoundType"> 🐭 Эко
            </label>
          </div>

          <p v-if="errorMessage" class="modal-error">{{ errorMessage }}</p>

          <div class="modal-footer">
            <button type="button" @click="closeModal" class="btn-cancel">Отмена</button>
            <button type="submit" class="btn-save">Сохранить</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'StratCard',
  props: {
    strat: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      showModal: false,
      editName: '',
      editDescription: '',
      editRoundType: 'full',
      errorMessage: ''
    }
  },
  computed: {
    roundTypeClass() {
      const types = {
        pistol: 'round-pistol',
        full: 'round-full',
        force: 'round-force',
        eco: 'round-eco'
      }
      return types[this.strat.roundType] || 'round-full'
    },
    roundIcon() {
      const icons = {
        pistol: '🔫',
        full: '💰',
        force: '⚡',
        eco: '🐭'
      }
      return icons[this.strat.roundType] || '💰'
    },
    roundText() {
      const texts = {
        pistol: 'Пистолетный раунд',
        full: 'Фулл бай',
        force: 'Форс бай',
        eco: 'Эко раунд'
      }
      return texts[this.strat.roundType] || 'Фулл бай'
    }
  },
  methods: {
    openEditModal() {
      this.editName = this.strat.name
      this.editDescription = this.strat.description
      this.editRoundType = this.strat.roundType || 'full'
      this.errorMessage = ''
      this.showModal = true
    },

    closeModal() {
      this.showModal = false
      this.errorMessage = ''
    },

    saveChanges() {
      const name = this.editName.trim()
      const description = this.editDescription.trim()

      if (!name || !description) {
        this.errorMessage = 'Заполните все поля!'
        return
      }

      this.$emit('edit', {
        name,
        description,
        roundType: this.editRoundType
      })

      this.closeModal()
    },

    formatDate(dateString) {
      if (!dateString) return 'Без даты'
      const date = new Date(dateString)
      if (Number.isNaN(date.getTime())) return 'Некорректная дата'
      return date.toLocaleDateString('ru-RU')
    }
  }
}
</script>

<style scoped>
.modal {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.85); /* Тёмный overlay без blur */
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  padding: 24px;
}

.modal-content {
  background: var(--panel-2); /* Чёткий тёмно-серый фон */
  border: 1px solid var(--line);
  border-radius: 20px;
  width: 100%;
  max-width: 520px;
  max-height: 90vh;
  overflow: hidden;
  box-shadow: 
    0 24px 70px rgba(0, 0, 0, 0.6),
    0 0 0 1px rgba(255, 255, 255, 0.05); /* Двойная тень для объёма */
  animation: fadeInScale 0.25s cubic-bezier(0.34, 1.56, 0.64, 1);
}

@keyframes fadeInScale {
  from { 
    opacity: 0; 
    transform: scale(0.95) translateY(-20px); 
  }
  to { 
    opacity: 1; 
    transform: scale(1) translateY(0); 
  }
}

.modal-header {
  background: rgba(17, 24, 39, 0.95); /* Более тёмный градиент */
  color: var(--text);
  padding: 20px 24px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid var(--line);
}

.modal-header h3 {
  margin: 0;
  font-size: 1.1rem;
  font-weight: 700;
}

.modal-close {
  background: transparent;
  border: none;
  color: var(--muted);
  font-size: 24px;
  width: 36px;
  height: 36px;
  border-radius: 50%;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
}

.modal-close:hover {
  background: rgba(255, 255, 255, 0.1);
  color: var(--text);
  transform: rotate(90deg) scale(1.05);
}

.modal-close:focus {
  outline: 2px solid var(--accent);
  outline-offset: 2px;
}

.modal-body {
  padding: 24px;
  max-height: 400px;
  overflow-y: auto;
}

.modal-body label {
  display: block;
  font-weight: 700;
  margin-top: 18px;
  margin-bottom: 8px;
  color: var(--text);
  font-size: 0.95rem;
}

.modal-body label:first-child {
  margin-top: 0;
}

.edit-input {
  width: 100%;
  padding: 14px 16px;
  border: 2px solid rgba(255, 255, 255, 0.08);
  border-radius: 14px;
  background: rgba(255, 255, 255, 0.02);
  color: var(--text);
  font-family: inherit;
  font-size: 15px;
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
}

.edit-input::placeholder {
  color: var(--muted);
}

.edit-input:focus {
  outline: none;
  border-color: var(--accent);
  background: rgba(255, 255, 255, 0.05);
  box-shadow: 
    0 0 0 4px rgba(109, 124, 255, 0.15),
    0 4px 12px rgba(0, 0, 0, 0.2);
}

.modal-round-options {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  margin-top: 8px;
}

.modal-round-options label {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  padding: 12px 18px;
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.06);
  border-radius: 12px;
  color: var(--text);
  font-weight: 500;
  font-size: 14px;
  transition: all 0.2s ease;
  white-space: nowrap;
}

.modal-round-options label:hover {
  background: rgba(109, 124, 255, 0.1);
  border-color: rgba(109, 124, 255, 0.3);
  transform: translateY(-1px);
}

.modal-round-options label.active {
  background: linear-gradient(135deg, var(--accent) 0%, var(--accent-2) 100%);
  border-color: var(--accent);
  color: white;
  box-shadow: 0 8px 20px rgba(109, 124, 255, 0.25);
}

.modal-round-options input {
  accent-color: var(--accent);
  margin-right: 8px;
}

.modal-footer {
  padding: 20px 24px 24px;
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  border-top: 1px solid var(--line);
  background: rgba(255, 255, 255, 0.02);
}

.btn-cancel,
.btn-save {
  padding: 12px 24px;
  border: none;
  border-radius: 12px;
  cursor: pointer;
  font-size: 15px;
  font-weight: 600;
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
  min-height: 48px;
}

.btn-cancel {
  background: rgba(255, 255, 255, 0.08);
  color: var(--text);
  border: 1px solid rgba(255, 255, 255, 0.12);
}

.btn-cancel:hover {
  background: rgba(255, 255, 255, 0.12);
  transform: translateY(-1px);
}

.btn-save {
  background: linear-gradient(135deg, var(--accent) 0%, var(--accent-2) 100%);
  color: white;
  box-shadow: 0 8px 20px rgba(109, 124, 255, 0.3);
}

.btn-save:hover {
  transform: translateY(-2px);
  box-shadow: 0 12px 28px rgba(109, 124, 255, 0.4);
}

.modal-error {
  color: var(--danger);
  margin-top: 12px;
  font-size: 14px;
  padding: 12px;
  background: rgba(239, 68, 68, 0.1);
  border: 1px solid rgba(239, 68, 68, 0.2);
  border-radius: 8px;
}

@media (max-width: 640px) {
  .modal-footer {
    flex-direction: column;
  }

  .btn-cancel,
  .btn-save {
    width: 100%;
  }

  .modal-round-options {
    flex-direction: column;
  }
}
</style>
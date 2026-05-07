<template>
  <div class="strat-card">
    <div class="strat-header">
      <h3>{{ strat.name }}</h3>
      <span class="strat-date">{{ formatDate(strat.date) }}</span>
    </div>
    <p class="strat-description">{{ strat.description }}</p>
    <div class="strat-footer">
      <div class="role-badges">
        <span v-for="(value, role) in strat.roles" :key="role" 
              v-if="value" class="role-badge">
          {{ role }}
        </span>
      </div>
      <div class="strat-actions">
        <button @click="$emit('edit')" class="btn-edit">✏️</button>
        <button @click="$emit('delete')" class="btn-delete">🗑️</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'StratCard',
  props: ['strat'],
  methods: {
    formatDate(dateString) {
      const date = new Date(dateString)
      return date.toLocaleDateString('ru-RU')
    }
  }
}
</script>

<style scoped>
.strat-card {
  background: #f8f9fa;
  border-radius: 12px;
  padding: 20px;
  margin-bottom: 15px;
  transition: all 0.3s ease;
  border-left: 4px solid #667eea;
}

.strat-card:hover {
  transform: translateX(5px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}

.strat-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.strat-header h3 {
  color: #333;
  font-size: 1.3em;
}

.strat-date {
  color: #999;
  font-size: 0.9em;
}

.strat-description {
  color: #555;
  line-height: 1.6;
  margin-bottom: 15px;
}

.strat-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 15px;
}

.role-badges {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.role-badge {
  background: #667eea;
  color: white;
  padding: 4px 12px;
  border-radius: 12px;
  font-size: 0.85em;
  font-weight: bold;
}

.strat-actions {
  display: flex;
  gap: 10px;
}

.btn-edit, .btn-delete {
  background: white;
  border: none;
  font-size: 1.2em;
  cursor: pointer;
  padding: 5px 10px;
  border-radius: 6px;
}

.btn-edit:hover {
  background: #4caf50;
  color: white;
}

.btn-delete:hover {
  background: #f44336;
  color: white;
}
</style>
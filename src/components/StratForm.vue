<template>
  <div class="strat-creator">
    <h2>➕ Новая стратегия</h2>
    <input 
      v-model="localStrat.name" 
      placeholder="Название стратегии"
      class="input-field"
    >
    <textarea 
      v-model="localStrat.description" 
      placeholder="Описание стратегии..."
      rows="3"
      class="input-field"
    ></textarea>
    <div class="role-tags">
      <label><input type="checkbox" v-model="localStrat.roles.AWP"> AWP</label>
      <label><input type="checkbox" v-model="localStrat.roles.Rifler"> Rifler</label>
      <label><input type="checkbox" v-model="localStrat.roles.Support"> Support</label>
      <label><input type="checkbox" v-model="localStrat.roles.Lurker"> Lurker</label>
    </div>
    <button @click="submitForm" class="btn-primary">➕ Добавить стратегию</button>
  </div>
</template>

<script>
export default {
  name: 'StratForm',
  data() {
    return {
      localStrat: {
        name: '',
        description: '',
        roles: { AWP: false, Rifler: false, Support: false, Lurker: false }
      }
    }
  },
  methods: {
    submitForm() {
      if (!this.localStrat.name || !this.localStrat.description) {
        alert('Заполните название и описание стратегии!')
        return
      }
      this.$emit('add-strat', { ...this.localStrat })
      this.resetForm()
    },
    resetForm() {
      this.localStrat = {
        name: '',
        description: '',
        roles: { AWP: false, Rifler: false, Support: false, Lurker: false }
      }
    }
  }
}
</script>

<style scoped>
.strat-creator {
  padding: 20px;
  background: #f8f9fa;
  border-bottom: 1px solid #e0e0e0;
}

.strat-creator h2 {
  color: #333;
  margin-bottom: 15px;
}

.input-field {
  width: 100%;
  padding: 12px;
  margin: 10px 0;
  border: 2px solid #ddd;
  border-radius: 8px;
  font-family: inherit;
  font-size: 16px;
}

.input-field:focus {
  outline: none;
  border-color: #667eea;
}

.role-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
  margin: 15px 0;
}

.role-tags label {
  display: flex;
  align-items: center;
  gap: 5px;
  cursor: pointer;
  padding: 8px 15px;
  background: white;
  border-radius: 20px;
  border: 1px solid #ddd;
}

.btn-primary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  padding: 12px 30px;
  border-radius: 25px;
  cursor: pointer;
  font-size: 16px;
  font-weight: bold;
}

.btn-primary:hover {
  transform: translateY(-2px);
}
</style>
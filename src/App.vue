<template>
  <div id="app">
    <div class="container">
      <header class="header">
        <h1>🎯 CS2 STRATBOOK</h1>
        <p>База стратегий для Counter-Strike 2</p>
      </header>

      <MapSelector v-model="selectedMap" />

      <StratForm @add-strat="addStrat" />

      <div class="strats-list">
        <h2>📋 Стратегии для {{ selectedMap }}</h2>
        <div class="stats">📊 Всего: {{ filteredStrats.length }}</div>
        
        <StratCard 
          v-for="strat in filteredStrats" 
          :key="strat.id"
          :strat="strat"
          @edit="editStrat(strat)"
          @delete="deleteStrat(strat.id)"
        />

        <div v-if="filteredStrats.length === 0" class="empty-state">
          <p>🎯 Нет стратегий для {{ selectedMap }}</p>
          <p>Добавьте первую стратегию!</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import MapSelector from './components/MapSelector.vue'
import StratForm from './components/StratForm.vue'
import StratCard from './components/StratCard.vue'

export default {
  name: 'App',
  components: { MapSelector, StratForm, StratCard },
  data() {
    return {
      selectedMap: 'Mirage',
      strats: []
    }
  },
  computed: {
    filteredStrats() {
      return this.strats.filter(strat => strat.map === this.selectedMap)
    }
  },
  mounted() {
    this.loadStrats()
  },
  methods: {
    addStrat(newStrat) {
      const strat = {
        id: Date.now(),
        ...newStrat,
        map: this.selectedMap,
        date: new Date().toISOString()
      }
      this.strats.unshift(strat)
      this.saveStrats()
    },
    deleteStrat(id) {
      if (confirm('Удалить стратегию?')) {
        this.strats = this.strats.filter(s => s.id !== id)
        this.saveStrats()
      }
    },
    editStrat(strat) {
      const newName = prompt('Новое название:', strat.name)
      if (newName && newName.trim()) {
        strat.name = newName
        this.saveStrats()
      }
    },
    saveStrats() {
      localStorage.setItem('cs2-stratbook', JSON.stringify(this.strats))
    },
    loadStrats() {
      const saved = localStorage.getItem('cs2-stratbook')
      if (saved) {
        this.strats = JSON.parse(saved)
      } else {
        this.strats = [
          {
            id: 1,
            name: 'A раш с флешками',
            description: 'Быстрый выход на A через аппартаменты. 2 флешки в сайт, молик на гулл.',
            roles: { AWP: true, Rifler: true, Support: false, Lurker: false },
            map: 'Mirage',
            date: new Date(Date.now() - 86400000).toISOString()
          },
          {
            id: 2,
            name: 'Медленный пик B',
            description: 'Занимаем контроль в тунелях. Вайпаем с помощью мелодрамы.',
            roles: { AWP: false, Rifler: true, Support: true, Lurker: false },
            map: 'Dust2',
            date: new Date(Date.now() - 172800000).toISOString()
          }
        ]
        this.saveStrats()
      }
    }
  }
}
</script>

<style>
* { margin: 0; padding: 0; box-sizing: border-box; }

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
}

#app { min-height: 100vh; padding: 20px; }

.container {
  max-width: 1200px;
  margin: 0 auto;
  background: white;
  border-radius: 20px;
  box-shadow: 0 20px 60px rgba(0,0,0,0.3);
  overflow: hidden;
}

.header {
  background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
  color: white;
  padding: 30px;
  text-align: center;
}

.header h1 { font-size: 2.5em; margin-bottom: 10px; }
.header p { font-size: 1.2em; opacity: 0.9; }

.strats-list { padding: 20px; }
.strats-list h2 { color: #333; margin-bottom: 10px; }

.stats {
  color: #666;
  margin-bottom: 20px;
  padding-bottom: 10px;
  border-bottom: 2px solid #f0f0f0;
}

.empty-state {
  text-align: center;
  padding: 60px 20px;
  color: #999;
}

.empty-state p { margin: 10px 0; font-size: 1.1em; }

@media (max-width: 768px) {
  .header h1 { font-size: 1.8em; }
  .strat-header { flex-direction: column; align-items: flex-start; gap: 8px; }
  .strat-footer { flex-direction: column; align-items: flex-start; }
}
</style>
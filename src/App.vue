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
          @edit="(updatedData) => editStrat(strat.id, updatedData)"
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

const STORAGE_KEY = 'cs2-stratbook'

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
        id: crypto.randomUUID(),
        name: newStrat.name.trim(),
        description: newStrat.description.trim(),
        roundType: newStrat.roundType || 'full',
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

    editStrat(id, updatedData) {
      this.strats = this.strats.map(strat =>
        strat.id === id
          ? {
              ...strat,
              name: updatedData.name.trim(),
              description: updatedData.description.trim(),
              roundType: updatedData.roundType
            }
          : strat
      )
      this.saveStrats()
    },

    saveStrats() {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(this.strats))
    },

    loadStrats() {
      const saved = localStorage.getItem(STORAGE_KEY)

      if (saved) {
        try {
          this.strats = JSON.parse(saved)
        } catch (error) {
          this.strats = []
          localStorage.removeItem(STORAGE_KEY)
        }
        return
      }

      this.strats = [
        {
          id: crypto.randomUUID(),
          name: 'Дефолт на A',
          description: 'Стандартный выход на A. Два игрока контролят пространство, AWP смотрит мид. Смоки на сайт и на ключевые позиции.',
          roundType: 'full',
          map: 'Mirage',
          date: new Date(Date.now() - 86400000).toISOString()
        },
        {
          id: crypto.randomUUID(),
          name: 'Пистолетный раш B',
          description: 'Быстрый выход на B с плотным разменом. Один игрок идёт первым, остальные страхуют и занимают плент.',
          roundType: 'pistol',
          map: 'Dust2',
          date: new Date(Date.now() - 172800000).toISOString()
        },
        {
          id: crypto.randomUUID(),
          name: 'Форс бай через аппартаменты',
          description: 'Покупаем форс-раунд и идём через аппартаменты на A. Один игрок остаётся в миду для информации и позднего выхода.',
          roundType: 'force',
          map: 'Mirage',
          date: new Date(Date.now() - 259200000).toISOString()
        },
        {
          id: crypto.randomUUID(),
          name: 'Эко лурк B',
          description: 'Технический эко. Один игрок собирает информацию и пробует найти килл, остальные играют от тайминга и сохранения экономики.',
          roundType: 'eco',
          map: 'Dust2',
          date: new Date(Date.now() - 345600000).toISOString()
        }
      ]

      this.saveStrats()
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  --bg: #0f1115;
  --panel: #151923;
  --panel-2: #1b2130;
  --text: #e8edf7;
  --muted: #9aa6bf;
  --line: rgba(255, 255, 255, 0.08);
  --accent: #6d7cff;
  --accent-2: #8a5cff;
  --success: #31c48d;
  --warning: #f59e0b;
  --danger: #ef4444;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background:
    radial-gradient(circle at top, rgba(109, 124, 255, 0.18), transparent 35%),
    radial-gradient(circle at bottom right, rgba(138, 92, 255, 0.14), transparent 30%),
    linear-gradient(135deg, #0b0e14 0%, #111827 55%, #0b0e14 100%);
  color: var(--text);
  min-height: 100vh;
}

#app {
  min-height: 100vh;
  padding: 24px;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  background: rgba(21, 25, 35, 0.92);
  border: 1px solid var(--line);
  border-radius: 24px;
  box-shadow: 0 24px 80px rgba(0, 0, 0, 0.45);
  overflow: hidden;
  backdrop-filter: blur(10px);
}

.header {
  background:
    linear-gradient(135deg, rgba(31, 41, 55, 0.95), rgba(17, 24, 39, 0.95));
  color: var(--text);
  padding: 36px 28px;
  text-align: center;
  border-bottom: 1px solid var(--line);
}

.header h1 {
  font-size: clamp(2rem, 4vw, 3rem);
  margin-bottom: 10px;
  letter-spacing: 0.5px;
}

.header p {
  font-size: 1.05rem;
  color: var(--muted);
}

.strats-list {
  padding: 24px;
}

.strats-list h2 {
  color: var(--text);
  margin-bottom: 10px;
  font-size: 1.35rem;
}

.stats {
  color: var(--muted);
  margin-bottom: 20px;
  padding-bottom: 12px;
  border-bottom: 1px solid var(--line);
}

.empty-state {
  text-align: center;
  padding: 72px 20px;
  color: var(--muted);
  border: 1px dashed var(--line);
  border-radius: 18px;
  background: rgba(255, 255, 255, 0.02);
}

.empty-state p {
  margin: 10px 0;
  font-size: 1rem;
}

@media (max-width: 768px) {
  #app {
    padding: 12px;
  }

  .header {
    padding: 24px 18px;
  }

  .strats-list {
    padding: 16px;
  }
}
</style>
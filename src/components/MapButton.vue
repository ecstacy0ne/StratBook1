<template>
  <button 
    class="map-entry-btn" 
    :class="{ active: isActive }" 
    @click="handleSelect"
    :aria-label="`Select map ${map}`"
    :title="isActive ? `Active: ${map}` : `Select ${map}`"
  >
    <div class="map-initial" aria-hidden="true">{{ mapInitial }}</div>
    <div class="map-info">
      <span class="map-designation">{{ map }}</span>
      <span v-if="isActive" class="map-status">ACTIVE SECTOR</span>
    </div>
    <div v-if="isActive" class="active-glow" aria-hidden="true"></div>
    <div v-if="isActive" class="active-indicator">●</div>
  </button>
</template>

<script>
export default {
  name: 'MapButton',
  
  props: { 
    map: { 
      type: String, 
      required: true,
      validator(value) {
        return value && value.trim().length > 0;
      }
    },
    isActive: { 
      type: Boolean, 
      default: false 
    } 
  },
  
  emits: ['select'],
  
  computed: {
    mapInitial() {
      if (!this.map) return '?';
      const words = this.map.trim().split(/\s+/);
      if (words.length === 1) {
        return this.map.charAt(0).toUpperCase();
      }
      return words.map(word => word.charAt(0).toUpperCase()).join('');
    }
  },
  
  methods: {
    handleSelect() {
      if (!this.isActive) {
        this.$emit('select', this.map);
      }
    }
  }
}
</script>

<style scoped>
.map-entry-btn { 
  width: 100%; 
  display: flex; 
  align-items: center; 
  padding: 15px 20px; 
  background: transparent; 
  border: 2px solid transparent; 
  cursor: pointer; 
  border-radius: 14px; 
  margin-bottom: 8px; 
  transition: all 0.25s ease; 
  position: relative; 
  overflow: hidden;
  text-align: left;
}

.map-entry-btn::before {
  content: '';
  position: absolute;
  left: 0;
  top: 0;
  bottom: 0;
  width: 0;
  background: linear-gradient(90deg, 
    rgba(0, 209, 255, 0.05) 0%, 
    transparent 100%);
  transition: width 0.3s ease;
}

.map-entry-btn:hover::before {
  width: 100%;
}

.map-initial { 
  width: 40px; 
  height: 40px; 
  background: var(--panel); 
  border: 2px solid var(--border); 
  border-radius: 12px; 
  display: flex; 
  align-items: center; 
  justify-content: center; 
  font-weight: 900; 
  font-size: 16px; 
  margin-right: 18px; 
  color: var(--muted); 
  transition: all 0.3s ease;
  flex-shrink: 0;
  position: relative;
  z-index: 1;
}

.map-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 4px;
  position: relative;
  z-index: 1;
}

.map-designation { 
  font-size: 15px; 
  font-weight: 700; 
  color: var(--muted); 
  text-transform: uppercase; 
  letter-spacing: 0.5px; 
  transition: all 0.2s ease;
  line-height: 1.2;
}

.map-status {
  font-size: 9px;
  font-weight: 900;
  color: var(--accent);
  letter-spacing: 0.8px;
  text-transform: uppercase;
  opacity: 0.8;
}

.map-entry-btn:hover { 
  background: var(--panel); 
  border-color: var(--border);
}

.map-entry-btn:hover .map-initial {
  border-color: var(--accent);
  color: var(--text);
}

.map-entry-btn:hover .map-designation {
  color: var(--text);
}

/* Активное состояние */
.map-entry-btn.active { 
  background: rgba(0, 209, 255, 0.05);
  border-color: rgba(0, 209, 255, 0.15);
}

.map-entry-btn.active .map-initial { 
  background: var(--accent); 
  color: #fff; 
  border-color: var(--accent);
  box-shadow: 0 0 20px rgba(0, 209, 255, 0.3);
  transform: scale(1.05);
}

.map-entry-btn.active .map-designation { 
  color: var(--text); 
  font-weight: 800;
}

/* Индикатор активной карты */
.active-indicator {
  position: absolute;
  right: 20px;
  top: 50%;
  transform: translateY(-50%);
  color: var(--accent);
  font-size: 8px;
  opacity: 0.7;
  animation: pulse-indicator 2s infinite;
  z-index: 1;
}

@keyframes pulse-indicator {
  0%, 100% { 
    opacity: 0.7; 
  }
  50% { 
    opacity: 0.3; 
  }
}

/* Светящаяся полоса */
.active-glow { 
  position: absolute; 
  left: 0; 
  top: 20%; 
  bottom: 20%; 
  width: 3px; 
  background: var(--accent); 
  border-radius: 0 3px 3px 0; 
  box-shadow: 
    0 0 10px var(--accent),
    0 0 20px rgba(0, 209, 255, 0.5),
    0 0 40px rgba(0, 209, 255, 0.3);
  animation: glow-pulse 2s infinite;
}

@keyframes glow-pulse {
  0%, 100% { 
    box-shadow: 
      0 0 10px var(--accent),
      0 0 20px rgba(0, 209, 255, 0.5),
      0 0 40px rgba(0, 209, 255, 0.3);
  }
  50% { 
    box-shadow: 
      0 0 15px var(--accent),
      0 0 30px rgba(0, 209, 255, 0.7),
      0 0 50px rgba(0, 209, 255, 0.4);
  }
}

/* Состояние фокуса для доступности */
.map-entry-btn:focus-visible {
  outline: 2px solid var(--accent);
  outline-offset: 2px;
}

/* Анимация при нажатии */
.map-entry-btn:active {
  transform: scale(0.98);
}

/* Адаптивность */
@media (max-width: 1200px) {
  .map-entry-btn {
    padding: 12px 15px;
  }
  
  .map-initial {
    width: 35px;
    height: 35px;
    font-size: 14px;
  }
  
  .map-designation {
    font-size: 13px;
  }
}
</style>
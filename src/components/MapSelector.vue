<template>
  <div class="map-selector">
    <button
      v-for="map in maps"
      :key="map"
      type="button"
      :class="{ active: modelValue === map }"
      :aria-pressed="modelValue === map"
      @click="$emit('update:modelValue', map)"
      class="map-btn"
    >
      {{ map }}
    </button>
  </div>
</template>

<script>
export default {
  name: 'MapSelector',
  props: {
    modelValue: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      maps: ['Mirage', 'Dust2', 'Inferno', 'Nuke', 'Overpass', 'Ancient']
    }
  }
}
</script>

<style scoped>
.map-selector {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  padding: 28px;
  background: 
    linear-gradient(180deg, rgba(27, 33, 48, 0.95), rgba(21, 25, 35, 0.98)),
    rgba(255, 255, 255, 0.015);
  border-bottom: 1px solid var(--line);
  backdrop-filter: blur(16px);
  border-radius: 0 0 24px 24px;
}

.map-btn {
  padding: 14px 28px;
  border: 2px solid rgba(255, 255, 255, 0.12);
  background: rgba(255, 255, 255, 0.03);
  border-radius: 999px;
  cursor: pointer;
  font-weight: 700;
  font-size: 15px;
  color: var(--text);
  letter-spacing: 0.3px;
  text-transform: uppercase;
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  position: relative;
  overflow: hidden;
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.2);
  min-height: 52px;
  min-width: 120px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.map-btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, 
    transparent, 
    rgba(255, 255, 255, 0.15), 
    transparent
  );
  transition: left 0.6s ease;
  z-index: 1;
}

.map-btn:hover::before {
  left: 100%;
}

.map-btn:hover {
  transform: translateY(-4px) scale(1.02);
  border-color: rgba(109, 124, 255, 0.5);
  background: rgba(109, 124, 255, 0.12);
  box-shadow: 
    0 16px 32px rgba(109, 124, 255, 0.25),
    0 8px 20px rgba(0, 0, 0, 0.3);
  color: var(--text);
}

.map-btn:focus {
  outline: none;
  box-shadow: 
    0 0 0 4px rgba(109, 124, 255, 0.3),
    0 16px 32px rgba(109, 124, 255, 0.25);
}

.map-btn.active {
  background: 
    linear-gradient(135deg, var(--accent) 0%, var(--accent-2) 100%);
  border-color: var(--accent);
  color: white;
  box-shadow: 
    0 0 0 1px rgba(109, 124, 255, 0.5),
    0 20px 40px rgba(109, 124, 255, 0.35),
    inset 0 1px 0 rgba(255, 255, 255, 0.2);
  transform: translateY(-2px) scale(1.03);
}

.map-btn.active::after {
  content: '';
  position: absolute;
  inset: 0;
  background: 
    radial-gradient(circle at center, 
      rgba(255, 255, 255, 0.25) 0%, 
      transparent 60%
    );
  animation: pulse-glow 2s infinite;
}

@keyframes pulse-glow {
  0%, 100% { 
    opacity: 1; 
    transform: scale(1); 
  }
  50% { 
    opacity: 0.6; 
    transform: scale(1.1); 
  }
}

.map-btn[aria-pressed="true"] {
  background: 
    linear-gradient(135deg, var(--accent) 0%, var(--accent-2) 100%);
  border-color: var(--accent);
}

/* Мобильная адаптация */
@media (max-width: 768px) {
  .map-selector {
    padding: 20px 16px;
    gap: 10px;
    justify-content: center;
  }

  .map-btn {
    padding: 16px 24px;
    font-size: 14px;
    min-width: 100px;
    flex: 1;
    max-width: 140px;
  }
}

@media (max-width: 480px) {
  .map-selector {
    padding: 18px 12px;
  }

  .map-btn {
    padding: 14px 20px;
    font-size: 13px;
    min-width: unset;
    flex: 1 1 45%;
  }
}
</style>
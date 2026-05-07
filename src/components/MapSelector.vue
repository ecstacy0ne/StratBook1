<template>
  <div class="map-selector">
    <button 
      v-for="map in maps" 
      :key="map"
      :class="{ active: modelValue === map }"
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
  props: ['modelValue'],
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
  gap: 10px;
  padding: 20px;
  background: #f8f9fa;
  border-bottom: 1px solid #e0e0e0;
}

.map-btn {
  padding: 10px 20px;
  border: 2px solid #667eea;
  background: white;
  border-radius: 25px;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.3s ease;
}

.map-btn:hover {
  transform: translateY(-2px);
  background: #f0f0ff;
}

.map-btn.active {
  background: #667eea;
  color: white;
  border-color: #667eea;
}
</style>
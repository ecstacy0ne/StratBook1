<template>
  <div id="app" :class="theme">
    <aside class="sidebar">
      <div class="sidebar-inner">
        <div class="logo-section">
          <div class="hex-logo">SB</div>
          <div class="brand-info">
            <span class="brand-name">STRATBOOK</span>
            <span class="brand-tag">TACTICAL INTERFACE</span>
          </div>
        </div>
        
        <nav class="map-registry">
  <div class="registry-header">
    <span>SECTOR REGISTRY</span>
    <button @click="view = 'settings'" class="btn-settings" title="Configuration">⚙️</button>
  </div>
  
  <!-- Кнопка Избранного -->
  <button 
    class="btn-favorites-nav" 
    :class="{ active: favoriteMode }"
    @click="favoriteMode = !favoriteMode; view = 'library'"
  >
    <span class="fav-icon">⭐</span>
    <span class="fav-label">FAVORITES</span>
    <span class="fav-count">{{ favoriteStrats.length }}</span>
  </button>

  <div class="map-list-area">
    <MapButton 
      v-for="m in maps" 
      :key="m" 
      :map="m" 
      :is-active="activeMap === m" 
      @select="changeMap" 
    />
  </div>
</nav>



        <button :disabled="!activeMap || maps.length === 0" class="btn-deploy-trigger" @click="view = 'creator'">
          DEPLOY STRATEGY
        </button>
      </div>
    </aside>

    <main class="viewport">
      <header class="top-nav">
        <div class="nav-left">
          <div class="version-tag">SYSTEM v3.4 // READY</div>
        </div>
        
        <div class="nav-right">
          <button @click="toggleTheme" class="btn-toggle-hud">
            {{ theme === 'dark' ? '☀️ LIGHT HUD' : '🌙 DARK HUD' }}
          </button>
          <div class="status-badge">
            <span class="s-label">DATABASE UNITS:</span>
            <span class="s-value">{{ filteredStrats.length }}</span>
          </div>
        </div>
      </header>

      <section v-if="view === 'library'" class="main-view">
        <div class="content-header">
  <h2 class="active-sector-title">{{ activeMap || 'OFFLINE' }}</h2>
  
  <div class="header-right">
  <!-- Кнопка поиска -->
  <button 
    class="btn-search-toggle"
    @click="showSearch = !showSearch"
    :class="{ active: showSearch }"
  >
    🔍
  </button>

  <div v-if="showSearch" class="search-input-wrapper">
    <input 
      v-model="searchQuery" 
      placeholder="SEARCH TACTICAL DATA..."
      class="hud-search-input"
      autofocus
    >
    <button v-if="searchQuery" class="search-clear-btn" @click="searchQuery = ''">✕</button>
  </div>

   <!-- Фильтры -->
   <div class="filter-panel">
     <!-- Side -->
     <div class="filter-group">
       <button v-for="s in ['all', 't', 'ct']" :key="s" 
         @click="fSide = s" 
         :class="['f-pill', s, { active: fSide === s }]">
         {{ s.toUpperCase() }}
       </button>
     </div>
 
     <!-- Site -->
     <div class="filter-group">
       <button v-for="site in ['all', 'A', 'B', 'Mid']" :key="site" 
         @click="fSite = site" 
         :class="['f-pill', { active: fSite === site }]">
         {{ site === 'all' ? 'ANY SITE' : site }}
       </button>
     </div>
 
     <!-- Round Type -->
     <div class="filter-group round-filter">
       <button 
         v-for="type in roundTypes" 
         :key="type.value"
         @click="roundFilter = type.value"
         :class="['f-pill', 'round', type.value, { active: roundFilter === type.value }]"
       >
         {{ type.icon }} {{ type.label }}
       </button>
     </div>
   </div>
 </div>
</div>

        <div v-if="filteredStrats.length === 0" class="empty-state-hud">
          <div class="hud-circle-icon">
            <div class="inner-pulse"></div>
            <span class="icon-char">!</span>
          </div>
          <h3>TACTICAL DATA STREAM: NULL</h3>
          <p>Sector {{ activeMap }} has no registered tactical deployments. Initialize manual entry.</p>
          <button v-if="activeMap" @click="view = 'creator'" class="btn-init-strat">
            INITIATE DEPLOYMENT
          </button>
        </div>
        
        <div v-else class="tactical-grid">
          <StratCard 
            v-for="item in filteredStrats" 
            :key="item.id" 
            :strat="item" 
            @delete="deleteStrat(item.id)" 
            @toggle-favorite="toggleFavorite"
          />
        </div>
      </section>

      <section v-if="view === 'settings'" class="main-view">
        <div class="content-header">
          <h2>MAP REGISTRY CONFIGURATION</h2>
          <button @click="view = 'library'" class="btn-exit-settings">EXIT TO HUD</button>
        </div>
        
        <div class="settings-container">
          <div class="settings-box add-sector">
            <span class="box-label">REGISTER NEW SECTOR</span>
            <div class="hud-input-group">
              <input v-model="newMapInput" @keyup.enter="addMap" placeholder="DESIGNATION..." class="hud-input-main">
              <button @click="addMap" class="btn-hud-confirm">ADD</button>
            </div>
          </div>

          <div class="settings-box registry-list-box">
            <span class="box-label">ACTIVE POOL REGISTRY</span>
            <div class="registry-scroller">
              <div v-for="m in maps" :key="m" class="registry-entry">
                <span class="entry-name">{{ m }}</span>
                <button @click="removeMap(m)" class="btn-entry-remove">TERMINATE</button>
              </div>
            </div>
            <button @click="resetToDefault" class="btn-hud-reset">RESTORE OFFICIAL ACTIVE DUTY</button>
          </div>
        </div>
      </section>

      <StratForm 
        v-if="view === 'creator'" 
        :active-map="activeMap" 
        @submit="saveStrat" 
        @close="view = 'library'" 
      />
    </main>
  </div>
  

</template>

<script>
import StratCard from './components/StratCard.vue'
import StratForm from './components/StratForm.vue'
import MapButton from './components/MapButton.vue'

const MAPS_KEY = 'cs_tactical_maps';
const DATA_KEY = 'cs_tactical_data';
const THEME_KEY = 'cs_tactical_theme';
const DEFAULT_MAPS = ['Mirage', 'Dust2', 'Inferno', 'Nuke', 'Anubis', 'Ancient', 'Overpass'];

export default {
  components: { StratCard, StratForm, MapButton },
  data() {
    const savedMaps = JSON.parse(localStorage.getItem(MAPS_KEY)) || DEFAULT_MAPS;
    return {
      view: 'library',
      theme: localStorage.getItem(THEME_KEY) || 'dark',
      maps: savedMaps,
      activeMap: savedMaps[0] || '',
      strats: JSON.parse(localStorage.getItem(DATA_KEY)) || [],
      fSide: 'all',
      fSite: 'all',
      favoriteMode: false,
      showSearch: false,
      searchQuery: '',
      roundFilter: 'all',
      roundTypes: [
  { value: 'all',    label: 'ALL',    icon: '🔄' },
  { value: 'pistol', label: 'PISTOL', icon: '🔫' },
  { value: 'eco',    label: 'ECO',    icon: '🐭' },
  { value: 'force',  label: 'FORCE',  icon: '⚡' },
  { value: 'full',   label: 'FULL',   icon: '💰' }
],
      newMapInput: ''
    }
  },
  computed: {
  filteredStrats() {
    let result = this.strats.filter(s => {
    const matchMap = s.map === this.activeMap;
    const matchSide = this.fSide === 'all' || s.side === this.fSide;
    const matchSite = this.fSite === 'all' || (s.site || 'Any') === this.fSite;
    const matchRound = this.roundFilter === 'all' || s.roundType === this.roundFilter;
    
    const matchSearch = !this.searchQuery || 
      (s.name && s.name.toLowerCase().includes(this.searchQuery.toLowerCase())) ||
      (s.description && s.description.toLowerCase().includes(this.searchQuery.toLowerCase()));
    
    return matchMap && matchSide && matchSite && matchRound && matchSearch;
  });

  if (this.favoriteMode) {
    result = result.filter(s => s.favorite);
  }
  return result;
},

  favoriteStrats() {
    return this.strats.filter(s => s.favorite === true);
    }
  },
  watch: {
    theme: {
      immediate: true,
      handler(val) {
        document.body.style.backgroundColor = val === 'dark' ? '#080a0f' : '#f1f3f6';
      }
    }
  },
  methods: {
    toggleTheme() {
      this.theme = this.theme === 'dark' ? 'light' : 'dark';
      localStorage.setItem(THEME_KEY, this.theme);
    },
    toggleFavorite(id) {
      const strat = this.strats.find(s => s.id === id);
      if (strat) {
    strat.favorite = !strat.favorite;
    this.persist();
  }
},
    changeMap(m) { this.activeMap = m; this.view = 'library'; },
    addMap() {
      const n = this.newMapInput.trim().toUpperCase();
      if (n && !this.maps.includes(n)) {
        this.maps.push(n);
        this.persist();
        this.newMapInput = '';
      }
    },
    removeMap(m) {
      if (this.maps.length > 1 && confirm(`SYSTEM: TERMINATE SECTOR ${m}?`)) {
        this.maps = this.maps.filter(x => x !== m);
        if (this.activeMap === m) this.activeMap = this.maps[0];
        this.persist();
      }
    },
    resetToDefault() {
      if (confirm('RE-INITIALIZE OFFICIAL POOL?')) {
        this.maps = [...DEFAULT_MAPS];
        this.activeMap = this.maps[0];
        this.persist();
      }
    },
    saveStrat(p) {
      this.strats.push({ id: Date.now(), date: new Date().toISOString(), map: this.activeMap, favorite: false, ...p });
      this.persist();
      this.view = 'library';
    },
    deleteStrat(id) {
      if (confirm('SYSTEM: DELETE STRAT?')) {
        this.strats = this.strats.filter(s => s.id !== id);
        this.persist();
      }
    },
    persist() {
      localStorage.setItem(MAPS_KEY, JSON.stringify(this.maps));
      localStorage.setItem(DATA_KEY, JSON.stringify(this.strats));
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

.search-container {
  margin-top: 15px;
  width: 100%;
  max-width: 420px;
}

body, html {
  width: 100%;
  height: 100%;
  overflow: hidden;
}

#app {
  display: flex;
  width: 100vw;
  height: 100vh;
  font-family: 'Inter', system-ui, -apple-system, sans-serif;
}

#app.dark {
  --bg: #080a0f;
  --surface: #11141b;
  --panel: rgba(255, 255, 255, 0.03);
  --border: rgba(255, 255, 255, 0.08);
  --text: #e2e8f0;
  --muted: #64748b;
  --accent: #00d1ff;
  --input: #000000;
  --overlay: rgba(0, 0, 0, 0.85);
  background: #080a0f;
  color: #e2e8f0;
}

#app.light {
  --bg: #f1f3f6;
  --surface: #ffffff;
  --panel: #e2e8f0;
  --border: rgba(0, 0, 0, 0.1);
  --text: #1e293b;
  --muted: #475569;
  --accent: #0066ff;
  --input: #f8fafc;
  --overlay: rgba(30, 41, 59, 0.5);
  background: #f1f3f6;
  color: #1e293b;
}

:root {
  --t-color: #ffa000;
  --ct-color: #00a2ff;
}

.sidebar {
  width: 300px;
  height: 100%;
  background: var(--surface);
  border-right: 1px solid var(--border);
  padding: 40px 25px;
  display: flex;
  flex-direction: column;
  flex-shrink: 0;
}

.sidebar-inner {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.logo-section {
  display: flex;
  align-items: center;
  gap: 15px;
  margin-bottom: 50px;
}

.hex-logo {
  width: 44px;
  height: 44px;
  background: var(--accent);
  color: #fff;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 900;
  font-size: 18px;
  box-shadow: 0 0 20px rgba(0, 209, 255, 0.2);
}

.brand-name {
  display: block;
  font-weight: 900;
  font-size: 14px;
  letter-spacing: 1px;
}

.brand-tag {
  display: block;
  font-size: 9px;
  font-weight: 800;
  color: var(--muted);
}

.map-registry {
  flex: 1;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.registry-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 10px;
  font-weight: 900;
  color: var(--muted);
  margin-bottom: 20px;
  letter-spacing: 1px;
}

.btn-settings {
  background: none;
  border: none;
  cursor: pointer;
  color: var(--muted);
  font-size: 16px;
  transition: 0.3s;
}

.btn-settings:hover {
  color: var(--accent);
  transform: rotate(90deg);
}

.btn-favorites-nav {
  width: 100%;
  display: flex;
  align-items: center;
  padding: 12px 16px;
  background: var(--panel);
  border: 1px solid var(--border);
  border-radius: 12px;
  margin-bottom: 15px;
  cursor: pointer;
  transition: all 0.2s ease;
  color: var(--muted);
  font-weight: 700;
  font-size: 13px;
}

.btn-favorites-nav:hover {
  background: rgba(255, 215, 0, 0.1);
  border-color: rgba(255, 215, 0, 0.3);
}

.btn-favorites-nav.active {
  background: rgba(255, 215, 0, 0.15);
  border-color: #ffd700;
  color: #ffd700;
}

.fav-icon {
  font-size: 18px;
  margin-right: 12px;
}

.fav-label {
  flex: 1;
  letter-spacing: 1px;
}

.fav-count {
  background: #ffd700;
  color: #000;
  padding: 2px 8px;
  border-radius: 10px;
  font-size: 11px;
  font-weight: 900;
  min-width: 24px;
  text-align: center;
}

.map-list-area {
  flex: 1;
  overflow-y: auto;
  padding-right: 5px;
}

.map-list-area::-webkit-scrollbar {
  width: 4px;
}

.map-list-area::-webkit-scrollbar-track {
  background: transparent;
}

.map-list-area::-webkit-scrollbar-thumb {
  background: var(--border);
  border-radius: 2px;
}

.btn-deploy-trigger {
  width: 100%;
  padding: 18px;
  background: var(--accent);
  color: #fff;
  border: none;
  border-radius: 14px;
  font-weight: 900;
  cursor: pointer;
  transition: 0.2s;
  margin-top: 20px;
  box-shadow: 0 4px 15px rgba(0, 209, 255, 0.2);
}

.btn-deploy-trigger:hover:not(:disabled) {
  transform: translateY(-2px);
  filter: brightness(1.1);
}

.btn-deploy-trigger:disabled {
  opacity: 0.15;
  cursor: not-allowed;
  filter: grayscale(1);
}

.viewport {
  flex: 1;
  height: 100%;
  overflow-y: auto;
  padding: 40px 50px;
  display: flex;
  flex-direction: column;
  position: relative;
}

.viewport::-webkit-scrollbar {
  width: 6px;
}

.viewport::-webkit-scrollbar-track {
  background: transparent;
}

.viewport::-webkit-scrollbar-thumb {
  background: var(--border);
  border-radius: 3px;
}

.top-nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 50px;
  flex-shrink: 0;
}

.version-tag {
  font-size: 10px;
  font-weight: 900;
  color: var(--muted);
  background: var(--panel);
  padding: 6px 12px;
  border-radius: 6px;
  letter-spacing: 1px;
}

.nav-right {
  display: flex;
  gap: 30px;
  align-items: center;
}

.btn-toggle-hud {
  background: var(--panel);
  border: 1px solid var(--border);
  color: var(--text);
  padding: 10px 20px;
  border-radius: 12px;
  cursor: pointer;
  font-size: 11px;
  font-weight: 800;
  transition: all 0.2s ease;
}

.btn-toggle-hud:hover {
  border-color: var(--accent);
  background: var(--surface);
}

.status-badge {
  background: var(--surface);
  border: 1px solid var(--border);
  padding: 10px 20px;
  border-radius: 10px;
  text-align: right;
}

.s-label {
  display: block;
  font-size: 9px;
  font-weight: 800;
  color: var(--muted);
  margin-bottom: 2px;
}

.s-value {
  font-size: 18px;
  font-weight: 900;
  color: var(--accent);
}

.main-view {
  flex: 1;
}

.content-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 40px;
  flex-wrap: wrap;
  gap: 20px;
}

.active-sector-title {
  font-size: 42px;
  font-weight: 900;
  letter-spacing: -2px;
  text-transform: uppercase;
}

.filter-panel {
  display: flex;
  gap: 15px;
  flex-wrap: wrap;
}

.filter-group {
  display: flex;
  background: var(--panel);
  padding: 4px;
  border-radius: 12px;
  border: 1px solid var(--border);
}

.f-pill {
  background: none;
  border: none;
  color: var(--muted);
  padding: 8px 18px;
  border-radius: 9px;
  cursor: pointer;
  font-size: 10px;
  font-weight: 900;
  transition: all 0.2s ease;
}

.f-pill:hover:not(.active) {
  color: var(--text);
  background: rgba(255, 255, 255, 0.05);
}

.f-pill.active {
  background: var(--surface);
  color: var(--text);
  box-shadow: 0 4px 15px rgba(0,0,0,0.1);
}

.tactical-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(340px, 1fr));
  gap: 30px;
  padding-bottom: 60px;
}

.empty-state-hud {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 120px 40px;
  border: 2px dashed var(--border);
  border-radius: 30px;
  text-align: center;
  background: var(--panel);
  margin-top: 20px;
}

.hud-circle-icon {
  position: relative;
  width: 90px;
  height: 90px;
  border: 3px solid var(--accent);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 30px;
}

.inner-pulse {
  position: absolute;
  inset: -10px;
  border: 1px solid var(--accent);
  border-radius: 50%;
  opacity: 0.3;
  animation: hud-pulse 2s infinite ease-out;
}

.icon-char {
  font-size: 40px;
  font-weight: 900;
  color: var(--accent);
}

.empty-state-hud h3 {
  font-size: 24px;
  font-weight: 900;
  margin-bottom: 12px;
  letter-spacing: 1px;
}

.empty-state-hud p {
  color: var(--muted);
  font-size: 15px;
  max-width: 400px;
  line-height: 1.6;
  margin-bottom: 35px;
}

.btn-init-strat {
  background: var(--accent);
  color: #fff;
  border: none;
  padding: 16px 32px;
  border-radius: 12px;
  font-weight: 900;
  font-size: 13px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.btn-init-strat:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(0, 209, 255, 0.3);
}

.settings-container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 40px;
  margin-top: 20px;
}

.settings-box {
  background: var(--surface);
  border: 1px solid var(--border);
  padding: 40px;
  border-radius: 24px;
}

.box-label {
  display: block;
  font-size: 11px;
  font-weight: 900;
  color: var(--accent);
  margin-bottom: 25px;
  letter-spacing: 1px;
}

.hud-input-group {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.hud-input-main {
  background: var(--input);
  border: 1px solid var(--border);
  color: var(--text);
  padding: 16px;
  border-radius: 12px;
  font-size: 15px;
  outline: none;
  transition: border-color 0.2s ease;
}

.hud-input-main:focus {
  border-color: var(--accent);
  box-shadow: 0 0 0 3px rgba(0, 209, 255, 0.1);
}

.hud-input-main::placeholder {
  color: var(--muted);
  opacity: 0.5;
}

.btn-hud-confirm {
  background: var(--accent);
  color: #fff;
  border: none;
  padding: 16px;
  border-radius: 12px;
  font-weight: 900;
  cursor: pointer;
  transition: all 0.2s ease;
}

.btn-hud-confirm:hover {
  filter: brightness(1.1);
  transform: translateY(-1px);
}

.registry-scroller {
  max-height: 350px;
  overflow-y: auto;
  margin-bottom: 25px;
  padding-right: 10px;
}

.registry-scroller::-webkit-scrollbar {
  width: 4px;
}

.registry-scroller::-webkit-scrollbar-track {
  background: transparent;
}

.registry-scroller::-webkit-scrollbar-thumb {
  background: var(--border);
  border-radius: 2px;
}

.registry-entry {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 20px;
  background: var(--panel);
  border-radius: 14px;
  margin-bottom: 10px;
  border: 1px solid var(--border);
  font-weight: 800;
  text-transform: uppercase;
  font-size: 14px;
  transition: background 0.2s ease;
}

.registry-entry:hover {
  background: var(--surface);
}

.entry-name {
  flex: 1;
  margin-right: 15px;
}

.btn-entry-remove {
  background: rgba(239, 68, 68, 0.1);
  color: #ef4444;
  border: 1px solid rgba(239, 68, 68, 0.2);
  padding: 8px 16px;
  border-radius: 8px;
  font-size: 10px;
  font-weight: 900;
  cursor: pointer;
  transition: all 0.2s ease;
  white-space: nowrap;
}

.btn-entry-remove:hover {
  background: rgba(239, 68, 68, 0.2);
  border-color: rgba(239, 68, 68, 0.4);
}

.btn-hud-reset {
  width: 100%;
  padding: 15px;
  background: none;
  border: 1px dashed var(--muted);
  color: var(--muted);
  border-radius: 12px;
  font-weight: 800;
  cursor: pointer;
  font-size: 12px;
  transition: all 0.2s ease;
}

.btn-hud-reset:hover {
  border-color: var(--accent);
  color: var(--accent);
  background: rgba(0, 209, 255, 0.05);
}

.btn-exit-settings {
  background: var(--panel);
  color: var(--text);
  border: 1px solid var(--border);
  padding: 12px 25px;
  border-radius: 12px;
  cursor: pointer;
  font-weight: 800;
  font-size: 12px;
  transition: all 0.2s ease;
}

.btn-exit-settings:hover {
  background: var(--surface);
  border-color: var(--accent);
}

.empty-registry {
  text-align: center;
  color: var(--muted);
  padding: 40px 20px;
  font-size: 14px;
}

.toast-notification {
  position: fixed;
  bottom: 30px;
  right: 30px;
  background: var(--surface);
  border: 1px solid var(--accent);
  color: var(--text);
  padding: 16px 24px;
  border-radius: 14px;
  font-weight: 700;
  font-size: 14px;
  z-index: 2000;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
  animation: slideIn 0.3s ease;
}

@keyframes slideIn {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

@keyframes hud-pulse {
  0% {
    transform: scale(0.8);
    opacity: 0.8;
  }
  100% {
    transform: scale(1.5);
    opacity: 0;
  }
}

@media (max-width: 1200px) {
  .sidebar {
    display: none;
  }
  
  .viewport {
    padding: 25px;
  }
  
  .active-sector-title {
    font-size: 32px;
  }
  
  .settings-container {
    grid-template-columns: 1fr;
    gap: 20px;
  }
}

@media (max-width: 768px) {
  .viewport {
    padding: 20px;
  }
  
  .content-header {
    flex-direction: column;
    align-items: flex-start;
  }
  
  .top-nav {
    flex-direction: column;
    gap: 15px;
    align-items: flex-start;
  }
  
  .tactical-grid {
    grid-template-columns: 1fr;
  }
  
  .nav-right {
    width: 100%;
    justify-content: space-between;
  }
}
.header-right {
  display: flex;
  align-items: center;
  gap: 12px;
  flex-wrap: wrap;
  justify-content: flex-end;
}

.btn-search-toggle {
  width: 48px;
  height: 48px;
  background: var(--panel);
  border: 1px solid var(--border);
  color: var(--text);
  border-radius: 12px;
  font-size: 20px;
  cursor: pointer;
  transition: all 0.25s ease;
  flex-shrink: 0;
}

.btn-search-toggle:hover {
  border-color: var(--accent);
  background: var(--surface);
}

.btn-search-toggle.active {
  background: var(--accent);
  color: #fff;
  border-color: var(--accent);
}

.search-input-wrapper {
  position: relative;
  min-width: 320px;
  max-width: 420px;
  min-width: 280px;
}

.hud-search-input {
  background: var(--input);
  border: 1px solid var(--border);
  color: var(--text);
  padding: 14px 50px 14px 20px;
  border-radius: 14px;
  width: 100%;
  font-size: 15px;
  outline: none;
  transition: all 0.2s ease;
}

.hud-search-input:focus {
  border-color: var(--accent);
  box-shadow: 0 0 0 3px rgba(0, 209, 255, 0.15);
}

.search-clear-btn {
  position: absolute;
  right: 12px;
  top: 50%;
  transform: translateY(-50%);
  background: none;
  border: none;
  color: var(--muted);
  font-size: 18px;
  cursor: pointer;
  padding: 4px;
}

.search-clear-btn:hover {
  color: #ef4444;
}

/* Адаптивность */
@media (max-width: 1100px) {
  .header-right {
    flex-direction: column;
    align-items: flex-end;
  }
  
}
/* === ROUND FILTER IN HEADER === */
.round-filter .f-pill {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 11px;
  padding: 8px 14px;
}

.round-filter .f-pill.round.pistol.active { background: #8b5cf6; color: white; }
.round-filter .f-pill.round.eco.active    { background: #eab308; color: #000; }
.round-filter .f-pill.round.force.active  { background: #f97316; color: white; }
.round-filter .f-pill.round.full.active   { background: #22c55e; color: white; }

/* === ROUND SWITCH IN FORM === */
.round-type-switch {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(110px, 1fr));
  gap: 12px;
}

.round-type-switch button {
  padding: 16px 12px;
  background: var(--panel);
  border: 2px solid var(--border);
  color: var(--muted);
  border-radius: 14px;
  cursor: pointer;
  font-weight: 800;
  font-size: 13px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  transition: all 0.25s ease;
}

.round-type-switch button:hover:not(.active) {
  border-color: var(--accent);
  color: var(--text);
  transform: translateY(-2px);
}

.round-type-switch button .rt-icon {
  font-size: 18px;
}

.round-type-switch button.active {
  border-color: var(--accent);
  background: var(--accent);
  color: white;
  box-shadow: 0 0 20px rgba(0, 209, 255, 0.3);
}
</style>
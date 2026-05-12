<template>
  <div class="strat-card" :class="strat.side">
    <div class="card-inner">
      <header class="card-header">
        <div class="site-tag" :class="getSiteClass(strat.site)">
          {{ strat.site || '?' }}
        </div>
        <div class="round-type-badge" :class="strat.roundType">
          {{ getRoundIcon(strat.roundType) }}
        </div>
        <div class="util-display" v-if="hasUtility">
          <div v-if="strat.utility.smokes > 0" class="u-unit">
            <span class="u-icon">☁️</span> 
            <span class="u-count">{{ strat.utility.smokes }}</span>
          </div>
          <div v-if="strat.utility.mollys > 0" class="u-unit">
            <span class="u-icon">🔥</span> 
            <span class="u-count">{{ strat.utility.mollys }}</span>
          </div>
          <div v-if="strat.utility.flashes > 0" class="u-unit">
            <span class="u-icon">✨</span> 
            <span class="u-count">{{ strat.utility.flashes }}</span>
          </div>
        </div>
      </header>
      
      <div class="card-content">
        <div class="side-id">
          {{ strat.side === 't' ? 'TERRORIST UNIT' : 'COUNTER-TERRORIST UNIT' }}
        </div>
        <h3 class="strat-name">{{ strat.name || 'UNNAMED STRATEGY' }}</h3>
        <p class="strat-description">
          {{ strat.description || 'No tactical data recorded' }}
        </p>
        
        <a 
          v-if="strat.videoUrl" 
          :href="sanitizedVideoUrl" 
          target="_blank" 
          rel="noopener noreferrer"
          class="btn-watch"
        >
          ANALYZE DATA STREAM
        </a>
      </div>
      <div class="hashtags" v-if="strat.hashtags && strat.hashtags.length">
  <span 
    v-for="tag in strat.hashtags" 
    :key="tag"
    class="hashtag-pill-small"
  >
    #{{ tag }}
  </span>
</div>

      <footer class="card-footer">
  <span class="entry-date">{{ formattedDate }}</span>
  
  <div class="footer-right">
    <button 
      @click="$emit('toggle-favorite', strat.id)" 
      class="fav-btn"
      :class="{ active: strat.favorite }"
      title="Add to favorites"
    >
      {{ strat.favorite ? '★' : '☆' }}
    </button>
    <button @click="handleDelete" class="btn-trash-icon" title="Delete strategy">
      🗑️
    </button>
  </div>
</footer>
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
  
 emits: ['delete', 'toggle-favorite'],
  
  computed: {
    sanitizedVideoUrl() {
      const url = this.strat.videoUrl;
      if (!url) return '#';
      
      try {
        new URL(url);
        return url;
      } catch {
        if (url.includes('.') && !url.includes(' ')) {
          return `https://${url}`;
        }
        return '#';
      }
    },
    
    formattedDate() {
      if (!this.strat.date) return 'No date';
      
      try {
        const date = new Date(this.strat.date);
        if (isNaN(date.getTime())) return 'Invalid date';
        
        return date.toLocaleDateString('ru-RU', {
          year: 'numeric',
          month: '2-digit',
          day: '2-digit'
        });
      } catch {
        return 'Invalid date';
      }
    },
    
    hasUtility() {
      if (!this.strat.utility) return false;
      const { smokes, mollys, flashes } = this.strat.utility;
      return (smokes > 0) || (mollys > 0) || (flashes > 0);
    }
  },
  
  methods: {
    getSiteClass(site) {
      if (!site) return '';
      const s = site.toLowerCase();
      if (['a', 'b', 'mid', 'any'].includes(s)) return s;
      return '';
    },
    getRoundIcon(type) {
      const map = { pistol: '🔫', eco: '🐭', force: '⚡', full: '💰' };
      return map[type] || '❓';
    },
    
    handleDelete() {
      if (confirm('Delete this strategy?')) {
        this.$emit('delete');
      }
    }
  }
}
</script>

<style scoped>
.footer-right {
  display: flex;
  align-items: center;
  gap: 12px;
}

.fav-btn {
  background: none;
  border: none;
  font-size: 22px;
  cursor: pointer;
  color: var(--muted);
  transition: all 0.2s ease;
  padding: 4px;
  border-radius: 8px;
}

.fav-btn:hover {
  color: #ffd700;
  transform: scale(1.2);
}

.fav-btn.active {
  color: #ffd700;
  animation: favPop 0.3s ease;
}

@keyframes favPop {
  0% { transform: scale(0.5); }
  50% { transform: scale(1.3); }
  100% { transform: scale(1); }
}
.strat-card { 
  background: var(--surface); 
  border: 1px solid var(--border); 
  border-radius: 24px; 
  transition: all 0.3s ease; 
  position: relative; 
  overflow: hidden; 
}

.strat-card:hover { 
  border-color: var(--accent); 
  transform: translateY(-6px); 
  box-shadow: 0 15px 35px rgba(0,0,0,0.25); 
}

.strat-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 3px;
  background: transparent;
}

.strat-card.t::before { background: var(--t-color); }
.strat-card.ct::before { background: var(--ct-color); }

.card-inner { 
  padding: 25px; 
  display: flex; 
  flex-direction: column; 
  min-height: 280px; 
}

.card-header { 
  display: flex; 
  justify-content: space-between; 
  align-items: center; 
  margin-bottom: 25px; 
}

.site-tag { 
  width: 36px; 
  height: 36px; 
  display: flex; 
  align-items: center; 
  justify-content: center; 
  border-radius: 10px; 
  font-weight: 900; 
  font-size: 14px; 
  background: var(--panel); 
  color: var(--text); 
  border: 1px solid var(--border); 
  text-transform: uppercase;
}

.site-tag.a { background: #ef4444; color: #fff; border: none; box-shadow: 0 0 15px rgba(239, 68, 68, 0.4); }
.site-tag.b { background: #3b82f6; color: #fff; border: none; box-shadow: 0 0 15px rgba(59, 130, 246, 0.4); }
.site-tag.mid { background: #a855f7; color: #fff; border: none; box-shadow: 0 0 15px rgba(168, 85, 247, 0.4); }
.site-tag.any { background: #10b981; color: #fff; border: none; box-shadow: 0 0 15px rgba(16, 185, 129, 0.4); }

.util-display { display: flex; gap: 12px; font-size: 13px; font-weight: 900; color: var(--muted); }
.u-unit { 
  display: flex; 
  align-items: center; 
  gap: 6px;
  background: var(--panel);
  padding: 4px 10px;
  border-radius: 8px;
  border: 1px solid var(--border);
}

.card-content { flex: 1; }

.side-id { 
  font-size: 9px; 
  font-weight: 900; 
  letter-spacing: 1.2px; 
  margin-bottom: 12px;
  padding: 4px 10px;
  display: inline-block;
  border-radius: 6px;
}

.t .side-id { color: var(--t-color); background: rgba(255, 160, 0, 0.1); border: 1px solid rgba(255, 160, 0, 0.2); }
.ct .side-id { color: var(--ct-color); background: rgba(0, 162, 255, 0.1); border: 1px solid rgba(0, 162, 255, 0.2); }

.strat-name { 
  font-size: 20px; 
  font-weight: 900; 
  margin-bottom: 10px; 
  color: var(--text); 
  line-height: 1.2;
  word-break: break-word;
}

.strat-description { 
  font-size: 14px; 
  color: var(--muted); 
  line-height: 1.6; 
  max-height: 4.8em; 
  overflow: hidden; 
  display: -webkit-box; 
  -line-clamp: 3; 
  -webkit-box-orient: vertical;
  word-break: break-word;
}

.btn-watch { 
  display: inline-flex; 
  align-items: center;
  gap: 8px;
  margin-top: 20px; 
  padding: 12px 18px; 
  background: var(--panel); 
  border: 1px solid var(--border); 
  color: var(--accent); 
  text-decoration: none; 
  border-radius: 14px; 
  font-size: 10px; 
  font-weight: 900; 
  letter-spacing: 1px; 
  transition: all 0.2s ease; 
}

.btn-watch::before { content: '▶'; font-size: 8px; }

.btn-watch:hover { 
  background: var(--accent); 
  color: #fff; 
  border-color: var(--accent);
  transform: translateY(-2px);
}

.card-footer { 
  margin-top: auto; 
  padding-top: 20px; 
  border-top: 1px solid var(--border); 
  display: flex; 
  justify-content: space-between; 
  align-items: center; 
}

.entry-date { 
  font-size: 11px; 
  color: var(--muted); 
  font-weight: 700; 
  font-family: monospace; 
}

.btn-trash-icon { 
  background: none; 
  border: none; 
  cursor: pointer; 
  color: var(--muted); 
  opacity: 0.4; 
  transition: all 0.2s ease; 
  font-size: 18px;
  padding: 5px;
  border-radius: 8px;
}

.btn-trash-icon:hover { 
  color: #ef4444; 
  opacity: 1; 
  transform: scale(1.1);
  background: rgba(239, 68, 68, 0.1);
}
.hashtags {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-top: 12px;
}

.hashtag-pill-small {
  background: var(--panel);
  color: var(--accent);
  font-size: 11px;
  padding: 3px 8px;
  border-radius: 9999px;
  border: 1px solid rgba(0, 209, 255, 0.2);
}
</style>
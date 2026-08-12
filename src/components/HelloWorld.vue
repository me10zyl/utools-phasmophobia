<template>
  <main class="ghost-helper">
    <section class="evidence-panel" aria-label="证据筛选">
      <div class="panel-heading">
        <div class="filter-summary">
          <strong>证据筛选</strong>
          <span>{{ activeEvidenceCount ? activeEvidenceCount + ' 项条件' : '左键确认 · 右键排除' }}</span>
        </div>
        <el-button class="reset-button" type="text" icon="el-icon-refresh" @click="reset">重置</el-button>
      </div>
      <div class="evidence-list">
        <button
          v-for="evidence in evidences"
          :key="evidence"
          class="evidence-button"
          :class="evidenceState(evidence)"
          type="button"
          @click="selectEvidence(evidence)"
          @contextmenu.prevent="excludeEvidence(evidence)"
        >
          <span class="evidence-mark">{{ evidenceMark(evidence) }}</span>
          <span class="evidence-name">{{ evidence }}</span>
          <span v-if="activeEvidenceCount" class="evidence-probability">{{ evidenceProbability(evidence) }}</span>
        </button>
      </div>
      <p v-if="activeEvidenceCount" class="probability-hint">概率：当前剩余鬼魂中具备该证据的比例</p>
    </section>

    <section class="result-header">
      <span>候选鬼魂 <strong>{{ filteredGhostTypes.length }}</strong> / {{ ghostTypes.length }}</span>
      <span class="legend"><i class="confirmed"></i>确认 <i class="excluded"></i>排除</span>
    </section>

    <section v-if="filteredGhostTypes.length" class="ghost-grid">
      <article v-for="ghost in filteredGhostTypes" :key="ghost.id" class="ghost-card">
        <div class="card-topline">
          <span class="category">{{ ghost.category }}</span>
          <span v-if="ghost.lineOfSightAcceleration === true" class="sight">视野加速</span>
          <span v-else-if="ghost.lineOfSightAcceleration === false" class="sight muted">无视野加速</span>
        </div>
        <div class="name-row">
          <h2>{{ ghost.name }}</h2>
          <span class="hunt-threshold">猎杀 {{ ghost.huntThreshold }}</span>
        </div>
        <p class="quick-memory">{{ ghost.quickMemory }}</p>

        <div class="evidence-tags" aria-label="证据">
          <span
            v-for="evidence in ghost.evidences"
            :key="evidence"
            class="evidence-tag"
            :class="evidenceState(evidence)"
          >{{ evidence }}<b v-if="activeEvidenceCount">{{ evidenceProbability(evidence) }}</b></span>
        </div>

        <dl class="stats">
          <div>
            <dt>移速</dt>
            <dd>{{ ghost.speed }}</dd>
          </div>
          <div>
            <dt>必特征</dt>
            <dd>{{ ghost.specialProof }}</dd>
          </div>
        </dl>

        <ul class="traits">
          <li v-for="trait in ghost.traits" :key="trait">{{ trait }}</li>
        </ul>
      </article>
    </section>

    <section v-else class="empty-state">
      <i class="el-icon-warning-outline"></i>
      没有符合当前条件的鬼魂，请调整证据。
    </section>
  </main>
</template>

<script>
import ghostTypes from '@/data/ghosts.json'

export default {
  name: 'HelloWorld',
  data () {
    return {
      ghostTypes: ghostTypes,
      selectedEvidences: [],
      excludedEvidences: []
    }
  },
  computed: {
    evidences () {
      const evidenceMap = {}
      this.ghostTypes.forEach(function (ghost) {
        ghost.evidences.forEach(function (evidence) {
          evidenceMap[evidence] = true
        })
      })
      return Object.keys(evidenceMap).sort()
    },
    activeEvidenceCount () {
      return this.selectedEvidences.length + this.excludedEvidences.length
    },
    filteredGhostTypes () {
      const selectedEvidences = this.selectedEvidences
      const excludedEvidences = this.excludedEvidences

      return this.ghostTypes.filter(function (ghost) {
        const hasAllSelectedEvidences = selectedEvidences.every(function (evidence) {
          return ghost.evidences.indexOf(evidence) !== -1
        })
        const hasExcludedEvidence = excludedEvidences.some(function (evidence) {
          return ghost.evidences.indexOf(evidence) !== -1
        })
        return hasAllSelectedEvidences && !hasExcludedEvidence
      })
    },
    evidenceProbabilities () {
      const remainingGhostCount = this.filteredGhostTypes.length
      const probabilities = {}

      this.evidences.forEach(function (evidence) {
        const ghostCount = this.filteredGhostTypes.filter(function (ghost) {
          return ghost.evidences.indexOf(evidence) !== -1
        }).length
        probabilities[evidence] = remainingGhostCount ? ghostCount / remainingGhostCount : 0
      }, this)

      return probabilities
    }
  },
  methods: {
    evidenceState (evidence) {
      if (this.selectedEvidences.indexOf(evidence) !== -1) {
        return 'is-selected'
      }
      if (this.excludedEvidences.indexOf(evidence) !== -1) {
        return 'is-excluded'
      }
      return ''
    },
    evidenceMark (evidence) {
      const state = this.evidenceState(evidence)
      if (state === 'is-selected') {
        return '✓'
      }
      if (state === 'is-excluded') {
        return '×'
      }
      return '+'
    },
    evidenceProbability (evidence) {
      const probability = this.evidenceProbabilities[evidence] || 0
      return Math.round(probability * 100) + '%'
    },
    selectEvidence (evidence) {
      if (this.selectedEvidences.indexOf(evidence) !== -1) {
        this.selectedEvidences = this.selectedEvidences.filter(function (item) {
          return item !== evidence
        })
        return
      }

      this.excludedEvidences = this.excludedEvidences.filter(function (item) {
        return item !== evidence
      })
      this.selectedEvidences.push(evidence)
    },
    excludeEvidence (evidence) {
      if (this.excludedEvidences.indexOf(evidence) !== -1) {
        this.excludedEvidences = this.excludedEvidences.filter(function (item) {
          return item !== evidence
        })
        return
      }

      this.selectedEvidences = this.selectedEvidences.filter(function (item) {
        return item !== evidence
      })
      this.excludedEvidences.push(evidence)
    },
    reset () {
      this.selectedEvidences = []
      this.excludedEvidences = []
    }
  }
}
</script>

<style scoped>
.ghost-helper {
  --ink: #253342;
  --muted: #788696;
  --line: #d7e1e8;
  --paper: #f7fafc;
  --blue: #2d7fba;
  --green: #438667;
  --red: #bf625b;
  box-sizing: border-box;
  min-height: 100vh;
  padding: 12px 16px 20px;
  color: var(--ink);
  background: linear-gradient(135deg, #edf4f7, #fafbf8 52%, #e9f2ef);
  font-family: Georgia, "Songti SC", "STSong", serif;
}

.evidence-panel,
.result-header,
.ghost-grid,
.empty-state {
  max-width: 1160px;
  margin-right: auto;
  margin-left: auto;
}

.evidence-panel {
  padding: 10px 12px 8px;
  border: 1px solid var(--line);
  border-radius: 7px;
  background: rgba(255, 255, 255, 0.83);
  box-shadow: 0 2px 8px rgba(46, 74, 94, 0.07);
}

.panel-heading,
.result-header,
.filter-summary,
.legend,
.evidence-list,
.evidence-button,
.card-topline,
.name-row,
.evidence-tags {
  display: flex;
  align-items: center;
}

.panel-heading,
.result-header,
.card-topline,
.name-row {
  justify-content: space-between;
}

.panel-heading {
  margin-bottom: 8px;
}

.filter-summary {
  gap: 9px;
}

.filter-summary strong {
  color: #1f4a63;
  font-size: 14px;
}

.filter-summary span,
.probability-hint {
  color: var(--muted);
  font-size: 11px;
}

.reset-button {
  padding: 0;
  color: #5d768a;
  font-family: inherit;
  font-size: 12px;
}

.reset-button:hover {
  color: var(--blue);
}

.evidence-list {
  flex-wrap: wrap;
  gap: 6px;
}

.evidence-button {
  gap: 5px;
  min-width: 91px;
  padding: 5px 7px;
  color: #425667;
  border: 1px solid #c9d8e1;
  border-radius: 4px;
  background: #fafdfe;
  cursor: pointer;
  font: inherit;
  font-size: 12px;
  transition: background 0.12s, border-color 0.12s, transform 0.12s;
}

.evidence-button:hover {
  border-color: #83a9c1;
  background: #eff8fc;
  transform: translateY(-1px);
}

.evidence-mark {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 14px;
  height: 14px;
  color: #7791a1;
  border: 1px solid currentColor;
  border-radius: 50%;
  font-size: 10px;
  font-weight: bold;
}

.evidence-name {
  white-space: nowrap;
}

.evidence-probability {
  margin-left: auto;
  color: #a77728;
  font-size: 11px;
  font-weight: bold;
}

.evidence-button.is-selected {
  color: #245f46;
  border-color: #86b89d;
  background: #eaf6ed;
}

.evidence-button.is-selected .evidence-probability {
  color: var(--green);
}

.evidence-button.is-excluded {
  color: #984842;
  border-color: #dba19b;
  background: #fff0ee;
}

.evidence-button.is-excluded .evidence-probability {
  color: var(--red);
}

.probability-hint {
  margin: 6px 0 0;
}

.result-header {
  margin-top: 12px;
  margin-bottom: 8px;
  color: #587083;
  font-size: 12px;
}

.result-header strong {
  color: #1d668f;
  font-size: 18px;
}

.legend {
  gap: 8px;
  color: #788696;
  font-size: 10px;
}

.legend i {
  width: 7px;
  height: 7px;
  margin-left: 2px;
  border-radius: 50%;
}

.legend .confirmed {
  background: #62a279;
}

.legend .excluded {
  background: #cf7770;
}

.ghost-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(232px, 1fr));
  gap: 9px;
}

.ghost-card {
  display: flex;
  flex-direction: column;
  min-height: 211px;
  padding: 10px 11px;
  border: 1px solid var(--line);
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.9);
  box-shadow: 0 2px 6px rgba(46, 74, 94, 0.05);
}

.category {
  padding: 2px 5px;
  color: #61717c;
  border-radius: 3px;
  background: #edf2f3;
  font-size: 10px;
}

.sight {
  color: #47805f;
  font-size: 10px;
}

.sight.muted {
  color: #8c9aa5;
}

.name-row {
  margin-top: 7px;
  gap: 6px;
}

.name-row h2 {
  margin: 0;
  color: #213e51;
  font-size: 19px;
  line-height: 1.1;
}

.hunt-threshold {
  flex: 0 0 auto;
  color: #a87524;
  font-size: 10px;
}

.quick-memory {
  min-height: 16px;
  margin: 4px 0 7px;
  color: #617383;
  font-size: 11px;
  line-height: 1.3;
}

.evidence-tags {
  flex-wrap: wrap;
  gap: 4px;
  margin-bottom: 7px;
}

.evidence-tag {
  padding: 2px 5px;
  color: #577083;
  border: 1px solid #d4e0e7;
  border-radius: 3px;
  background: #f7fbfd;
  font-size: 10px;
}

.evidence-tag b {
  margin-left: 2px;
  color: #a77728;
  font-size: 9px;
}

.evidence-tag.is-selected {
  color: #246247;
  border-color: #9cc9aa;
  background: #eff9f1;
}

.evidence-tag.is-selected b {
  color: var(--green);
}

.evidence-tag.is-excluded {
  color: #a34d47;
  border-color: #e2aaa4;
  background: #fff2f0;
}

.evidence-tag.is-excluded b {
  color: var(--red);
}

.stats {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 5px;
  margin: 0 0 6px;
}

.stats div {
  min-width: 0;
  padding: 4px 5px;
  border-left: 2px solid #bed1db;
  background: #f5f8f8;
}

.stats dt {
  margin-bottom: 2px;
  color: #82929d;
  font-size: 9px;
}

.stats dd {
  margin: 0;
  color: #526778;
  font-size: 9px;
  line-height: 1.25;
}

.traits {
  display: -webkit-box;
  margin: auto 0 0;
  padding-left: 13px;
  overflow: hidden;
  color: #667886;
  font-size: 10px;
  line-height: 1.35;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 3;
}

.traits li + li {
  margin-top: 2px;
}

.traits li::marker {
  color: #6391a8;
}

.empty-state {
  padding: 30px 16px;
  color: #718493;
  border: 1px dashed #aabfc9;
  border-radius: 6px;
  background: rgba(255, 255, 255, 0.55);
  font-size: 13px;
  text-align: center;
}

.empty-state i {
  margin-right: 5px;
  color: #bd7961;
}

@media (max-width: 600px) {
  .ghost-helper {
    padding: 8px 9px 14px;
  }

  .ghost-grid {
    grid-template-columns: repeat(auto-fill, minmax(205px, 1fr));
    gap: 6px;
  }

  .evidence-panel {
    padding: 8px;
  }
}
</style>

<template>
  <main class="ghost-helper">
    <header class="hero">
      <div>
        <p class="eyebrow">PHASMOPHOBIA</p>
        <h1>恐鬼症证据推理</h1>
        <p class="subtitle">左键确认已发现的证据，右键排除当前证据。</p>
      </div>
      <el-button class="reset-button" type="text" icon="el-icon-refresh" @click="reset">
        重置筛选
      </el-button>
    </header>

    <section class="evidence-panel" aria-label="证据筛选">
      <div class="panel-heading">
        <h2>证据</h2>
        <span>{{ activeEvidenceCount }} 项条件已启用</span>
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
          {{ evidence }}
        </button>
      </div>
      <p class="hint"><span>左键</span> 确认 &nbsp; <span>右键</span> 排除 &nbsp;·&nbsp; 再次点击同一状态可取消</p>
    </section>

    <section class="result-header">
      <div>
        <p class="eyebrow">推理结果</p>
        <h2>可能的鬼魂 <strong>{{ filteredGhostTypes.length }}</strong> / {{ ghostTypes.length }}</h2>
      </div>
      <div class="legend">
        <span><i class="legend-dot confirmed"></i>已确认</span>
        <span><i class="legend-dot excluded"></i>已排除</span>
      </div>
    </section>

    <section v-if="filteredGhostTypes.length" class="ghost-grid">
      <article v-for="ghost in filteredGhostTypes" :key="ghost.id" class="ghost-card">
        <div class="card-topline">
          <span class="category">{{ ghost.category }}</span>
          <span v-if="ghost.lineOfSightAcceleration === true" class="sight">视野加速</span>
          <span v-else-if="ghost.lineOfSightAcceleration === false" class="sight muted">无视野加速</span>
        </div>
        <div class="name-row">
          <h3>{{ ghost.name }}</h3>
          <span class="hunt-threshold">猎杀 {{ ghost.huntThreshold }}</span>
        </div>
        <p class="quick-memory">{{ ghost.quickMemory }}</p>

        <dl class="stats">
          <div>
            <dt>特殊移速</dt>
            <dd>{{ ghost.speed }}</dd>
          </div>
          <div>
            <dt>必特征证据</dt>
            <dd>{{ ghost.specialProof }}</dd>
          </div>
        </dl>

        <div class="evidence-tags" aria-label="证据">
          <span
            v-for="evidence in ghost.evidences"
            :key="evidence"
            class="evidence-tag"
            :class="evidenceState(evidence)"
          >{{ evidence }}</span>
        </div>

        <ul class="traits">
          <li v-for="trait in ghost.traits" :key="trait">{{ trait }}</li>
        </ul>
      </article>
    </section>

    <section v-else class="empty-state">
      <i class="el-icon-warning-outline"></i>
      <h2>没有符合当前条件的鬼魂</h2>
      <p>请检查已确认或已排除的证据，或重置后重新推理。</p>
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
  box-sizing: border-box;
  min-height: 100vh;
  padding: 40px clamp(20px, 5vw, 72px) 72px;
  color: #e9e6de;
  background:
    radial-gradient(circle at 10% 0%, rgba(151, 44, 43, 0.2), transparent 28rem),
    radial-gradient(circle at 90% 20%, rgba(63, 101, 94, 0.18), transparent 30rem),
    #141615;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "PingFang SC", "Microsoft YaHei", sans-serif;
}

.hero,
.result-header {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  gap: 24px;
  max-width: 1440px;
  margin: 0 auto;
}

.eyebrow {
  margin: 0 0 7px;
  color: #ad9b72;
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.18em;
}

h1,
h2,
h3,
p {
  margin-top: 0;
}

h1 {
  margin-bottom: 8px;
  color: #f7f2e8;
  font-size: clamp(27px, 4vw, 42px);
  line-height: 1.15;
  letter-spacing: 0.02em;
}

.subtitle {
  margin-bottom: 0;
  color: #aaa79e;
  font-size: 14px;
}

.reset-button {
  padding: 8px 0;
  color: #d6ccba;
  font-size: 14px;
}

.reset-button:hover {
  color: #f1c56b;
}

.evidence-panel {
  max-width: 1440px;
  margin: 34px auto 44px;
  padding: 22px 24px 17px;
  border: 1px solid #343631;
  border-radius: 12px;
  background: rgba(28, 31, 29, 0.82);
  box-shadow: 0 12px 35px rgba(0, 0, 0, 0.18);
}

.panel-heading {
  display: flex;
  align-items: baseline;
  gap: 12px;
  margin-bottom: 18px;
}

.panel-heading h2,
.result-header h2 {
  margin-bottom: 0;
  color: #f1ede2;
  font-size: 19px;
}

.panel-heading span {
  color: #85877f;
  font-size: 12px;
}

.evidence-list {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.evidence-button {
  display: inline-flex;
  align-items: center;
  gap: 7px;
  min-width: 102px;
  padding: 9px 13px;
  overflow: hidden;
  color: #d5d4ce;
  border: 1px solid #4b4d46;
  border-radius: 6px;
  outline: none;
  background: #252825;
  cursor: pointer;
  font: inherit;
  font-size: 14px;
  transition: 0.16s ease;
}

.evidence-button:hover {
  border-color: #979179;
  color: #fff8e8;
  transform: translateY(-1px);
}

.evidence-mark {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 17px;
  height: 17px;
  color: #a9aaa2;
  border: 1px solid currentColor;
  border-radius: 50%;
  font-size: 12px;
  font-weight: 700;
}

.evidence-button.is-selected {
  color: #e5f0d1;
  border-color: #718c58;
  background: #35442f;
}

.evidence-button.is-excluded {
  color: #f2d1cb;
  border-color: #9f554e;
  background: #4b2d2a;
}

.hint {
  margin: 16px 0 0;
  color: #86877f;
  font-size: 12px;
}

.hint span {
  color: #c5b695;
}

.result-header {
  align-items: center;
  margin-bottom: 20px;
}

.result-header h2 strong {
  color: #e7bd68;
  font-family: Georgia, serif;
  font-size: 28px;
}

.legend {
  display: flex;
  flex-wrap: wrap;
  gap: 14px;
  color: #95978e;
  font-size: 12px;
}

.legend span {
  display: inline-flex;
  align-items: center;
  gap: 6px;
}

.legend-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
}

.legend-dot.confirmed {
  background: #9cbd75;
}

.legend-dot.excluded {
  background: #d57970;
}

.ghost-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 16px;
  max-width: 1440px;
  margin: 0 auto;
}

.ghost-card {
  display: flex;
  flex-direction: column;
  min-height: 318px;
  padding: 19px 20px 18px;
  border: 1px solid #353833;
  border-radius: 10px;
  background: linear-gradient(145deg, rgba(42, 45, 41, 0.98), rgba(29, 32, 30, 0.98));
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.13);
}

.card-topline,
.name-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}

.category,
.sight {
  color: #b8b59e;
  font-size: 11px;
}

.category {
  padding: 3px 7px;
  border-radius: 3px;
  background: #33372f;
}

.sight {
  color: #9fb983;
}

.sight.muted {
  color: #84877e;
}

.name-row {
  margin-top: 15px;
}

.name-row h3 {
  margin-bottom: 0;
  color: #f3efe5;
  font-size: 23px;
  line-height: 1.2;
}

.hunt-threshold {
  flex: 0 0 auto;
  color: #e4c169;
  font-size: 12px;
}

.quick-memory {
  min-height: 38px;
  margin: 7px 0 15px;
  color: #c5c5bc;
  font-size: 13px;
  line-height: 1.45;
}

.stats {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
  margin: 0 0 15px;
}

.stats div {
  min-width: 0;
  padding: 8px;
  border-left: 2px solid #575447;
  background: rgba(17, 19, 18, 0.25);
}

.stats dt {
  margin-bottom: 4px;
  color: #85877f;
  font-size: 10px;
}

.stats dd {
  margin: 0;
  color: #d2d0c7;
  font-size: 11px;
  line-height: 1.35;
}

.evidence-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-bottom: 14px;
}

.evidence-tag {
  padding: 3px 7px;
  color: #b9bbb2;
  border: 1px solid #51534c;
  border-radius: 3px;
  font-size: 11px;
}

.evidence-tag.is-selected {
  color: #e2f1cb;
  border-color: #78975d;
  background: #33442e;
}

.evidence-tag.is-excluded {
  color: #f0c9c3;
  border-color: #98574f;
  background: #4a2e2a;
}

.traits {
  margin: auto 0 0;
  padding-left: 17px;
  color: #aeb0a6;
  font-size: 12px;
  line-height: 1.55;
}

.traits li + li {
  margin-top: 5px;
}

.traits li::marker {
  color: #bca56c;
}

.empty-state {
  max-width: 1440px;
  margin: 0 auto;
  padding: 55px 24px;
  color: #a9aaa2;
  border: 1px dashed #494b45;
  border-radius: 10px;
  text-align: center;
}

.empty-state i {
  color: #d9b55e;
  font-size: 31px;
}

.empty-state h2 {
  margin: 11px 0 5px;
  color: #e9e5db;
  font-size: 18px;
}

.empty-state p {
  margin: 0;
  font-size: 13px;
}

@media (max-width: 600px) {
  .ghost-helper {
    padding: 28px 16px 48px;
  }

  .hero,
  .result-header {
    align-items: flex-start;
    flex-direction: column;
    gap: 14px;
  }

  .evidence-panel {
    padding: 18px 16px;
  }

  .evidence-button {
    min-width: 0;
  }

  .legend {
    gap: 10px;
  }
}
</style>

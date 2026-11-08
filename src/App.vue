<script setup>
import { computed, onMounted, onUnmounted, ref, watch } from 'vue'
import DamageCalculator from './components/DamageCalculator.vue'
import Navbar from './components/Navbar.vue'
import TokenCard from './components/TokenCard.vue'

const palette = [
  { bg: '#dce9ce', ink: '#29412e', accent: '#7c9668', icon: '✳' },
  { bg: '#ecd4c3', ink: '#61402e', accent: '#bd8060', icon: '◆' },
  { bg: '#d5e2e6', ink: '#334b57', accent: '#718d98', icon: '◈' },
  { bg: '#e8dfb9', ink: '#5d5130', accent: '#aa995b', icon: '✦' },
  { bg: '#e2d6e8', ink: '#55415f', accent: '#9979a5', icon: '❋' },
]

const saved = (() => { try { return JSON.parse(localStorage.getItem('mana-table-state')) } catch { return null } })()
const tokens = ref(saved?.tokens ?? [])
const life = ref(saved?.life ?? 20)
const showName = ref(false)
const newName = ref('')
const selectionMode = ref(false)
const showDamage = ref(false)
const toast = ref('')
let toastTimer

const selected = computed(() => tokens.value.filter(token => token.selected).length)
const allSelected = computed(() => tokens.value.length > 0 && selected.value === tokens.value.length)
const totalPower = computed(() => tokens.value.reduce((sum, token) => sum + token.power, 0))

watch([tokens, life], ([cards, total]) => localStorage.setItem('mana-table-state', JSON.stringify({ tokens: cards, life: total })), { deep: true })

function addToken() {
  const name = newName.value.trim() || 'Token'
  tokens.value.push({ id: Date.now() + Math.random(), name, power: 1, toughness: 1, color: Math.floor(Math.random() * palette.length), selected: false })
  showName.value = false
  newName.value = ''
  notify(name + ' added to the battlefield')
}
function adjust(token, stat, delta) { token[stat] = Math.max(0, token[stat] + delta) }
function clone(token) { tokens.value.push({ ...token, id: Date.now() + Math.random(), selected: false }); notify(token.name + ' cloned') }
function toggleAll() { tokens.value.forEach(token => { token.selected = !allSelected.value }) }
function removeSelected() {
  const count = selected.value
  tokens.value = tokens.value.filter(token => !token.selected)
  selectionMode.value = false
  notify(count + ' token' + (count === 1 ? '' : 's') + ' removed')
}
function notify(message) {
  toast.value = message
  clearTimeout(toastTimer)
  toastTimer = setTimeout(() => { toast.value = '' }, 2200)
}
function onKey(event) { if (event.key === 'Escape') showDamage.value = false }
onMounted(() => window.addEventListener('keydown', onKey))
onUnmounted(() => { window.removeEventListener('keydown', onKey); clearTimeout(toastTimer) })
</script>

<template>
  <div class="app">
    <Navbar :life="life" @open-damage="showDamage = true" />
    <main class="layout">
      <section class="board">
        <div class="heading">
          <div>
            <div class="eyebrow">YOUR SIDE OF THE TABLE</div>
            <h1>Battlefield</h1>
          </div>
          <div class="count"><b>{{ tokens.length }}</b><small>{{ tokens.length === 1 ? 'CREATURE' : 'CREATURES' }}</small></div>
        </div>
        <div class="toolbar">
          <div class="tools">
            <button class="tool" :class="{ active: selectionMode }" @click="selectionMode = !selectionMode; tokens.forEach(token => token.selected = false)">
              <svg viewBox="0 0 24 24"><path d="M5 5h5v5H5zM14 5h5v5h-5zM5 14h5v5H5zM14 14h5v5h-5z" /></svg>{{ selectionMode ? 'Done' : 'Select' }}
            </button>
            <button v-if="selectionMode" class="tool" @click="toggleAll">
              <i class="mini-check" :class="{ checked: allSelected }">{{ allSelected ? '✓' : '' }}</i>{{ allSelected ? 'Deselect all' : 'Select all' }}
            </button>
          </div>
          <button class="tool damage-link" @click="showDamage = true"><svg viewBox="0 0 24 24"><path d="m13 2-3 8h7l-6 12 1-9H6z" /></svg>Damage</button>
        </div>
        <div v-if="tokens.length" class="grid" :class="{ selecting: selectionMode }">
          <TokenCard v-for="(token, index) in tokens" :key="token.id" :token="token" :palette="palette"
            :selection-mode="selectionMode" :order="index"
            @adjust="(stat, delta) => adjust(token, stat, delta)"
            @clone="clone(token)" @toggle-selection="token.selected = !token.selected" />
        </div>
        <div v-else class="empty">
          <div>✳</div>
          <h2>A clean battlefield.</h2>
          <p>Add a token to start building your side of the table.</p>
        </div>
        <div class="add-area">
          <form v-if="showName" class="name-form" @submit.prevent="addToken">
            <label for="token-name">TOKEN TYPE <span>· OPTIONAL</span></label>
            <div class="name-row">
              <input id="token-name" v-model="newName" placeholder="e.g. Treasure, Spirit..." maxlength="24" autofocus>
              <button class="confirm" type="submit">＋ Add token</button>
              <button class="close" type="button" aria-label="Close" @click="showName = false; newName = ''">×</button>
            </div>
            <small>Leave blank to create a generic token.</small>
          </form>
          <button v-else class="add-token" @click="showName = true; $nextTick(() => document.getElementById('token-name')?.focus())">
            <span class="add-icon">+</span><span><b>Add a token</b><small>Bring something onto the battlefield</small></span><i>＋</i>
          </button>
        </div>
        <div v-if="selectionMode && selected" class="selection-bar">
          <span><b>{{ selected }}</b> selected</span>
          <button @click="removeSelected"><svg viewBox="0 0 24 24"><path d="M4 7h16M10 11v6M14 11v6M5 7l1 14h12l1-14M9 7V4h6v3" /></svg>Remove selected</button>
        </div>
      </section>
      <aside class="aside">
        <div class="aside-card">
          <div class="aside-label">SESSION NOTES</div>
          <h2>A little help at the table.</h2>
          <p>Tap <b>+</b> to add counters. Clone a creature when your board grows. Select several to clear them at once.</p>
          <div class="rule"></div>
          <div class="board-power"><span>BOARD POWER</span><b>{{ totalPower }}<small> total ATK</small></b></div>
          <button class="aside-calc" @click="showDamage = true"><i>↗</i> Open damage calculator</button>
        </div>
      </aside>
    </main>
    <DamageCalculator v-model:show="showDamage" v-model:life="life" :total-power="totalPower" />
    <Transition name="toast">
      <div v-if="toast" class="toast"><span>✳</span>{{ toast }}</div>
    </Transition>
  </div>
</template>

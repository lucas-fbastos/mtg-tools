<script setup>
import { computed, onMounted, onUnmounted, ref, watch } from 'vue'
import DamageCalculator from './components/DamageCalculator.vue'
import Navbar from './components/Navbar.vue'
import SessionNotes from './components/SessionNotes.vue'
import TokenCreator from './components/TokenCreator.vue'
import TokenCard from './components/TokenCard.vue'

const palette = [
  { id: 'white', name: 'White', bg: '#d7c98f', ink: '#332d20', accent: '#9b8548', icon: '☼', dot: '#f3e8c2' },
  { id: 'blue', name: 'Blue', bg: '#76a9c2', ink: '#182f3d', accent: '#315e79', icon: '◈', dot: '#4b91b8' },
  { id: 'black', name: 'Black', bg: '#403b43', ink: '#f2ecdf', accent: '#b2a2b4', icon: '☾', dot: '#2e2932' },
  { id: 'red', name: 'Red', bg: '#c96a4d', ink: '#421f1b', accent: '#85402f', icon: '◆', dot: '#bb4e38' },
  { id: 'green', name: 'Green', bg: '#78956a', ink: '#203221', accent: '#456346', icon: '✳', dot: '#638451' },
  { id: 'colorless', name: 'Colorless', bg: '#a9aaa5', ink: '#30332f', accent: '#6e746f', icon: '◇', dot: '#a5aaa8' },
]

const saved = (() => { try { return JSON.parse(localStorage.getItem('mana-table-state')) } catch { return null } })()
const tokens = ref(saved?.tokens ?? [])
const life = ref(saved?.life ?? 20)
const selectionMode = ref(false)
const useStacks = ref(false)
const showDamage = ref(false)
const toast = ref('')
let toastTimer

const selected = computed(() => tokens.value.filter(token => token.selected).length)
const allSelected = computed(() => tokens.value.length > 0 && selected.value === tokens.value.length)
const totalPower = computed(() => tokens.value.reduce((sum, token) => sum + token.power, 0))
const displayedTokens = computed(() => {
  if (!useStacks.value) return tokens.value.map(token => ({ key: token.id, token, members: [token] }))

  const stacks = new Map()
  for (const token of tokens.value) {
    const tokenColor = typeof token.color === 'number' ? palette[token.color]?.id ?? token.color : token.color
    const key = JSON.stringify([token.name, tokenColor, token.power, token.toughness])
    if (!stacks.has(key)) stacks.set(key, { key, token, members: [] })
    stacks.get(key).members.push(token)
  }
  return [...stacks.values()]
})

watch([tokens, life], ([cards, total]) => localStorage.setItem('mana-table-state', JSON.stringify({ tokens: cards, life: total })), { deep: true })

function addToken({ name, color }) {
  tokens.value.push({ id: Date.now() + Math.random(), name, power: 1, toughness: 1, color, selected: false })
  notify(name + ' added to the battlefield')
}
function adjust(token, stat, delta) { token[stat] = Math.max(0, token[stat] + delta) }
function toggleStackSelection(stack) {
  const shouldSelect = !stack.members.every(token => token.selected)
  stack.members.forEach(token => { token.selected = shouldSelect })
}
function clone(token) { tokens.value.push({ ...token, id: Date.now() + Math.random(), selected: false }); notify(token.name + ' cloned') }
function removeToken(token) {
  tokens.value = tokens.value.filter(card => card.id !== token.id)
  notify(token.name + ' removed')
}
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
            <h1>Manage your Tokens</h1>
          </div>
          <div class="count"><b>{{ tokens.length }}</b><small>{{ tokens.length === 1 ? 'CREATURE' : 'CREATURES' }}</small></div>
        </div>
        <div class="toolbar">
          <div class="tools">
            <button class="tool" :class="{ active: selectionMode }" @click="selectionMode = !selectionMode; tokens.forEach(token => token.selected = false)">
              <svg viewBox="0 0 24 24"><path d="M5 5h5v5H5zM14 5h5v5h-5zM5 14h5v5H5zM14 14h5v5h-5z" /></svg>{{ selectionMode ? 'Done' : 'Select' }}
            </button>
            <button class="tool stack-toggle" :class="{ active: useStacks }" :aria-pressed="useStacks" @click="useStacks = !useStacks">
              <svg viewBox="0 0 24 24"><rect x="5" y="4" width="13" height="15" rx="2"/><path d="M8 7h7M8 10h7M8 13h4M8 21h11a2 2 0 0 0 2-2V7"/></svg>Use stacks
            </button>
            <button v-if="selectionMode" class="tool" @click="toggleAll">
              <i class="mini-check" :class="{ checked: allSelected }">{{ allSelected ? '✓' : '' }}</i>{{ allSelected ? 'Deselect all' : 'Select all' }}
            </button>
          </div>
          <button class="tool damage-link" @click="showDamage = true"><svg viewBox="0 0 24 24"><path d="m13 2-3 8h7l-6 12 1-9H6z" /></svg>Damage</button>
        </div>
        <div v-if="tokens.length" class="grid" :class="{ selecting: selectionMode, 'stacked-grid': useStacks }">
          <TokenCard v-for="(stack, index) in displayedTokens" :key="stack.key" :token="stack.token" :palette="palette"
            :stack-count="stack.members.length" :stacked="useStacks && stack.members.length > 1"
            :selection-mode="selectionMode" :order="index"
            @adjust="(stat, delta) => stack.members.forEach(member => adjust(member, stat, delta))"
            @clone="clone(stack.token)" @delete="removeToken(stack.token)"
            @toggle-selection="toggleStackSelection(stack)" />
        </div>
        <div v-else class="empty">
          <div>✳</div>
          <h2>A clean battlefield.</h2>
          <p>Add a token to start building your side of the table.</p>
        </div>
        <div v-if="selectionMode && selected" class="selection-bar">
          <span><b>{{ selected }}</b> selected</span>
          <button @click="removeSelected"><svg viewBox="0 0 24 24"><path d="M4 7h16M10 11v6M14 11v6M5 7l1 14h12l1-14M9 7V4h6v3" /></svg>Remove selected</button>
        </div>
      </section>
      <aside class="aside">
        <TokenCreator :palette="palette" @create="addToken" />
        <SessionNotes :total-power="totalPower" @open-damage="showDamage = true" />
      </aside>
    </main>
    <DamageCalculator v-model:show="showDamage" v-model:life="life" :total-power="totalPower" />
    <Transition name="toast">
      <div v-if="toast" class="toast"><span>✳</span>{{ toast }}</div>
    </Transition>
  </div>
</template>

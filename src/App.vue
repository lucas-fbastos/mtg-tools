<script setup>
import { computed, onMounted, onUnmounted, ref, watch } from 'vue'
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
const showName = ref(false), newName = ref(''), selectionMode = ref(false), showDamage = ref(false)
const attack = ref(0), blocker = ref(4), calcMode = ref('combat'), toast = ref('')
let toastTimer
const selected = computed(() => tokens.value.filter(t => t.selected).length)
const allSelected = computed(() => tokens.value.length > 0 && selected.value === tokens.value.length)
const totalPower = computed(() => tokens.value.reduce((sum, t) => sum + t.power, 0))
const lethal = computed(() => blocker.value > 0 ? Math.floor(Math.max(0, attack.value) / blocker.value) : 0)
watch([tokens, life], ([cards, total]) => localStorage.setItem('mana-table-state', JSON.stringify({ tokens: cards, life: total })), { deep: true })
function addToken() {
  const name = newName.value.trim() || 'Token'
  tokens.value.push({ id: Date.now() + Math.random(), name, power: 1, toughness: 1, color: Math.floor(Math.random() * palette.length), selected: false })
  showName.value = false; newName.value = ''; notify(name + ' added to the battlefield')
}
function adjust(token, stat, delta) { token[stat] = Math.max(0, token[stat] + delta) }
function clone(token) { tokens.value.push({ ...token, id: Date.now() + Math.random(), selected: false }); notify(token.name + ' cloned') }
function toggleAll() { tokens.value.forEach(t => { t.selected = !allSelected.value }) }
function removeSelected() { const count = selected.value; tokens.value = tokens.value.filter(t => !t.selected); selectionMode.value = false; notify(count + ' token' + (count === 1 ? '' : 's') + ' removed') }
function notify(message) { toast.value = message; clearTimeout(toastTimer); toastTimer = setTimeout(() => { toast.value = '' }, 2200) }
function onKey(e) { if (e.key === 'Escape') showDamage.value = false }
onMounted(() => window.addEventListener('keydown', onKey))
onUnmounted(() => { window.removeEventListener('keydown', onKey); clearTimeout(toastTimer) })
</script>
<template>
  <div class="app">
    <header class="topbar">
      <a class="brand" href="#"><span class="brand-mark">✳</span><span class="brand-name">MTG <span>Tools</span></span></a>
      <button class="life-pill" @click="showDamage=true"><span>YOUR LIFE</span><b>{{ life }}</b><em>♥</em></button>
    </header>
    <main class="layout">
      <section class="board">
        <div class="heading"><div><div class="eyebrow">YOUR SIDE OF THE TABLE</div><h1>Battlefield<span>.</span></h1></div><div class="count"><b>{{ tokens.length }}</b><small>{{ tokens.length === 1 ? 'CREATURE' : 'CREATURES' }}</small></div></div>
        <div class="toolbar"><div class="tools">
          <button class="tool" :class="{active:selectionMode}" @click="selectionMode=!selectionMode; tokens.forEach(t=>t.selected=false)"><svg viewBox="0 0 24 24"><path d="M5 5h5v5H5zM14 5h5v5h-5zM5 14h5v5H5zM14 14h5v5h-5z"/></svg>{{ selectionMode?'Done':'Select' }}</button>
          <button v-if="selectionMode" class="tool" @click="toggleAll"><i class="mini-check" :class="{checked:allSelected}">{{ allSelected?'✓':'' }}</i>{{ allSelected?'Deselect all':'Select all' }}</button>
        </div><button class="tool damage-link" @click="showDamage=true"><svg viewBox="0 0 24 24"><path d="m13 2-3 8h7l-6 12 1-9H6z"/></svg>Damage</button></div>
        <div v-if="tokens.length" class="grid" :class="{selecting:selectionMode}">
          <article v-for="(token,i) in tokens" :key="token.id" class="token" :class="{chosen:token.selected}" :style="{'--bg':palette[token.color]?.bg||palette[0].bg,'--ink':palette[token.color]?.ink||palette[0].ink,'--accent':palette[token.color]?.accent||palette[0].accent,'--order':i}">
            <button v-if="selectionMode" class="check" :aria-label="token.selected?'Deselect token':'Select token'" @click="token.selected=!token.selected">{{ token.selected?'✓':'' }}</button>
            <div class="card-top"><span class="symbol">{{ palette[token.color]?.icon||'✳' }}</span><span class="kind">CREATURE TOKEN</span><button class="clone" aria-label="Clone token" @click="clone(token)"><svg viewBox="0 0 24 24"><path d="M8 8V5a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v9a2 2 0 0 1-2 2h-3M5 8h9a2 2 0 0 1 2 2v9a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-9a2 2 0 0 1 2-2Z"/></svg></button></div>
            <div class="art"><i></i><i></i><span>{{ palette[token.color]?.icon||'✳' }}</span></div>
            <div class="token-name">{{ token.name }}</div><div class="subtitle">TOKEN CREATURE</div>
            <div class="stats">
              <div class="stat"><button @click="adjust(token,'power',1)" aria-label="Increase attack">+</button><small>ATK</small><b>{{ token.power }}</b></div>
              <button class="pump" aria-label="Add plus one plus one counter" @click="adjust(token,'power',1);adjust(token,'toughness',1)">+<small>/+</small></button>
              <div class="stat"><button @click="adjust(token,'toughness',1)" aria-label="Increase defense">+</button><small>DEF</small><b>{{ token.toughness }}</b></div>
            </div>
            <div class="substats"><button @click="adjust(token,'power',-1)">− ATK</button><i></i><button @click="adjust(token,'toughness',-1)">− DEF</button></div>
          </article>
        </div>
        <div v-else class="empty"><div>✳</div><h2>A clean battlefield.</h2><p>Add a token to start building your side of the table.</p></div>
        <div class="add-area">
          <form v-if="showName" class="name-form" @submit.prevent="addToken"><label for="token-name">TOKEN TYPE <span>· OPTIONAL</span></label><div class="name-row"><input id="token-name" v-model="newName" placeholder="e.g. Treasure, Spirit..." maxlength="24" autofocus><button class="confirm" type="submit">＋ Add token</button><button class="close" type="button" aria-label="Close" @click="showName=false;newName=''">×</button></div><small>Leave blank to create a generic token.</small></form>
          <button v-else class="add-token" @click="showName=true;$nextTick(()=>document.getElementById('token-name')?.focus())"><span class="add-icon">+</span><span><b>Add a token</b><small>Bring something onto the battlefield</small></span><i>＋</i></button>
        </div>
        <div v-if="selectionMode&&selected" class="selection-bar"><span><b>{{ selected }}</b> selected</span><button @click="removeSelected"><svg viewBox="0 0 24 24"><path d="M4 7h16M10 11v6M14 11v6M5 7l1 14h12l1-14M9 7V4h6v3"/></svg>Remove selected</button></div>
      </section>
      <aside class="aside"><div class="aside-card"><div class="aside-label">SESSION NOTES</div><h2>A little help at the table.</h2><p>Tap <b>+</b> to add counters. Clone a creature when your board grows. Select several to clear them at once.</p><div class="rule"></div><div class="board-power"><span>BOARD POWER</span><b>{{ totalPower }}<small> total ATK</small></b></div><button class="aside-calc" @click="showDamage=true"><i>↗</i> Open damage calculator</button></div></aside>
    </main>
    <div v-if="showDamage" class="backdrop" @click.self="showDamage=false">
      <section class="modal" role="dialog" aria-modal="true" aria-labelledby="calc-title"><button class="modal-close" @click="showDamage=false" aria-label="Close">×</button><div class="eyebrow">THE MATH, MADE EASY</div><h2 id="calc-title">Damage calculator<span>.</span></h2>
        <div class="tabs"><button :class="{current:calcMode==='combat'}" @click="calcMode='combat'">Combat damage</button><button :class="{current:calcMode==='life'}" @click="calcMode='life'">Life total</button></div>
        <template v-if="calcMode==='combat'"><p class="intro">See how many blockers your creatures can take down.</p>
          <div class="calc-inputs"><label><span>YOUR TOTAL ATTACK</span><div class="num"><button @click="attack=Math.max(0,attack-1)">−</button><input v-model.number="attack" type="number" min="0" inputmode="numeric"><button @click="attack++">+</button></div></label><b class="divide">÷</b><label><span>BLOCKER TOUGHNESS</span><div class="num"><button @click="blocker=Math.max(1,blocker-1)">−</button><input v-model.number="blocker" type="number" min="1" inputmode="numeric"><button @click="blocker++">+</button></div></label></div>
          <div class="result"><div class="result-symbol">✳</div><div><span>YOU CAN DEAL LETHAL TO</span><b>{{ lethal }} <small>{{ lethal===1?'CREATURE':'CREATURES' }}</small></b></div><p>{{ attack }} total damage ÷ {{ blocker }} toughness, rounded down</p></div><button class="fill" @click="attack=totalPower">Use board power <span>{{ totalPower }} ATK →</span></button>
        </template>
        <template v-else><p class="intro">Adjust your life total as damage comes in.</p><div class="life-adjust"><button @click="life=Math.max(0,life-1)" aria-label="Lose one life">−</button><div><span>YOUR LIFE</span><b>{{ life }}</b></div><button @click="life++" aria-label="Gain one life">+</button></div><div class="quick"><span>QUICK DAMAGE</span><button v-for="n in [1,2,3,5]" :key="n" @click="life=Math.max(0,life-n)">−{{ n }}</button></div></template>
        <button class="done" @click="showDamage=false">Back to the battlefield</button>
      </section>
    </div>
    <Transition name="toast"><div v-if="toast" class="toast"><span>✳</span>{{ toast }}</div></Transition>
  </div>
</template>

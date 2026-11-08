<script setup>
import { computed } from 'vue'

const props = defineProps({
  token: { type: Object, required: true },
  palette: { type: Array, required: true },
  selectionMode: { type: Boolean, default: false },
  order: { type: Number, default: 0 },
})
const emit = defineEmits(['adjust', 'clone', 'toggle-selection'])
const colors = computed(() => props.palette[props.token.color] || props.palette[0])
const cardStyle = computed(() => ({
  '--bg': colors.value.bg,
  '--ink': colors.value.ink,
  '--accent': colors.value.accent,
  '--order': props.order,
}))
function adjust(stat, delta) { emit('adjust', stat, delta) }
</script>

<template>
  <article class="token" :class="{ chosen: token.selected }" :style="cardStyle">
    <button v-if="selectionMode" class="check" :aria-label="token.selected ? 'Deselect token' : 'Select token'"
      @click="emit('toggle-selection')">{{ token.selected ? '✓' : '' }}</button>
    <div class="card-top">
      <span class="token-name">Token</span>
      <button class="clone" aria-label="Clone token" @click="emit('clone')">
        <svg viewBox="0 0 24 24"><path d="M8 8V5a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v9a2 2 0 0 1-2 2h-3M5 8h9a2 2 0 0 1 2 2v9a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-9a2 2 0 0 1 2-2Z" /></svg>
      </button>
    </div>
    <div class="art"><i></i><i></i><span>{{ colors.icon }}</span></div>
    <div class="subtitle"><span class="symbol">{{ colors.icon }}</span><span class="type-label">Token Creature — {{ token.name }}</span></div>
    <div class="card-footer">
      <button class="pump" aria-label="Increase attack and defense by one"
        @click="adjust('power', 1); adjust('toughness', 1)">+1/+1</button>
      <div class="power-toughness" :aria-label="`${token.power} attack, ${token.toughness} defense`">
        <div class="pt-stat">
          <b>{{ token.power }}</b>
          <span class="pt-arrows">
            <button aria-label="Increase attack" @click="adjust('power', 1)"><span class="triangle up"></span></button>
            <button aria-label="Decrease attack" @click="adjust('power', -1)"><span class="triangle down"></span></button>
          </span>
        </div>
        <span class="pt-slash">/</span>
        <div class="pt-stat">
          <b>{{ token.toughness }}</b>
          <span class="pt-arrows">
            <button aria-label="Increase defense" @click="adjust('toughness', 1)"><span class="triangle up"></span></button>
            <button aria-label="Decrease defense" @click="adjust('toughness', -1)"><span class="triangle down"></span></button>
          </span>
        </div>
      </div>
    </div>
  </article>
</template>

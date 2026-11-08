<script setup>
defineProps({
  totalPower: { type: Number, required: true },
})

const emit = defineEmits(['open-damage'])
</script>

<template>
  <div class="aside-card">
    <div class="aside-label">SESSION NOTES</div>
    <h2>A little help at the table.</h2>
    <p>Tap <b>+</b> to add counters. Clone a creature when your board grows. Select several to clear them at once.</p>
    <div class="rule"></div>
    <div class="board-power"><span>BOARD POWER</span><b>{{ totalPower }}<small> total ATK</small></b></div>
    <button class="aside-calc" @click="emit('open-damage')"><i>↗</i> Open damage calculator</button>
  </div>
</template>

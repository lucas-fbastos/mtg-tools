<script setup>
defineProps({ life: { type: Number, required: true } })
defineEmits(['open-damage'])
</script>

<template>
  <header class="topbar">
    <a class="brand" href="#"><span class="brand-mark">✳</span><span class="brand-name">MTG <span>Tools</span></span></a>
    <button class="life-pill" @click="$emit('open-damage')"><span>YOUR LIFE</span><b>{{ life }}</b><em>♥</em></button>
  </header>
</template>

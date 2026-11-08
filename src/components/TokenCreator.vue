<script setup>
import { computed, nextTick, ref } from 'vue'

const props = defineProps({
  palette: { type: Array, required: true },
})
const emit = defineEmits(['create'])

const showForm = ref(false)
const name = ref('')
const color = ref('colorless')
const showColorPicker = ref(false)
const selectedColor = computed(() => props.palette.find(option => option.id === color.value) || props.palette.at(-1))

function openForm() {
  showForm.value = true
  nextTick(() => document.getElementById('token-name')?.focus())
}

function closeForm() {
  showForm.value = false
  name.value = ''
  color.value = 'colorless'
  showColorPicker.value = false
}

function createToken() {
  emit('create', { name: name.value.trim() || 'Token', color: color.value })
  closeForm()
}
</script>

<template>
  <div class="token-creator">
    <form v-if="showForm" class="name-form" @submit.prevent="createToken">
      <label for="token-name">TOKEN TYPE <span>· OPTIONAL</span></label>
      <div class="name-row">
        <input id="token-name" v-model="name" placeholder="e.g. Treasure, Spirit..." maxlength="24" autofocus>
        <button class="confirm" type="submit">＋ Add token</button>
        <button class="close" type="button" aria-label="Close" @click="closeForm">×</button>
      </div>
      <div class="color-picker-row">
        <span class="selected-color"><i :style="{ '--dot-color': selectedColor.dot }"></i>{{ selectedColor.name }}</span>
        <button class="pick-color" type="button" :aria-expanded="showColorPicker" @click="showColorPicker = !showColorPicker">
          {{ showColorPicker ? 'Close colors' : 'Pick color' }} <span aria-hidden="true">⌄</span>
        </button>
      </div>
      <div v-if="showColorPicker" class="color-picker" role="radiogroup" aria-label="Token color">
        <button v-for="(option, index) in palette" :key="option.id" type="button" role="radio"
          :aria-checked="color === option.id" :aria-label="option.name" :title="option.name"
          class="color-choice" :class="[`color-${option.id}`, { selected: color === option.id }]"
          :style="{ '--dot-color': option.dot, '--x': index === 0 ? '50%' : index === 1 ? '12%' : index === 2 ? '28%' : index === 3 ? '72%' : index === 4 ? '88%' : '50%', '--y': index === 0 ? '12%' : index === 1 ? '34%' : index === 2 ? '78%' : index === 3 ? '78%' : index === 4 ? '34%' : '50%' }"
          @click="color = option.id">
          <i></i>
        </button>
      </div>
      <small>Leave blank to create a generic token.</small>
    </form>
    <button v-else class="add-token" aria-label="Add a token" @click="openForm">
      <span class="add-icon">+</span><span><b>Add a token</b><small>Bring something onto the battlefield</small></span><i>＋</i>
    </button>
  </div>
</template>

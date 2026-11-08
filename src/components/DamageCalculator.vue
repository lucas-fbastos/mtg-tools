<script setup>
import { computed, ref } from 'vue'

const props = defineProps({
  show: { type: Boolean, default: false },
  life: { type: Number, required: true },
  totalPower: { type: Number, required: true },
})
const emit = defineEmits(['update:show', 'update:life'])
const attack = ref(0)
const blocker = ref(4)
const calcMode = ref('combat')
const lethal = computed(() => blocker.value > 0 ? Math.floor(Math.max(0, attack.value) / blocker.value) : 0)
function setLife(value) { emit('update:life', Math.max(0, value)) }
</script>

<template>
  <div v-if="show" class="backdrop" @click.self="emit('update:show', false)">
    <section class="modal" role="dialog" aria-modal="true" aria-labelledby="calc-title">
      <button class="modal-close" @click="emit('update:show', false)" aria-label="Close">×</button>
      <h2 id="calc-title">Damage calculator<span>.</span></h2>
      <div class="tabs">
        <button :class="{ current: calcMode === 'combat' }" @click="calcMode = 'combat'">Combat damage</button>
        <button :class="{ current: calcMode === 'life' }" @click="calcMode = 'life'">Life total</button>
      </div>
      <template v-if="calcMode === 'combat'">
        <p class="intro">See how many blockers your creatures can take down.</p>
        <div class="calc-inputs">
          <label><span>YOUR TOTAL ATTACK</span>
            <div class="num"><button @click="attack = Math.max(0, attack - 1)">−</button><input v-model.number="attack" type="number" min="0" inputmode="numeric"><button @click="attack++">+</button></div>
          </label>
          <b class="divide">÷</b>
          <label><span>BLOCKER TOUGHNESS</span>
            <div class="num"><button @click="blocker = Math.max(1, blocker - 1)">−</button><input v-model.number="blocker" type="number" min="1" inputmode="numeric"><button @click="blocker++">+</button></div>
          </label>
        </div>
        <div class="result">
          <div class="result-symbol">✳</div>
          <div><span>YOU CAN DEAL LETHAL TO</span><b>{{ lethal }} <small>{{ lethal === 1 ? 'CREATURE' : 'CREATURES' }}</small></b></div>
          <p>{{ attack }} total damage ÷ {{ blocker }} toughness, rounded down</p>
        </div>
        <button class="fill" @click="attack = totalPower">Use board power <span>{{ totalPower }} ATK →</span></button>
      </template>
      <template v-else>
        <p class="intro">Adjust your life total as damage comes in.</p>
        <div class="life-adjust">
          <button @click="setLife(props.life - 1)" aria-label="Lose one life">−</button>
          <div><span>YOUR LIFE</span><b>{{ life }}</b></div>
          <button @click="setLife(props.life + 1)" aria-label="Gain one life">+</button>
        </div>
        <div class="quick"><span>QUICK DAMAGE</span><button v-for="n in [1, 2, 3, 5]" :key="n" @click="setLife(props.life - n)">−{{ n }}</button></div>
      </template>
      <button class="done" @click="emit('update:show', false)">Back to the battlefield</button>
    </section>
  </div>
</template>

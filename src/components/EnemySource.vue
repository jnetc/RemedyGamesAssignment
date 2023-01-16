<script setup lang="ts">
import { defineEmits, defineProps, toRef, ref } from 'vue';

let oneEnemy = ref(500);
const emit = defineEmits<{
  (event: 'onClick', payload: number): number;
}>();
</script>

<template>
  <div class="input-element">
    <label for="one-source">One enemy source {{ oneEnemy }}</label>
    <input type="text" name="one" id="one-source" v-model.number="oneEnemy" />
    <button class="add-source-btn" @click="emit('onClick', oneEnemy)">Check</button>
  </div>
</template>

<style></style>

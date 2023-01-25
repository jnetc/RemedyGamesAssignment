<script setup lang="ts">
import SecondApproach from './components/SecondApproach.vue'
import FirstApproach from './components/FirstApproach.vue'
</script>

<template>
  <first-approach></first-approach>
  <second-approach></second-approach>
</template>

<style>
:root {
  --ui-element: hsl(60, 14%, 99%);
  --bar-color: hsl(187, 86%, 41%);
  --bar-backdrop: hsl(162, 13%, 26%);
  --add-hp: hsl(169, 67%, 43%);
  --remove-hp: hsl(0, 67%, 43%);
}

.scene {
  justify-self: end;
  width: 400px;
  aspect-ratio: 1;
  position: relative;
}

/* UI CONTROL PANEL */
.elements-ui {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.5rem 1rem;
}

.element__input {
  display: flex;
  flex-direction: column;
  gap: 0.3rem;
}

.element__input label {
  width: max-content;
  font-size: 0.9rem;
}

.element__input input {
  padding: 0.3rem 0.1rem;
  width: 100%;
  font-size: 1.1rem;
}

.element__input button {
  margin-block-start: 0.3rem;
  padding: 0.4rem 1rem;
  font-size: 1.1rem;
  font-weight: bold;
}

.add-source-btn {
  color: var(--bar-backdrop);
  background-color: var(--add-hp);
}

.remove-source-btn {
  color: var(--ui-element);
  background-color: var(--remove-hp);
}
</style>

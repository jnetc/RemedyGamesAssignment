<script setup lang="ts">
import { ref, computed } from 'vue';

const PROGRESS_BAR_WIDTH = 160; // Units '160px' = '10rem'
const ALL_SOURCE_POINTS = 2000; // All points includes in progress bar
let COUNTER_SPEED = 20; // min 15 ->
const COUNTER_DURATION = ref(500); // 500ms
let DEATH_VALUE: number = 10; // 10% of 100
const PROGRESS = ref(0);
const ONCE = ref(false); // Prevent to store the animation events

const initSourceValue = ref(0);
const initCounterValue = ref(0);

const sourceUIRef = ref<HTMLElement | null>(null);
const counterRef = ref<HTMLElement | null>(null);
const amountRef = ref<HTMLElement | null>(null);
const progressRef = ref<HTMLElement | null>(null);

// Initial enemy values
let oneEnemy: number = 400;
let secondEnemy: number = 330;
let thirdEnemy: number = 250;
let fourthEnemy: number = 100;

// Randomize resouce value
const max = ref(0);
const min = ref(0);
const forMax = 300;
const forMin = 100;

function increaseValue(source: number) {
  // :style="{ transitionDuration: `${currTime}s` }"
  // initCounterDuration.value = source / INITIAL_TIME;

  // Calculate min and max values for each iteration in current source
  max.value = (source / ALL_SOURCE_POINTS) * forMax;
  min.value = (source / ALL_SOURCE_POINTS) * forMin;

  sourceUIRef.value?.classList.add('source-UI__show');

  // Calling the counter when source-UI class will be visible
  sourceUIRef.value?.addEventListener('transitionend', () => counterCalc(source), { once: true });
}

function counterCalc(source: number) {
  const step = setInterval(() => {
    // Increasing and saving random value for each iteration
    let accumulation = initCounterValue.value + Math.floor(Math.random() * (max.value - min.value));
    // Checking, and then stopping, resseting iteration
    if (accumulation >= source) {
      // Adding the last value to the counter
      initCounterValue.value = source;

      counterRef.value?.addEventListener('animationend', (e: AnimationEvent) => steps(e, source), {
        once: ONCE.value,
      });

      sourceUIRef.value?.addEventListener('transitionend', () => increaseProgress(source), {
        once: true,
      });

      clearInterval(step);
      return;
    }

    counterRef.value?.classList.add('move-to-show');
    // Seting accumulation value to the initial value
    initCounterValue.value = accumulation;
  }, COUNTER_SPEED);
}

// Checking the current animation name and doing some action inside each step
function steps(event: AnimationEvent, source: number) {
  if (event.animationName === 'counter-show') {
    console.log('step 1', event.animationName);

    commonSourceCalc(source);
    counterRef.value?.classList.replace('move-to-show', 'move-to-hide');
  }
  if (event.animationName === 'counter-hide') {
    console.log('step_2', event.animationName);

    initCounterValue.value = 0;
    counterRef.value?.classList.remove('move-to-hide');
    sourceUIRef.value?.classList.remove('source-UI__show');
    ONCE.value = true;
  }
}

function commonSourceCalc(source: number) {
  const accBreak = initSourceValue.value + source;

  const step = setInterval(() => {
    // Increasing and saving random value for each iteration
    let accumulation = initSourceValue.value + Math.floor(Math.random() * (max.value - min.value));
    // Checking, and then stopping iteration and cleaning up
    if (accumulation >= accBreak) {
      // Adding the last value to the counter
      initSourceValue.value = accBreak;
      clearInterval(step);
      return;
    }
    // Seting accumulation value to the initial value
    initSourceValue.value = accumulation;
  }, COUNTER_SPEED);
}

function increaseProgress(step: number) {
  // Calculating, how many px we will add on this step
  let calc = (PROGRESS_BAR_WIDTH * step) / ALL_SOURCE_POINTS;
  // Adding the calculated step to previous value
  return (PROGRESS.value += calc);
}

function switchProgressClass(event: Event) {
  let text = event.currentTarget as HTMLButtonElement;
  if (text.textContent === 'Smooth') {
    text.textContent = 'Steps';
  } else {
    text.textContent = 'Smooth';
  }
  progressRef.value?.classList.toggle('steps');
}
</script>

<template>
  <main>
    <section class="scene">
      <div
        class="source-UI"
        ref="sourceUIRef"
        :style="{ transitionDuration: `${COUNTER_DURATION}ms` }"
      >
        <div class="source-UI-icon">
          <svg viewBox="0 0 34 32" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path
              d="M0.998199 31.9837L6.43541 11.4877C6.46181 11.3882 6.60326 11.3887 6.62891 11.4885L11.8997 31.9837H0.998199Z"
              fill="white"
            />
            <path
              d="M23.0895 31.9837L28.5267 11.4877C28.5531 11.3882 28.6946 11.3887 28.7202 11.4885L33.991 31.9837H23.0895Z"
              fill="white"
            />
            <path
              d="M11.9192 21.3689L17.3564 0.872988C17.3828 0.773454 17.5243 0.773991 17.5499 0.873723L22.8207 21.3689H11.9192Z"
              fill="white"
            />
          </svg>
        </div>
        <span class="source-UI-amount" ref="amountRef">{{ initSourceValue }}</span>
        <span
          class="source-UI-counter"
          ref="counterRef"
          :style="{ animationDuration: `${COUNTER_DURATION}ms` }"
          >{{ initCounterValue }}</span
        >
      </div>
      <div class="progress-bar">
        <div class="progress" ref="progressRef" :style="{ width: `${PROGRESS}px` }"></div>
      </div>
      <img src="/bg.webp" alt="bg" />
    </section>
    <section class="ui-elements">
      <div class="input-element">
        <label for="one-source">One enemy source</label>
        <input type="text" name="one" id="one-source" v-model.number.trim="oneEnemy" />
        <button class="add-source-btn" @click="increaseValue(oneEnemy)">Check</button>
      </div>
      <div class="input-element">
        <label for="two-source">Second enemy source</label>
        <input type="text" name="two" id="two-source" v-model.number.trim="secondEnemy" />
        <button class="add-source-btn" @click="increaseValue(secondEnemy)">Check</button>
      </div>
      <div class="input-element">
        <label for="three-source">Third enemy source</label>
        <input type="text" name="three" id="three-source" v-model.number.trim="thirdEnemy" />
        <button class="add-source-btn" @click="increaseValue(thirdEnemy)">Check</button>
      </div>
      <div class="input-element">
        <label for="fourth-source">Fourth enemy source</label>
        <input type="text" name="fourth" id="fourth-source" v-model.number.trim="fourthEnemy" />
        <button class="add-source-btn" @click="increaseValue(fourthEnemy)">Check</button>
      </div>
      <div class="input-element">
        <label for="remove-source">Remove source %</label>
        <input type="text" name="remove" id="remove-source" v-model.number.trim="DEATH_VALUE" />
        <button class="remove-source-btn">Remove</button>
      </div>
      <div class="input-element">
        <label for="remove-source">Bar animation smooth/steps</label>
        <button class="source-btn" @click="switchProgressClass">Smooth</button>
      </div>
      <div class="input-element">
        <label for="duration">Counter animation (ms)</label>
        <input
          type="text"
          name="duration"
          id="duration"
          placeholder="Default 500ms"
          v-model.number="COUNTER_DURATION"
        />
      </div>
    </section>
  </main>
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
img {
  max-width: 100%;
  height: auto;
}
/* PROGRESS BAR */
.progress-bar {
  width: 10rem;
  height: 1rem;
  position: absolute;
  left: 3rem;
  bottom: 5rem;
  background-color: var(--bar-backdrop);
}
/* ICON cross  */
.progress-bar::after,
.progress-bar::before {
  content: '';
  height: inherit;
  aspect-ratio: 1 / 3.1;
  position: absolute;
  top: 0;
  left: -1.1rem;
  background-color: var(--bar-color);
}
.progress-bar::before {
  transform: rotate(90deg);
}
.progress {
  position: absolute;
  inset-block: 0 0;
  background-color: var(--bar-color);
  transition-duration: 0.5s;
  transition-property: width;
  transition-timing-function: ease-in-out;
}
.progress.steps {
  transition-timing-function: steps(4, jump-start);
}

/* SOURCE component*/
.source-UI {
  display: grid;
  grid-template-columns: 1.3rem auto;
  column-gap: 0.5rem;
  align-items: baseline;
  position: absolute;
  left: 3rem;
  bottom: 7.5rem;
  opacity: 0;
  /* duration changes dynamically */
  /* transition-duration: 0.5s; */
  transition-property: opacity;
  transition-timing-function: ease-in-out;
}
.source-UI__show {
  opacity: 1;
}

.source-UI-amount,
.source-UI-counter {
  grid-column: 2;
  grid-row: 1;
  font-family: sans-serif;
  font-weight: bold;
  user-select: none;
}

.source-UI-counter {
  opacity: 0;
  color: var(--add-hp);
  transform: translate3d(0px, -2.25rem, 0);
}
/* ICON cross */
.source-UI-counter::after,
.source-UI-counter::before {
  content: '';
  height: 0.5rem;
  aspect-ratio: 1 / 3.3;
  position: absolute;
  inset-block: 25%;
  left: -0.5rem;
  background-color: var(--add-hp);
}
.source-UI-counter::before {
  transform: rotate(90deg);
}

/* transition counter classes */
.move-to-show,
.move-to-hide {
  animation-timing-function: ease-in-out;
  animation-fill-mode: forwards;
}
.move-to-show {
  animation-name: counter-show;
}
.move-to-hide {
  animation-name: counter-hide;
}

@keyframes counter-show {
  0% {
    transform: translate3d(0px, -2.25rem, 0);
    opacity: 0;
  }
  100% {
    transform: translate3d(0px, -1.125rem, 0);
    opacity: 1;
  }
}
@keyframes counter-hide {
  from {
    transform: translate3d(0px, -1.125rem, 0);
    opacity: 1;
  }
  to {
    transform: translate3d(0px, 0, 0);
    opacity: 0;
  }
}

/* UI CONTROL PANEL */
.ui-elements {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.5rem 1rem;
}
.input-element {
  /* width: min-content; */
  display: flex;
  flex-direction: column;
  gap: 0.3rem;
}
.input-element label {
  width: max-content;
  font-size: 0.9rem;
}
.input-element input {
  padding: 0.3rem 0.1rem;
  width: 100%;
  font-size: 1.1rem;
}
.input-element button {
  margin-block-start: 0.3rem;
  padding: 0.4rem 1rem;
  font-size: 1.1rem;
  font-weight: bold;
}
.input-element:last-child button {
  margin-block-start: 0;
}
.add-source {
  /* color: var(--ui-element); */
  background-color: var(--bar-backdrop);
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

<script setup lang="ts">
import { ref, onMounted, onUpdated } from 'vue';

const PROGRESS_BAR_WIDTH = 160; // Units '160px' = '10rem'
const ALL_SOURCE_POINTS = 2000; // All points includes in progress bar
const INITIAL_TIME = 500; // ms
let SPEED_ANIMATION = 20;
let DEATH_VALUE: number = 10; // 10% of 100
let PROGRESS = 0;
// const PLAY = ref(false);

const initialCounterValue = ref(0);
const initialSourceValue = ref(0);
// const currTime = ref(0);

const counterRef = ref<HTMLElement | null>(null);
const amountRef = ref<HTMLElement | null>(null);
const sourceUIRef = ref<HTMLElement | null>(null);

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
const setToPauseCounter = 500; // ms
const hideCounterAfter = setToPauseCounter + 500; // where 500 = 0.5s in css 'source-UI-counter' class

function increaseValue(source: number) {
  // Find appropriate duration time for current source
  // :style="{ transitionDuration: `${currTime}s` }"
  // currTime.value = source / INITIAL_TIME;

  max.value = (source / ALL_SOURCE_POINTS) * forMax;
  min.value = (source / ALL_SOURCE_POINTS) * forMin;

  sourceUIRef.value?.classList.add('source-UI__show');

  sourceUIRef.value?.addEventListener(
    'transitionend',
    () => {
      counterCalc(source);
    },
    { once: true }
  );
}

function counterCalc(source: number) {
  const step = setInterval(() => {
    // Increasing and saving random value for each iteration
    let accumulation =
      initialCounterValue.value + Math.floor(Math.random() * (max.value - min.value));
    // Checking, and then stopping iteration and cleaning up
    if (accumulation >= source) {
      // Adding the last value to the counter
      initialCounterValue.value = source;
      // Small break before continuing
      const addPause = setTimeout(() => {
        counterRef.value?.classList.replace('move-to-show', 'move-to-hide');
        increaseProgress(source);
        commonSourceCalc(source);
      }, setToPauseCounter);
      // Hiding the counter and cleanig timeouts / interval
      const removePause = setTimeout(() => {
        counterRef.value?.classList.remove('move-to-hide');
        resetToInitialValue();
      }, hideCounterAfter);

      // Hide
      counterRef.value?.addEventListener(
        'transitionend',
        () => {
          const hideSourceUI = setTimeout(() => {
            sourceUIRef.value?.classList.remove('source-UI__show');
            // Clear all timeouts
            clearTimeout(addPause);
            clearTimeout(removePause);
            clearTimeout(hideSourceUI);
          }, 1500);
        },
        { once: true }
      );
      clearInterval(step);
      return;
    }

    // console.log('_source');
    counterRef.value?.classList.add('move-to-show');
    // Seting accumulation value to the initial value
    initialCounterValue.value = accumulation;
  }, SPEED_ANIMATION);
}

function commonSourceCalc(source: number) {
  const accBreak = initialSourceValue.value + source;

  const step = setInterval(() => {
    // Increasing and saving random value for each iteration
    let accumulation =
      initialSourceValue.value + Math.floor(Math.random() * (max.value - min.value));
    // Checking, and then stopping iteration and cleaning up
    if (accumulation >= accBreak) {
      // Adding the last value to the counter
      initialSourceValue.value = accBreak;
      clearInterval(step);
      return;
    }
    // Seting accumulation value to the initial value
    initialSourceValue.value = accumulation;
  }, SPEED_ANIMATION);
}

function increaseProgress(step: number) {
  // Calculating, how many px we will add on this step
  let calc = (PROGRESS_BAR_WIDTH * step) / ALL_SOURCE_POINTS;
  // Adding the calculated step to previous value
  return (PROGRESS += calc);
}

function hideUIElement(event: Event) {
  // console.log(event);
}

function resetToInitialValue() {
  // Resetting counter to initial value for prepare to the next iteration
  const resetCounterValue = setTimeout(() => {
    initialCounterValue.value = 0;
    clearTimeout(resetCounterValue);
  }, 1000);
}
</script>

<template>
  <main>
    <section class="scene">
      <div class="source-UI" ref="sourceUIRef">
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
        <!-- <span class="source-UI-amount" ref="amountRef">+0</span> -->
        <span class="source-UI-amount" ref="amountRef">{{ initialSourceValue }}</span>
        <span class="source-UI-counter" ref="counterRef" @transitionend="hideUIElement">{{
          initialCounterValue
        }}</span>
      </div>
      <div class="progress-bar">
        <div class="progress" :style="{ width: `${PROGRESS}px` }"></div>
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
        <label for="speed">Speed animation</label>
        <input type="text" name="speed" id="speed" v-model.number="SPEED_ANIMATION" />
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
  resize: both;
}
img {
  max-width: 100%;
  height: auto;
}
/* Progress bar */
.progress-bar {
  width: 10rem;
  height: 1rem;
  position: absolute;
  left: 3rem;
  bottom: 5rem;
  background-color: var(--bar-backdrop);
}
/* Cross ICON */
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
  transition: all 0.3s ease-in-out;
}

/* UI save up component*/
.source-UI {
  display: grid;
  grid-template-columns: 1.3rem auto;
  column-gap: 0.5rem;
  align-items: baseline;
  position: absolute;
  left: 3rem;
  bottom: 8.5rem;
  opacity: 0; /* METKA */
  transition: opacity 0.5s ease-in-out;
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
  opacity: 0; /* METKA */
  color: var(--add-hp);
  transition-duration: 0.5s;
  transition-property: opacity transform;
  transition-timing-function: ease-in-out;
  transform: translate3d(0px, -2.25rem, 0);
}
/* Cross ICON */
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
/* transition counter */
.move-to-show {
  opacity: 1;
  transform: translate3d(0px, -1.125rem, 0);
}
.move-to-hide {
  transform: translate3d(0px, 0, 0);
  opacity: 0;
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
  padding: 0.5rem 1rem;
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

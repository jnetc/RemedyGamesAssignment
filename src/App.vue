<script setup lang="ts">
import { ref, Ref, onMounted, onUpdated, computed } from 'vue';
import EnemySource from './components/EnemySource.vue';

const PROGRESS_BAR_WIDTH = 160; // Units '160px' = '10rem'
const ALL_SOURCE_POINTS = 2000; // All points includes in progress bar
let COUNTER_SPEED = 20; // min 15 ->
let COUNTER_DURATION = ref(500); // 500ms
let PROGRESS = 0;

const initSourceValue = ref(0);
const initCounterValue = ref(0);
const initCounterSum = ref(0);

// HTML Elements
const sourceUIRef = ref<HTMLElement | null>(null);
const counterRef = ref<HTMLElement | null>(null);
const amountRef = ref<HTMLElement | null>(null);
const progressRef = ref<HTMLElement | null>(null);

// Initial enemy values
let ONE_ENEMY: number = 1800;
let SECOND_ENEMY: number = 530;
let THIRD_ENEMY: number = 250;
let FOURTH_ENEMY: number = 100;
let DEATH_VALUE: number = 10; // 10% of 100

// Randomize resouce value
const max = ref(0);
const min = ref(0);
const forMax = 300;
const forMin = 100;

// Timeout dalays
const setToPauseCounter = computed(() => COUNTER_DURATION.value); // 500 ms
const prepareToHideCounter = computed(() => COUNTER_DURATION.value * 2); //  500ms + 500ms
const prepareToHideSource = computed(() => COUNTER_DURATION.value * 3);

const counterIncreaseAnimation = ref(false)

function sourceUI(source: number, calculation: 'increase' | 'decrease') {
  // Converting percents to points
  if (calculation === 'decrease') {
    source = ALL_SOURCE_POINTS * (source / 100)
  }

  // Calculate min and max values for each iteration in current source
  max.value = (source / ALL_SOURCE_POINTS) * forMax;
  min.value = (source / ALL_SOURCE_POINTS) * forMin;

  sourceUIRef.value?.classList.add('show-element');
  counterRef.value?.classList.add('reset-counter-position')

  // Calling the counter when source class will be visible
  sourceUIRef.value?.addEventListener('transitionend', function () {
    counterRef.value?.classList.remove('reset-counter-position')

    if (calculation === 'decrease') {
      counterIncreaseAnimation.value = false
      lostSource(source)
      return
    }
    counterIncreaseAnimation.value = true
    collectSource(source)
  },
    { once: true }
  );
}
function lostSource(source: number) {
  showCounter(counterRef.value)
  decreaseTotalSource(source)
  decreaseCounterSource(source)
}

function showCounter(elCounter: HTMLElement | null) {
  const showCounter = setTimeout(() => {
    elCounter?.classList.add('show-element')
    clearTimeout(showCounter)
  }, 0)
}

function decreaseTotalSource(source: number) {
  const accBreak = initSourceValue.value - source;

  const interval = setInterval(() => {
    // Increasing and saving random value for each iteration
    let accumulation = initSourceValue.value - Math.floor(Math.random() * (max.value - min.value));

    // If the accumulation egual or below 0,
    // the iteration will stop.
    if (initSourceValue.value <= 0) {
      initSourceValue.value = 0;
      clearInterval(interval);
      return;
    }
    // If the accumulation less or equal total source
    if (accumulation <= accBreak) {
      // Adding the last value to the counter
      initSourceValue.value = accBreak;
      clearInterval(interval);
      return;
    }
    // Seting accumulation value to the initial value
    initSourceValue.value = accumulation;
  }, COUNTER_SPEED);
}

function decreaseCounterSource(source: number) {
  // If source points less than
  source = source <= initCounterSum.value ? source : initCounterSum.value

  if (source < DEATH_VALUE) {
    // Calculate min and max values for the last iteration,
    // if the sum of the source is less than DEATH_VALUE
    max.value = (source / ALL_SOURCE_POINTS) * forMax;
    min.value = (source / ALL_SOURCE_POINTS) * forMin;
  }

  const interval = setInterval(() => {
    // Increasing and saving random value for each iteration
    let accumulation = initCounterValue.value + Math.floor(Math.random() * (max.value - min.value));
    // Checking, and then stopping, resseting iteration
    if (accumulation >= source) {
      // Adding the last value to the counter
      initCounterValue.value = source;
      initCounterSum.value = initCounterSum.value <= 0 ? 0 : initCounterSum.value - source;
      // Showing counter after starting decrease total source
      firstStage(counterRef.value)
      // Hiding the source and resetting after the counter hidded
      counterRef.value?.addEventListener('transitionend', function () {
        secondStage(source, progressRef.value)
        resetAndHide(initCounterValue, sourceUIRef.value, this)
      },
        { once: true }
      );
      clearInterval(interval);
      return;
    }
    // Seting accumulation value to the initial value
    initCounterValue.value = accumulation;
  }, COUNTER_SPEED);
}

// INCREASE SOURCE
function collectSource(source: number) {

  const interval = setInterval(() => {
    // Increasing and saving random value for each iteration
    let accumulation = initCounterValue.value + Math.floor(Math.random() * (max.value - min.value));
    // Checking, and then stopping, resseting iteration
    if (accumulation >= source) {
      // Adding the last value to the counter
      initCounterValue.value = source;
      initCounterSum.value = initCounterSum.value >= ALL_SOURCE_POINTS ? ALL_SOURCE_POINTS : initCounterSum.value + source;
      // Small break before continuing
      firstStage(counterRef.value, source);
      // Hiding the source and resetting after the counter hidded
      counterRef.value?.addEventListener('transitionend', function () {
        secondStage(source, progressRef.value, counterRef.value);
        resetAndHide(initCounterValue, sourceUIRef.value, this)
      },
        { once: true }
      );
      clearInterval(interval);
      return;
    }

    counterRef.value?.classList.add('move-down-and-show');
    // Seting accumulation value to the initial value
    initCounterValue.value = accumulation;
  }, COUNTER_SPEED);
}

// Swap classes to hide counter
// and calling function to increase total source points
function firstStage(elCounter: HTMLElement | null, source?: number) {
  const timeout = setTimeout(() => {
    // if decreasing
    if (!source) {
      elCounter?.classList.replace('show-element', 'move-up-and-hide')
      return
    }
    // Increase
    elCounter?.classList.replace('move-down-and-show', 'move-down-and-hide');
    totalSourceCalc(source);
    clearTimeout(timeout);
  }, setToPauseCounter.value);
}

// 1) If increase points, removing the counter class
// 2) Calling function to increase or decrease progress bar width
function secondStage(source: number, elProgress: HTMLElement | null, elCounter?: HTMLElement | null) {
  const timeout = setTimeout(() => {
    // only for increase
    elCounter ? elCounter?.classList.remove('move-down-and-hide') : null;
    // work in increase | decrease
    sourceProgress(source, elProgress);
    clearTimeout(timeout);
  }, prepareToHideCounter.value);
}

function totalSourceCalc(source: number) {
  const accBreak = initSourceValue.value + source;

  const interval = setInterval(() => {
    // Increasing and saving random value for each iteration
    let accumulation = initSourceValue.value + Math.floor(Math.random() * (max.value - min.value));
    // If the accumulation exceeds the max. amount of source points,
    // the iteration will stop.
    if (accumulation >= ALL_SOURCE_POINTS) {
      initSourceValue.value = ALL_SOURCE_POINTS;
      clearInterval(interval);
      return;
    }
    // If the accumulation more or equal total source
    if (accumulation >= accBreak) {
      // Adding the last value to the counter
      initSourceValue.value = accBreak;
      clearInterval(interval);
      return;
    }
    // Seting accumulation value to the initial value
    initSourceValue.value = accumulation;
  }, COUNTER_SPEED);
}

function sourceProgress(points: number, elProgress: HTMLElement | null) {
  // Calculating, how many px we will add on progress bar
  let calc = (PROGRESS_BAR_WIDTH * points) / ALL_SOURCE_POINTS;
  let step

  if (!counterIncreaseAnimation.value) {
    // Subtracting the calculated step to previous value of the progress
    step = (PROGRESS -= calc);
    // If step less the entire progress bar
    if (step <= PROGRESS_BAR_WIDTH) {

      elProgress?.addEventListener('transitionend', function () {
        this.classList.remove('is-full-bar');
      },
        { once: true }
      );
    }
  }

  if (counterIncreaseAnimation.value) {
    // Adding the calculated step to previous value of the progress
    step = (PROGRESS += calc);
    // If step exceed the entire progress bar
    if (step >= PROGRESS_BAR_WIDTH) {
      PROGRESS = PROGRESS_BAR_WIDTH;

      elProgress?.addEventListener('transitionend', function () {
        this.classList.add('is-full-bar');
      },
        { once: true }
      );
      return PROGRESS;
    }
  }
  return step;
}

function resetAndHide(counter: Ref<number>, elSource: HTMLElement | null, elCounter?: HTMLElement | null) {
  const timeout = setTimeout(() => {
    // Resetting counter to initial value for prepare to the next iteration
    counter.value = 0;
    elSource?.classList.remove('show-element');
    // // only for increase
    elCounter ? elCounter?.classList.remove('move-up-and-hide') : null
    clearTimeout(timeout);
  }, prepareToHideSource.value);
}

// Switching between transition
function switchProgressClass(event: Event) {
  let text = (event.currentTarget as HTMLButtonElement).textContent;

  if (text === 'Smooth') {
    (event.currentTarget as HTMLButtonElement).textContent = 'Steps';
  } else {
    (event.currentTarget as HTMLButtonElement).textContent = 'Smooth';
  }
  progressRef.value?.classList.toggle('steps');
}
</script>

<template>
  <main>
    <section class="scene">
      <div class="source" ref="sourceUIRef" :style="{ transitionDuration: `${COUNTER_DURATION}ms` }">
        <div class="source-icon">
          <svg viewBox="0 0 34 32" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path
              d="M0.998199 31.9837L6.43541 11.4877C6.46181 11.3882 6.60326 11.3887 6.62891 11.4885L11.8997 31.9837H0.998199Z"
              fill="white" />
            <path
              d="M23.0895 31.9837L28.5267 11.4877C28.5531 11.3882 28.6946 11.3887 28.7202 11.4885L33.991 31.9837H23.0895Z"
              fill="white" />
            <path
              d="M11.9192 21.3689L17.3564 0.872988C17.3828 0.773454 17.5243 0.773991 17.5499 0.873723L22.8207 21.3689H11.9192Z"
              fill="white" />
          </svg>
        </div>
        <span class="source-amount" ref="amountRef">{{ initSourceValue }}</span>
        <span class="source-counter" :class="[counterIncreaseAnimation ? 'source-increase' : 'source-decrease']"
          ref="counterRef" :style="{ transitionDuration: `${COUNTER_DURATION}ms` }">
          {{ initCounterValue }}
        </span>
      </div>
      <div class="progress-bar">
        <div class="progress" ref="progressRef" :style="{ width: `${PROGRESS}px` }"></div>
      </div>
      <img src="/bg.webp" alt="bg" />
    </section>
    <section class="ui-elements">
      <div class="input-element">
        <label for="one-source">One enemy source</label>
        <input type="text" name="one" id="one-source" v-model.number.trim="ONE_ENEMY" />
        <button class="add-source-btn" @click="sourceUI(ONE_ENEMY, 'increase')">Check</button>
      </div>
      <div class="input-element">
        <label for="two-source">Second enemy source</label>
        <input type="text" name="two" id="two-source" v-model.number.trim="SECOND_ENEMY" />
        <button class="add-source-btn" @click="sourceUI(SECOND_ENEMY, 'increase')">Check</button>
      </div>
      <div class="input-element">
        <label for="three-source">Third enemy source</label>
        <input type="text" name="three" id="three-source" v-model.number.trim="THIRD_ENEMY" />
        <button class="add-source-btn" @click="sourceUI(THIRD_ENEMY, 'increase')">Check</button>
      </div>
      <div class="input-element">
        <label for="fourth-source">Fourth enemy source</label>
        <input type="text" name="fourth" id="fourth-source" v-model.number.trim="FOURTH_ENEMY" />
        <button class="add-source-btn" @click="sourceUI(FOURTH_ENEMY, 'increase')">Check</button>
      </div>
      <div class="input-element">
        <label for="duration">Counter animation (ms)</label>
        <input type="text" name="duration" id="duration" placeholder="Default 500ms"
          v-model.number="COUNTER_DURATION" />
      </div>
      <div class="input-element">
        <label for="speed">Counter speed (min 15)</label>
        <input type="text" name="speed" id="speed" placeholder="Default 20" v-model.number="COUNTER_SPEED" />
      </div>
      <div class="input-element">
        <label for="remove-source">Remove source %</label>
        <input type="text" name="remove" id="remove-source" v-model.number.trim="DEATH_VALUE" />
        <button class="remove-source-btn" @click="sourceUI(DEATH_VALUE, 'decrease')">Remove</button>
      </div>
      <div class="input-element">
        <label for="remove-source">Bar animation smooth/steps</label>
        <button class="add-source-btn" @click="switchProgressClass">Smooth</button>
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
  transition-property: width, transform;
  transition-timing-function: ease-in-out;
  box-shadow: 0 0 0 0px hsl(0 0% 255% / 0.5);
}

.is-full-bar {
  animation: full-bar 0.8s ease-out infinite;
}

.progress.steps {
  transition-timing-function: steps(4, jump-start);
}

@keyframes full-bar {
  100% {
    box-shadow: 0 0 0 0.3rem hsl(0 0% 255% / 0);
  }
}

/* SOURCE component*/
.source {
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

.source-amount,
.source-counter {
  grid-column: 2;
  grid-row: 1;
  font-family: sans-serif;
  font-weight: bold;
  user-select: none;
}

.source-counter {
  opacity: 0;
  /* duration changes dynamically */
  /* transition-duration: 0.5s; */
  transition-property: opacity, transform;
  transition-timing-function: ease-in-out;

}

.source-increase {
  color: var(--add-hp);
  transform: translate3d(0, -2.25rem, 0);
}

.source-decrease {
  --add-hp: var(--remove-hp);
  color: var(--add-hp);
  transform: translate3d(0, -1.125rem, 0);
}

.reset-counter-position {
  display: none;
}

/* ICON cross */
.source-increase::after,
.source-increase::before,
.source-decrease::after {
  content: '';
  width: 0.5rem;
  aspect-ratio: 3.3 / 1;
  position: absolute;
  inset-block: 45%;
  left: -0.625rem;
  background-color: var(--add-hp);
}

.source-increase::before {
  transform: rotate(90deg);
}

/* transition counter classes */
.show-element {
  opacity: 1;
}

.move-up-and-hide {
  opacity: 0;
  transform: translate3d(0, -2.25rem, 0);
}

.move-down-and-show {
  opacity: 1;
  transform: translate3d(0, -1.125rem, 0);
}

.move-down-and-hide {
  opacity: 0;
  transform: translate3d(0, 0, 0);
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

.add-source-btn {
  color: var(--bar-backdrop);
  background-color: var(--add-hp);
}

.remove-source-btn {
  color: var(--ui-element);
  background-color: var(--remove-hp);
}
</style>

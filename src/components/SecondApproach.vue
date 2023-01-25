<template>
  <main>
    <section class="scene">
      <div class="source" ref="sourceRef">
        <div class="source__icon">
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
        <span class="source__amount" ref="amountRef">{{ initSourceValue }}</span>
        <span class="source__counter" ref="counterRef">{{ initCounterValue }}</span>
      </div>
      <div class="progress">
        <div class="progress__wrapper" ref="progressWrapperRef">
          <div class="progress__bar" ref="progressBarRef"
            :style="{ width: `${PROGRESS_BAR_WIDTH}px`, left: `-${PROGRESS_BAR_WIDTH}px`, transform: `translateX(${PROGRESS}px)` }">
          </div>
          <div class="progress__prediction" ref="progressPredictRef" :style="{ width: `${PROGRESS_PREVIEW}px` }">
          </div>
        </div>
      </div>
      <img src="/bg.webp" alt="bg" />
    </section>
    <section class="elements-ui">
      <div class="element__input">
        <label for="one-source">One enemy source</label>
        <input type="text" name="one" id="one-source" v-model.number.trim="ONE_ENEMY" />
        <button class="add-source-btn" @click="sourceHandler(ONE_ENEMY, true)">Add</button>
      </div>
      <div class="element__input">
        <label for="remove-source">Remove source %</label>
        <input type="text" name="remove" id="remove-source" v-model.number.trim="DEATH_PERCENT" />
        <button class="remove-source-btn" @click="sourceHandler(DEATH_PERCENT, false)">Remove</button>
      </div>
    </section>
  </main>
</template>

<script setup lang="ts">
import { ref } from 'vue';
const PROGRESS_BAR_WIDTH = 160; // '160px' = '10rem'
const ALL_SOURCE_POINTS = 2000; // All points includes in progress bar
const COUNTER_SPEED = 20;
const TIMEOUT = 1000 // ms
let PROGRESS = 0;
let PROGRESS_PREVIEW = 0;
let RANDOM_RANGE = 15
let ONE_ENEMY = 150;
let DEATH_PERCENT = 10; // 10% of 100

// Randomize resouce value
const max = ref(0);
const min = ref(0);
const forMax = 300;
const forMin = 100;

const initSourceValue = ref(0);
const initCounterValue = ref(0);
const initCounterSum = ref(0);

// HTML Elements
const sourceRef = ref<HTMLElement | null>(null);
const counterRef = ref<HTMLElement | null>(null);
const amountRef = ref<HTMLElement | null>(null);
const progressBarRef = ref<HTMLElement | null>(null);
const progressWrapperRef = ref<HTMLElement | null>(null);
const progressPredictRef = ref<HTMLElement | null>(null); // EXTRA: Predictional

function sourceHandler(source: number, isAccumulate: boolean) {

  if (isAccumulate) {
    // Randomizing range between positive and negative RANDOM_RANGE value
    const randomizeSource = Math.ceil(Math.random() * RANDOM_RANGE) * (Math.round(Math.random()) ? 1 : -1)
    // Setting randomized source value
    source = source + randomizeSource
  }

  // Getting the DEATH_PERCENT number
  if (!isAccumulate) {
    source = ALL_SOURCE_POINTS * (source / 100)
  }
  // If source remains less than 100 points
  if (!isAccumulate && 100 >= initSourceValue.value) {
    source = initSourceValue.value
  }

  // Calculate min and max values for each iteration in current source
  max.value = (source / ALL_SOURCE_POINTS) * forMax;
  min.value = (source / ALL_SOURCE_POINTS) * forMin;

  sourceRef.value?.classList.add('show')

  // EXTRA: calculate preview width and
  let memorize = (PROGRESS_BAR_WIDTH * source) / ALL_SOURCE_POINTS;

  sourceRef.value?.addEventListener('transitionend', function () {

    PROGRESS_PREVIEW = predictionStep(memorize, isAccumulate)
    if (initSourceValue.value <= 0 && !isAccumulate) {
      const timeout = setTimeout(() => {
        sourceRef.value?.classList.remove('show')
        clearTimeout(timeout)
      }, 300)
      return
    }

    const counterAnimationClasse = isAccumulate ? 'add-source' : 'remove-source'
    const counterStyleClasse = isAccumulate ? 'source-increase' : 'source-decrease'

    counterRef.value?.classList.add(counterAnimationClasse, counterStyleClasse)
    progressPredictRef.value?.classList.add('show-element')
    // STEP I
    isAccumulate ? collectSource(source) : null

    // STEP II
    const stemTwo = setTimeout(() => {
      counterRef.value?.classList.add('pause')
      if (!isAccumulate) {
        totalSource(source, 'decrease')
      }
    }, TIMEOUT / 2)

    // STEP III
    const stemThree = setTimeout(() => {
      if (!isAccumulate) {
        loseSource(source)
      }
      if (isAccumulate) {
        counterRef.value?.classList.remove('pause')
        totalSource(source, 'increase')
      }
    }, TIMEOUT)

    // STEP IV
    counterRef.value?.addEventListener('animationend', function () {
      this.classList.remove('add-source', 'remove-source', 'source-increase', 'source-decrease')


      const stepFour = setTimeout(() => {
        sourceRef.value?.classList.remove('show')
        sourceProgress(source, isAccumulate ? 'increase' : 'decrease')
        initCounterValue.value = 0

        clearTimeout(stemTwo)
        clearTimeout(stemThree)
        clearTimeout(stepFour)
      }, TIMEOUT)

    }, { once: true })
  }, { once: true })
}

function predictionStep(memo: number, isAcc: boolean) {
  if (!isAcc && PROGRESS_PREVIEW < memo) {
    return 0
  }
  if (!isAcc && PROGRESS_PREVIEW > memo) {
    return PROGRESS_PREVIEW -= memo
  }
  if (isAcc && memo + PROGRESS_PREVIEW >= PROGRESS_BAR_WIDTH) {
    return PROGRESS_BAR_WIDTH
  }
  if (isAcc && memo + PROGRESS_PREVIEW < PROGRESS_BAR_WIDTH) {
    return PROGRESS_PREVIEW += memo
  }
  return 0
}

function collectSource(source: number) {
  const interval = setInterval(() => {
    // Increasing and saving random value for each iteration
    let accumulation = initCounterValue.value + Math.floor(Math.random() * (max.value - min.value));
    // Checking, and then stopping, resseting iteration
    if (accumulation >= source) {
      // Adding the last value to the counter
      initCounterValue.value = source;
      initCounterSum.value = initCounterSum.value >= ALL_SOURCE_POINTS ? ALL_SOURCE_POINTS : initCounterSum.value + source;
      clearInterval(interval);
      return;
    }
    // Seting accumulation value to the initial value
    initCounterValue.value = accumulation;
  }, COUNTER_SPEED);
}

function loseSource(source: number) {
  // If source points less than
  source = source <= initCounterSum.value ? source : initCounterSum.value

  if (source < DEATH_PERCENT) {
    // Calculate min and max values for the last iteration,
    // if the sum of the source is less than DEATH_PERCENT
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
      // Remove pause animation
      counterRef.value?.classList.remove('pause')
      clearInterval(interval);
      return;
    }
    // Seting accumulation value to the initial value
    initCounterValue.value = accumulation;
  }, COUNTER_SPEED);
}


function totalSource(source: number, calculation: 'increase' | 'decrease') {
  const limit = calculation === "increase" ? initSourceValue.value + source : initSourceValue.value - source;
  let intervalExtraBreak = 0
  let accumulation = 0

  if (calculation === 'decrease') {
    const interval = setInterval(() => {
      // Increasing and saving random value for each iteration
      accumulation = initSourceValue.value - Math.floor(Math.random() * (max.value - min.value));
      // If the accumulation egual or below 0,
      // the iteration will stop.
      if (initSourceValue.value <= 0 || accumulation < 0) {
        initSourceValue.value = 0;
        clearInterval(interval);
        return;
      }
      // If the accumulation less or equal total source
      if (accumulation <= limit) {
        // Adding the last value to the counter
        initSourceValue.value = limit;
        clearInterval(interval);
        return;
      }
      // Extra break after 80 iteractions to prevent infinite loop
      intervalExtraBreak++
      if (intervalExtraBreak === 60) {
        initSourceValue.value = limit;
        clearInterval(interval);
        return;
      }
      // Seting accumulation value to the initial value
      initSourceValue.value = accumulation;
    }, COUNTER_SPEED);
  }

  if (calculation === 'increase') {
    const interval = setInterval(() => {

      // Increasing and saving random value for each iteration
      accumulation = initSourceValue.value + Math.floor(Math.random() * (max.value - min.value));
      // If the accumulation exceeds the max. amount of source points,
      // the iteration will stop.
      if (accumulation >= ALL_SOURCE_POINTS) {
        initSourceValue.value = ALL_SOURCE_POINTS;
        clearInterval(interval);
        return;
      }
      // If the accumulation more or equal total source
      if (accumulation >= limit) {
        // Adding the last value to the counter
        initSourceValue.value = limit;
        clearInterval(interval);
        return;
      }
      // Extra break after 80 iteractions to prevent infinite loop
      intervalExtraBreak++
      if (intervalExtraBreak === 60) {
        initSourceValue.value = limit;
        clearInterval(interval);
        return;
      }
      // Seting accumulation value to the initial value
      initSourceValue.value = accumulation;
    }, COUNTER_SPEED);
  }
}

function sourceProgress(source: number, calculation: 'increase' | 'decrease') {
  // Calculating, how many px we will add on progress bar
  let calc = (PROGRESS_BAR_WIDTH * source) / ALL_SOURCE_POINTS;
  let step = 0

  if (calculation === 'decrease') {
    // Subtracting the calculated step to previous value of the progress
    step = (PROGRESS -= calc);
    // If step less the entire progress bar
    if (step <= 0) PROGRESS = 0

    if (step <= PROGRESS_BAR_WIDTH) {

      // EXTRA: Removing a class after the width has decreased
      progressBarRef.value?.addEventListener('transitionend', () => {
        progressWrapperRef.value?.classList.remove('is-full-bar');
      }, { once: true });
      return
    }
  }

  if (calculation === 'increase') {
    // Adding the calculated step to previous value of the progress
    step = (PROGRESS += calc);
    // If step exceed the entire progress bar
    if (step >= PROGRESS_BAR_WIDTH) {
      PROGRESS = PROGRESS_BAR_WIDTH;
      // EXTRA: Indicate that the entire progress bar has been filled
      // after the width transition ends
      progressBarRef.value?.addEventListener('transitionend', function () {
        progressPredictRef.value?.classList.remove('show-element')
        progressWrapperRef.value?.classList.add('is-full-bar');
      }, { once: true })
      return PROGRESS;
    }
  }
}
</script>

<style scoped>
/* PROGRESS BAR */
.progress {
  width: 10rem;
  height: 1rem;
  position: absolute;
  left: 3rem;
  bottom: 5rem;
  background-color: var(--bar-backdrop);
}

/* ICON cross  */
.progress::after,
.progress::before {
  content: '';
  height: inherit;
  aspect-ratio: 1 / 3.1;
  position: absolute;
  top: 0;
  left: -1.1rem;
  background-color: var(--bar-color);
}

.progress::before {
  transform: rotate(90deg);
}

.progress__wrapper {
  overflow: hidden;
  position: absolute;
  inset: 0;
  box-shadow: 0 0 0 0 hsl(0 0% 100% / .7);
}

.progress__bar {
  position: absolute;
  inset-block: 0 0;
  background-color: var(--bar-color);
  transition-duration: 0.5s;
  transition-property: transform;
  transition-timing-function: ease-in-out;
  z-index: 5;
}

.progress__prediction {
  position: absolute;
  inset-block: 0 0;
  overflow: hidden;
  opacity: 0;
  background-color: hsl(0 0% 255% / .2);
  transition: opacity .3s ease-in-out;
  z-index: 1;
}

.show-element {
  opacity: 1;
}

@keyframes indicate {
  100% {
    box-shadow: 0 0 0 2px hsl(0 0% 255% / 0);
  }
}

.is-full-bar {
  animation: full-bar 0.8s .5s ease-out 3;

}

.progress__bar.steps {
  transition-timing-function: steps(4, jump-start);
}

@keyframes full-bar {
  100% {
    box-shadow: 0 0 0 0.3rem hsla(0, 0%, 100%, 0);
  }
}

.source {
  display: grid;
  grid-template-columns: 1.3rem auto;
  column-gap: 0.5rem;
  align-items: baseline;
  position: absolute;
  left: 3rem;
  bottom: 7.5rem;
  opacity: 0;
  transition: opacity 1s ease-in-out;
  font-family: sans-serif;
  font-weight: bold;
  user-select: none;
}

.source__icon {
  grid-column: 1;
  grid-row: 2;
}

.source__amount,
.source__counter {
  grid-column: 2;
}

.source__amount {
  grid-row: 2;
  /* opacity: 0; */
  /* transition: opacity 1s ease-in-out; */
}

.source__counter {
  grid-row: 1;
  opacity: 0;
  position: relative;
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

/* ICON plus / minus  */
.source-increase::after,
.source-decrease::after {
  position: absolute;
  font-size: 1.2em;
  font-weight: bold;
}

.source-increase::after {
  content: '+';
  top: -0.125rem;
  left: -0.8rem;
}

.source-decrease::after {
  content: '-';
  top: -0.225rem;
  left: -0.6rem;
}

.add-source {
  animation: ascent 1s ease-in-out forwards;
}

.remove-source {
  animation: descent 1s ease-in-out forwards;
}

.pause {
  animation-play-state: paused;
}

.show {
  opacity: 1;
}

@keyframes ascent {
  0% {
    transform: translate3d(0, -1.125rem, 0);
    opacity: 0;
  }

  50% {
    transform: translate3d(0, 0rem, 0);
    opacity: 1;
  }

  100% {
    transform: translate3d(0, 1.125rem, 0);
    opacity: 0;
  }
}

@keyframes descent {
  0% {
    transform: translate3d(0, 0rem, 0);
    opacity: 0;
  }

  50% {
    transform: translate3d(0, 0rem, 0);
    opacity: 1;
  }

  100% {
    transform: translate3d(0, -1.125rem, 0);
    opacity: 0;
  }
}
</style>

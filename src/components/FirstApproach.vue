<script setup lang="ts">
import { ref, Ref, computed } from 'vue';

const PROGRESS_BAR_WIDTH = 160; // '160px' = '10rem'
const ALL_SOURCE_POINTS = 2000; // All points includes in progress bar
let COUNTER_SPEED = 20;
let COUNTER_DURATION = ref(500); // 500ms
let PROGRESS = 0;
let PROGRESS_PREVIEW = 0;
let RANDOM_RANGE = 15

// Initial enemy values
let ONE_ENEMY = 500;
let SECOND_ENEMY = 400;
let THIRD_ENEMY = 333;
let FOURTH_ENEMY = 55;
let DEATH_PERCENT = 10; // 10% of 100

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

// Randomize resouce value
const max = ref(0);
const min = ref(0);
const forMax = 300;
const forMin = 100;

// Timeout dalays
const setToPauseCounter = computed(() => COUNTER_DURATION.value); // 500 ms
const prepareToHideCounter = computed(() => COUNTER_DURATION.value * 2); // 500ms + 500ms
const prepareToHideSource = computed(() => COUNTER_DURATION.value * 3); // 500ms + 500ms + 500ms

const counterIncreaseAnimation = ref(false)
const randomizerSwitcher = ref(false)
// Disable buttons to play the animation and prevent spam events
const disableButton = ref(false)

function sourceHandler(source: number, calculation: 'increase' | 'decrease') {

  disableButton.value = true

  if (calculation === 'increase') {
    // Randomizing range between positive and negative RANDOM_RANGE value
    const randomizeSource = Math.ceil(Math.random() * RANDOM_RANGE) * (Math.round(Math.random()) ? 1 : -1)
    // Setting randomized source value
    source = randomizerSwitcher.value ? source + randomizeSource : source
  }

  // Converting percents to points
  if (calculation === 'decrease') {
    source = ALL_SOURCE_POINTS * (source / 100)
  }

  // Calculate min and max values for each iteration in current source
  max.value = (source / ALL_SOURCE_POINTS) * forMax;
  min.value = (source / ALL_SOURCE_POINTS) * forMin;

  sourceRef.value?.classList.add('show-element');
  // Adding class to resetting counter position
  counterRef.value?.classList.add('reset-counter-position')

  // EXTRA: calculate preview width and
  let memorize = (PROGRESS_BAR_WIDTH * source) / ALL_SOURCE_POINTS;
  // Calling the counter when source class will be visible
  sourceRef.value?.addEventListener('transitionend', function () {
    counterRef.value?.classList.remove('reset-counter-position')

    if (calculation === 'decrease') {
      // Switching animation direction
      counterIncreaseAnimation.value = false
      // If the source has '0', breaking the counter animation / transition
      // Only quickly show the source element
      if (PROGRESS_PREVIEW === 0) {
        this.classList.remove('show-element');
        disableButton.value = false
        return
      }
      lostSource(source)

      // EXTRA: Show preview progress before main progress
      // EXTRA: add class to 'predict' how many source will be lost
      if (PROGRESS_PREVIEW < memorize) {
        PROGRESS_PREVIEW = 0
        return
      }
      PROGRESS_PREVIEW -= memorize
      return
    }
    // Switching animation direction
    counterIncreaseAnimation.value = true
    collectSource(source)

    // EXTRA: Show preview progress before main progress
    // EXTRA: add a class to 'predict' how many source will be added
    progressPredictRef.value?.classList.add('show-element')
    if (memorize + PROGRESS_PREVIEW >= PROGRESS_BAR_WIDTH) {
      PROGRESS_PREVIEW = PROGRESS_BAR_WIDTH
      return
    }
    PROGRESS_PREVIEW += memorize
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
      // Showing counter after starting decrease total source
      firstStage(counterRef.value)
      // Hiding the source and resetting after the counter hidded
      counterRef.value?.addEventListener('transitionend', function () {
        secondStage(source, progressBarRef.value)
        resetAndHide(initCounterValue, sourceRef.value, this)
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
        secondStage(source, progressBarRef.value, counterRef.value);
        resetAndHide(initCounterValue, sourceRef.value, this)
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

  disableButton.value = false

  // Calculating, how many px we will add on progress bar
  let calc = (PROGRESS_BAR_WIDTH * points) / ALL_SOURCE_POINTS;
  let step = 0

  if (!counterIncreaseAnimation.value) {
    // Subtracting the calculated step to previous value of the progress
    step = (PROGRESS -= calc);
    // If step less the entire progress bar
    if (step <= 0) PROGRESS = 0

    if (step <= PROGRESS_BAR_WIDTH) {
      // EXTRA: Removing a class after the width has decreased
      elProgress?.addEventListener('transitionend', () => {
        progressWrapperRef.value?.classList.remove('is-full-bar');

      }, { once: true });
      return
    }
  }

  if (counterIncreaseAnimation.value) {
    // Adding the calculated step to previous value of the progress
    step = (PROGRESS += calc);
    // If step exceed the entire progress bar
    if (step >= PROGRESS_BAR_WIDTH) {
      PROGRESS = PROGRESS_BAR_WIDTH;

      // EXTRA: Indicate that the entire progress bar has been filled
      // after the width transition ends
      elProgress?.addEventListener('transitionend', function () {
        progressPredictRef.value?.classList.remove('show-element')
        progressWrapperRef.value?.classList.add('is-full-bar');
      }, { once: true })
      return PROGRESS;
    }
  }
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

// UI elements handlers
// Switching between transition
function switchProgressClass(event: Event) {
  let text = (event.currentTarget as HTMLButtonElement).textContent;

  if (text === 'Smooth') {
    (event.currentTarget as HTMLButtonElement).textContent = 'Steps';
  } else {
    (event.currentTarget as HTMLButtonElement).textContent = 'Smooth';
  }
  progressBarRef.value?.classList.toggle('steps');
}
function randomizerHandler() {
  randomizerSwitcher.value = !randomizerSwitcher.value
}
</script>

<template>
  <main>
    <section class="scene">
      <div class="source" ref="sourceRef" :style="{ transitionDuration: `${COUNTER_DURATION}ms` }">
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
        <span class="source__counter" :class="[counterIncreaseAnimation ? 'source-increase' : 'source-decrease']"
          ref="counterRef" :style="{ transitionDuration: `${COUNTER_DURATION}ms` }">
          {{ initCounterValue }}
        </span>
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
        <button class="add-source-btn" :disabled="disableButton"
          @click="sourceHandler(ONE_ENEMY, 'increase')">Add</button>
      </div>
      <div class="element__input">
        <label for="two-source">Second enemy source</label>
        <input type="text" name="two" id="two-source" v-model.number.trim="SECOND_ENEMY" />
        <button class="add-source-btn" :disabled="disableButton"
          @click="sourceHandler(SECOND_ENEMY, 'increase')">Add</button>
      </div>
      <div class="element__input">
        <label for="three-source">Third enemy source</label>
        <input type="text" name="three" id="three-source" v-model.number.trim="THIRD_ENEMY" />
        <button class="add-source-btn" :disabled="disableButton"
          @click="sourceHandler(THIRD_ENEMY, 'increase')">Add</button>
      </div>
      <div class="element__input">
        <label for="fourth-source">Fourth enemy source</label>
        <input type="text" name="fourth" id="fourth-source" v-model.number.trim="FOURTH_ENEMY" />
        <button class="add-source-btn" :disabled="disableButton"
          @click="sourceHandler(FOURTH_ENEMY, 'increase')">Add</button>
      </div>
      <div class="element__input">
        <label for="remove-source">Remove source %</label>
        <input type="text" name="remove" id="remove-source" v-model.number.trim="DEATH_PERCENT" />
        <button class="remove-source-btn" :disabled="disableButton"
          @click="sourceHandler(DEATH_PERCENT, 'decrease')">Remove</button>
      </div>
      <div class="element__input">
        <label for="random-source">Randomize source %</label>
        <input type="text" name="random" id="random-source" v-model.number.trim="RANDOM_RANGE" />
        <button :class="[randomizerSwitcher ? 'add-source-btn' : 'remove-source-btn']" :disabled="disableButton"
          @click="randomizerHandler">{{
            randomizerSwitcher? 'On'
              : 'Off'
          }}</button>
      </div>
      <div class="element__input">
        <label>Progress bar animation</label>
        <button class="add-source-btn" @click="switchProgressClass">Smooth</button>
      </div>
      <div class="element__input">
        <label for="duration">Counter animation (ms)</label>
        <input type="text" name="duration" id="duration" placeholder="Default 500ms"
          v-model.number="COUNTER_DURATION" />
      </div>
      <div class="element__input">
        <label for="speed">Counter speed</label>
        <input type="text" name="speed" id="speed" placeholder="Default 20" v-model.number="COUNTER_SPEED" />
      </div>
    </section>
  </main>
</template>

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

.source__amount,
.source__counter {
  grid-column: 2;
  grid-row: 1;
  font-family: sans-serif;
  font-weight: bold;
  user-select: none;
}

.source__counter {
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
</style>

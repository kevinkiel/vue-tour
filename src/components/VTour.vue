<template>
  <div class="v-tour">
    <slot
      :current-step="currentStep"
      :steps="steps"
      :previous-step="previousStep"
      :next-step="nextStep"
      :stop="stop"
      :is-first="isFirst"
      :is-last="isLast"
      :labels="customOptions.labels"
      :highlight="customOptions.highlight"
    >
      <!--Default slot {{ currentStep }}-->
      <v-step
        v-if="currentStep === index"
        v-for="(step, index) of steps"
        :key="index"
        :step="step"
        :previous-step="previousStep"
        :next-step="nextStep"
        :stop="stop"
        :is-first="isFirst"
        :is-last="isLast"
        :labels="customOptions.labels"
        :highlight="customOptions.highlight"
      >
        <!--<div v-if="index === 2" slot="actions">
          <a @click="nextStep">Next step</a>
        </div>-->
      </v-step>
    </slot>
  </div>
</template>

<script>
import { DEFAULT_CALLBACKS, DEFAULT_OPTIONS, KEYS, HIGHLIGHT } from '../shared/constants'

export default {
  name: 'v-tour',
  props: {
    steps: {
      type: Array,
      default: () => []
    },
    name: {
      type: String
    },
    options: {
      type: Object,
      default: () => { return DEFAULT_OPTIONS }
    },
    callbacks: {
      type: Object,
      default: () => { return DEFAULT_CALLBACKS }
    }
  },
  data () {
    return {
      currentStep: -1
    }
  },
  mounted () {
    this.$tours[this.name] = this

    if (this.customOptions.useKeyboardNavigation) {
      window.addEventListener('keyup', this.handleKeyup)
    }
  },
  beforeDestroy () {
    // Remove the keyup listener if it has been defined
    if (this.customOptions.useKeyboardNavigation) {
      window.removeEventListener('keyup', this.handleKeyup)
    }
  },
  computed: {
    // Allow us to define custom options and merge them with the default options.
    // Since options is a computed property, it is reactive and can be updated during runtime.
    customOptions () {
      return {
        ...DEFAULT_OPTIONS,
        ...this.options
      }
    },
    customCallbacks () {
      return {
        ...DEFAULT_CALLBACKS,
        ...this.callbacks
      }
    },
    // Return true if the tour is active, which means that there's a VStep displayed
    isRunning () {
      return this.currentStep > -1 && this.currentStep < this.numberOfSteps
    },
    isFirst () {
      return this.currentStep === 0
    },
    isLast () {
      return this.currentStep === this.steps.length - 1
    },
    numberOfSteps () {
      return this.steps.length
    }
  },
  methods: {
    start (startStep) {
      // Wait for the DOM to be loaded, then start the tour
      setTimeout(() => {
        this.customCallbacks.onStart()
        this.currentStep = typeof startStep !== 'undefined' ? parseInt(startStep, 10) : 0
      }, this.customOptions.startTimeout)
    },
    previousStep () {
      if (this.currentStep > 0) {
        this.customCallbacks.onPreviousStep(this.currentStep)
        this.currentStep--
      }
    },
    nextStep () {
      if (this.currentStep < this.numberOfSteps - 1 && this.currentStep !== -1) {
        this.customCallbacks.onNextStep(this.currentStep)
        this.currentStep++
      }
    },
    stop () {
      this.customCallbacks.onStop()
      document.body.classList.remove(HIGHLIGHT.ACTIVE_TOUR)
      this.currentStep = -1
    },

    handleKeyup (e) {
      // TODO: debug mode
      // console.log('[Vue Tour] A keyup event occured:', e)
      switch (e.keyCode) {
        case KEYS.ARROW_RIGHT:
          this.nextStep()
          break
        case KEYS.ARROW_LEFT:
          this.previousStep()
          break
        case KEYS.ESCAPE:
          this.stop()
          break
      }
    }
  }
}
</script>

<style lang="scss">
  body.v-tour-active {
    pointer-events: none;
  }

  .v-tour {
    pointer-events: auto;
  }

  .v-tour-highlight {
    pointer-events: auto;
    z-index: 9999;
  }

  .v-tour-highlight:after {
    content: "";
    position: absolute;
    z-index: -1;
    box-shadow: 0 0 50px 25px rgba(0, 0, 0, 0.5);
    width: calc(100% + 10px);
    height: calc(100% + 10px);
    left: -5px;
    top: -5px;
  }

  .v-tour-position {
    position: relative;
  }
</style>

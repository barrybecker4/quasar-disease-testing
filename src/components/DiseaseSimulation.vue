<template>
  <q-card class="card-md">
      <div class="inputs">
      <span class="input-line">The incidence of the disease in the population is
         <span id="probability-diseased">{{initialPctDiseased}}%</span>
         <span id="probability-diseased-slider" class="slider"></span>
      </span>
         <div class="input-line">The disease testing accuracy is <span id="test-accuracy">{{initialTestAccuracy}}%</span>
             <div id="test-accuracy-slider" class="slider"></div>
         </div>
      </div>
      <q-slider v-model="standard" :min="0" :max="50" label/>
      <q-slider v-model="standard" :min="0" :max="50"  label color="green"/>

      <bayes-rule-view :graph="this.graph" :totalPopulation="this.totalPopulation"></bayes-rule-view>
      <!--sankey-view :graph="this.graph" @highlight="onHighlight" @unhighlight="onUnhighlight"></sankey-view>
      <venn-diagram-view :graph="this.graph" :totalPopulation="this.totalPopulation"></venn-diagram-view -->
      <notes-content :testPositive="this.testPositive" :totalPopulation="this.totalPopulation"></notes-content>
  </q-card>
</template>

<script>

import diseaseConsts from './diseaseConsts.js';
import BayesRuleView from 'components/BayesRuleView.vue';
import NotesContent from 'components/NotesContent.vue';

export default {

  components: {
      BayesRuleView,
      //SankeyView,
      //VennDiagramView,
      NotesContent,
  },

  setup() {
    return {}
  },

  props: {
    totalPopulation: { //  a number in the range [100, 1,000,000,000,000]
      type: Number,
      default: 100000
    },
    initialPctDiseased: { //  probability of having the disease. In range [0.1, 10]
      type: Number,
      default: 1
    },
    initialTestAccuracy: { // percent of time that the test is correct. In range [90, 99.5]
      type: Number,
      default: 90,
    }
  },

  data() {
     return {
       graph: {
         nodes: diseaseConsts.NODES,
         links: [0, 0, 0],
       },
       probDiseased: this.initialPctDiseased,
       testAccuracy: this.initialTestAccuracy,
     }
  },

  computed: {
     diseasedPop: function() {
         return this.probDiseased * this.totalPopulation;
     },
     healthyPop: function() {
         return this.totalPopulation - this.diseasedPop;
     },
     testNegAndHealthy: function() {
         return this.testAccuracy * this.healthyPop;
     },
     testNegButDiseased: function() {
         return (1.0 - this.testAccuracy) * this.diseasedPop;
     },
     testPositiveAndDiseased: function() {
         return this.diseasedPop - this.testNegButDiseased;
     },
     testPositiveButHealthy: function() {
         return this.healthyPop - this.testNegAndHealthy;
     },
     testPositive: function() {
         return this.testPositiveAndDiseased + this.testPositiveButHealthy;
     },
  },

  mounted() {
    this.init();
  },

  methods: {
    init: function() {
      this.initializeInputSection(this.initialPctDiseased, this.initialTestAccuracy);
      this.updateViews();
    },

    /**
     * Show two sliders that allow changing the incidence and accuracy.
     */
    initializeInputSection: function(initialPctDiseased, initialTestAccuracy) {
      let probDiseasedSlider = $("#probability-diseased-slider");
      let testAccuracySlider = $("#test-accuracy-slider");

      // Using integer values to avoid rounding problems at the max value
      probDiseasedSlider.slider({
          value: Math.log10(initialPctDiseased),
          min: -2,
          max: 1.0,
          step: 0.1,
          height: "10px",
          slide: this.getSliderChangedHandler("#probability-diseased", this.pctDiseasedConverter),
          stop: this.clearThumbTip
      });

      testAccuracySlider.slider({
          value: initialTestAccuracy * 10,
          min: 800,
          max: 999,
          step: 1,
          slide: this.getSliderChangedHandler("#test-accuracy", this.testAccuracyConverter),
          stop: this.clearThumbTip
      });
    },

    pctDiseasedConverter: function(sliderValue) {
      return diseaseConsts.format(Math.pow(10, sliderValue), 2) + "%";
    },

    testAccuracyConverter: function(sliderValue) {
      return sliderValue / 10 + "%";
    },

    /**
     * @param sliderEl jquery selector for slider
     * @param convert function used to map slider value to actual value
     * @returns {Function} slider changed callback
     */
    getSliderChangedHandler: function(sliderEl, convert) {
      let vm = this;
      return function (event, ui) {
        // update value in text
        var value = convert(ui.value);
        $(sliderEl).text(value);

        // current value (when sliding) or initial value (at start)
        var tooltip = '<div class="tooltip"><div class="tooltip-inner">' + value
            + '</div><div class="tooltip-arrow"></div></div>';
        $(sliderEl + "-slider").find('.ui-slider-handle').html(tooltip);
        vm.updateViews();
      }
    },

    updateViews: function() {
      this.probDiseased = parseFloat($("#probability-diseased").text()) / 100.0;
      this.testAccuracy = parseFloat($("#test-accuracy").text()) / 100.0;

      let links = [
          {"source": 0, "target": 2, "value": this.testNegButDiseased},
          {"source": 0, "target": 3, "value": this.testPositiveAndDiseased},
          {"source": 1, "target": 3, "value": this.testPositiveButHealthy},
          {"source": 1, "target": 4, "value": this.testNegAndHealthy}
      ];
      //Vue.set(this.graph, 'links', links); // no longer needed for Vue to recognizes changes in Vue 3
    },

    clearThumbTip: function(event, ui) {
      $("#probability-diseased-slider").find('.ui-slider-handle').empty();
      $("#test-accuracy-slider").find('.ui-slider-handle').empty();
    },

    onHighlight(value) {
      console.log("highlight:" + value);
    },
    onUnhighlight(value) {
      console.log("unhighlight:" + value);
    }
  }
}
</script>

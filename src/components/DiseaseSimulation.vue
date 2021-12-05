<template>
  <q-card class="card-md">
      <div class="inputs">
        <span class="input-line">The incidence of the disease in the population is
          <span id="probability-diseased">{{probDiseasedLabel}}</span>
          <span class="slider">
            <q-slider v-model="rawProbDiseased" :min="-2" :max="1" :step="0.1" label :label-value="probDiseasedLabel" />
          </span>
        </span>
        <div class="input-line">The disease testing accuracy is
           <span id="test-accuracy">{{testAccuracyLabel}}</span>
           <span class="slider">
             <q-slider v-model="rawTestAccuracy" :min="800" :max="999" label :label-value="testAccuracyLabel" />
           </span>
        </div>
      </div>

      <bayes-rule-view :graph="this.graph" :totalPopulation="this.totalPopulation"></bayes-rule-view>
      <sankey-view :graph="this.graph" @highlight="onHighlight" @unhighlight="onUnhighlight"></sankey-view>
      <venn-diagram-view :graph="this.graph" :totalPopulation="this.totalPopulation"></venn-diagram-view>
      <notes-content :testPositive="this.testPositive" :totalPopulation="this.totalPopulation"></notes-content>
  </q-card>
</template>

<script>

import diseaseConsts from './diseaseConsts.js';
import BayesRuleView from 'components/BayesRuleView.vue';
import SankeyView from 'components/SankeyView.vue';
import VennDiagramView from 'components/VennDiagramView.vue';
import NotesContent from 'components/NotesContent.vue';

export default {

  components: {
      BayesRuleView,
      SankeyView,
      VennDiagramView,
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
      default: 90
    }
  },

  data() {
     return {
       graph: {
         nodes: diseaseConsts.NODES,
         links: [0, 0, 0],
       },
       rawProbDiseased: Math.log10(this.initialPctDiseased),
       rawTestAccuracy: this.initialTestAccuracy * 10,
     }
  },

  computed: {
     probDiseased: function() {
       return this.pctDiseasedConverter(this.rawProbDiseased);
     },
     probDiseasedLabel: function() {
       return diseaseConsts.format(this.probDiseased, 2) + '%';
     },
     testAccuracy: function() {
       return this.testAccuracyConverter(this.rawTestAccuracy);
     },
     testAccuracyLabel: function() {
       return this.testAccuracy + '%';
     },
     diseasedPop: function() {
         return this.probDiseased * this.totalPopulation / 100.0;
     },
     healthyPop: function() {
         return this.totalPopulation - this.diseasedPop;
     },
     testNegAndHealthy: function() {
         return this.testAccuracy * this.healthyPop / 100.0;
     },
     testNegButDiseased: function() {
         return (1.0 - this.testAccuracy / 100.0) * this.diseasedPop;
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

  watch: {
    probDiseased(value, oldValue) {
      this.updateViews();
    },
    testAccuracy(value, oldValue) {
      this.updateViews();
    },
  },

  methods: {
    init: function() {
      this.updateViews();
    },

    pctDiseasedConverter: function(sliderValue) {
      return Math.pow(10, sliderValue);
    },

    testAccuracyConverter: function(sliderValue) {
      return sliderValue / 10;
    },

    updateViews: function() {
      let links = [
          {"source": 0, "target": 2, "value": this.testNegButDiseased},
          {"source": 0, "target": 3, "value": this.testPositiveAndDiseased},
          {"source": 1, "target": 3, "value": this.testPositiveButHealthy},
          {"source": 1, "target": 4, "value": this.testNegAndHealthy}
      ];
      this.graph.links = links;
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

<style scoped>

  .inputs {
    width: 860px;
  }

  .slider {
    float: right;
    width: 400px;
  }

  .q-slider--h {
    height: 20px;
  }

  #bayes-rule-view {
    width: 99%;
    padding: 10px;
  }
  #sankey-view {
    float: left;
    width: 50%;
    height: 650px;
  }
  #venn-diagram-view {
    float: right;
    width: 50%;
    height: 650px;
  }

  #notes {
    float: left;
    width: 100%;
  }
</style>

<style>
  body {
    font-family: Calibri, SansSerif, sans-serif;
    min-width: 900px;
    margin: 20px;
  }

  .diseased {
    font-weight: bold;
    color: #990000;
  }

  .healthy {
    font-weight: bold;
    color: #009900;
  }

  /* if this changes significantly, also update in disease.js. This version should be a little darker */
  .positive {
    font-weight: bold;
    color: #cc9900;
  }

  .big {
    font-size: 24px;
  }

</style>




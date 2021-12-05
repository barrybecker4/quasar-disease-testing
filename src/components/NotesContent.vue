<template>
  <q-card class="card-md" id="notes">
    <q-card-section class="title">Notes:</q-card-section>
    <q-card-section>
      <ul>
           <li>p(<span class="diseased">D</span> | <span class="positive">positive</span>)
               means the probability of having the disease given that you have tested positive.
               This is the notation for representing a conditional probability.
           </li>
           <li>p(<span class="positive">positive</span> | <span class="diseased">D</span>) =
               p(<span class="negative">negative</span> | <span class="healthy">H</span>)
               = <span id="accuracy-fn">test accuracy, by definition.</span>
           </li>
           <li>p(<span class="positive">positive</span>) is the total number of people that test positive
               ({{this.testPositiveLabel}}) divided by the total population
               ({{this.totalPopulation.toLocaleString()}})
               = {{this.probPositive}}</li>
           <li>The data is shown on the left
               with a <a href="https://bost.ocks.org/mike/sankey/">Sankey diagram</a>,
               and on the right with a <a href="https://en.wikipedia.org/wiki/Venn_diagram">Venn diagram</a>
           </li>
      </ul>
    </q-card-section>
  </q-card>
</template>

<script>

import diseaseConsts from './diseaseConsts.js'

export default {
  // name: 'ComponentName',
  setup () {
    return {}
  },

  props: {
    testPositive: {
      type: Number,
      default: 0,
    },
    totalPopulation: {
      type: Number,
      default: 1,
    },
  },

  computed: {
     testPositiveLabel: function() {
       return Math.round(this.testPositive).toLocaleString();
     },
     probPositive: function() {
       return diseaseConsts.format(this.testPositive / this.totalPopulation, 4);
     },
  },
}
</script>


<style scoped>
  .title {
      font-weight: bold;
      color: #333333;
      border-bottom: 1px solid black;
  }

</style>


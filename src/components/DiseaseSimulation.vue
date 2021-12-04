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
      <!--bayes-rule-view :graph="this.graph" :totalPopulation="this.totalPopulation"></bayes-rule-view>
      <sankey-view :graph="this.graph" @highlight="onHighlight" @unhighlight="onUnhighlight"></sankey-view>
      <venn-diagram-view :graph="this.graph" :totalPopulation="this.totalPopulation"></venn-diagram-view -->
      <!--notes-content :testPositive="this.testPositive" :totalPopulation="this.totalPopulation"></notes-content-->
  </q-card>
</template>

<script>

import IntroductionContent from 'components/NotesContent.vue';

export default {

  // name: 'ComponentName',

  components: {
      //BayesRuleView,
      //SankeyView,
      //VennDiagramView,
      //NotesContent
  },

  setup () {
    return {}
  }


}
</script>

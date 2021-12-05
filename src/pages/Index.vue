<template>
  <q-page class="column">
    <introduction-content/>
    <disease-simulation
            :total-population="10000000"
            :initial-pct-diseased="1"
            :initial-test-accuracy="90">
    </disease-simulation>


    <!--q-card class="card-md">
      <q-card-section class="bg-blue">
        Title
      </q-card-section>
      <q-card-section class="bg-orange">
        Body
      </q-card-section>
      <q-card-section>
        Body 2
      </q-card-section>
    </q-card-->

  </q-page>
</template>

<script>
import { defineComponent } from 'vue';
import IntroductionContent from 'components/IntroductionContent.vue';
import DiseaseSimulation from 'components/DiseaseSimulation.vue';


export default defineComponent({

  name: 'PageIndex',

  components: {
    IntroductionContent,
    DiseaseSimulation,
  },
})
</script>



<style scoped>

  /* For mobile */
  [class*="card-"] {
    font-size: 12px;
    margin: 10px;
    padding: 10px;
  }

  @media only screen and (min-width: 768px) {
    /* For desktop: */
    .card-md {
      font-size: 18px;
      margin: 20px;
      padding: 20px;
      box-sizing: border-box;
    }
  }

</style>


<template>
  <q-card class="card-md">
     <p>Suppose your doctor tests you for a terrible disease, and the results are
         <span class="positive">positive</span>!
         How worried should you be?
         <a href="https://en.wikipedia.org/wiki/Bayes%27_theorem">Bayes' theorem</a>,
         can be used to answer this question.
     </p>

     <p>Suppose the total population is <span id="total-population"></span> people.
         Let <span class="diseased">D</span> = <span class="diseased">Diseased</span>
         and <span class="healthy">H</span> = <span class="healthy">Healthy</span>.
     </p>
  </q-card>
</template>


<script>
  export default {
  }
</script>


<style scoped>
  .diseased {
      font-weight: bold;
      color: #990000;
  }

  .healthy {
      font-weight: bold;
      color: #009900;
  }
</style>

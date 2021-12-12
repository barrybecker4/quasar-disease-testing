<template>
  <div>
    <table class="bayes-rule-exp" style="margin:0; border-collapse: collapse;">
       <tr>
           <td rowspan="2" nowrap="nowrap">
               p(<span class="diseased">D</span> | <span class="positive">positive</span>)&nbsp; = &nbsp;
           </td>
           <td class="numerator">
               p(<span class="diseased">D</span>) &nbsp; p(<span class="positive">positive</span> | <span class="diseased">D</span>)
           </td>
           <td rowspan="2" nowrap="nowrap"> &nbsp; = &nbsp;</td>
           <td class="numerator"><span class="prob-diseased">
               {{probDiseasedTxt}}</span> * <span class="prob-pos-given-diseased">{{probPositiveGivenDiseasedTxt}}</span>
           </td>
           <td rowspan="2" nowrap="nowrap"> &nbsp; = &nbsp;</td>
           <td rowspan="2" width="100%">
               <span class="prob-diseased-result">{{probDiseasedGivenPositiveTxt}}&nbsp;</span> chance that you are infected.
               <span class="prob-diseased-worry" :style='{color: worryAttrs.color}'>{{worryAttrs.howMuch}}</span>
           </td>
       </tr>
       <tr>
           <td class="upper_line">p(<span class="positive">positive</span>)</td>
           <td class="upper_line"><span class="prob-positive">{{probPositiveTxt}}</span></td>
       </tr>
    </table>
  </div>
</template>

<script>
import diseaseConsts from './diseaseConsts.js';

export default {

  props: {
    graph: {
      type: Object, // Object or array defaults must be returned from a factory function
      default() {
        return {};
      }
    },
    totalPopulation: {
      type: Number,
      default: 0,
    }
  },

  computed: {
     numPositiveAndDiseased: function() {
         return this.graph.links[1].value;
     },
     numPositiveAndHealthy: function() {
         return this.graph.links[2].value;
     },
     numDiseased: function() {
         return this.graph.links[0].value + this.numPositiveAndDiseased;
     },
     numPositive: function() {
         return this.numPositiveAndDiseased + this.numPositiveAndHealthy;
     },
     probPositiveGivenDiseased: function() {
         return this.numPositiveAndDiseased / this.numDiseased;
     },
     probDiseasedTxt: function() {
         return diseaseConsts.format(this. numDiseased / this.totalPopulation, 5)
     },
     probPositiveTxt: function() {
         return diseaseConsts.format(this.numPositive / this.totalPopulation, 4);
     },
     probPositiveGivenDiseasedTxt: function() {
         return diseaseConsts.format(this.probPositiveGivenDiseased, 4);
     },
     probDiseasedGivenPositive: function() {
        return (100 * this.numDiseased * this.probPositiveGivenDiseased) / this.numPositive;
    },
     probDiseasedGivenPositiveTxt: function() {
         return diseaseConsts.format(this.probDiseasedGivenPositive, 2) + "%  ";
     },
     worryAttrs: function() {
         let howMuch = "Panic!!!";
         let color = "#dd0000";
         let prob = this.probDiseasedGivenPositive;

         if (prob <= 5) {
             howMuch = "Don't worry at all!";
             color = "#00dd00";
         }
         else if (prob <= 10) {
             howMuch = "Don't worry.";
             color = "#22cc00";
         }
         else if (prob <= 15) {
             howMuch = "Perhaps you should worry a little...";
             color = "#77aa00";
         }
         else if (prob <= 20) {
             howMuch = "Be concerned, but don't panic.";
             color = "#997700";
         }
         else if (prob <= 50) {
             howMuch = "You should worry.";
             color = "#bb6600";
         }
         else if (prob <= 80) {
             howMuch = "Yes, you should be very worried.";
             color = "#cc3300";
         }
         return { howMuch, color };
     }
  }
}
</script>


<style scoped>

  table.bayes-rule-exp {
    padding: 15px;
  }

  td.upper_line {
    border-top:solid 1px black;
    margin: 2px;
    text-align: center;
  }

  td.numerator {
    white-space: nowrap;
    padding: 3px;
  }

  table.fraction {
    text-align: center;
    vertical-align: middle;
     margin-top:0.5em;
     margin-bottom:0.5em;
     line-height: 2em;
  }

  .prob-diseased-result {
    font-size: 20px;
    font-weight: bold;
  }

  .prob-diseased-worry {
    font-size: 24px;
    padding: 6px;
    font-style: italic;
  }

</style>


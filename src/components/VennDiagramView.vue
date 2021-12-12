<template>
  <div id="venn-diagram-view"></div>
</template>

<script>

import * as d3 from 'd3'
import diseaseConsts from './diseaseConsts.js'
import circleUtils from './circleUtils.js'
import emitter from './emitter.js'



/** all circles will be relative to the test positive circle */
const TEST_POS_CIRCLE_RADIUS = 230;
const DURATION = 300;
const POP_LABEL_X = 50;
const POP_LABEL_Y = 240;

let margin = {top: 10, bottom: 10, left: 0};

export default {

  props: {
     graph: {
       type: Object, // Object or array defaults must be returned from a factory function
          default() { return {}; }
        },
       totalPopulation: {
         type: Number,
         default: 0,
       }
     },

     mounted() {
         this.init();
     },

     watch: {
       graph: {
         handler(g){
           this.render();
         },
         deep: true
        }
     },

     methods: {

       /** Add the initial svg structure */
       init: function() {

         const rootSvg = d3.selectAll("#" + this.$el.id).append("svg");

         const svg = rootSvg.append("g")
           .attr("transform",
                "translate(" + margin.left + "," + margin.top + ")");

         this.addPopulationCircle(svg);
         this.addTestPositiveCircle(svg);
         this.addDiseasedCircle(svg);
         this.addSubsetRegions(svg);

         window.addEventListener('resize', this.render);

         diseaseConsts.NAMED_INTERSECTIONS.forEach(eventName => {
           emitter.on(eventName, eventOpts => {
             eventOpts.action === 'highlight' ? this.doHighlight(eventName) : this.doUnhighlight(eventName);
           });
         });
       },

       addPopulationCircle(svg) {
         const vm = this;
         svg.append("circle")
           .attr("class", diseaseConsts.HEALTHY_TEST_NEG)
           .style("fill-opacity", 0.3)
           .attr("fill", diseaseConsts.TEST_NEG_HEALTHY_COLOR)
           .on("mouseover", function(d) {
             emitter.emit(diseaseConsts.HEALTHY_TEST_NEG, {action: "highlight"} );
           })
           .on("mouseout", function(d) {
             emitter.emit(diseaseConsts.HEALTHY_TEST_NEG, {action: "unhighlight"} );
           })
           .append("title").text("The whole population of " + this.totalPopulation.toLocaleString() +
           " people. \nThose outside the red circle are healthy");

         svg.append("text")
           .attr("class", "venn-label population")
           .attr("x", POP_LABEL_X)
           .attr("y", POP_LABEL_Y)
           .text("Whole Population");
       },

       addTestPositiveCircle(svg) {
         svg.append("circle")
           .attr("class", "test-positive-circle")
           .style("fill-opacity", 0.2)
           .attr("fill", diseaseConsts.POSITIVE_COLOR);
       },

       addDiseasedCircle(svg) {
         svg.append("text")
           .attr("class", "venn-label diseased")
           .text("Diseased");

         svg.append("line")
           .attr("class", "venn-line diseased");

         svg.append("circle")
           .attr("class", "diseased-circle")
           .style("fill-opacity", 0.1)
           .attr("fill", diseaseConsts.DISEASED_COLOR)
           .append("title");
       },

       addSubsetRegions(svg) {
         const vm = this;
         svg.append("path")
           .attr("class", diseaseConsts.DISEASED_TEST_POS)
           .style("fill-opacity", 0.4)
           .attr("fill", "#ffaa00")
           .on("mouseover", function(d) {
             emitter.emit(diseaseConsts.DISEASED_TEST_POS, {action: "highlight"} );
           })
           .on("mouseout", function(d) {
             emitter.emit(diseaseConsts.DISEASED_TEST_POS, {action: "unhighlight"} );
           })
           .append("title");

         svg.append("path")
           .attr("class", diseaseConsts.DISEASED_TEST_NEG)
           .style("fill-opacity", 0.5)
           .attr("fill", diseaseConsts.TEST_NEG_DISEASED_COLOR)
           .on("mouseover", function(d) {
             emitter.emit(diseaseConsts.DISEASED_TEST_NEG, {action: "highlight"} );
           })
           .on("mouseout", function(d) {
             emitter.emit(diseaseConsts.DISEASED_TEST_NEG, {action: "unhighlight"} );
           })
           .append("title");

         svg.append("path")
           .attr("class", diseaseConsts.HEALTHY_TEST_POS)
           .style("fill-opacity", 0.1)
           .attr("fill", "#55ee00")
           .on("mouseover", function(d) {
             emitter.emit(diseaseConsts.HEALTHY_TEST_POS, {action: "highlight"} );
           })
           .on("mouseout", function(d) {
             emitter.emit(diseaseConsts.HEALTHY_TEST_POS, {action: "unhighlight"} );
           })
           .append("title");

         svg.append("text")
           .attr("class", "venn-label positive")
           .text("Tested Positive");
       },

       /** update the Venn diagram */
       render: function() {

         const el = this.$el;
         const chartWidth = el.clientWidth;
         const chartHeight = el.clientHeight;
         const chartWidthD2 = chartWidth / 2;
         const chartHeightD2 = chartHeight / 2 + 30;

         const svg = d3.select("#" + el.id + " svg")
              .attr("width", chartWidth)
              .attr("height", chartHeight);

         const numPositiveAndDiseased = this.graph.links[1].value;
         const numPositiveAndHealthy =  this.graph.links[2].value;
         const testNegButDiseased = Math.round(this.graph.links[0].value);
         const numDiseased = testNegButDiseased + numPositiveAndDiseased;
         const numPositive = numPositiveAndDiseased + numPositiveAndHealthy;

         const testPositiveRad = TEST_POS_CIRCLE_RADIUS;
         const scaleFactor = Math.sqrt(this.totalPopulation / numPositive);
         const diseasedRad = testPositiveRad * Math.sqrt(numDiseased / numPositive);
         const popRad = testPositiveRad * scaleFactor;
         const diseaseArea = Math.PI * diseasedRad * diseasedRad;
         const overlap = (numPositiveAndDiseased / numDiseased) * diseaseArea;

         const distance = circleUtils.findCircleSeparation({
            radiusA: testPositiveRad,
            radiusB: diseasedRad,
            overlap: overlap
          });

         const  centerX = chartWidthD2 + testPositiveRad - 180;
         const diseasedCenterY = chartHeightD2 - distance;
         const popCircleCenterX = Math.max(chartWidthD2 - popRad, 0) + popRad + 10;

         svg.selectAll("circle.healthy--test-negative-healthy")
            .attr("cx", popCircleCenterX)
            .attr("cy", chartHeightD2)
            .attr("r", popRad);

         const rotation = 30 + 60 * (1.0 - TEST_POS_CIRCLE_RADIUS / popRad);
         let diseasedTop = diseasedCenterY - diseasedRad;
         svg.selectAll("text.venn-label.positive")
              .attr("x", centerX - 110)
              .attr("y", 0.54 * chartHeight);
         svg.selectAll("text.venn-label.diseased")
              .attr("x", centerX + 30)
              .attr("y", diseasedTop - 5);

         svg.selectAll("line.venn-line.diseased")
              .attr("x1", centerX)
              .attr("y1", diseasedTop + 1)
              .attr("x2", centerX + 29)
              .attr("y2", diseasedTop - 10);
         svg.selectAll("text.venn-label.population")
              .attr("transform", "rotate(" + -rotation + " " + POP_LABEL_X + " " + POP_LABEL_Y + ")");

         svg.selectAll("circle.test-positive-circle")
              .attr("cx", centerX)
              .attr("cy", chartHeightD2)
              .attr("r", testPositiveRad);

         svg.selectAll("circle.diseased-circle")
              .attr("cx", centerX)
              .attr("cy", diseasedCenterY)
              .attr("r", diseasedRad);

         // Draw paths for 2 halves of disease circle - the part in the intersection, and outside of it.
         // See https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial/Paths
         const pdPath = svg.selectAll("path.diseased--test-positive");
         const pctDiseasedGivenPositive = diseaseConsts.format(100 * numPositiveAndDiseased / numPositive, 2);
         pdPath.attr("d", this.pathFunc(centerX, chartHeightD2, testPositiveRad,
              centerX, diseasedCenterY, diseasedRad, 0, 1,   1, 1));
         pdPath.select("title").text(numPositiveAndDiseased.toLocaleString() +
              " (" + pctDiseasedGivenPositive + "%) " + " of the " +
              numPositive.toLocaleString() +
              " that tested positive\nactually have the disease.");

         const ndPath = svg.selectAll("path.diseased--test-negative-diseased");
         ndPath.attr("d", this.pathFunc(centerX, chartHeightD2, testPositiveRad,
              centerX, diseasedCenterY, diseasedRad, 0, 1, 0,0));
         ndPath.select("title").text(testNegButDiseased.toLocaleString() + " out of "
              + numDiseased.toLocaleString() + "\nwith the disease test negative.");

         const phPath = svg.selectAll("path.healthy--test-positive");
         phPath.attr("d", this.pathFunc(centerX, chartHeightD2, testPositiveRad,
              centerX, diseasedCenterY, diseasedRad, 1, 0,  1, 1));
         phPath.select("title")
              .text(numPositiveAndHealthy.toLocaleString()  + " are healthy out of the\n" +
                  numPositive.toLocaleString() + " that tested positive.");
       },

       pathFunc: function(x1, y1, rad1, x2, y2, rad2, largeArcFlag1, sweepFlag1, largeArcFlag2, sweepFlag2) {
         let interPoints = circleUtils.circleIntersection(x1, y1, rad1,  x2, y2, rad2);
         if (interPoints[0] === interPoints[2])
            return "M0,0";
         else {
            let rotation = 0;
            return "M" +
                 interPoints[0] + "," + interPoints[2] + "A" + rad2 + "," + rad2 + " " + rotation + " " +
                 largeArcFlag2 + " " + sweepFlag2 + " " +
                 interPoints[1] + "," + interPoints[3]+ "A" + rad1 + "," + rad1 +  " " + rotation + " " +
                 largeArcFlag1 + " " + sweepFlag1 + " " +
                 interPoints[0] + "," + interPoints[2];
         }
       },

       doHighlight: function(id) {

           let styles = {};
           let svg = getSvg();
           switch(id) {
               case diseaseConsts.HEALTHY_TEST_NEG:
                   styles.fillOpacity = 0.4;
                   styles.strokeWidth = 3;
                   styles.strokeOpacity = 0.6;
                   break;
               case diseaseConsts.DISEASED_TEST_POS:
                   styles.fillOpacity = 1.0;
                   styles.strokeWidth = 1;
                   styles.strokeOpacity = 1.0;
                   svg.select("circle.test-positive-circle").transition("tooltip").duration(DURATION)
                      .style("fill-opacity", 0.4)
                      .style("stroke", "black")
                      .style("stroke-width", 1)
                      .style("stroke-opacity", 0.3);
                   break;
               case diseaseConsts.HEALTHY_TEST_POS:
                   styles.fillOpacity = 0.5;
                   styles.strokeWidth = 1;
                   styles.strokeOpacity = 1.0;
                   svg.select("circle.test-positive-circle").transition("tooltip").duration(DURATION)
                       .style("stroke", "black")
                       .style("stroke-width", 2)
                       .style("stroke-opacity", 0.7);
                   break;
               case diseaseConsts.DISEASED_TEST_NEG:
                   styles.fillOpacity = 1.0;
                   styles.strokeWidth = 1;
                   styles.strokeOpacity = 1.0;
                   svg.select("circle.diseased-circle").transition("tooltip").duration(DURATION)
                       .style("stroke", "black")
                       .style("stroke-width", 1)
                       .style("stroke-opacity", 0.3);
                   break;
               default:
               throw "Unexpected style: " + id;
           }
           d3.select("#venn-diagram-view ." + id).transition().duration(DURATION)
                .style("stroke", "black")
                .style("fill-opacity", styles.fillOpacity)
                .style("stroke-width", styles.strokeWidth)
                .style("stroke-opacity", styles.strokeOpacity);
           ;
       },

       doUnhighlight: function(id) {
           let styles = {};
           let svg = getSvg();
           switch(id) {
               case diseaseConsts.HEALTHY_TEST_NEG:
                   styles.fillOpacity = 0.3;
                   styles.strokeWidth = 0;
                   styles.strokeOpacity = 0.0;
                   break;
               case diseaseConsts.DISEASED_TEST_POS:
                   styles.fillOpacity = 0.2;
                   styles.strokeWidth = 0;
                   styles.strokeOpacity = 0.0;
                   svg.select("circle.test-positive-circle").transition("tooltip").duration(DURATION)
                       .style("fill-opacity", 0.3)
                       .style("stroke-width", 0)
                       .style("stroke-opacity", 0.0);
                   break;
              case diseaseConsts.HEALTHY_TEST_POS:
                  styles.fillOpacity = 0.1;
                  styles.strokeWidth = 0;
                  styles.strokeOpacity = 0.0;
                  svg.select("circle.test-positive-circle").transition("tooltip").duration(DURATION)
                      .style("fill-opacity", 0.2)
                      .style("stroke-width", 0)
                      .style("stroke-opacity", 0.0);
                  break;
              case diseaseConsts.DISEASED_TEST_NEG:
                  styles.fillOpacity = 0.5;
                  styles.strokeWidth = 0;
                  styles.strokeOpacity = 0.0;
                  svg.select("circle.diseased-circle").transition("tooltip").duration(DURATION)
                      .style("stroke-width", 0)
                      .style("stroke-opacity", 0.0);
                  break;
              default:
                 throw "Unexpected style: " + id;
           }
           d3.select("#venn-diagram-view ." + id).transition().duration(DURATION)
               .style("stroke", "none")
               .style("fill-opacity", styles.fillOpacity)
               .style("stroke-width", styles.strokeWidth)
               .style("stroke-opacity", styles.strokeOpacity);
       },
     }
}

function getSvg() {
    return d3.select("#venn-diagram-view svg");
}
</script>

<style>

  .venn-label {
    font-size: 32px;
    font-weight: bold;
  }

  .venn-label.population {
    fill: #339933;
  }
  .venn-label.positive {
    fill: #999900;
  }
  .venn-label.diseased {
    fill: #aa0000;
  }

  .venn-line {
    stroke: #444444;
    stroke-width: 1px;
    stroke-opacity: 0.7;
  }

</style>

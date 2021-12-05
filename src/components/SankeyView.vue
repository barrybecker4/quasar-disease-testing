<template>
  <div id="sankey-view"></div>
</template>

<script>
import * as d3 from 'd3'
import * as d3Sankey from "d3-sankey";
import diseaseConsts from './diseaseConsts.js'

let sankey= d3Sankey.sankey()
    .nodeWidth(15)
    .nodePadding(30);

let defs, linksEl, nodesEl;
let width, height;
let links;
let margin = {top: 10, right: 10, bottom: 10, left: 10};
const DURATION = 300;
let colorScale = d3.scale.ordinal()
    .range([
        diseaseConsts.DISEASED_COLOR,
        diseaseConsts.HEALTHY_COLOR,
        diseaseConsts.TEST_NEG_DISEASED_COLOR,
        diseaseConsts.POSITIVE_COLOR,
        diseaseConsts.TEST_NEG_HEALTHY_COLOR
    ])
    .domain([
        diseaseConsts.DISEASED,
        diseaseConsts.HEALTHY,
        diseaseConsts.TEST_NEG_DISEASED,
        diseaseConsts.TEST_POS,
        diseaseConsts.TEST_NEG_HEALTHY
    ]);

export default {

  setup () {
    return {};
  },

  props: {
    graph: {
      type: Object,
      default: () => {}
    },
  },

  mounted() {
     let vm = this;
     this.init();

     /* This has changed in Vue 3
     this.$root.$on('highlight', data => {
         vm.doHighlight(data);
     });
     this.$root.$on('unhighlight', data => {
         vm.doUnhighlight(data);
     });*/
  },

  watch: {
     graphLinks(value) { this.render() },
  },

  computed: {
    graphLinks: function() {
      return this.graph.links;
    }
  },

  methods: {
    /** Add the initial svg structure */
    init: function() {
       let svg = d3.selectAll("#sankey-view").append("svg")
           .append("g")
           .attr("transform",
               "translate(" + margin.left + "," + margin.top + ")");

       defs = svg.append("defs");
       linksEl = svg.append("g");
       nodesEl = svg.append("g");

       window.addEventListener('resize', this.render);
    },

    /** update the sankey diagram */
    render: function() {
       let el = this.$el;
       let chartWidth = el.clientWidth;
       let chartHeight = el.clientHeight;
       width = chartWidth - margin.left - margin.right;
       height = chartHeight - margin.top - margin.bottom;

       // append the svg canvas to the page
       let svg = d3.select("#" + el.id + " svg")
           .attr("width", chartWidth)
           .attr("height", chartHeight);

       sankey(this.graph);
       sankey.size([width, height]);
       //sankey.update(this.graph);

       this.addColorGradients();
       this.addLinks();
       this.addNodes();
    },

    addLinks: function() {
      let vm = this;
      links = linksEl.selectAll(".link").data(this.graph.links, getGradientLinkId);

      let linkEnter = links.enter()
          .append("path")
          /*
          .on("mouseover", function(d) {
              vm.$root.$emit('highlight', getLinkId(d));
          })
          .on("mouseout", function(d) {
              vm.$root.$emit('unhighlight', getLinkId(d));
           })*/
          .attr("class", function(d) { return "link " + getLinkId(d); })
          .style("stroke-opacity", 0.3)
          .append("title");

      links
          .attr("d", d3Sankey.sankeyLinkHorizontal())
          .style("stroke", function(d) {
              return "url(#" + getGradientLinkId(d) + ")";
          })
          .style("stroke-width", function (d) {
              return Math.max(1, d.width);
          })
          .sort(function (a, b) {
              return b.width - a.width;
          });

      // add the link titles
      links.selectAll("path title")
          .text(function (d) {
              let data = d3.select(this.parentNode).datum();
              return d.source.name + " -> " + d.target.name + " (" + data.value.toLocaleString() + " people)";
          });
    },

    /**
      * consider foreign object for html styling
      * <foreignobject x="10" y="10" width="100" height="150">
      *   <body xmlns="http://www.w3.org/1999/xhtml">
      *   <div>Here is a <strong>paragraph</strong> that requires <em>word wrap</em></div>
      *  </body>
      */
    addNodes: function() {
      let nodes = nodesEl.selectAll(".node").data(this.graph.nodes);
      let nodeEnter = nodes.enter();

      let nodeG = nodeEnter.append("g")
          .attr("class", "node");

      nodes.attr("transform", function (d) {
          return "translate(" + d.x0 + "," + d.y0 + ")";
      });

      // add the rectangles for the nodes
      nodeG.append("rect")
          .attr("width", sankey.nodeWidth())
          .style("fill", nodeColor)
          .style("fill-opacity", 0.5)
          .style("stroke", function (d) {
              return d3.rgb(d.color).darker(1);
          })
          .on("mouseover", function(d) {
               d3.select(this).transition("tooltip").duration(DURATION)
                   .style("fill-opacity", 0.9);
               })
          .on("mouseout", function(d) {
               d3.select(this).transition("tooltip").duration(DURATION)
                   .style("fill-opacity", 0.5);
               })
          .append("title");
      nodes.select("rect title")
          .text(function (d) {
              return d.name + "\n" + d.node.toLocaleString();
          });

      nodes.select("rect")
          .attr("height", function (d) {
              return d.y1 - d.y0; //d.dy;  width?
          });

      // add in the title for the nodes
      nodeG.append("text")
          .attr("x", -6)
          .attr("dy", ".35em")
          .attr("text-anchor", "end")
          .attr("transform", null)
          .text(function (d) {
              return d.name;
          })
          .filter(function (d) {
              return d.x < width / 2;
          })
          .attr("x", 6 + sankey.nodeWidth())
          .attr("text-anchor", "start");

      nodes.select("text")
          .attr("y", function (d) {
              return (d.y1 - d.y0) / 2;   // d.dy / 2;  width/2
          });
    },

    /** add link color gradients */
    addColorGradients: function() {

      let grads = defs.selectAll("linearGradient")
          .data(this.graph.links, getGradientLinkId);

      grads.enter().append("linearGradient")
          .attr("id", getGradientLinkId)
          .attr("gradientUnits", "userSpaceOnUse");

      grads.html("") // erase any existing <stop> elements on update
          .append("stop")
          .attr("offset", "0%")
          .attr("stop-color", function (d) {
              return nodeColor((+d.source.x0 <= +d.target.x0) ? d.source : d.target);
          });

      grads.append("stop")
          .attr("offset", "100%")
          .attr("stop-color", function (d) {
              return nodeColor((+d.source.x0 > +d.target.x0) ? d.source : d.target)
          });
    },

    doHighlight: function(id) {
         d3.select("#sankey-view ." + id).transition().duration(DURATION)
             // opacity change is less for large area
             .style("stroke-opacity", function(d) { return d.target.node == "4" ? 0.45 : 0.7; });
         ;
    },
    doUnhighlight: function(id) {
         d3.select("#sankey-view ." + id).transition().duration(DURATION)
             .style("stroke-opacity", 0.3);
    },
  },
}


function getLinkId(d) {
   return d.source.id + "--" + d.target.id;
}

function getGradientLinkId(d) {
   return "gradient-" + getLinkId(d);
}

function nodeColor(d) {
   return d.color = colorScale(makeValid(d.id));
}

function makeValid(s) {
   return s.replace(/ /g, "").replace(/,/g, "");
}

</script>

<style>

  .node rect {
    cursor: move;
    shape-rendering: crispEdges;
  }

  .node text {
    pointer-events: none;
    text-shadow: 0 1px 0 #fff;
  }

  .link {
    fill: none;
  }

</style>

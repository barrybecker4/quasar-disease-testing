<template>
  <div id="sankey-view"></div>
</template>

<script>
import * as d3 from 'd3'
import * as d3Sankey from "d3-sankey";
import diseaseConsts from './diseaseConsts.js'
import emitter from './emitter.js'

emitter.on('my-event', e => console.log('my-event', e) );


let sankey= d3Sankey.sankey()
    .nodeWidth(15)
    .nodePadding(30);

let defs, linksEl, nodesEl;
let width, height;
let links;
let margin = {top: 10, right: 10, bottom: 10, left: 10};
const DURATION = 300;
const NODE_TEXT_OFFSET = 6;

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

  props: {
    graph: {
      type: Object,
      default: () => {}
    },
  },

  mounted() {
    //let vm = this;
    this.init();
  },

  watch: {
    graphLinks: {
      //immediate: true,
      handler(value) {
        this.render();
      }
    },
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

       setTimeout(function() {
         window.dispatchEvent(new Event('resize'));
       }, 10);

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

       this.addLinkColorGradients();
       this.addLinks();
       this.addNodes();
    },

    addLinks: function() {
      links = linksEl.selectAll(".link").data(this.graph.links, getGradientLinkId);

      let linkEnter = links.enter()
          .append("path")
          .attr("class", function(d) { return "link " + getLinkId(d); })
          .style("stroke-opacity", 0.3)
          .on("mouseover", function(d) {
            d3.select(this).transition("tooltip").duration(DURATION)
              .style("stroke-opacity", function(d) { return d.target.node == "4" ? 0.45 : 0.7; });
          })
          .on("mouseout", function(d) {
            d3.select(this).transition("tooltip").duration(DURATION)
              .style("stroke-opacity", 0.3);
           })
          .append("title");

      links
          .attr("d", d3Sankey.sankeyLinkHorizontal())
          .style("stroke", function(d) {
              return "url(#" + getGradientLinkId(d) + ")";
          })
          .style("stroke-width", function (d) {
              return Math.max(1, d.width);
          });

      // add the link titles
      links.selectAll("path title")
          .text(function (d) {
              let data = d3.select(this.parentNode).datum();
              return d.source.name + " -> " + d.target.name + " (" + data.value.toLocaleString() + " people)";
          });
    },

    addNodes: function() {
      let nodes = nodesEl.selectAll(".node").data(this.graph.nodes);
      let nodeEnter = nodes.enter();

      this.addNodeGeometry(nodeEnter);
      this.updateNodeGeometry(nodes);
    },

    addNodeGeometry(nodeEnter) {
      let nodeG = nodeEnter.append("g")
        .attr("class", "node");

      nodeG.append("rect")
        .attr("width", sankey.nodeWidth())
        .style("fill", nodeColor)
        .style("fill-opacity", 0.5)
        .style("stroke", function (d) {
          return d3.rgb(d.color).darker(1);
        })
        .on("mouseover", function(d) {
          d3.select(this).transition("tooltip").duration(DURATION)
            .style("fill-opacity", 0.99)
            .style("stroke-width", 2);
        })
        .on("mouseout", function(d) {
          d3.select(this).transition("tooltip").duration(DURATION)
            .style("fill-opacity", 0.5)
            .style("stroke-width", 1);
        })
        .append("title");

      // add in the title for the nodes
      nodeG.append("text")

        .attr("dy", ".35em")
        .attr("text-anchor", "end")
        .attr("transform", null)
        .text(function (d) {
          return d.name;
        });
    },

    updateNodeGeometry(nodes) {
      nodes.attr("transform", (d) => "translate(" + d.x0 + "," + d.y0 + ")");

      nodes.select("rect title")
        .text((d) => {
          return d.name + "\n" + d.value.toLocaleString();
        })

      nodes.select("rect")
        .attr("height", (d) => d.y1 - d.y0);

      nodes.select("text")
        .attr("y", (d) => (d.y1 - d.y0) / 2)
        .attr("x", (d) => d.x0 < (width / 2) ? NODE_TEXT_OFFSET + sankey.nodeWidth() : -NODE_TEXT_OFFSET)
        .attr("text-anchor", (d) => d.x0 < (width / 2) ? "start" : "end");
    },

    addLinkColorGradients: function() {

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

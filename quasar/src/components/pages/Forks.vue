<template>
  <div class="pools">
    <h2> Fork Monitor </h2>
    <div v-if="is_loaded">
    </div>
    <loading v-else></loading>
    <svg id="svg-forks" width="680" height="510" :class="getDisplayClass()">
      <g transform="translate(80, 40)" id="root"></g>
      <g stroke="#fec02c" transform="translate(60, 40)" id="axis"></g>
    </svg>
  </div>

</template>

<script>
const d3 = require('d3');
import BlocksService from '@/services/BlocksService'
import Loading from '@/components/utils/Loading'

export default {
  name: 'stars',

  components:{ Loading, },

  data () {
    return {
      forks: [],
      is_loaded: false,
      pixels_per_second: 2,
      node_width: 300,
      node_height: 20
    }
  },
  mounted () {
    this.getForks()
  },
  methods: {
    getDisplayClass() {
      if (this.is_loaded) {
        return "displaySVG"
      }
      return "displayNOSVG"
    },
    async getForks() {
      this.is_loaded = false
      let nodes = {
        "0x69d1f8cef325b99d1ac2ee3acb92214bd774daee16d0409bffdaca22ef2b1461" : {
            "number": 7248048,
            "timestamp": 1550736462,
            "hash": "0x69d1f8cef325b99d1ac2ee3acb92214bd774daee16d0409bffdaca22ef2b1461",
            "difficulty": 2978675520298708,
            "parents": [
                "0x9890232c149e2236bfa4525b70d273f4dc41a151e06e41c32aa7f4cc39ab44c8"
            ]
        },
        "0x38523445b8babd9960d293e909ad69976bf861863bc29ca5cbbb2b04d27c0ac1" :         {
            "number": 7248049,
            "timestamp": 1550736505,
            "hash": "0x38523445b8babd9960d293e909ad69976bf861863bc29ca5cbbb2b04d27c0ac1",
            "difficulty": 2978675520298708,
            "parents": [
                "0x69d1f8cef325b99d1ac2ee3acb92214bd774daee16d0409bffdaca22ef2b1461"
            ]
        },
        "0x37523445b8babd9960d293e909ad69976bf861863bc29ca5cbbb2b04d27c0ac1" :         {
            "number": 7248050,
            "timestamp": 1550736605,
            "hash": "0x37523445b8babd9960d293e909ad69976bf861863bc29ca5cbbb2b04d27c0ac1",
            "difficulty": 2978675520298708,
            "parents": [
                "0x38523445b8babd9960d293e909ad69976bf861863bc29ca5cbbb2b04d27c0ac1"
            ]
        },
        "0x36523445b8babd9960d293e909ad69976bf861863bc29ca5cbbb2b04d27c0ac1" :         {
            "number": 7248050,
            "timestamp": 1550736605,
            "hash": "0x36523445b8babd9960d293e909ad69976bf861863bc29ca5cbbb2b04d27c0ac1",
            "difficulty": 2978675520298608,
            "parents": [
                "0x38523445b8babd9960d293e909ad69976bf861863bc29ca5cbbb2b04d27c0ac1"
            ]
        },
        "0x35523445b8babd9960d293e909ad69976bf861863bc29ca5cbbb2b04d27c0ac1" :         {
            "number": 7248051,
            "timestamp": 1550736629,
            "hash": "0x35523445b8babd9960d293e909ad69976bf861863bc29ca5cbbb2b04d27c0ac1",
            "difficulty": 2978675520298708,
            "parents": [
                "0x37523445b8babd9960d293e909ad69976bf861863bc29ca5cbbb2b04d27c0ac1",
                "0x36523445b8babd9960d293e909ad69976bf861863bc29ca5cbbb2b04d27c0ac1"
            ]
        },
      }
      this.buildGraph(nodes)
      this.is_loaded = true
    },
    async drawGraph(nodes, edges) {
            var svg = d3.select("svg"),
                g = svg.select("#root");

            var xmax = Math.max.apply(null, nodes.map(function(d) { return d.x; }));
            var ymax = Math.max.apply(null, nodes.map(function(d) { return d.y; }));

            svg.attr("height", ymax + this.node_height * 2);
            svg.attr("width", xmax + this.node_width + 60);


            var nodeset = g.selectAll(".node")
                .data(nodes, function(d) { return d.block.hash; });
            var t = nodeset.transition()
                .attr("class", function(d) { return "node" + (d.canonical?" canonical-node":" uncle-node"); })
                .attr("transform", function(d) { return "translate(" + d.x + ", " + d.y + ")"; });

            var node = nodeset.enter().append("g")
                .attr("class", function(d) { return "node" + (d.canonical?" canonical-node":" uncle-node"); })
                .attr("transform", function(d) { return "translate(" + d.x + ", " + d.y + ")"; });
            node.append("circle")
                .attr("r", 2.5)
                .attr("fill", "#666")
            node.append("text")
                .attr("class", "nodehash")
                .attr("dy", 13)
                .attr("x", 8)
                .attr("fill", "#00c000")
                .style("text-anchor", "start")
                .text(function(d) { return d.block.hash.substring(0, 10); });
            node.append("text")
                .attr("class", "nodenum")
                .attr("fill", "#00c000")
                .attr("dy", -3)
                .attr("x", 8)
                .style("text-anchor", "start")
                .text(function(d) { return d.block.number; });
            nodeset.exit().remove();

            var link = g.selectAll(".link")
                .data(edges, function(d) { return [d[0].block.hash, d[1].block.hash]; });
            link.transition(t)
                .attr("class", function(d) { 
                    return "link" + ((d[0].canonical && d[1].canonical)?" canonical-link":" uncle-link"); 
                })
                .attr("d", function(d) {
                    var fromNode = d[0],
                        toNode = d[1];

                    return "M" + toNode.x + "," + toNode.y
                        + "C" + toNode.x  + "," + (toNode.y + 15)
                        + " " + fromNode.x + "," + (fromNode.y - 15)
                        + " " + fromNode.x + "," + fromNode.y;
                });
            link.enter().append("path")
                .attr("class", function(d) { 
                    return "link" + ((d[0].canonical && d[1].canonical)?" canonical-link":" uncle-link"); 
                })
                .attr("d", function(d) {
                    var fromNode = d[0],
                        toNode = d[1];

                    return "M" + toNode.x + "," + toNode.y
                        + "C" + toNode.x  + "," + (toNode.y + 15)
                        + " " + fromNode.x + "," + (fromNode.y - 15)
                        + " " + fromNode.x + "," + fromNode.y;
                });
            link.exit().remove();

            var earliest = Math.min.apply(null, nodes.map(function(n) { return n.block.timestamp; }));
            var latest = Math.max.apply(null, nodes.map(function(n) { return n.block.timestamp; }));
            var timeScale = d3.scaleTime()
                .domain([new Date(latest * 1000), new Date(earliest * 1000)])
                .range([0, ymax]);
            var timeAxis = d3.axisLeft(timeScale)
                .ticks(d3.timeMinute.every(1));
            var axisG = svg.select("#axis");
            axisG.enter()
                .call(timeAxis);
            axisG.transition()
                .call(timeAxis);
    },
    async layoutNodes(nodes, mincol, latest) {
            var columns = [];

            for(var i = 0; i < nodes.length; i++) {
                var node = nodes[i];
                node.y = (latest - node.block.timestamp) * this.pixels_per_second;
                for(var j = 0; j < columns.length; j++) {
                    let self = this
                    if(j == 0 && !node.canonical)
                        continue;
                    if(columns[j] < node.y)
                        continue;
                    if(node.siblings.map(function(n) { return n.x == j * self.node_width; }).reduce(function(a, b) { return a | b; }, false))
                        continue;
                    node.x = (j + mincol) * this.node_width;
                    columns[j] = node.y - this.node_height;
                    break;
                }
                if (node.x == undefined) {
                    node.x = (columns.length + mincol) * this.node_width;
                    columns.push(node.y + this.node_height);
                }
            }
            return mincol + columns.length;
    },
    async layoutSubgraph(nodes, mincol, latest) {
            // Populate siblings and find roots
            var canonical = {}; // Traces the canonical chain
            for(var i = nodes.length - 1; i >= 0; i--) {
                var node = nodes[i];

                if(node.children.length == 0 || canonical[node.block.hash] != undefined) {
                    node.canonical = true;
                    if(node.parents.length > 0)
                        canonical[node.parents[0].block.hash] = true;
                }

                for(var j = 0; j < node.parents.length; j++) {
                    for(var k = 0; k < node.parents[j].children.length; k++) {
                        if(node.parents[j].children[k] != node)
                            node.siblings.push(node.parents[j].children[k]);
                    }
                }

                for(var j = 0; j < node.children.length; j++) {
                    for(var k = 0; k < node.children[j].parents.length; k++) {
                        if(node.children[j].parents[k] != node)
                            node.siblings.push(node.children[j].parents[k]);
                    }
                }
            }

            // Lay out the physical graph
            return await this.layoutNodes(nodes, mincol, latest);
    },
    async findSubgraphs(nodes) {
            var subgraphs = [];
            var subgraphMap = {};
            var totalNodes = 0;
            var nextSubgraph = 0;

            for(var i = 0; i < nodes.length; i++) {
                if(subgraphMap[nodes[i].block.hash] == undefined) {
                    var frontier = [nodes[i]];
                    var subgraph = [];
                    subgraphs[nextSubgraph++] = subgraph;

                    while(frontier.length != 0) {
                        var node = frontier.pop();
                        if(subgraphMap[node.block.hash] != undefined)
                            continue;
                        subgraphMap[node.block.hash] = node;
                        subgraph.push(node);
                        for(var j = 0; j < node.parents.length; j++)
                            frontier.push(node.parents[j]);
                        for(var j = 0; j < node.children.length; j++)
                            frontier.push(node.children[j]);
                    }
                }
            }

            for(var i = 0; i < subgraphs.length; i++)
                subgraphs[i].sort(function(a, b) { return a.block.timestamp - b.block.timestamp; });
            return subgraphs;
    },
    async buildGraph(blocks) {
     var earliest = undefined;

            // Build a map of new node objects
            var nodeMap = {}
            for(var hash in blocks) {
                var block = blocks[hash];
                var node = {
                    block: block,
                    parents: [],
                    children: [],
                    siblings: [],
                    canonical: false,
                }
                nodeMap[block.hash] = node;
            }

            // Generate node and edge lists, and populate parents and children
            var nodes = [];
            var edges = [];
            for(var hash in nodeMap) {
                var node = nodeMap[hash];

                for(var i = 0; i < node.block.parents.length; i++) {
                    var parentNode = nodeMap[node.block.parents[i]];
                    if(parentNode != undefined) {
                        node.parents.push(parentNode);
                        edges.push([parentNode, node]);
                        parentNode.children.push(node);
                    }
                }

                nodes.push(node);
            }

            var latest = nodes.map(function(n) { return n.block.timestamp; }).reduce(function(a, b) { return Math.max(a, b); }, 0);
            
            var subgraphs = await this.findSubgraphs(nodes);
            subgraphs.sort(function(a, b) {
                function totalDifficulty(nodes) {
                    return nodes.map(function(a) { return a.block.difficulty; }).reduce(function(a, b) { return a + b;}, 0);
                }
                return totalDifficulty(b) - totalDifficulty(a);
            });

            var mincol = 0;
            for(var i = 0; i < subgraphs.length; i++) {
                mincol = await this.layoutSubgraph(subgraphs[i], mincol, latest);
            }

            await this.drawGraph(nodes, edges); 
    }
  }
}
</script>

<style type="text/css">
.displaySVG {
  display: block;
}

#svg-forks {
  margin: 0 auto;
  color: #00c000;
}
.displayNOSVG {
  display: none;
}

#axis path {
  stroke: #fff
}

#root path {
  stroke: #666
}
</style>

<template>
  <!-- <div id="app"> -->
  <v-app id="app">
    <h3>Array:</h3>
    <p v-for="item in array" :key="item.message">{{ item }}</p>

    <h3>Min from d3:</h3>
    <p>{{ minFromD3 }}</p>

    <h3>Mean from d3:</h3>
    <p>{{ meanFromD3 }}</p>

    <h3>Max from d3:</h3>
    <p>{{ maxFromD3 }}</p>

    <v-container>
      <v-card class="ml-10 mr-10 mb-10" elevation="2" outlined>
        <h3>Graph with d3:</h3>
        <div id="cool-graph"></div>
      </v-card>
    </v-container>
  </v-app>
  <!-- </div> -->
</template>


<script>
const d3 = require("d3");

export default {
  name: "App",
  data() {
    return {
      array: [
        { date: "2013-04-01", value: 22 },
        { date: "2013-04-02", value: 23 },
        { date: "2013-04-03", value: 22 },
        { date: "2013-04-04", value: 23 },
        { date: "2013-04-05", value: 22 },
        { date: "2013-04-06", value: 24 },
        { date: "2013-04-07", value: 21 },
        { date: "2013-04-08", value: 22 },
        { date: "2013-04-09", value: 23 },
        { date: "2013-04-10", value: 22 },
        { date: "2013-04-11", value: 22 },
        { date: "2013-04-12", value: 23 },
        { date: "2013-04-13", value: 21 },
        { date: "2013-04-14", value: 22 },
        { date: "2013-04-15", value: 27 },
        { date: "2013-04-16", value: 23 },
        { date: "2013-04-17", value: 22 },
        { date: "2013-04-18", value: 23 },
        { date: "2013-04-19", value: 22 },
        { date: "2013-04-20", value: 21 },
        { date: "2013-04-21", value: 22 },
      ],

      minFromD3: 0,
      meanFromD3: 0,
      maxFromD3: 0,
    };
  },
  methods: {
    doD3Stuff() {
      this.minFromD3 = d3.min(this.array, (d) => {
        return d.value;
      });
      this.meanFromD3 = d3.mean(this.array, (d) => {
        return d.value;
      });
      this.maxFromD3 = d3.max(this.array, (d) => {
        return d.value;
      });

      // graph vars
      let margin = { top: 20, right: 30, bottom: 30, left: 40 };
      let height = 100;
      let width = 600;

      // Prepare dates from data
      let formattedData = this.array.map((item) => {
        return { date: d3.timeParse("%Y-%m-%d")(item.date), value: item.value };
      });

      // SVG
      let svg = d3
        .select("#cool-graph")
        .append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom);

      // X AXIS
      let x = d3
        .scaleTime()
        .domain(
          d3.extent(formattedData, function (d) {
            return d.date;
          })
        )
        .range([margin.left, width - margin.right]);

      svg
        .append("g")
        .attr("transform", `translate(0, ${height - margin.bottom})`)
        .call(
          d3
            .axisBottom(x)
            .tickArguments([d3.timeDay.every(1)])
            .tickFormat(d3.timeFormat("%a %d"))
        )
        .selectAll("text")
        .style("text-anchor", "end")
        .attr("dx", "-.8em")
        .attr("dy", ".15em")
        .attr("transform", "rotate(-65)");

      // Y AXIS
      let y = d3
        .scaleLinear()
        .domain([
          d3.min(formattedData, function (d) {
            return d.value;
          }) / 1.25,
          d3.max(formattedData, function (d) {
            return d.value;
          }),
        ])
        .range([height - margin.bottom, margin.top]);

      svg
        .append("g")
        .attr("transform", `translate(${margin.left}, 0)`)
        .call(d3.axisLeft(y).ticks(3));

      // Add the line
      svg
        .append("path")
        .datum(formattedData)
        .attr("fill", "#cce5df")
        .attr("stroke", "#69b3a2")
        .attr("stroke-width", 1.5)
        .attr(
          "d",
          d3
            .area()
            .x(function (d) {
              return x(d.date);
            })
            .y0(
              y(
                d3.min(formattedData, function (d) {
                  return d.value;
                }) / 1.25
              )
            )
            .y1(function (d) {
              return y(d.value);
            })
            .curve(d3.curveMonotoneX)
        );
    },
  },
  mounted() {
    this.doD3Stuff();
  },
};
</script>


<style scoped>
h3 {
  font-family: "Courier New", Courier, monospace;
  font-weight: bold;
  padding: 0;
  margin: 0;
}
p {
  font-family: "Courier New", Courier, monospace;
  font-weight: bold;
  padding: 0;
  margin: 0;
  font-size: 9pt;
}
</style>

<template>
  <v-app id="app">
    <v-container>
      <v-card elevation="2" outlined>
        <v-card-title
          ><v-icon>mdi-chart-line</v-icon> Graph with d3:</v-card-title
        >
        <div id="cool-graph"></div>
        <v-row>
          <v-col cols="12" sm="4">
            <v-card-text
              ><span class="font-weight-bold"
                ><v-icon>mdi-code-less-than</v-icon> Min:</span
              >
              {{ minFromD3 }}</v-card-text
            >
          </v-col>
          <v-col cols="12" sm="4">
            <v-card-text
              ><span class="font-weight-bold"
                ><v-icon>mdi-approximately-equal-box</v-icon> Mean:</span
              >
              {{ meanFromD3 }}</v-card-text
            >
          </v-col>
          <v-col cols="12" sm="4">
            <v-card-text
              ><span class="font-weight-bold"
                ><v-icon>mdi-code-greater-than</v-icon> Max:</span
              >
              {{ maxFromD3 }}</v-card-text
            >
          </v-col>
        </v-row>
        <v-card-text class="pt-0"
          >This line chart displays an x-axis slanted tick for each day. The
          y-axis is from the lowest data value divided by 1.25 and reaches to
          the highest data value.
        </v-card-text>
      </v-card>

      <v-card elevation="2" outlined class="mt-10">
        <v-simple-table>
          <template v-slot:default>
            <thead>
              <tr class="grey lighten-3">
                <th class="text-left pl-0">
                  <v-card-title>Data</v-card-title>
                  <v-card-subtitle>({{ array.length }} items)</v-card-subtitle>
                </th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in array" :key="item.date">
                <td>{{ item }}</td>
              </tr>
            </tbody>
          </template>
        </v-simple-table>
      </v-card>
    </v-container>
  </v-app>
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
      let strokeWidth = 1.5;
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
        .attr("preserveAspectRatio", "xMinYMin meet")
        .attr("viewBox", "0 0 600 100");

      // X AXIS
      let x = d3
        .scaleTime()
        .domain(
          d3.extent(formattedData, function (d) {
            return d.date;
          })
        )
        .range([margin.left + strokeWidth / 2, width - margin.right]);

      svg
        .append("g")
        .attr("transform", `translate(0, ${height - margin.bottom})`)
        .call(
          d3
            .axisBottom(x)
            .tickArguments([d3.timeDay.every(1)])
            .tickFormat(d3.timeFormat("%a %d"))
        )
        .attr("color", "#333333")
        .selectAll("text")
        .style("text-anchor", "end")
        .attr("dx", "-.8em")
        .attr("dy", ".15em")
        .attr("font-size", "5pt")
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
        .call(d3.axisLeft(y).ticks(3))
        .attr("color", "#333333")
        .selectAll("text")
        .attr("font-size", "5pt");

      // Add the line
      let path = svg
        .append("path")
        .datum(formattedData)
        .attr("fill", "#cce5df")
        .attr("stroke", "#69b3a2")
        .attr("stroke-width", strokeWidth)
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
            .curve(d3.curveCatmullRom.alpha(0.5))
        );

      // Draw line animation
      let totalLength = path.node().getTotalLength();
      path
        .attr("stroke-dasharray", totalLength + " " + totalLength)
        .attr("stroke-dashoffset", totalLength)
        .transition() // Call Transition Method
        .duration(2500) // Set Duration timing (ms)
        .ease(d3.easeLinear) // Set Easing option
        .attr("stroke-dashoffset", 0);


    },
  },
  mounted() {
    this.doD3Stuff();
  },
};
</script>


<style scoped>
</style>

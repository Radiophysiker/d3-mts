<template>
  <div>
    <div ref="chart" class="hello"></div>
    <select v-model="selectedUser" @change="drawChart(selectedUser)" >
      <option v-for="item in data" :value="item.date" :key="String(item.date)">{{item.date}}</option>
    </select>
  </div>
</template>

<script>










import * as d3 from 'd3'

export default {
  name: 'Chart',
  data() {
    return {
      data: [],
      selectedUser: '',
      yMin: 0,
      yMax: 0,
      date: {
        min: new Date(2014, 0, 1),
        max: new Date(2018, 0, 1),
      },
      color: ['steelblue', 'red'],
      chartSize: {
        width: 960,
        height: 500,
        margin: {
          top: 10,
          right: 30,
          bottom: 30,
          left: 80,
        },
      },
    }
  },
  created() {
    // получаем данные
    this.fetch()
  },
  methods: {
    fetch() {
      d3.dsv(
        ';',
        'https://raw.githubusercontent.com/Radiophysiker/mts/master/data/d3_sample1.csv',
        (d) => {
          return {
            date: d.xdate, // конвектируем столбцы под наше названия
            value: d.value,
          }
        },
      )
        .then((data) => {
          let parseDate = d3.timeParse('%d.%m.%Y')
          this.data = data.filter((elem) => elem.value) // убираем пустые значения
          this.data.forEach((d) => {
            d.date = parseDate(d.date) // парсим дату
            d.value = parseFloat(d.value)
          })
          // находим максимумы и мин
          this.yMax = d3.max(this.data, (d) => d.value)
          this.yMin = d3.min(this.data, (d) => d.value)
          this.date.min = d3.min(this.data, (d) => d.date)
          this.date.max = d3.max(this.data, (d) => d.date)
          this.drawChart() // после получения данных рисуем график
        })
        .catch(function(e) {
          throw e
        })
    },
    drawChart(currentMaxDate = this.date.max) {
      let width = this.chartSize.width - this.chartSize.margin.left - this.chartSize.margin.right
      let height = this.chartSize.height - this.chartSize.margin.top - this.chartSize.margin.bottom
      // set the ranges
      var x = d3
        .scaleTime()
        .domain([this.date.min, currentMaxDate])
        .rangeRound([0, width])
      var y = d3.scaleLinear().range([height, 0])

      // set the parameters for the histogram
      var histogram = d3
        .histogram()
        .value((d) => d.date)
        .domain(x.domain())
        .thresholds(x.ticks(d3.timeMonth))

      // append the svg object to the body of the page
      // append a 'group' element to 'svg'
      // moves the 'group' element to the top left margin
      var svg = d3
        .select(this.$refs.chart)
        .append('svg')
        .attr('width', this.chartSize.width)
        .attr('height', this.chartSize.height)
        .append('g')
        .attr(
          'transform',
          'translate(' + this.chartSize.margin.left + ',' + this.chartSize.margin.top + ')',
        )

      var bins = histogram(this.data)

      this.date.min = d3.min(bins, (d) => d[0].date)
      this.date.max = d3.max(bins, (d) => d[0].date)
      let colorScale = d3
        .scaleLinear()
        .domain([this.yMin, this.yMax])
        .range([d3.rgb(this.color[0]), d3.rgb(this.color[1])])
      // Scale the range of the data in the y domain
      y.domain([0, this.yMax])
      // append the bar rectangles to the svg element
      svg
        .selectAll('rect')
        .data(bins)
        .enter()
        .append('rect')
        .attr('class', 'bar')
        .attr('x', 1)
        .attr('transform', (d) => `translate(${x(d.x0)}, ${height * (1 - d[0].value / this.yMax)})`)
        .attr('width', (d) => (x(d.x1) - x(d.x0) - 1 < 0 ? 0 : x(d.x1) - x(d.x0) - 1))
        .attr('height', (d) => (height * d[0].value) / this.yMax)
        .attr('fill', (d) => colorScale(d[0].value))
      // add the x Axis
      svg
        .append('g')
        .attr('transform', 'translate(0,' + height + ')')
        .call(d3.axisBottom(x))

      // add the y Axis
      svg.append('g').call(d3.axisLeft(y))
    },
  },
}</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>

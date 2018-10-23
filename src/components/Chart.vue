<template>
  <div>
    C:
    <el-select
      v-model="selectedMin"
      size="small"
      popper-class="flex"
      placeholder="Выберите дату"
      @change="drawChart(selectedMin, selectedMax)">
      <el-option
        v-for="item in data.filter((d) => {
          (d.date.getTime() < (new Date(moment(selectedMax).add(-6, 'months'))).getTime())
        })"
        :key="String(item.date)"
        :label="moment(item.date).format('MMMM YYYY')"
        :value="String(item.date)"/>
    </el-select>
    До:
    <el-select
      v-model="selectedMax"
      size="small"
      popper-class="flex"
      placeholder="Выберите дату"
      @change="drawChart(selectedMin, selectedMax)">
      <el-option
        v-for="item in data.filter((d) => {
          return (d.date.getTime() > (new Date(moment(selectedMin).add(6, 'months'))).getTime())
        })"
        :key="String(item.date)"
        :label="moment(item.date).format('MMMM YYYY')"
        :value="String(item.date)"/>
    </el-select>
    <div
      ref="chart"
      class="hello"/>
  </div>
</template>

<script>
import * as d3 from 'd3';
import moment from 'moment';

export default {
  name: 'Chart',
  data() {
    return {
      data: [],
      selectedMin: String(new Date(2014, 0, 1)),
      selectedMax: String(new Date(2018, 0, 1)),
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
    };
  },
  created() {
    // получаем данные
    this.fetch();
  },
  methods: {
    moment,
    fetch() {
      d3.dsv(
        ';',
        'https://raw.githubusercontent.com/Radiophysiker/d3-mts/master/data/d3_sample1.csv',
        d => ({
          date: d.xdate, // конвектируем столбцы под наше названия
          value: d.value,
        }),
      )
        .then((data) => {
          const parseDate = d3.timeParse('%d.%m.%Y');
          this.data = data.filter(elem => elem.value); // убираем пустые значения
          this.data.forEach((d) => {
            d.date = parseDate(d.date); // парсим дату
            d.value = parseFloat(d.value);
          });
          // находим максимумы и мин
          this.date.min = d3.min(this.data, d => d.date);
          this.selectedMin = String(this.date.min);
          this.date.max = d3.max(this.data, d => d.date);
          this.selectedMax = String(this.date.max);
          this.drawChart(); // после получения данных рисуем график
        })
        .catch((e) => {
          throw e;
        });
    },
    drawChart(selectedMin = this.date.min, selectedMax = this.date.max) {
      const currentMaxDate = new Date(selectedMax);
      const currentMinDate = new Date(selectedMin);
      const localData = this.data.filter(elem => (elem.date.getTime() >= currentMinDate.getTime())
        && (elem.date.getTime() <= currentMaxDate.getTime()));
      const localMax = d3.max(localData, d => d.value);
      const localMin = d3.min(localData, d => d.value);
      d3.select(this.$refs.chart)
        .select('svg')
        .remove();
      const width =
        this.chartSize.width -
        this.chartSize.margin.left -
        this.chartSize.margin.right;
      const height =
        this.chartSize.height -
        this.chartSize.margin.top -
        this.chartSize.margin.bottom;
      const x = d3
        .scaleTime()
        .domain([currentMinDate, currentMaxDate])
        .rangeRound([0, width]);
      const y = d3.scaleLinear().range([height, 0]).nice();

      // устанавливаем параметры для гистограммы
      const histogram = d3
        .histogram()
        .value(d => d.date)
        .domain(x.domain())
        .thresholds(x.ticks(d3.timeMonth));

      const svg = d3
        .select(this.$refs.chart)
        .append('svg')
        .attr('width', this.chartSize.width)
        .attr('height', this.chartSize.height)
        .append('g')
        .attr(
          'transform',
          `translate(${this.chartSize.margin.left},${
            this.chartSize.margin.top
          })`,
        );
      const bins = histogram(localData);
      const colorScale = d3
        .scaleLinear()
        .domain([localMin, localMax])
        .range([d3.rgb(this.color[0]), d3.rgb(this.color[1])]);
      y.domain([0, localMax]);
      // добавляем гистограммы
      svg
        .selectAll('rect')
        .data(bins)
        .enter()
        .append('rect')
        .attr('class', 'bar')
        .attr('x', 1)
        .attr(
          'transform',
          d =>
            `translate(${x(d.x0)}, ${height * (-(d[0].value / localMax) + 1)})`,
        )
        .attr(
          'width',
          d => (x(d.x1) - x(d.x0) - 1 < 0 ? 0 : x(d.x1) - x(d.x0) - 1),
        )
        .attr('height', d => (height * d[0].value) / localMax)
        .attr('fill', d => colorScale(d[0].value));

      // добавлена оси X
      svg
        .append('g')
        .attr('transform', `translate(0,${height})`)
        .call(d3.axisBottom(x));

      //  добавлена оси Y
      svg.append('g').call(d3.axisLeft(y));
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->

<style>
.el-select {
  margin-left: 10px;
  margin-right: 10px;
}
.flex {
  display: flex;
  flex-direction: column;
}
.el-input__inner {
  width: 150px;
}
</style>

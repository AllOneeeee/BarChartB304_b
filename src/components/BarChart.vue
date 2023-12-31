<template>
  <div>
    <div ref="chart"></div>
    <button @click="sortData">Sort Data</button>
  </div>
</template>

<script>
import * as d3 from 'd3';
import 'd3-fetch';

export default {
  data() {
    return {
      isSorted: false,
      data: null,
      xScale: null,
      svg: null,
    };
  },
  mounted() {
    this.drawChart();
  },
  methods: {
    async drawChart() {
      // URL Gist GitHub
      const gistUrl = 'https://gist.githubusercontent.com/AllOneeeee/ded5fc8e4a8499352edea658a69bdeb3/raw/159afbce81fa5d9cf2b845e9153006fda54f9077/data_tk.csv';

      // Baca data dari URL Gist GitHub
      this.data = await d3.csv(gistUrl);

      // Konfigurasi skala
      this.xScale = d3.scaleBand().domain(this.data.map(d => d['Tingkat Pendidikan'])).range([0, 500]).padding(0.1);
      const yScale = d3.scaleLinear().domain([0, d3.max(this.data, d => d.frekuensi)]).range([200, 0]);

      // Membuat SVG dan elemen-elemen grafik
      this.svg = d3.select(this.$refs.chart)
        .append('svg')
        .attr('width', 600)
        .attr('height', 380)
        .style('border', '1px solid black')  // Border 
        .append('g')
        .attr('transform', 'translate(50, 50)'); // Margin 

      // Menambahkan garis axis x
      this.svg.append('g')
        .attr('transform', `translate(0, ${200})`) // Memindahkan axis ke bawah
        .attr('class', 'x-axis') // Add class for updating later
        .call(d3.axisBottom(this.xScale))
        .selectAll('text')
        .style('text-anchor', 'end') 
        .attr('transform', 'rotate(-45)') 
        .style('font-family', 'Poppins')
        .style('font-size', '14px');
        
      // Menambahkan garis axis y
      const yAxis = this.svg.append('g')
        .call(d3.axisLeft(yScale).ticks(5))
        .style('font-family', 'Poppins')
        .style('font-size', '14px');

      // Menambahkan elemen(bars)
      this.svg.selectAll('rect')
        .data(this.data)
        .enter()
        .append('rect')
        .style("mix-blend-mode", "multiply")
        .attr('x', d => this.xScale(d['Tingkat Pendidikan']))
        .attr('y', d => yScale(d.frekuensi))
        .attr('width', this.xScale.bandwidth())
        .attr('height', d => 200 - yScale(d.frekuensi))
        .attr('fill', '#FFC837')
        .style('font-family', 'Poppins')
        .attr('stroke', 'black') // Border warna
        .attr('stroke-width', 1)  // Border width
        .on('mouseover', (event, d) => {
          d3.select(event.currentTarget)
            .attr('fill', '#FBAC1B'); // Ganti warna bar saat dihover

          this.svg.append('text')
            .attr('class', 'label')
            .attr('x', this.xScale(d['Tingkat Pendidikan']) + this.xScale.bandwidth() / 2)
            .attr('y', yScale(d.frekuensi) - 5)
            .attr('text-anchor', 'middle')
            .text(d.frekuensi)
            .style('font-family', 'Poppins')
            .style('font-size', '18px')
            .style('font-weight', 'bold');
        })
        .on('mouseout', (event, d) => {
          // Menghilangkan label pada mouseout
          d3.select(event.currentTarget)
            .attr('fill', '#FFC837'); // Kembalikan warna bar ke warna asli
          
          this.svg.selectAll('.label').remove();
        });

      // Menambahkan label axis x
      this.svg.append('text')
        .attr('x', 250) // Posisi label pada pertengahan axis x
        .attr('y', 275) // Posisi label di bawah axis x
        .style('text-anchor', 'middle')
        .text('Tingkat Pendidikan')
        .style('font-family', 'Poppins');

      this.sortData();
    },
    sortData() {
      this.isSorted = !this.isSorted;

      // Sort data 
      const sortedData = this.isSorted
        ? [...this.data].sort((a, b) => d3.descending(+a.frekuensi, +b.frekuensi))
        : this.data;

      // Update xScale domain 
      this.xScale.domain(sortedData.map(d => d['Tingkat Pendidikan']));

      // Update bars 
      this.svg.selectAll('rect')
        .data(sortedData, d => d['Tingkat Pendidikan'])
        .transition()
        .duration(500)
        .attr('x', d => this.xScale(d['Tingkat Pendidikan']));

      // Update x-axis 
      this.svg.select('.x-axis')
        .transition()
        .duration(500)
        .call(d3.axisBottom(this.xScale));
    },
  },
};
</script>

<style scoped>

</style>

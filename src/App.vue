<template>
  <div id="app">
    <virtual-list :list="tableData" :total="total" @fetch="fetchData">
      <el-table-column prop="$index" label="序号" width="180"></el-table-column>
      <el-table-column prop="name" label="名字" width="180"></el-table-column>
      <el-table-column prop="name" label="名字" width="180"></el-table-column>
      <el-table-column prop="name" label="名字" width="180"></el-table-column>
      <el-table-column prop="name" label="名字" width="180"></el-table-column>
      <el-table-column prop="name" label="名字" width="180"></el-table-column>
      <el-table-column prop="name" label="名字" width="180"></el-table-column>
      <el-table-column prop="name" label="名字" width="180"></el-table-column>
    </virtual-list>
    <div>总数量：{{ total }}</div>
  </div>
</template>

<script>
import VirtualList from './components/virtual-list'
export default {
  name: 'App',
  components: {
    VirtualList
  },
  data() {
    return {
      tableData: [],
      total: 0
    }
  },
  created() {
    this.fetchData()
  },
  methods: {
    fetchData() {
      const newArr = []
      const total = this.total || 0
      for (let i = 1; i <= 20; i++) {
        newArr.push({ name: `测试用户${this.total + i}` })
      }
      let timer = window.setTimeout(() => {
        this.total = this.tableData.length + 20
        this.tableData = [...this.tableData, ...newArr]
        console.log('0.0')
        clearTimeout(timer)
      }, 120)
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
  height: 800px;
}
</style>

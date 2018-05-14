<template>
  <div id="app">
    <el-row>
      <el-col :span="24">
        <el-table
          :data="analysisResultList">
          <el-table-column label="" width="30">
            <template slot-scope="scope">
              <!-- `checked` 为 true 或 false -->
              <el-checkbox v-model="checked"></el-checkbox>
            </template>
          </el-table-column>
          <el-table-column
            align="center"
            prop="analysisResultId"
            label="ID"
            width="180">
          </el-table-column>

          <el-table-column
            align="center"
            prop="analysisTime"
            label="Analysis Time"
            width="300">
          </el-table-column>
          <el-table-column
            align="center"
            prop="symbol"
            label="Symbol"
            width="180">
          </el-table-column>
          <el-table-column
            align="center"
            prop="sHour"
            label="Hour"
            width="180"
            sortable="true">
          </el-table-column>
          <el-table-column
            align="center"
            prop="t"
            label="T"
            width="180">
          </el-table-column>
          <el-table-column
            align="center"
            prop="income"
            label="Income"
            width="180">
          </el-table-column>
          <el-table-column
            align="center"
            prop="scale"
            label="Scale"
            width="180">
          </el-table-column>
        </el-table>
      </el-col>
    </el-row>
    <el-row style="margin-top: 10px">
      <el-col :span="24">
        <el-button @click="submit()">提交</el-button>
      </el-col>
    </el-row>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        analysisResultList:[]
      }
    }, created() {
      this.load();
    }, mounted() {

    },
    methods: {
      async load() {

        const response = await fetch('/api/analysis/list', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({})
        });
        const bodyText = await response.text();
        this.analysisResultList = JSON.parse(bodyText);
      },
      submit() {
        this.$alert('这是一段内容', '标题名称', {
          confirmButtonText: '确定',
        });
      }
    }
  }
</script>

<style>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
</style>

<template>
  <div id="app">
    <el-row>
      <el-col :span="24">
        <el-table
          :data="analysisResultList"
          :row-class-name="tableRowClassName">
          <el-table-column label="" width="180">
            <template slot-scope="scope">
              <!-- `checked` 为 true 或 false -->
              <el-button @click="showOrder(scope.row)" v-show="scope.row.isPendingOrder == false" type="primary">下单
              </el-button>


              <el-tooltip class="item" effect="light" placement="right">
                <div slot="content">数量: {{scope.row.amount}}<br/>eth: {{scope.row.eth}}</div>
                <el-button @click="cancelOrder(scope.row.orderId)" v-show="scope.row.isPendingOrder == true"
                           type="danger" >撤单
                </el-button>
              </el-tooltip>

              <a :href="scope.row.exchangeUrl" style="margin-left: 4px;" target="_blank">行情</a>
            </template>
          </el-table-column>
          <el-table-column
            align="center"
            prop="analysisResultId"
            label="ID"
            width="120">
          </el-table-column>

          <el-table-column
            align="center"
            prop="analysisTime"
            label="Analysis Time"
            width="230">
          </el-table-column>
          <el-table-column
            align="center"
            prop="symbol"
            label="Symbol"
            width="160">
          </el-table-column>
          <el-table-column
            align="center"
            prop="sHour"
            label="Hour"
            width="120"
            sortable="true">
          </el-table-column>
          <el-table-column
            align="center"
            prop="t"
            label="T"
            width="120">
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
          <el-table-column
            align="center"
            prop="price"
            label="Price"
            sortable
            width="120">
          </el-table-column>
          <el-table-column
            align="center"
            prop="vol"
            label="Vol"
            sortable
            width="120">
          </el-table-column>
          <el-table-column
            align="center"
            prop="amount"
            label="Amount"
            sortable
            width="120">
          </el-table-column>
          <el-table-column
            align="center"
            prop="days"
            label="Days"
            sortable
            width="120">
          </el-table-column>
        </el-table>
      </el-col>
    </el-row>

    <el-dialog title="订单信息" :visible.sync="dialogFormVisible" width="30%">
      <el-form :model="formOrder">

        <el-form-item label="Symbol:">
          <el-input v-model="formOrder.symbol" :disabled="true"></el-input>
        </el-form-item>
        <el-form-item label="T:">
          <el-input v-model="formOrder.t" :disabled="true"></el-input>
        </el-form-item>
        <el-form-item label="Buy Time:">
          <el-input v-model="formOrder.buyTimeString" :disabled="true"></el-input>
        </el-form-item>
        <el-form-item label="Price:">
          <el-input v-model="formOrder.price" :disabled="true"></el-input>
        </el-form-item>
        <el-form-item label="Amount:">
          <el-input v-model="formOrder.amount"></el-input>
        </el-form-item>
        <el-form-item label="ETH:">
          {{this.eth}}
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveOrder()">确 定</el-button>
      </div>
    </el-dialog>

  </div>
</template>

<script>
  export default {
    data() {
      return {
        analysisResultList: [],
        pendingOrderList: [],
        dialogFormVisible: false,
        formOrder: {}
      }
    }, created() {
      this.loadData();
    }, mounted() {

    },
    methods: {
      tableRowClassName({row, rowIndex}) {
        console.log(row);
        if (row.days < 2000) {
          return 'warning-row';
        }
        return '';
      },
      async loadAnalysisResultList() {
        const response = await fetch('/api/analysis/list', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({})
        });
        const bodyText = await response.text();
        this.analysisResultList = JSON.parse(bodyText);


      }, async loadPendingOrderList() {
        const response = await fetch('/api/order/pending/list', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({})
        });
        const bodyText = await response.text();
        this.pendingOrderList = JSON.parse(bodyText);
      }, async loadData() {
        await this.loadAnalysisResultList();
        await this.loadPendingOrderList();

        this.analysisResultList.forEach((analysisResult) => {
          analysisResult.isPendingOrder = false;
          this.pendingOrderList.forEach((pendingOrder) => {
            if (analysisResult.symbol == pendingOrder.symbol) {
              analysisResult.isPendingOrder = true;
              analysisResult.orderId = pendingOrder.orderItemId;
              // 订单数量
              analysisResult.amount = pendingOrder.amount;
              // eth 数量
              analysisResult.eth = analysisResult.price * analysisResult.amount;
            }
          });
        });
        let json = JSON.stringify(this.analysisResultList);
        this.analysisResultList = JSON.parse(json);

      }, showOrder(analysisResult) {
        let nowXDate = new XDate();
        let year = nowXDate.getFullYear();
        let month = nowXDate.getMonth();
        let date = nowXDate.getDate();
        let hour = analysisResult.sHour;

        let buyTimeXDate = new XDate(year, month, date, hour, 0, 0, 0);
        if (buyTimeXDate.valueOf() < nowXDate.valueOf()) {
          buyTimeXDate = buyTimeXDate.addDays(1);
        }

        this.formOrder = {
          symbol: analysisResult.symbol,
          t: analysisResult.t,
          buyTime: buyTimeXDate.toDate(),
          buyTimeString: buyTimeXDate.toString("yyyy-MM-dd HH:mm:ss"),
          price: analysisResult.price
        };
        this.dialogFormVisible = true;
      }, async saveOrder() {
        const response = await fetch('/api/order/add', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(this.formOrder)
        });
        const bodyText = await response.text();

        // 刷新列表
        this.loadData();

        this.formOrder = {};
        this.dialogFormVisible = false;
      }, async cancelOrder(orderId) {

        const response = await fetch('/api/order/delete', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({"orderItemId": orderId})
        });
        const bodyText = await response.text();

        // 刷新列表
        this.loadData();


      }
    },
    computed: {
      // 计算属性的 getter
      eth: function () {
        // `this` 指向 vm 实例
        if (this.formOrder.price == null) {
          return 0;
        }

        let s = this.formOrder.price * this.formOrder.amount;
        return s;
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

  .el-table .warning-row {
    background: #FF8C00;
  }
</style>

<template>
  <div>
    <div class="container">
      <h1 class="title">記帳 631 理財小幫手</h1>
      <div class="row account">
        <div class="col-md-3">
          <div class="item salary">
            <p class="item-title">當月薪資</p>
            <p class="item-money">{{salary | commaFormat | priceFormat}}</p>
          </div>
        </div>
        <div class="col-md-3">
          <div
            class="item expend"
            :class="{'coloring': livingExpenseDegree > 0}"
          >
            <p class="item-title">生活支出</p>
            <p
              class="item-money"
              :class="{'not-zero': livingExpense > 0}"
            >
              {{ livingExpense | commaFormat | priceFormat}}
            </p>
            <div
              v-if="livingExpenseDegree < 100"
              class="wave"
              :style="{'bottom': livingExpenseDegree + '%'}"
            ></div>
          </div>
        </div>
        <div class="col-md-3">
          <div
            class="item expend"
            :class="{'coloring': investDegree > 0}"
          >
            <p class="item-title">投資理財</p>
            <p
              class="item-money"
              :class="{'not-zero': invest > 0}"
            >
              {{ invest | commaFormat | priceFormat}}
            </p>
            <div
              v-if="investDegree < 100"
              class="wave"
              :style="{'bottom': investDegree + '%'}"
            ></div>
          </div>
        </div>
        <div class="col-md-3">
          <div
            class="item expend"
            :class="{'coloring': savingsDegree > 0}"
          >
            <p class="item-title">儲蓄</p>
            <p
              class="item-money"
              :class="{'not-zero': savings > 0}"
            >
              {{ savings | commaFormat | priceFormat}}
            </p>
            <div
              v-if="savingsDegree < 100"
              class="wave"
              :style="{'bottom': savingsDegree + '%'}"
            ></div>
          </div>
        </div>
      </div>
      <div class="input-info">
        <div class="info-contain">
          <div class="row info-contain-left">
            <div class="col-3 d-flex mb-20">
              <div class="info-item col-form-label">當月薪資</div>
              <input
                type="text"
                class="info-input form-control"
                v-model="salary"
                @keyup="onlyNum($event, 'salary')"
              >
            </div>
            <div class="col-3 d-flex mb-20 align-middle">
              <div class="info-item col-form-label">紀錄日期</div>
              <input
                type="text"
                id="datepicker"
                class="info-input form-control"
                v-model="date"
              >
            </div>
            <div class="col-3 d-flex mb-20">
              <div class="info-item col-form-label">項目金額</div>
              <input
                type="text"
                class="info-input form-control"
                v-model="cash"
                @keyup="onlyNum($event, 'cash')"
              >
            </div>
            <div class="col-3 d-flex mb-20">
              <div class="info-item col-form-label">項目類別</div>
              <select
                class="info-input form-control"
                v-model="category"
              >
                <option value="01">生活支出</option>
                <option value="02">投資理財</option>
                <option value="03">儲蓄</option>
              </select>
            </div>
            <div class="col-12 d-flex">
              <div class="info-item description col-form-label">項目描述</div>
              <input
                type="text"
                class="info-input form-control"
                v-model="disc"
              >
            </div>
          </div>
          <div
            class="info-btn"
            @click="create"
          >
            <p class="create">新增</p>
          </div>
        </div>
      </div>
      <div class="account-list">
        <table class="table table-striped">
          <thead class="thead-header">
            <tr>
              <th scope="col" class="table-item">紀錄日期</th>
              <th scope="col" class="table-item">消類金額</th>
              <th scope="col" class="table-item">項目類別</th>
              <th scope="col" class="table-desc">項目描述</th>
              <th scope="col" class="table-edit"></th>
              <th scope="col" class="table-delete"></th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in list" :key="index">
              <td>{{item.date}}</td>
              <td>{{item.cash}}</td>
              <td>{{item.category}}</td>
              <td class="text-left">{{item.disc}}</td>
              <td
                class="edit-icon"
                @click="editItem(item, index)"
              >
                <i class="bi bi-pencil-square"></i>
              </td>
              <td
                class="edit-icon"
                @click="deleteItem(item)"
              >
                <i class="bi bi-trash"></i>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- EditModal -->
    <div
      class="modal fade"
      id="editItem"
      tabindex="-1"
      aria-labelledby="editItem"
      aria-hidden="true"
    >
      <EditDialog
        :editData="editSelected"
        @editedData="editedData"
      />
    </div>

    <!-- DeleteModal -->
    <div
      class="modal fade"
      id="deleteItem"
      tabindex="-1"
      aria-labelledby="deleteItem"
      aria-hidden="true"
    >
      <DeleteDialog
        :deleteData="deleteSelected"
        @deletedItem="deletedItem"
      />
    </div>
  </div>
</template>

<script>
import $ from 'jquery';
import 'jquery-ui-dist/jquery-ui';
import 'jquery-ui-dist/jquery-ui.min.css';

import moment from 'moment';
import DeleteDialog from './components/DeleteDialog.vue';
import EditDialog from './components/EditDialog.vue';

export default {
  components: {
    EditDialog,
    DeleteDialog
  },
  data(){
    return {
      salary                 : 0,
      livingExpense          : 0,
      invest                 : 0,
      savings                : 0,
      list                   : [],
      date                   : '',
      cash                   : 0,
      category               : '',
      disc                   : '',
      editSelected           : null,
      deleteSelected         : null,
      openEditDialog         : false,
      openDeleteDialog       : false,
    }
  },
  metaInfo: {
    title: '631 記帳財務管理',
    lang: 'zh-Hant'
  },
  filters: {
    priceFormat (val) {
      return '$' + val
    },
    commaFormat (val) {
      if (val && val > 0) {
        return val.toString().replace(/^(-?\d+?)((?:\d{3})+)(?=\.\d+$|$)/, function(all, pre, groupOf3Digital){
          return pre + groupOf3Digital.replace(/\d{3}/g, ',$&');
        })
      } else {
        return 0
      }
    }
  },
  computed: {
    livingExpenseDegree () {
      if (this.livingExpense > 0) {
        return (this.livingExpense / (this.salary * 0.6)) * 100
      } else {
        return -4
      }
    },
    investDegree () {
      if (this.invest > 0) {
        return (this.invest / (this.salary * 0.3)) * 100
      } else {
        return -4
      }
    },
    savingsDegree () {
      if (this.savings > 0) {
        return (this.savings / (this.salary * 0.1)) * 100
      } else {
        return -4
      }
    }
  },
  mounted () {
    $( "#datepicker" ).datepicker(
      {
        dateFormat: 'yy/mm/dd',
        onSelect: this.setDateValue
      }
    );
    
    const ORG_LIST = localStorage.getItem('DATA_LIST')

    if (ORG_LIST) {
      const dataList = JSON.parse(ORG_LIST)
      this.list = dataList
      this.salary = localStorage.getItem('DATA_SALARY')
      this.livingExpense = localStorage.getItem('LIVING_EXP')
      this.invest = localStorage.getItem('INVEST')
      this.savings = localStorage.getItem('SAVINGS')
    }
  },
  methods: {
    setDateValue (val) {
      this.date = val
    },
    create () {
      // 取得 datepicker 資訊 - https://www.codegrepper.com/code-examples/javascript/jquery+datepicker+get+selected+date
      this.date = $('#datepicker').datepicker('getDate')

      const salary = this.salary
      const date = moment(this.date).format('YYYY/MM/DD')
      const cash = this.cash
      const category = this.category
      const disc = this.disc.trim()

      this.list.push({
        index: Math.random(),
        date,
        cash,
        category : this.categorySwitch(category),
        categoryNo: category,
        disc
      })
      this.date = '';
      this.cash = 0;
      this.category = '';
      this.disc = '';

      this.calculate()
      localStorage.setItem('DATA_SALARY', salary)
      localStorage.setItem('DATA_LIST', JSON.stringify(this.list))
    },
    categorySwitch (val) {
      switch (val) {
        case '01':
          return '生活支出';
        case '02':
          return '投資理財';
        case '03':
          return '儲蓄';
      }
    },
    calculate () {
      this.reset();

      // 對 list 分堆並計算各分類金額
      console.log('this.list', this.list)
      const groupByCategoryNo = this.groupArrayData(this.list, 'categoryNo')

      Object.entries(groupByCategoryNo).forEach((item) => {
        switch (item[0]) {
          case '01':
            this.livingExpense = item[1].reduce((acc, current) => Number(acc) + Number(current.cash), 0)
            localStorage.setItem('LIVING_EXP', this.livingExpense)
            break;
          case '02':
            this.invest = item[1].reduce((acc, current) => Number(acc) + Number(current.cash), 0)
            localStorage.setItem('INVEST', this.invest)
            break;
          case '03':
            this.savings = item[1].reduce((acc, current) => Number(acc) + Number(current.cash), 0)
            localStorage.setItem('SAVINGS', this.savings)
            break;
        }
      })
    },
    reset() {
      this.livingExpense=0;
      this.invest=0;
      this.savings=0;
      localStorage.setItem('LIVING_EXP', this.livingExpense);
      localStorage.setItem('INVEST', this.invest);
      localStorage.setItem('SAVINGS', this.savings);
    },
    groupArrayData (list, key) {
      return list.reduce((acc, current) => {
        (acc[current[key]] = acc[current[key]] || []).push(current)
        return acc
      }, {})
    },
    editItem (item, index) {
      this.editSelected = {
        index,
        ...item
      }
      this.openEditDialog = true
      $('#editItem').modal('show')
    },
    editedData ($event) {
      const findIndex = this.list.findIndex(item => item.index === $event.index)
      this.list[findIndex] = $event
      this.calculate()
      localStorage.setItem('DATA_LIST', JSON.stringify(this.list))
      $('#editItem').modal('hide')
    },
    deleteItem (item) {
      this.deleteSelected = item
      this.openDeleteDialog = true
      $('#deleteItem').modal('show')
    },
    deletedItem (list) {
      const newList = this.list.findIndex((item) => {
        return list.index === item.index
      })
      this.list.splice(newList, 1)

      this.calculate()
      localStorage.setItem('DATA_LIST', JSON.stringify(this.list))

      $('#deleteItem').modal('hide')
    },
    onlyNum(val, key) {
      let keyInVal = val.target.value
      switch(key) {
        case 'salary':
          this.salary = keyInVal.toString().replace(/[^0-9.]/g, '')
          break;
        case 'cash':
          this.cash = keyInVal.toString().replace(/[^0-9.]/g, '')
          break;
      }
      
    }
  }
}
</script>

<style lang="scss" scoped>
@import '@/assets/styles/main.scss';

.container {
  padding-top: 50px;
  padding-bottom: 50px;
}
.title {
  @include font-setting;
  font-size: 40px;
  font-weight: 600;
  color: #2A9D8F;
  text-align: center;
  margin-bottom: 50px;
}
.account {
  margin-bottom: 50px;
  .item {
    padding: 20px;
    background-color: white;
    border-radius: 10px;
    height: 245px;
    position: relative;
    border-radius: 50%;
    overflow: hidden;
    border: 6px solid #F4A261;
    z-index: 100;
    &.expend {
      // background-color: white;
      .item-title {
        text-align: center;
        font-size: 28px;
        color: #707070;
      }
      .item-money {
        font-size: 48px;
        font-weight: 600;
        color: #DDDBCB;
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        &.not-zero {
          color: #F4A261;
        }
      }
    }
    .item-title {
      text-align: center;
      font-size: 28px;
      color: white;
    }
    .item-money {
      font-size: 48px;
      font-weight: 600;
      color: white;
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
    }
  }
  .salary {
    background-color: #F4A261;
  }
  .coloring {
    background-color: #F5F1E3;
  }
  // 水波紋效果 https://blog.csdn.net/ak852369/article/details/100705102
  .wave {
    position: absolute;
    left: 50%;
    width: 480px;
    height: 500px;
    background-color: #fff;
    border-radius: 40%;
    animation-duration: 6s;
    animation-name: rotate;
    animation-iteration-count: infinite;
    animation-timing-function: linear;
    z-index: -1;
  }
}
@keyframes rotate {
  0% {
    transform: translate(-50%, 0) rotateZ(0deg);
  }
  50% {
    transform: translate(-50%, -1%) rotateZ(180deg);
  }
  100% {
    transform: translate(-50%, 0) rotateZ(360deg);
  }
}
.input-info {
  padding: 20px 20px;
  border: 2px solid #2A9D8F;
  border-radius: 10px;
  margin-bottom: 50px;
  .info-contain {
    display: flex;
    justify-content: space-between;
    align-items: stretch;
    .info-contain-left {
      flex: 1;
      margin-right: 15px;
      .info-item {
        margin-right: 12px;
        font-size: 18px;
        color: #7D7D7D;
        display: inline-block;
        width: 132px;
        &.description {
          width: 80px;
        }
      }
      .info-input {
        border: 1px solid #2A9D8F;
        border-radius: 5;
      }
    }
    .info-btn {
      background-color: #2A9D8F;
      border-radius: 5px;
      color: white;
      width: 100px;
      position: relative;
      cursor: pointer;
      .create {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
      }
    }
  }
}
.account-list {
    border: 1px solid #264653;
  .table {
    margin-bottom: 0;
    border: none;
  }
  .thead-header {
    background-color: #264653;
    color: white;
    font-size: 20px;
    text-align: center;
    padding: 16px 0;
  }
  th {
    border-top: 0;
  }
  td {
    border: none;
    text-align: center;
  }
  .table-striped tbody tr:nth-of-type(odd) {
    background-color: #EDF6F9;
  }
  .table-item {
    width: 12%;
  }
  .table-edit,
  .table-delete {
    width: 5%;
  }
  .edit-icon {
    cursor: pointer;
  }
}
</style>
<template>
  <div>
    <div class="container">
      <h1 class="title">記帳 631 理財小幫手</h1>
      <div class="row account">
        <div class="col-md-3">
          <div class="item salary">
            <p class="item-title">當月薪水</p>
            <p class="item-money">${{total | commaFormat}}</p>
          </div>
        </div>
        <div class="col-md-3">
          <div class="item expend" :class="{'coloring': livingExpenseDegree > 0}">
            <p class="item-title">生活支出</p>
            <p class="item-money" :class="{'not-zero': livingExpense > 0}">${{ livingExpense | commaFormat }}</p>
            <div v-if="livingExpenseDegree < 100" class="wave" :style="{'bottom': livingExpenseDegree + '%'}"></div>
          </div>
        </div>
        <div class="col-md-3">
          <div class="item expend" :class="{'coloring': investDegree > 0}">
            <p class="item-title">投資</p>
            <p class="item-money" :class="{'not-zero': invest > 0}">${{ invest | commaFormat }}</p>
            <div v-if="investDegree < 100" class="wave" :style="{'bottom': investDegree + '%'}"></div>
          </div>
        </div>
        <div class="col-md-3">
          <div class="item expend" :class="{'coloring': savingsDegree > 0}">
            <p class="item-title">儲蓄</p>
            <p class="item-money" :class="{'not-zero': savings > 0}">${{ savings | commaFormat }}</p>
            <div v-if="savingsDegree < 100" class="wave" :style="{'bottom': savingsDegree + '%'}"></div>
          </div>
        </div>
      </div>
      <div class="input-info">
        <div class="info-contain">
          <div class="row info-contain-left">
            <div class="col-4 d-flex mb-20 align-middle">
              <div  class="info-item col-form-label">紀錄日期</div>
              <input type="text" id="datepicker" class="info-input form-control" v-model="date">
            </div>
            <div class="col-4 d-flex mb-20">
              <div class="info-item col-form-label">項目金額</div>
              <input type="text" class="info-input form-control" v-model="cash">
            </div>
            <div class="col-4 d-flex mb-20">
              <div class="info-item col-form-label">項目類別</div>
              <select class="info-input form-control" v-model="category">
                <option value="01">生活支出</option>
                <option value="02">理財</option>
                <option value="03">儲蓄</option>
              </select>
            </div>
            <div class="col-12 d-flex">
              <div class="info-item description col-form-label">項目描述</div>
              <input type="text" class="info-input form-control" v-model="disc">
            </div>
          </div>
          <div class="info-btn" @click="create">
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
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in list" :key="index">
              <td>{{item.date}}</td>
              <td>{{item.cash}}</td>
              <td>{{item.category}}</td>
              <td class="text-left">{{item.disc}}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import $ from 'jquery';
import 'jquery-ui-dist/jquery-ui';
import 'jquery-ui-dist/jquery-ui.min.css';

import moment from 'moment';

export default {
  data(){
    return {
      total                  : 40000,
      livingExpense          : 0,
      invest                 : 0,
      savings                : 0,
      list                   : [],
      date                   : '',
      cash                   : 0,
      category               : '',
      disc                   : '',
    }
  },
  metaInfo: {
    title: '631 記帳財務管理',
    lang: 'zh-Hant'
  },
  filters: {
    commaFormat (val) {
      return val.toString().replace(/^(-?\d+?)((?:\d{3})+)(?=\.\d+$|$)/, function(all, pre, groupOf3Digital){
        return pre + groupOf3Digital.replace(/\d{3}/g, ',$&');
      })
    }
  },
  computed: {
    livingExpenseDegree () {
      if (this.livingExpense > 0) {
        return (this.livingExpense / (this.total * 0.6)) * 100
      } else {
        return -4
      }
    },
    investDegree () {
      if (this.invest > 0) {
        return (this.invest / (this.total * 0.3)) * 100
      } else {
        return -4
      }
    },
    savingsDegree () {
      if (this.savings > 0) {
        return (this.savings / (this.total * 0.1)) * 100
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
  },
  methods: {
    setDateValue (val) {
      this.date = val
    },
    create () {
      // 取得 datepicker 資訊 - https://www.codegrepper.com/code-examples/javascript/jquery+datepicker+get+selected+date
      this.date = $('#datepicker').datepicker('getDate')

      const date = moment(this.date).format('YYYY/MM/DD')
      const cash = this.cash
      const category = this.category
      const disc = this.disc.trim()

      this.list.push({
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
    },
    categorySwitch (val) {
      switch (val) {
        case '01':
          return '生活支出';
        case '02':
          return '投資';
        case '03':
          return '儲蓄';
      }
    },
    calculate () {
      // 對 list 分堆並計算各分類金額
      const groupByCategoryNo = this.groupArrayData(this.list, 'categoryNo')
      console.log(Object.entries(groupByCategoryNo))

      Object.entries(groupByCategoryNo).forEach((item) => {
        switch (item[0]) {
          case '01':
            this.livingExpense = item[1].reduce((acc, current) => Number(acc) + Number(current.cash), 0)
            break;
          case '02':
            this.invest = item[1].reduce((acc, current) => Number(acc) + Number(current.cash), 0)
            break;
          case '03':
            this.savings = item[1].reduce((acc, current) => Number(acc) + Number(current.cash), 0)
            break;
        }
      })
    },
    groupArrayData (list, key) {
      return list.reduce((acc, current) => {
        (acc[current[key]] = acc[current[key]] || []).push(current)
        return acc
      }, {})
    }
  }
}
</script>

<style lang="scss" scoped>
@import './assets/styles/main.scss';

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
        margin-right: 26px;
        font-size: 18px;
        color: #7D7D7D;
        display: inline-block;
        width: 120px;
        &.description {
          width: 84px;
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
    width: 15%;
  }
}
</style>
<template>
  <div>
    <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="exampleModalLabel">編輯</h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <div class="modal-body">
            <form>
              <div class="form-group">
                <label for="innerDatepicker">紀錄日期</label>
                <input
                  type="text"
                  id="innerDatepicker"
                  class="form-control"
                  v-model="Data.date"
                >
              </div>
              <div class="form-group">
                <label>消費金額</label>
                <input
                  type="text"
                  class="form-control"
                  v-model="Data.cash"
                >
              </div>
              <div class="form-group">
                <label>項目類別</label>
                <select
                  class="form-control"
                  v-model="Data.categoryNo"
                >
                  <option value="01">生活支出</option>
                  <option value="02">投資理財</option>
                  <option value="03">儲蓄</option>
                </select>
              </div>
              <div class="form-group">
                <label>項目描述</label>
                <input
                  type="text"
                  class="form-control"
                  v-model="Data.disc"
                >
              </div>
            </form>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-dismiss="modal">取消</button>
            <button type="button" class="btn btn-primary" @click="save()">儲存</button>
          </div>
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
  props: {
    editData : {
      type    : Object,
      default : null
    }
  },
  data() {
    return {
      Data: {
        index       : null,
        date        : '',
        cash        : '',
        category    : '',
        categoryNo  : '',
        disc        : ''
      }
    }
  },
  watch: {
    editData () {
      this.Data.index = this.editData.index
      this.Data.date = this.editData.date
      this.Data.cash = this.editData.cash
      this.Data.category = this.editData.category
      this.Data.categoryNo = this.editData.categoryNo
      this.Data.disc = this.editData.disc
    }
  },
  mounted() {
    $("#innerDatepicker").datepicker(
      {
        dateFormat: 'yy/mm/dd',
        onSelect: this.setDateValue
      }
    );
  },
  methods: {
    setDateValue (val) {
      this.Data.date = val
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
    save () {
      const index = this.Data.index
      const date = moment(this.Data.date).format('YYYY/MM/DD')
      const cash = this.Data.cash
      const categoryNo = this.Data.categoryNo
      const disc = this.Data.disc.trim()
      const editedData = {
        index,
        date,
        cash,
        category : this.categorySwitch(categoryNo),
        categoryNo,
        disc
      }
      
      this.$emit('editedData', editedData)
    }
  }
}
</script>

<style lang="scss" scoped>
@import '@/assets/styles/main.scss';

// Dialog 設定
.modal.fade .modal-dialog{
  max-width: 600px;
  width: 600px;
  margin: 0;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  .modal-body {
    padding: 30px 40px;
  }
}
</style>
<template>
  <div>
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="exampleModalLabel">刪除</h5>
          <button type="button" class="close" data-dismiss="modal" aria-label="Close">
            <span aria-hidden="true">&times;</span>
          </button>
        </div>
        <div class="modal-body">
          您是否要刪除 <span class="hint">{{deleteData ? deleteData.category : ''}}</span> 項目一筆金額為 <span class="hint">${{deleteData ? deleteData.cash : ''}}</span> 元的紀錄?
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" data-dismiss="modal">取消</button>
          <button type="button" class="btn btn-danger" @click="deletedItem(deleteData)">刪除</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    deleteData: {
      type: Object,
      default: null
    }
  },
  watch: {
    deleteData () {
      console.log('deleteData', this.deleteData)
    }
  },
  methods: {
    deletedItem () {
      this.$emit('deletedItem', this.deleteData)
    }
  }
}
</script>

<style lang="scss" scoped>
@import '@/assets/styles/main.scss';

.hint {
  color: rgb(171, 19, 19);
  font-weight: normal;
  font-size: 18px;
}

// Dialog 設定
.modal.fade .modal-dialog{
  margin: 0;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  .modal-body {
    padding: 40px;
  }
}
</style>
<template>
  <div>
    <div>
      <b-row class="justify-content-center">
        <b-col sm="4" md="2">
          <b-button block @click="addOperator()">+ 事業者追加</b-button>
        </b-col>
        <b-col sm="4" md="2">
          <b-button block>作成</b-button>
        </b-col>
      </b-row>
      <b-modal v-model="showModal">
        <template slot="modal-title">{{ modalTitle }}</template>
        <b-form-group label-for="inputname">
          <b-form-radio-group
            :plain="true"
            :options="[
              {text: '新規 ', value: true},
              {text: '既存 ', value: false}
              ]"
            :checked="1"
            v-model="isNew">
          </b-form-radio-group>
          <b-form-group
            label="ID"
            label-for="id"
            :label-cols="2">
            <b-form-input id="id" v-model="itemId" v-bind:disabled="isNew"></b-form-input>
          </b-form-group>
          <b-form-group
            label="名称"
            label-for="name"
            :label-cols="2">
            <b-form-input id="name" v-model="itemName"></b-form-input>
          </b-form-group>
          <div v-if="itemType === 'st'">
            <b-form-group
              label="端末台数"
              label-for="number"
              :label-cols="2">
              <b-form-input id="number" v-model="itemNum"></b-form-input>
            </b-form-group>
          </div>
        </b-form-group>
         <template slot="modal-footer" slot-scope="{ ok, cancel }">
           <b-button size="sm" @click="cancel()">Cancel</b-button>
           <b-button size="sm" variant="primary" @click="add(), showModal=false">OK</b-button>
         </template>
      </b-modal>
    </div>
    <ul v-for="(data, index) in treeData" id="tree" :key="index">
      <tree-item
        class="item"
        :item="data"
        @add-item="addItem">
      </tree-item>
    </ul>
  </div>
</template>

<script>
import treeItem from './TreeItem.vue'
import sampleData from '../sample_data.js'

export default {
  name: 'tree',
  components: {treeItem},
  data: () => {
    return {
      showModal: false,
      treeData: sampleData,
      isNew: true,
      itemName: '',
      itemId: '',
      itemNum: '',
      itemType: '',
      item: Object,
      modalTitle: ''
    }
  },
  methods: {
    addOperator: function () {
      this.modalTitle = "事業者登録"
      this.itemType = 'op'
      this.showModal = true
    },
    addItem: function (item) {
      switch (item.type) {
        case "op":
          if (!item.merchants) {
            this.$set(item, 'merchants', [])
          }
          this.modalTitle = "加盟店登録"
          this.itemType = "mc"
          break;
        case "mc":
          if (!item.stores) {
            this.$set(item, 'stores', [])
          }
          this.modalTitle = "設置店舗登録"
          this.itemType = "st"
          break;
        default:
      }
      this.item = item
      this.showModal = true
    },
    add: function () {
      const itemType = this.itemType
      const itemNum = this.itemNum
      let data = {
        name: this.itemName, 
        id: this.itemId,
        type: this.itemType
      }
      switch (itemType) {
        case "op":
          this.treeData.push(data)
          break;
        case "mc":
          this.item.merchants.push(data)
          break;
        case "st":
          data["terminals"] = itemNum
          this.item.stores.push(data)
          break;
        default:
      }
      this.clearItemName()
    },
    
    clearItemName: function () {
      this.itemName = ''
      this.itemNum = ''
      this.itemId = ''
    },
    deleteType(item){
      let vm = this
      item.forEach(function(value) {
        delete value.type
        if (value.merchants) {
          vm.deleteType(value.merchants)
        } else if (value.stores) {
          vm.deleteType(value.stores)
        }
      })
    }
  }
}
</script>

<style>
#tree {
  width: 500px;
  background: #dadada;
  border-radius :8px;
  box-shadow :0px 0px 5px silver;
  padding: 0.5em 0.5em 0.5em 2em;
  margin: auto;
  margin-top: 20px;
}
</style>

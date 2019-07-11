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
      
      <b-modal size="xl" v-model="showModal">
        <template slot="modal-title">{{ modalTitle }}</template>
        <b-form-radio-group
          :plain="true"
          :options="[
            {text: '新規 ', value: true},
            {text: '既存 ', value: false}
            ]"
          :checked="1"
          v-model="isNew">
        </b-form-radio-group>
          
        <b-row>
          <b-col md="4">
            <b-form-group label="ID" label-for="id">
              <b-form-textarea id="id" v-model="input.id" v-bind:disabled="isNew" rows="20"></b-form-textarea>
            </b-form-group>
          </b-col>
          <b-col md="4">
            <b-form-group label="名称" label-for="name">
              <b-form-textarea id="name" v-model="input.name" rows="20"></b-form-textarea>
            </b-form-group>
          </b-col>
          <b-col md="4">
            <div v-if="itemType === 'store'">
              <b-form-group label="端末台数" label-for="number">
                <b-form-textarea id="number" v-model="input.num" rows="20"></b-form-textarea>
              </b-form-group>
            </div>
          </b-col>  
        </b-row>
        
         <template slot="modal-footer">
           <b-button size="sm" @click="clearItem(), showModal=false">キャンセル</b-button>
           <b-button size="sm" variant="primary" @click="add(), showModal=false">追加</b-button>
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
      modalTitle: '',
      isNew: true,
      input: {
        id: '',
        name: '',
        num: ''
      },
      item: Object,
      itemType: '' 
    }
  },
  methods: {
    addOperator: function () {
      this.modalTitle = "事業者登録"
      this.itemType = 'enterprise'
      this.showModal = true
    },
    addItem: function (item) {
      switch (item.type) {
        case "enterprise":
          if (!item.merchants) {
            this.$set(item, 'merchants', [])
          }
          this.modalTitle = "加盟店登録"
          this.itemType = "merchant"
          break;
        case "merchant":
          if (!item.stores) {
            this.$set(item, 'stores', [])
          }
          this.modalTitle = "設置店舗登録"
          this.itemType = "store"
          break;
        default:
      }
      this.item = item
      this.showModal = true
    },
    createData (namearr, idarr, type) {
      let data = []
      const arr = this.isNew ? namearr : idarr
      for (let i = 0; i < arr.length; i++){
        data.push({
          id: idarr[i] ? idarr[i] : null, 
          name: namearr[i] ? namearr[i] : null,
          type: type
        })
      }
      return data
    },
    add: function () {
      const namearr = this.input.name.split(/\r\n|\n/)
      const idarr = this.input.id.split(/\r\n|\n/)
      const numarr = this.input.num.split(/\r\n|\n/)
      
      let data = this.createData(namearr, idarr, this.itemType)
      
      switch (this.itemType) {
        case "enterprise":
          this.treeData = this.treeData.concat(data)
          break;
        case "merchant":
          this.item.merchants = this.item.merchants.concat(data)
          break;
        case "store":
          for (let i = 0; i < data.length; i++){
            data[i]["terminals"] = numarr[i] ? numarr[i] : null
          }
          this.item.stores = this.item.stores.concat(data)
          break;
        default:
      }
      this.clearItem()
    },
    clearItem () {
      this.input = { name: '', num: '', id: ''}
    },
    deleteType (item) {
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

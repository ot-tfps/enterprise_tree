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
              <b-form-group label="端末台数" label-for="terminal">
                <b-form-textarea id="terminal" v-model="input.terminal" rows="20"></b-form-textarea>
              </b-form-group>
            </div>
          </b-col>  
        </b-row>
        
        <p v-if="errors.length">
          <ul class="text-danger pl-0">
            <li class="pb-0" v-for="(error,index) in errors" :key="index">{{ error }}</li>
          </ul>
        </p>
        
         <template slot="modal-footer">
           <b-button size="sm" @click="closeModal()">キャンセル</b-button>
           <b-button size="sm" variant="primary" @click="add()">追加</b-button>
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
        terminal: ''
      },
      item: Object,
      itemType: '',
      disable: true,
      errors: []
    }
  },
  methods: {
    addOperator () {
      this.modalTitle = "事業者登録"
      this.itemType   = 'enterprise'
      this.showModal  = true
    },
    addItem (item) {
      switch (item.type) {
        case "enterprise":
          if (!item.merchants) {
            this.$set(item, 'merchants', [])
          }
          this.modalTitle = "加盟店登録"
          this.itemType   = "merchant"
          break;
        case "merchant":
          if (!item.stores) {
            this.$set(item, 'stores', [])
          }
          this.modalTitle = "設置店舗登録"
          this.itemType   = "store"
          break;
        default:
      }
      this.item = item
      this.showModal = true
    },
    createData (nameList, idList, type) {
      let data = []
      const arr = this.isNew ? nameList : idList
      for (let i = 0; i < arr.length; i++){
        data.push({
          id: idList[i] ? idList[i] : null, 
          name: nameList[i] ? nameList[i] : null,
          type: type
        })
      }
      return data
    },
    add () {
      //textareaを改行で分解して配列に格納
      const nameList = this.input.name.split(/\r\n|\n/)
      const idList = this.input.id.split(/\r\n|\n/)
      const terminalList = this.input.terminal.split(/\r\n|\n/)
      
      //validation
      this.errors = [];
      if (this.isNew && this.isIncludeNull(nameList)) {
        this.errors.push("名称に空文字が含まれています")
      }
      if (!this.isNew && this.isIncludeNull(idList)) {
        this.errors.push("IDに空文字が含まれています")
      } else if (!this.isOnlyNumId(idList)) {
        this.errors.push("IDは半角数字で入力してください")
      }
      if (this.itemType === "store" && !this.isOnlyNumId(terminalList)) {
        this.errors.push("端末台数は半角数字で入力してください")
      }
      if (this.errors.length > 0) {
        return
      }
      
      //追加するdataを作成
      let data = this.createData(nameList, idList, this.itemType)
      
      //現在のtypeによって追加先のプロパティを切り分け
      switch (this.itemType) {
        case "enterprise":
          this.treeData = this.treeData.concat(data)
          break;
        case "merchant":
          this.item.merchants = this.item.merchants.concat(data)
          break;
        case "store":
          for (let i = 0; i < data.length; i++) {
            data[i]["terminals"] = terminalList[i] ? terminalList[i] : null
          }
          this.item.stores = this.item.stores.concat(data)
          break;
        default:
      }
      this.closeModal()
    },
    isIncludeNull (arr) {
      return arr.includes("")
    },
    isOnlyNumId (arr) {
      return arr.every(value => !isNaN(value))
    },
    closeModal () {
      this.showModal = false
      this.input = { id: '', name: '', terminal: ''}
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

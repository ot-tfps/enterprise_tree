<template>
  <div>
    <div>
      
      <b-row class="ml-3">
        <b-col sm="4" md="2">
          <b-button block @click="addOperator()">+ 事業者追加</b-button>
        </b-col>
        <b-col sm="4" md="2">
          <b-button block @click="sendJson()">作成</b-button>
        </b-col>
      </b-row>
      
      <ul v-for="(data, index) in item" class="tree" :key="index">
        <tree-item
          class="item"
          :item="data"
          @add-button-clicked="showAddModal"
          @delete-button-clicked="showDeleteModal">
        </tree-item>
      </ul>
      
      <b-modal v-model="showJson" hide-footer @hidden="clear">
        <div class="card mr-auto ml-auto">
          <div class="card-header">
            <span>json</span>
          </div>
          <div class="card-body p-2">
            {{jsonToSend}}
          </div>
        </div>
      </b-modal>
      
      <b-modal size="xl" v-model="showModal" @show="resetModal">
        <template slot="modal-title">{{ modalTitle }}</template>
        <div class="ye"></div>
        
        <div class="text-center loading" v-if="loading">
          <b-spinner style="width: 4rem; height: 4rem;" label="Text Centered"></b-spinner>
        </div>
        
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
            <div v-show="itemType === 'store'">
              <b-form-group label="端末台数" label-for="terminal">
                <b-form-textarea id="terminal" v-model="input.terminal" rows="20"></b-form-textarea>
              </b-form-group>
            </div>
          </b-col>
        </b-row>
        
        <p v-show="errors.length">
          <ul class="text-danger pl-0">
            <li class="pb-0" v-for="(error,index) in errors" :key="index">{{ error }}</li>
          </ul>
        </p>
        
        <template slot="modal-footer">
          <b-button size="sm" @click="showModal=false">キャンセル</b-button>
          <b-button size="sm" variant="primary" @click="addButtonClicked">追加</b-button>
        </template>
      </b-modal>
      
      <b-modal size="sm" v-model="deleteModal" hide-header @ok="deleteTrue">
        <p class="my-4">削除してよろしいですか？</p>
      </b-modal>
      
    </div>
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
      deleteModal: false,
      isNew: true,
      loading: false,
      showJson: false,
      modalTitle: '',
      itemType: '',
      input: {
        id: '',
        name: '',
        terminal: ''
      },
      item: sampleData,
      tempItem: Object,
      errors: [],
      deleteData: [],
      jsonToSend: []
    }
  },
  methods: {
    addOperator () {
      this.modalTitle = "事業者登録"
      this.itemType   = 'enterprise'
      this.showModal  = true
    },
    deleteTrue() {
      this.deleteItem()
    },
    showDeleteModal (data) {
      this.deleteModal = true
      this.deleteData = data
    },
    deleteItem () {
      const parent = this.deleteData[0]
      const child  = this.deleteData[1]
      let parentItem
      
      switch (child.type) {
        case "enterprise":
          parentItem = parent.item
          break;
        case "merchant":
          parentItem = parent.item["merchants"]
          break;
        case "store":
          parentItem = parent.item["stores"]
          break;
        default:
      }
      for (let i = 0, len = parentItem.length; i < len; i++) {
        if (parentItem[i] === child) {
          parentItem.splice(i, 1)
          break
        }
      }
    },
    showAddModal (item) {
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
      this.tempItem  = item
      this.showModal = true
    },
    requestAnimation (callback) {
      requestAnimationFrame(() => {
        requestAnimationFrame(callback)
      })
    },
    createData (nameList, idList, terminalList, type) {
      let data  = []
      const arr = this.isNew ? nameList : idList
      for (let i = 0; i < arr.length; i++){
        data.push({
          id: !this.isNew ? idList[i] : null, 
          name: nameList[i] ? nameList[i] : null,
          type: type
        })
        if (this.itemType == "store") {
          data[i]["terminals"] = terminalList[i] ? terminalList[i] : 0
        }
      }
      return data
    },
    addButtonClicked () {
      //textareaを改行で分解して配列に格納
      const nameList     = this.input.name.split(/\r\n|\n/)
      const idList       = this.input.id.split(/\r\n|\n/)
      const terminalList = this.input.terminal.split(/\r\n|\n/)
      //validation
      //name
      this.errors = []
      if (this.isNew && this.isIncludeNull(nameList)) {
        this.errors.push("名称に空文字が含まれています")
      }
      //id
      if (!this.isNew && this.isIncludeNull(idList)) {
        this.errors.push("IDに空文字が含まれています")
      }
      if (!this.isNew && !this.isOnlyNum(idList)) {
        this.errors.push("IDは半角数字で入力してください")
      }
      //terminal
      if (this.itemType == "store" && !this.isOnlyNum(terminalList)) {
        this.errors.push("端末台数は半角数字で入力してください")
      }
      if (this.errors.length > 0) {
        return
      }
      //追加するdataを作成
      let data = this.createData(nameList, idList, terminalList, this.itemType)
      // loading開始
      const self = this
      this.loading = true
      // loading animationが開始されてから後続処理実行
      this.requestAnimation(() => {
        //input dataをtree itemに追加
        self.addItem(data)
        self.loading   = false
        self.showModal = false
      })
    },
    addItem (data) {
      //typeによって追加先のプロパティを切り分け
      switch (this.itemType) {
        case "enterprise":
          this.item = this.item.concat(data)
          break;
        case "merchant":
          this.tempItem.merchants = this.tempItem.merchants.concat(data)
          break;
        case "store":
          this.tempItem.stores = this.tempItem.stores.concat(data)
          break;
        default:
      }
    },
    isIncludeNull (arr) {
      return arr.includes("")
    },
    isOnlyNum (arr) {
      return arr.every(value => !isNaN(value))
    },
    resetModal () {
      this.errors    = []
      this.input     = { id: '', name: '', terminal: ''}
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
      return item
    },
    sendJson () {
      const jsonToSend = this.deleteType(this.item)
      this.jsonToSend  = jsonToSend
      this.showJson    = true
    },
    clear () {
      this.item = []
    }
  }
}
</script>

<style>
.tree {
  background: #dadada;
  border-radius :8px;
  box-shadow :0px 0px 5px silver;
  padding: 0.5em 0.5em 0.5em 2em;
  margin: 30px 30px 0 30px;
  margin-top: 20px;
}
.loading{
  position: absolute;
  z-index: 10;
  left: 50%;
  top: 40%;
}
/* textarea {
  background-image:
    repeating-linear-gradient(
      #fff,
      #fff calc(1.5rem - 1px),
      #ced4da calc(1.5rem - 1px),
      #ced4da 1.5rem,
      #fff 1.5rem
    );
  background-origin: content-box;
  background-clip: content-box;
  background-attachment: local;
} */

</style>

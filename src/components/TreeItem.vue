<template>
  <li class="tree-col">
    <div :class="{bold: isFolder}" class="col-width" @mouseover="isHover=true" @mouseout="isHover=false">
      <i v-show="isFolder" @click="toggle" class="fas fa-fw" :class="caretClass" aria-hidden="true"></i>
      <i v-show="item.type=='enterprise'" class="far fa-building fa-fw"></i>
      <i v-show="item.type=='merchant'" class="fas fa-store-alt fa-fw"></i>
      <i v-show="item.type=='store'" class="fas fa-tablet-alt fa-fw"></i>

      <div v-if="item.id" class="d-inline">
        <span v-if="!editableId" @click="editableId=true, setEditable()">{{ item.id }} </span>
        <input v-else type="text" size="8" :value="item.id" ref="ref" @input="updateId" @blur="editableId=false" @keyup.enter="editableId=false">
      </div>
      
      <div v-if="item.name" class="d-inline">
        <span v-if="!editableName" @click="editableName=true, setEditable()">{{item.name | truncate(10)}}</span>
        <input v-else type="text" :value="item.name" ref="ref" @input="updateName" @blur="editableName=false" @keyup.enter="editableName=false">
      </div>
      
      <span v-if="!editableTerminal" v-show="item.terminals!=undefined" @click="editableTerminal=true, setEditable()">： {{item.terminals!=null?item.terminals:0}}台</span>
      <input v-else type="text" size="3" :value="item.terminals" ref="ref" @input="updateTerminal" @blur="editableTerminal=false" @keyup.enter="editableTerminal=false">

      <span v-show="isHover">
        <i class="far fa-trash-alt fa-fw" @click="deleteItem(item)"></i>
      </span>
      
      <b-button v-if="showAdd" variant="outline-secondary" class="d-block mb-2" size="sm" @click="$emit('add-button-clicked', item)">{{addName}}を追加する</b-button>
    </div>
    <ul v-show="isOpen">
      <tree-item
        class="item"
        v-for="(merchant, index) in children"
        :key="index"
        :item="merchant"
        @click="toggle"
        @add-button-clicked="$emit('add-button-clicked', $event)"
        @delete-button-clicked="$emit('delete-button-clicked', $event)">
      </tree-item>
    </ul>
  </li>
</template>

<script>
export default {
  name: 'tree-item',
  props: {
    item: Object
  },
  data: () => {
    return {
      isOpen: true,
      isHover: false,
      editableId: false,
      editableName: false,
      editableTerminal: false,
    }
  },
  computed: {
    isFolder: function () {
      return this.children &&
        this.children.length
    },
    showAdd: function () {
      return this.item.type != 'store'
    },
    children: function () {
      return this.item.merchants || this.item.stores
    },
    caretClass () {
      return this.isOpen ? 'fa-caret-down' : 'fa-caret-right'
    },
    addName () {
      return this.item.type == "enterprise" ? "加盟店" : "設置店舗"
    }
  },
  filters: {
    truncate: function (text, length) {
      if (text.length <= length) {
        return text;
      } else {
        return text.substring(0, length) + '...';
      }
    } 
  },
  methods: {
    toggle () {
      if (this.isFolder) {
        this.isOpen = !this.isOpen
      }
    },
    updateId (e) {
      if (e.target.value && !isNaN(e.target.value)) {
        this.item.id = e.target.value
      }
    },
    updateName (e) {
      if (this.item.id) {
        this.item.name = e.target.value
      } else if (e.target.value) {
        this.item.name = e.target.value
      }
    },
    updateTerminal (e) {
      if (!e.target.value) {
        this.item.terminals = 0
      } else if (!isNaN(e.target.value)) {
        this.item.terminals = e.target.value
      }
    },
    setEditable () {
      this.$nextTick(() => this.$refs.ref.focus())
    },
    deleteItem () {
      const data = [this.$parent, this.item]
      this.$emit('delete-button-clicked', data)
    }
  }
}
</script>

<style>
.tree-col {
  display: flex;
  padding: 0;
  margin-right: -15px;
  margin-left: -15px;
}
.col-width{
  width: 350px;
}
.caretClass {
  margin-left: -1rem;
}
</style>
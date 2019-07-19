<template>
  <li class="tree-col">
    <div :class="{bold: isFolder}" class="col-width">
      <i v-show="isFolder" @click="toggle" class="fas fa-fw" :class="caretClass" aria-hidden="true"></i>
      <i v-show="item.type=='enterprise'" class="fa fa-address-book fa-sm"></i>
      <i v-show="item.type=='merchant'" class="fa fa-institution fa-sm"></i>
      <i v-show="item.type=='store'" class="fa fa-building-o fa-sm"></i>
      {{ item.id }} {{ item.id ? ':' : '' }} {{ item.name | truncate(10) }} 
      <span v-show="item.terminals != undefined">{{'：' + item.terminals + '台'}}</span>
      <i v-show="showAdd" class="fas fa-plus fa-fw" aria-hidden="true" @click="$emit('add-item', item)"></i>
      <i class="fas fa-edit fa-fw"></i>
      <i class="far fa-trash-alt fa-fw"></i>
    </div>
    <ul v-show="isOpen">
      <tree-item
        class="item"
        v-for="(merchant, index) in children"
        :key="index"
        :item="merchant"
        @click="toggle"
        @add-item="$emit('add-item', $event)"
        @delete-item="$emit('delete-item', $event)">
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
      isOpen: true
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
    toggle: function () {
      if (this.isFolder) {
        this.isOpen = !this.isOpen
      }
    deleteItem () {
      const data = [this.$parent, this.item]
      this.$emit('delete-item', data)
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
<template>
  <li class="tree-col">
    <div :class="{bold: isFolder}" class="col-width">
      <span v-show="isFolder" @click="toggle">[ {{ isOpen ? '-' : '+' }} ]</span>
      <i v-show="item.type=='enterprise'" class="fa fa-address-book fa-sm"></i>
      <i v-show="item.type=='merchant'" class="fa fa-institution fa-sm"></i>
      <i v-show="item.type=='store'" class="fa fa-building-o fa-sm"></i>
      {{ item.id }} {{ item.id ? ':' : '' }} {{ item.name | truncate(10) }} 
      <span v-show="item.terminals != undefined">{{'：' + item.terminals + '台'}}</span>
      <span v-show="showAdd" class="add" @click="$emit('add-item', item)"> +</span>
    </div>
    <ul v-show="isOpen">
      <tree-item
        class="item"
        v-for="(merchant, index) in children"
        :key="index"
        :item="merchant"
        @click="toggle"
        @add-item="$emit('add-item', $event)">
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
</style>
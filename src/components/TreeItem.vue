<template>
  <li>
    <div
      :class="{bold: isFolder}"
      @click="toggle">
      <i v-show="item.type==='op'" class="fa fa-address-book fa-sm"></i>
      <i v-show="item.type==='mc'" class="fa fa-institution fa-sm"></i>
      <i v-show="item.type==='st'" class="fa fa-building-o fa-sm"></i>
      <i v-show="item.type==='tm'" class="fa fa-tablet fa-sm"></i>
      {{ item.id }} {{ item.id ? ':' : '' }} {{ item.name }} {{ item.type == 'tm' ? '台' : '' }}
      <span v-show="item.children != undefined">[ {{ isOpen ? '-' : '+' }} ]</span>
    </div>
    <ul v-show="isOpen">
      <li v-show="showAdd" class="add" @click="$emit('add-item', item)">+</li>
      <tree-item
        class="item"
        v-for="(child, index) in item.children"
        :key="index"
        :item="child"
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
      return this.item.children &&
        this.item.children.length
    },
    showAdd: function () {
      return this.item.type != 'st' && this.item.type != 'tm'
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

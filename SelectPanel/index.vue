<template>
  <div class="cas-select-panel">
    <div class="cas-select-panel__header">
      <div class="cas-select-panel__title">{{title}}</div>
      <div class="cas-select-panel__clear" @click="handleClearAll">清空全部</div>
    </div>
    <div class="cas-select-panel__content">
      <div class="cas-select-panel__item" v-for="item in data" :key="item.id">
        <span class="cas-select-panel__item-text">{{item.text}}</span>
        <span class="cas-select-panel__item-remove" @click="handleRemove(item.id)">x</span>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: "CasSelectPanel",
  props: {
    data: {
      type: Array
    },
    title: {
      type: String,
      default: "已选"
    }
  },
  methods: {
    handleClearAll() {
      this.$emit("clear-all", this.data.map(item => item.id));
    },
    handleRemove(id) {
      this.$emit("remove", id);
    }
  }
};
</script>
<style lang="scss">
$prefix: "cas-select-panel";
.#{$prefix} {
  padding: 10px;
  background-color: #fff;
  border: 1px solid #dee4f5;
  border-radius: 2px;
}

.#{$prefix}__header {
  display: flex;
  justify-content: space-between;
}

.#{$prefix}__clear {
  color: blue;
  cursor: pointer;
}

.#{$prefix}__content {
}

.#{$prefix}__item {
  display: flex;
  justify-content: space-between;
  padding: 5px;
  background: antiquewhite;
  cursor: pointer;
  margin: 10px 0;
}
</style>

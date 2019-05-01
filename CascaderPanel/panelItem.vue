<template>
  <div class="cascader-panel-item">
    <div class="cascader-panel-item__header">{{title}}</div>
    <div class="cascader-panel-item__content">
      <div class="cascader-panel-item__list-item allcheck" v-if="hasAllCheck && data.length">
        <el-checkbox
          class="cascader-panel-item__checkbox"
          v-model="isAllCheck"
          @change="handleAllCheckChange"
        ></el-checkbox>
        <span
          class="cascader-panel-item__text"
          :class="{'cascader-panel-item__textChecked':isAllCheck}"
        >全选</span>
      </div>
      <div
        class="cascader-panel-item__list-item"
        v-for="(item,index) in data"
        :key="item.id"
        :class="getItemClass(item)"
        @click="handleSelect(item,index,$event)"
      >
        <el-checkbox
          :disabled="item.disabled"
          class="cascader-panel-item__checkbox"
          v-model="checkList[index].checked"
          v-show="showCheckBox && checkList[index]"
          @change="checked => handleChange(checked,item)"
        ></el-checkbox>
        <span
          class="cascader-panel-item__text"
          :class="{'cascader-panel-item__textChecked':checkList[index].checked}"
        >{{item.text}}</span>
        <span
          v-if="item.childNodes && item.childNodes.length"
          class="cascader-panel-item__toright"
          :class="{'cascader-panel-item__textChecked':checkList[index].checked}"
        >
          <i class="el-icon-arrow-right"></i>
        </span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "CascaderItem",
  props: {
    title: {
      type: String,
      required: true
    },
    data: {
      type: Array
    },
    level: {
      type: Number
    },
    showCheckBox: {
      type: Boolean,
      default: true
    },
    hasAllCheck: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      checkList: this.data.map(item => {
        return { checked: item.checked };
      })
    };
  },
  watch: {
    data: {
      deep: true,
      handler() {
        this.checkList = this.data.map(item => {
          return { checked: item.checked };
        });
      }
    }
  },
  computed: {
    isAllCheck: {
      get() {
        return this.checkList.every(item => item.checked);
      },
      set() {}
    }
  },
  methods: {
    handleSelect({ id, childNodes }, index, e) {
      if (
        (e.target.matches(".el-checkbox__original") ||
          e.target.matches(".el-checkbox__inner")) &&
        (!childNodes || !childNodes.length)
      ) {
        return;
      }
      this.$emit("select", id);
      //如果是leaf节点，则相当于直接进行一次check
      if (!childNodes || !childNodes.length) {
        this.checkList[index].checked = !this.checkList[index].checked;
        this.$emit("check", this.checkList[index].checked, id);
      }
    },
    handleChange(checked, { id }) {
      this.$emit("check", checked, id);
    },
    getItemClass(item) {
      return item.selected && (item.childNodes && item.childNodes.length)
        ? "is-select"
        : "";
    },
    handleAllCheckChange(checked) {
      this.checkList.forEach(item => (item.checked = checked));
      this.$emit("check-all", checked, this.data[0].parent.id);
    }
  }
};
</script>

<style lang="scss">
$prefix: "cascader-panel-item";

.#{$prefix} {
  padding: 10px;
  background-color: #fff;
  border: 1px solid #DDDEE1;
  border-radius: 4px;
}
.#{$prefix}__header {
  font-family: PingFangSC-Medium;
  font-size: 12px;
  color: #1c2438;
  letter-spacing: 0;
  line-height: 14px;
}
.#{$prefix}__list-item {
  margin: 5px 0;
  cursor: pointer;
  padding: 10px 0;

  &.is-select,
  &:hover {
    background: #f5fbff;
    z-index: 9;
  }
}

.#{$prefix}__checkbox {
  margin-right: 10px;
}

.#{$prefix}__text {
  font-family: MicrosoftYaHei;
  font-size: 12px;
  color: #1c2438;
  letter-spacing: 0;
  line-height: 12px;
}
.#{$prefix}__textChecked {
  color: #0590ff;
}
.#{$prefix}__toright {
  float: right;
}
</style>

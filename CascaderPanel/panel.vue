<script>
import { flattenData } from "./util";
import CascaderItem from "./panelItem";
import Node from "./model/node";
import SelectPanel from "components/SelectPanel";

export default {
  components: { CascaderItem, SelectPanel },
  name: "CascaderPanel",
  props: {
    data: {
      type: Array,
      required: true
    },
    props: {
      type: Object,
      default: function() {
        return {
          key: "id",
          label: "label"
        };
      }
    },
    settings: {
      type: Array,
      required: true
    },
    checkedLevel: {
      type: Number,
      required: true
    },
    zipLevel: {
      type: Number,
      required: true
    },
    isSingle: {
      type: Boolean,
      default: false
    }
  },
  render(h) {
    let data = this.rootNode.childNodes;
    return (
      <div class="cascader-panel">
        <div class="cascader-panel__select">
          {this.curSelectData.map((item, index) => {
            const {
              level,
              title,
              showCheckBox,
              hasAllCheck
            } = this.settings.find(item => item.level === index + 1);
            return this.curShowList[index] ? (
              <CascaderItem
                key={index}
                data={item}
                title={title}
                level={level}
                showCheckBox={showCheckBox}
                hasAllCheck={hasAllCheck}
                on-select={this.handleSelect}
                on-check={this.handleCheck}
                on-check-all={(checked, pid) =>
                  this.handleAllCheck(checked, pid, index)
                }
              />
            ) : null;
          })}
        </div>
        <SelectPanel
          class="cascader-panel__choosen"
          data={this.checkedData}
          on-clear-all={this.handleClearAll}
          on-remove={this.handleRemoveItem}
        />
        {this.renderSlot()}
      </div>
    );
  },
  data() {
    return {
      rootNode: null,
      flattenData: [],
      curShowList: [],
      checkedData: []
    };
  },
  watch: {
    data: {
      deep: true,
      handler() {
        this.rootNode = new Node({
          data: this.data,
          props: this.props
        });
        this.flattenData = flattenData(this.rootNode);
        this.curShowList = new Array(this.maxLevel)
          .fill()
          .map((item, index) => {
            let isShow = index === 0 ? true : false;
            return isShow;
          });
      }
    },
    checkedData: {
      deep: true,
      handler(val, old) {
        this.$emit("result-change", val);
      }
    }
  },
  computed: {
    maxLevel() {
      return this.settings.length;
    },
    curSelectData() {
      const result = [];
      let data = this.rootNode;
      for (let i = 0; i < this.maxLevel; i++) {
        //有可能没有选中的
        if (!data || !data.childNodes) {
          result.push([]);
          continue;
        }
        result.push(data.childNodes);
        let selectNode = data.childNodes.find(item => item.selected) || [];
        data = selectNode;
      }
      return result;
    }
  },
  created() {
    this.rootNode = new Node({
      data: this.data,
      props: this.props
    });
    this.flattenData = flattenData(this.rootNode);
    //console.log(this.flattenData);
    this.curShowList = new Array(this.maxLevel).fill().map((item, index) => {
      let isShow = index === 0 ? true : false;
      return isShow;
    });
  },
  methods: {
    handleSelect(id) {
      //单选
      const selectNode = this.flattenData.find(item => item.id === id);
      selectNode.parent.childNodes.forEach(node => (node.selected = false));
      selectNode.selected = true;

      //下一级展示出来,更深的层级不渲染
      if (selectNode.level < this.maxLevel) {
        this.curShowList[selectNode.level] = !!selectNode.childNodes.length;
        for (let i = selectNode.level + 1; i < this.maxLevel; i++) {
          this.curShowList[i] = false;
        }
      }
      //单选模式，逻辑变为类似级联选择器，选择非最深层次的节点直接清空当前节点下所有的checked结果，视为重新选择
      if (this.isSingle && selectNode.level !== this.maxLevel) {
        this.flattenData.forEach(p => (p.checked = false));
        this.getCheckedData();
      }
    },
    handleCheck(isCheck, id, immediate = true) {
      const checkedLevel = this.checkedLevel;
      //勾选当前级别及子级
      let selectNode = this.flattenData.find(item => item.id === id);
      if (!selectNode) {
        return;
      }
      //递归
      //由父到子
      function setCheck(node) {
        node.checked = isCheck;
        if (!node.childNodes.length && node.level < checkedLevel) {
          node.noChildChecked = isCheck;
        }
        if (!Array.isArray(node.childNodes) || !node.childNodes.length) {
          return;
        }
        node.childNodes.forEach(node => {
          setCheck(node);
        });
      }
      //由子到父
      function setParentCheck(parent) {
        if (!parent || !parent.parent) {
          return;
        }
        parent.checked = parent.childNodes.every(child => {
          return child.checked === true;
        });
        setParentCheck(parent.parent);
      }

      setCheck(selectNode);
      setParentCheck(selectNode.parent);
      if (immediate) {
        this.getCheckedData();
      }
    },
    handleAllCheck(isCheck, parentId, index) {
      //console.log("触发全选");
      const selectNode = this.flattenData.find(item => item.id === parentId);
      // 第一级全选性能优化
      if (index === 0) {
        this.flattenData.forEach(p => (p.checked = isCheck));
        if (isCheck) {
          this.checkedData = this.flattenData.filter(p => p.level === 1);
        } else {
          this.checkedData = [];
        }
      } else {
        selectNode.checked = isCheck;
        selectNode.childNodes.forEach(item => {
          this.handleCheck(isCheck, item.id, false);
        });
        this.getCheckedData();
      }
    },
    getCheckedData() {
      //压缩数据 一级一级压缩
      let result = [];
      const toZipData = this.flattenData.filter(p => p.level === this.zipLevel);
      function step(nodes) {
        if (!nodes || !nodes.length) {
          return;
        }
        const curSelectData = nodes.filter(p => p.checked || p.noChildChecked);
        const noSelectData = nodes.filter(
          p => !(p.checked || p.noChildChecked)
        );
        result.push(...curSelectData);
        noSelectData.forEach(p => step(p.childNodes));
      }
      step(toZipData);
      this.checkedData = result;
    },
    handleRemoveItem(id) {
      this.handleCheck(false, id);
    },
    handleClearAll(ids) {
      ids.forEach(id => {
        this.handleCheck(false, id);
      });
    },
    setCheckedNode(key, immediate = true) {
      const selectNode = this.flattenData.find(
        item => item.data[this.props.key] === key
      );
      if (selectNode) {
        this.handleCheck(true, selectNode.id, immediate);
      }
    },
    setCheckedNodes(keys) {
      /* public API */
      keys.forEach(key => {
        this.setCheckedNode(key, false);
      });
      this.getCheckedData();
    },
    getCheckedNodes(isZip = true) {
      /* public API */
      if (isZip) {
        return this.checkedData.map(item => {
          return {
            id: item.id,
            text: item.text,
            data: item.data,
            level: item.level
          };
        });
      } else {
        return this.flattenData.filter(p => p.checked || p.noChildChecked);
      }
    },
    //清空所有节点
    refreshAll() {
      /* public API */
      const ids = this.flattenData
        .filter(
          p => (p.checked && p.level === this.checkedLevel) || p.noChildChecked
        )
        .map(p => p.id);
      this.handleClearAll(ids);
      this.curShowList = new Array(this.maxLevel).fill().map((item, index) => {
        let isShow = index === 0 ? true : false;
        return isShow;
      });
    }
  }
};
</script>
<style lang="scss">
$prefix: "cascader";

.#{$prefix}-panel {
  display: flex;
  background: #fff;
  height: 300px;
}

.#{$prefix}-panel__select {
  display: flex;
  flex: 2;
}

.#{$prefix}-panel__choosen {
  flex: 1;
}

.#{$prefix}-panel-item {
  flex: 1;
  overflow-y: auto;
}
</style>

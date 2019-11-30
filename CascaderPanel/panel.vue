<script>
import { flattenData } from "./util";
import CascaderItem from "./panelItem";
import CascaderSearch from "./searchPanel";
import Node from "./model/node";
import SelectPanel from "./selectPanel";

export default {
  components: { CascaderItem, CascaderSearch, SelectPanel },
  name: "CascaderPanel",
  props: {
    data: {
      type: Array,
      required: true
    },
    props: {
      type: Object,
      default() {
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
    },
    isSearch: {
      type: Boolean,
      default: false
    },
    isCollapse: {
      type: Boolean,
      default: false
    },
    disabled: {
      type: Boolean,
      default: false
    },
    keyword: {
      type: String
    },
    customKeywordMethod: {
      type: Function
    },
    customFilterMethod: {
      type: Function
    },
    formatterSelectText: {
      type: Function
    }
  },
  render(h) {
    let data = this.rootNode.childNodes;
    return (
      <div class="cascader-panel">
        <div class="cascader-panel__select" v-show={!this.keyword}>
          {this.curSelectData.map((item, index) => {
            const {
              level,
              title,
              showCheckBox,
              hasAllCheck
            } = this.settings.find(item => item.level === index + 1);
            return this.curShowList[index] ? (
              <CascaderItem
                style={{
                  width: `calc(100% / ${
                    this.curShowList.filter(c => c).length
                  })`
                }}
                key={index}
                disabled={this.disabled}
                isSearch={this.isSearch}
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
        <div class="cascader-panel__select" v-show={this.keyword}>
          <cascader-search
            data={this.searchData}
            keyword={this.keyword}
            on-check={this.handleCheck}
            on-check-all={(checked, ids) =>
              this.handleSearchAllCheck(checked, ids)
            }
          />
        </div>
        {this.$scopedSlots.select ? (
          this.$scopedSlots.select({
            data: this.wholeCheckedData,
            remove: this.handleRemoveItem,
            clearAll: this.handleClearAll
          })
        ) : (
          <SelectPanel
            class="cascader-panel__choosen"
            data={this.wholeCheckedData}
            disabled={this.disabled}
            on-clear-all={this.handleClearAll}
            on-remove={this.handleRemoveItem}
          />
        )}
        {this.renderSlot()}
      </div>
    );
  },
  data() {
    return {
      rootNode: null,
      searchData: [],
      lastCheckedData: [],
      flattenData: [],
      curShowList: [],
      checkedData: []
    };
  },
  watch: {
    data: {
      deep: true,
      handler() {
        this.init();

        if (!this.isSearch) {
          this.checkedData = [];
        } else {
          //搜索模式下,先把之前的选择结果存起来
          this.lastCheckedData.push(...this.checkedData);
          this.checkedData = [];
        }
      }
    },
    checkedData: {
      deep: true,
      handler(val, old) {
        const wholeData = this.flattenData.filter(
          p => p.checked || p.noChildChecked
        );
        this.$emit("result-change", val, wholeData);
      }
    }
  },
  computed: {
    maxLevel() {
      return this.settings.length;
    },
    wholeCheckedData() {
      if (!this.isSearch) {
        return this.checkedData;
      }

      const result = [];
      this.checkedData.forEach(curCheckedData => {
        const id = curCheckedData.data[this.props.key];
        const index = this.lastCheckedData.findIndex(
          p => p.data[this.props.key] === id
        );
        if (index > -1) {
          //如果在上一次的选择中已经选择过,则去掉这个结果
          this.lastCheckedData.splice(index, 1);
        }
      });
      return this.lastCheckedData.concat(this.checkedData);
    },
    //获取每一级的数据
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

      //console.log("当前选中数组", result);
      return result;
    }
  },
  created() {
    this.init();
    this.$watch(
      "keyword",
      //模糊搜索,debounce
      this.$root._.debounce(async val => {
        let filterMethod = node => {
          return node.text && node.text.includes(val);
        };
        //自定义搜索，目前需要返回原始数据的id集合用来做筛选
        if (this.customKeywordMethod) {
          const needCustomFilter = this.customKeywordMethod(val);
          if (needCustomFilter) {
            const ids = await this.customFilterMethod(val);
            filterMethod = node => {
              return ids.includes(node.data[this.props.key]);
            };
          }
        }

        this.searchData = val
          ? this.flattenData.filter(node => filterMethod(node))
          : [];
      })
    );
  },
  methods: {
    handleSelect(id) {
      //单选
      const selectNode = this.flattenData.find(item => item.id === id);
      selectNode.parent.childNodes.forEach(node => (node.selected = false));
      selectNode.selected = true;

      //下一级展示出来,更深的层级不渲染
      if (selectNode.level < this.maxLevel && this.isCollapse) {
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

      this.$emit("select", selectNode);
      //console.log("handleSelect", id);
    },
    handleCheck(isCheck, id, immediate = true) {
      const checkedLevel = this.checkedLevel;
      //勾选当前级别及子级
      let selectNode = this.flattenData.find(item => item.id === id);
      if (!selectNode) {
        //找不到去lastData里找
        const index = this.lastCheckedData.findIndex(item => item.id === id);
        this.lastCheckedData.splice(index, 1);
        return;
      }

      //递归
      //由父到子
      function setCheck(node) {
        node.checked = isCheck;
        node.indeterminate = false;
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
        const validChildren = parent.childNodes.filter(p => !p.disabled);
        const checked = validChildren.length
          ? validChildren.every(p => p.checked)
          : false;
        const totalNum = parent.childNodes.length;
        const checkedNum = parent.childNodes.reduce((c, p) => {
          const num = p.checked ? 1 : p.indeterminate ? 0.5 : 0;
          return c + num;
        }, 0);

        parent.checked = checked;
        parent.indeterminate = checkedNum !== totalNum && checkedNum > 0;
        setParentCheck(parent.parent);
      }

      setCheck(selectNode);
      setParentCheck(selectNode.parent);
      if (immediate) {
        this.getCheckedData();
      }
      //console.log("handleCheck", isCheck, id);
    },
    handleAllCheck(isCheck, parentId, index) {
      //console.log("触发全选");
      const selectNode = this.flattenData.find(item => item.id === parentId);
      selectNode.checked = isCheck;
      selectNode.childNodes.forEach(item => {
        this.handleCheck(isCheck, item.id, false);
      });
      this.getCheckedData();
    },
    //处理搜索出来的结果的全选
    handleSearchAllCheck(isCheck, ids) {
      ids.forEach(id => {
        this.handleCheck(isCheck, id, false);
        this.getCheckedData();
      });
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
      //console.log(result, "压缩过后的数据");
      this.checkedData = result.map(item => {
        return {
          ...item,
          text: this.formatterSelectText
            ? this.formatterSelectText(item)
            : item.text
        };
      });
      //不开启压缩
      //this.checkedData = this.flattenData.filter(p => p.checked || p.noChildChecked)
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
      /* public API */
      const selectNode = this.flattenData.find(
        item => item.data[this.props.key] === key
      );
      if (selectNode) {
        this.handleCheck(true, selectNode.id, immediate);
      }
    },
    setCheckedNodes(keys) {
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
            level: item.level,
            parent: item.parent
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
      this.curShowList = this.getShowList();
    },
    getShowList() {
      return new Array(this.maxLevel).fill().map((item, index) => {
        let isShow = this.isCollapse ? (index === 0 ? true : false) : true;
        return isShow;
      });
    },
    init() {
      this.rootNode = new Node({
        data: this.data,
        props: this.props
      });
      this.flattenData = flattenData(this.rootNode);
      this.curShowList = this.getShowList();

      // if (!this.isCollapse) {
      //   let node = this.rootNode;
      //   while (node && node.childNodes) {
      //     node.selected = true;
      //     node = node.childNodes[0];
      //   }
      // }
    }
  }
};
</script>
<style lang="scss">
$prefix: "cascader";

.#{$prefix}-panel {
  display: flex;
  height: 300px;
}

.#{$prefix}-panel__select {
  display: flex;
  flex: 47;
  border: 1px solid #dddee1;
  border-radius: 4px;
  margin-right: 20px;
  background: #fff;
  overflow-x: hidden;
}

.#{$prefix}-panel__choosen {
  flex: 18;
  min-width: 180px;
}
</style>

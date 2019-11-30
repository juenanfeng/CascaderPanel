<template>
<div class="cascader-panel-item cascader-panel--search" v-if="keyword">
    <div class="cascader-panel-item__header">{{title}}</div>
    <div class="cascader-panel-item__content">
        <slot name="empty" v-if="!data.length">
            <div class="empty-slot-wrapper">
                <!-- <img src="../../assets/images/cas-empty.png"> -->
                <span>无搜索内容</span>
            </div>
        </slot>
        <div class="cascader-panel-item__list-item allcheck" v-if="hasAllCheck && data.length">
            <el-checkbox class="cascader-panel-item__checkbox" :indeterminate="isIndeterminate" :value="isAllCheck" @change="handleAllCheckChange"></el-checkbox>
            <span
          class="cascader-panel-item__text"
          :class="{'cascader-panel-item__textChecked':isAllCheck}"
        >全选</span>
        </div>
        <div
            class="cascader-panel-item__list-item"
            v-for="(item,index) in checkList"
            :key="item.id"
            :class="getItemClass(item)">
            <el-checkbox
                :disabled="item.disabled"
                class="cascader-panel-item__checkbox"
                :value="item.checked"
                @change="checked => handleChange(checked,item)"></el-checkbox>
            <span class="cascader-panel-item__text">
          <span :class="{'is-checked':item.checked}">
            <span v-html="item.rawText"></span>
            </span>
            <span class="cas-info">{{item.casInfo}}</span>
            </span>
        </div>
    </div>
</div>
</template>

<script>
export default {
    name: "CascaderSearchPanel",
    props: {
        title: {
            type: String,
            default: "搜索结果"
        },
        keyword: {
            type: String
        },
        data: {
            type: Array
        },
        hasAllCheck: {
            type: Boolean,
            default: true
        }
    },
    data() {
        return {
            delay: 300
        };
    },
    computed: {
        isAllCheck() {
            return this.checkList.every(item => item.checked);
        },
        checkList() {
            return this.getCheckList();
        },
        isIndeterminate(){
            const vaildData = this.checkList.filter(item => !item.disabled)
            const checkedNum = vaildData.filter(item => item.checked).length
            return checkedNum > 0 && checkedNum < vaildData.length
        }
    },
    methods: {
        handleChange(checked, {
            id
        }) {
            this.$emit("check", checked, id);
        },
        getItemClass(item) {
            return item.selected && (item.childNodes && item.childNodes.length) ?
                "is-select" :
                "";
        },
        handleAllCheckChange(checked) {
            this.$emit("check-all", checked, this.data.map(p => p.id));
        },
        getCasInfo(data) {
            const info = [];
            let node = data.parent;

            while (node && node.parent) {
                info.push(node.text);
                node = node.parent;
            }

            return info;
        },
        getRawText(text) {
            // return text.split('').reduce((acc, curr) => {
            //     return acc += this.keyword.includes(curr) ? `<span class="is-high-light">${curr}</span>` : curr
            // }, '')
            const reg = new RegExp(`${this.keyword}`, "g");
            return text.replace(
                reg,
                `<span class="is-high-light">${this.keyword}</span>`
            );
        },
        getCheckList() {
            const sourceData = this.data;

            return this.data.map(p => {
                return {
                    id: p.id,
                    disabled: p.disabled,
                    checked: p.checked,
                    rawText: this.getRawText(p.text),
                    casInfo: this.getCasInfo(p).join(" | ")
                };
            });
        }
    }
};
</script>

<style lang="scss">
.cascader-panel-item.cascader-panel--search {
    width: 100%;
}

.cascader-panel-item__text {
    .is-high-light {
        color: #0590ff;
    }

    &.is-checked {
        color: #0590ff;
    }

    .cas-info {
        margin-left: 8px;
        color: #80848f;
    }
}
</style>

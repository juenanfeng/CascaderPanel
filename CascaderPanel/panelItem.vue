<template>
<div class="cascader-panel-item">
    <div class="cascader-panel-item__header">{{title}}</div>
    <div class="cascader-panel-item__content">
        <slot name="empty" v-if="!data.length && level === 1 && isSearch">
            <div class="empty-slot-wrapper">
                <img src="../../assets/images/cas-empty.png">
                <span>无搜索内容</span>
            </div>
        </slot>
        <div class="cascader-panel-item__list-item allcheck" v-if="hasAllCheck && data.length">
            <el-checkbox
                class="cascader-panel-item__checkbox"
                :disabled="disabled"
                :indeterminate="isIndeterminate"
                v-model="isAllCheck"></el-checkbox>
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
            @click="handleSelect(item,index,$event)">
            <el-checkbox
                :disabled="disabled || item.disabled"
                class="cascader-panel-item__checkbox"
                :indeterminate="item.indeterminate"
                :value="item.checked"
                v-if="showCheckBox"
                @change="checked => handleChange(checked,item)"></el-checkbox>
            <span
          :title="item.text"
          class="cascader-panel-item__text"
          :class="{'cascader-panel-item__textChecked':item.checked || item.indeterminate}"
        >{{item.text}}</span>
            <span
          v-if="item.childNodes && item.childNodes.length"
          class="cascader-panel-item__toright"
          :class="{'cascader-panel-item__textChecked':item.checked || item.indeterminate}"
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
        },
        disabled: {
            type: Boolean,
            default: false
        },
        isSearch: {
            type: Boolean,
            default: false
        }
    },
    data() {
        return {
            delay: 300
        };
    },
    computed: {
        isAllCheck: {
            get() {
                return this.data.every(item => item.checked);
            },
            set(checked) {
                this.$emit("check-all", checked, this.data[0].parent.id);
            }
        },
        isIndeterminate() {
            const vaildData = this.data.filter(item => !item.disabled);
            const checkedNum = vaildData.filter(item => item.checked).length;
            return checkedNum > 0 && checkedNum < vaildData.length;
        }
    },
    methods: {
        handleSelect({
            id,
            childNodes,
            checked
        }, index, e) {
            if (
                (e.target.matches(".el-checkbox__original") ||
                    e.target.matches(".el-checkbox__inner")) &&
                (!childNodes || !childNodes.length)
            ) {
                return;
            }
            this.$emit("select", id);
            //如果是leaf节点，则相当于直接进行一次check
            if (!this.disabled && (!childNodes || !childNodes.length)) {
                this.$emit("check", !checked, id);
            }
        },
        handleChange(checked, {
            id
        }) {
            this.$emit("check", checked, id);
        },
        getItemClass(item) {
            return item.selected && (item.childNodes && item.childNodes.length) ?
                "is-select" :
                "";
        }
    }
};
</script>

<style lang="scss">
$prefix: "cascader-panel-item";

.#{$prefix} {
    //flex: 1;
    border-right: 1px solid #dddee1;

    &:last-of-type {
        border-right: 0;
    }

    .empty-slot-wrapper {
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -65%);
        color: #c0c4cc;

        img {
            width: 100px;
            height: 66px;
            display: block;
        }

        span {
            display: inline-block;
            margin-top: 4px;
            margin-left: 16px;
        }
    }
}

.#{$prefix}__header {
    font-size: 12px;
    color: #1c2438;
    letter-spacing: 0;
    line-height: 14px;
    height: 32px;
    border-bottom: 1px solid #dddee1;
    line-height: 32px;
    padding-left: 20px;
}

.#{$prefix}__list-item {
    display: flex;
    align-items: center;
    cursor: pointer;
    padding: 10px 20px;
    height: 32px;
    position: relative;

    &.is-select,
    &:hover {
        background: #f5fbff;
        z-index: 9;
    }
}

.#{$prefix}__content {
    position: relative;
    overflow-y: auto;
    height: calc(300px - 34px);
}

.el-checkbox.cascader-panel-item__checkbox {
    margin-right: 10px;
}

.#{$prefix}__text {
    //font-family: MicrosoftYaHei;
    font-size: 12px;
    color: #1c2438;
    letter-spacing: 0;
    // line-height: 12px;
    white-space: nowrap;
    text-overflow: ellipsis;
    overflow: hidden;
    max-width: calc(100% - 36px);
    display: inline-block;
}

.#{$prefix}__textChecked {
    color: #0590ff;
}

.#{$prefix}__toright {
    position: absolute;
    right: 20px;
}
</style>

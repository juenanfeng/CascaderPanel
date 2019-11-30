<template>
<div class="select-panel">
    <div class="select-panel__header">
        <div class="select-panel__title">{{title}}</div>
        <div class="select-panel__clear" @click="handleClearAll">清空全部</div>
    </div>
    <div class="select-panel__content">
        <slot name="empty" v-if="!data.length && type === 'search'">
            <div class="empty-slot-wrapper">
                <!-- <img src="../../assets/images/cas-empty.png"> -->
                <span>无搜索内容</span>
            </div>
        </slot>
        <div class="select-panel__item" v-for="item in data" :key="item.id">
            <el-tooltip
                placement="top"
                :content="item.text"
                effect="light"
                :open-delay="delay">
                <span class="select-panel__item-text">{{item.text}}</span>
            </el-tooltip>
            <span class="select-panel__item-remove" @click="handleRemove(item.id)">
              <i class="el-icon-close"></i>
            </span>
        </div>
    </div>
</div>
</template>

<script>
export default {
    name: "SnSelectPanel",
    data() {
        return {
            delay: 300
        }
    },
    props: {
        data: {
            type: Array,
            default: () => []
        },
        title: {
            type: String,
            default: "已选"
        },
        itemWidth: {
            type: [Number, String],
            default: 140
        },
        disabled: {
            type: Boolean,
            default: false
        },
        type: {
            type: String,
            default: "cas"
        }
    },
    computed: {
        _itemWidth() {
            return this.itemWidth + "px"
        }
    },
    methods: {
        handleClearAll() {
            // if (this.disabled) return;
            this.$emit("clear-all", this.data.map(item => item.id));
        },
        handleRemove(id) {
            //if (this.disabled) return;
            this.$emit("remove", id);
        }
    }
};
</script>

<style lang="scss">
$prefix: "select-panel";

.#{$prefix} {
    background-color: #fff;
    border: 1px solid #dee4f5;
    border-radius: 2px;

    .empty-slot-wrapper {
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -65%);
        color: #C0C4CC;

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
    display: flex;
    justify-content: space-between;
    padding: 8px 20px;
    border-bottom: 1px solid #dddee1;
    height: 32px;

}

.#{$prefix}__title {
    font-size: 12px;
    color: #1c2438;
    letter-spacing: 0;
    line-height: 14px;
}

.#{$prefix}__clear {
    font-size: 12px;
    color: #0590ff;
    letter-spacing: 0;
    line-height: 14px;
    cursor: pointer;
}

.#{$prefix}__content {
    height: 265px;
    overflow-y: auto;
    padding: 0 20px;
    position: relative;
    // display: flex;
    // flex-wrap: wrap;
    // align-content: flex-start;
    // justify-content: space-between;
}

.#{$prefix}__item {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 5px;
    background: #f6f7fb;
    cursor: pointer;
    margin: 2px 0;
    font-size: 12px;
    color: #1c2438;
    height: 26px;
    width: 100%;
}

.#{$prefix}__item-text {
    text-overflow: ellipsis;
    white-space: nowrap;
    overflow: hidden;
    padding-right: 12px;
}

.#{$prefix}__item-remove {
    color: #dddee1;
    font-size: 15px;

    i {
        transition: all .3s;
    }

    &:hover {
        i {
            transform: rotate(180deg);
        }
    }
}
</style>

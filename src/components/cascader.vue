<template>
  <div class="h-cascader" ref="cascader" v-click-outside="close">
    <div class="trigger" @click="toggle">
      {{result || '&nbsp;'}}
    </div>
    <div class="popover-wrapper" v-if="popoverVisible">
      <cascader-items :items="source" class="popover" :loadData="loadData"
        :loading-item="loadingItem"
        :height="popoverHeight" :selected="selected" @update:selected="onUpdateSelected"></cascader-items>
    </div>
  </div>
</template>

<script>
import popover from "./popover.vue";
import CascaderItems from "./cascader-item.vue";
import ClickOutside from "./click-outside.js";
export default {
  name: "hCascader",
  components: { CascaderItems, popover },
  directives: { ClickOutside },
  props: {
    source: {
      type: Array,
    },
    popoverHeight: {
      type: String,
    },
    selected: {
      type: Array,
      default: () => {
        return [];
      },
    },
    loadData: {
      type: Function,
    },
  },
  data() {
    return {
      popoverVisible: false,
      loadingItem: {},
    };
  },
  updated() {},
  methods: {
    open() {
      this.popoverVisible = true;
    },
    close() {
      this.popoverVisible = false;
    },
    toggle() {
      if (this.popoverVisible === true) {
        this.close();
      } else {
        this.open();
      }
    },
    onUpdateSelected(newSelected) {
      this.$emit("update:selected", newSelected);
      let lastItem = newSelected[newSelected.length - 1];
      let simplest = (children, id) => {
        return children.filter((item) => item.id === id)[0];
      };
      let complex = (children, id) => {
        let noChildren = [];
        let hasChildren = [];
        children.forEach((item) => {
          if (item.children) {
            hasChildren.push(item);
          } else {
            noChildren.push(item);
          }
        });
        let found = simplest(noChildren, id);
        if (found) {
          return found;
        } else {
          found = simplest(hasChildren, id);
          if (found) {
            return found;
          } else {
            for (let i = 0; i < hasChildren.length; i++) {
              found = complex(hasChildren[i].children, id);
              if (found) {
                return found;
              }
            }
            return undefined;
          }
        }
      };
      let updateSource = (result) => {
        this.loadingItem = {};
        let copy = JSON.parse(JSON.stringify(this.source));
        let toUpdate = complex(copy, lastItem.id);
        toUpdate.children = result;
        this.$emit("update:source", copy);
      };
      if (!lastItem.isLeaf && this.loadData) {
        this.loadData(lastItem, updateSource); // 回调:把别人传给我的函数调用一下
        // 调回调的时候传一个函数,这个函数理论应该被调用
        this.loadingItem = lastItem;
      }
    },
  },
  computed: {
    result() {
      return this.selected.map((item) => item.name).join("/");
    },
  },
};
</script>

<style lang="scss">
$border-color-hover: #666;
$border-color: #999;
$border-color-light: lighten($border-color, 30%);
$border-radius: 4px;
$box-shadow-color: rgba(0, 0, 0, 0.5);
$button-active-bg: #eee;
$button-bg: white;
$button-height: 32px;
$color: #333;
$font-size: 14px;
$input-height: 32px;
$red: #f1453d;
$grey: #eee;
.box-shadow {
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.15);
}
@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
.h-cascader {
  display: inline-block;
  position: relative;
  .trigger {
    background: white;
    height: $input-height;
    display: inline-flex;
    align-items: center;
    padding: 0 1em;
    min-width: 10em;
    border: 1px solid $border-color;
    border-radius: $border-radius;
  }
  .popover-wrapper {
    position: absolute;
    top: 100%;
    left: 0;
    background: white;
    display: flex;
    margin-top: 8px;
    z-index: 1;
    @extend .box-shadow;
  }
}
</style>

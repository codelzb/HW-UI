<template>
  <div class="h-popover" ref="popover">
    <div ref="contentWrapper" class="h-popover-content-wrapper" v-show="visible" :class="{ [`${position}`]: true }">
      <slot name="content"></slot>
    </div>
    <span ref="trigger" id="hiddenDiv">
      <slot></slot>
    </span>
  </div>
</template>
<script>
(function() {
  var title = document.getElementsByClassName[0]("h-popover-content-wrapper");
  var hiddenDiv = document.getElementById("hiddenDiv");
  var timer = null;

  hiddenDiv.onmouseover = title.onmouseover = function() {
    if (timer) clearTimeout(timer);
    hiddenDiv.style.display = "block";
  };

  hiddenDiv.onmouseout = title.onmouseout = function() {
    timer = setTimeout(function() {
      hiddenDiv.style.display = "none";
    }, 400);
  };
})();
</script>
<script>
export default {
  name: "hPopover",
  props: {
    position: {
      type: String,
      default: "top",
      validator(value) {
        return ["top", "right", "left", "bottom"].indexOf(value) >= 0;
      },
    },
    trigger: {
      type: String,
      default: "click",
      validator(value) {
        return ["click", "hover"].indexOf(value) >= 0;
      },
    },
    openDelay: {
      type: Number,
      default: 0,
    },
    closeDelay: {
      type: Number,
      default: 200,
    },
  },
  data() {
    return { visible: false, timer: null };
  },
  computed: {
    openEvent() {
      if (this.trigger === "click") {
        return "click";
      } else {
        return "mouseenter";
      }
    },
    closeEvent() {
      if (this.trigger === "click") {
        return "click";
      } else {
        return "mouseleave";
      }
    },
  },
  methods: {
    positionContent() {
      let content = this.$refs.contentWrapper;
      document.body.appendChild(content);
      let { top, left, bottom, right } = this.$refs.trigger.getBoundingClientRect();
      let width = right - left;
      let height = bottom - top;
      if (this.position === "top") {
        content.style.top = `${top + window.scrollY}px`;
        content.style.left = `${left + window.scrollX}px`;
      } else if (this.position === "bottom") {
        content.style.top = `${top + height + window.scrollY}px`;
        content.style.left = `${left + window.scrollX}px`;
      } else if (this.position === "left") {
        content.style.top = `${top + height / 2 + window.scrollY}px`;
        content.style.left = `${left + window.scrollX}px`;
      } else if (this.position === "right") {
        content.style.top = `${top + height / 2 + window.scrollY}px`;
        content.style.left = `${left + width + window.scrollX}px`;
      }
    },
    onClickDocument(e) {
      if (this.$refs.popover === e.target || this.$refs.popover.contains(e.target)) {
        return;
      } else if (this.$refs.contentWrapper && (this.$refs.contentWrapper === e.target || this.$refs.contentWrapper.contains(e.target))) {
        return;
      }
      this.close();
    },
    open() {
      if (this.trigger === "click") {
        this.visible = true;
        this.$nextTick(() => {
          this.positionContent();
          document.addEventListener("click", this.onClickDocument);
        });
      } else {
        clearTimeout(this.timer);
        if (this.openDelay) {
          this.timer = setTimeout(() => {
            this.visible = true;
          }, this.openDelay);
        } else {
          this.visible = true;
        }
      }
    },
    onClick(e) {
      if (this.$refs.trigger.contains(e.target)) {
        if (this.visible === true) {
          this.close();
        } else {
          this.open();
        }
      }
    },
    close() {
      if (this.trigger === "click") {
        this.visible = false;
        document.removeEventListener("click", this.onClickDocument);
      } else {
        clearTimeout(this.timer);
        if (this.closeDelay) {
          this.timer = setTimeout(() => {
            this.visible = false;
          }, this.closeDelay);
        } else {
          this.visible = false;
        }
      }
    },
    cleanup() {
      if (this.openDelay || this.closeDelay) {
        clearTimeout(this._timer);
      }
    },
  },
  mounted() {
    let popover = this.$refs.popover;
    if (this.trigger === "click") {
      popover.addEventListener("click", this.onClick);
    } else {
      popover.addEventListener("mouseenter", this.open);
      popover.addEventListener("mouseleave", this.close);
    }
  },
  beforeDestroy() {
    let popover = this.$refs.popover;
    this.cleanup();
    if (this.trigger === "click") {
      popover.removeEventListener("click", this.onClick);
    } else {
      popover.removeEventListener("mouseenter", this.open);
      popover.removeEventListener("mouseleave", this.close);
    }
  },
  deactivated() {
    this.cleanup();
  },
};
</script>
<style lang="scss">
@import "./color.scss";
.h-popover {
  display: inline-block;
  vertical-align: top;
  > span {
    display: inline-block;
    vertical-align: top;
  }
}
.h-popover-content-wrapper {
  border: $borderbase;
  border-color: #dbdfeb;
  box-shadow: 0 2px 12px 0 rgba(0,0,0,0.1);
  border-radius: 4px;
  background: #fdfefe;
  padding: 0.5em 1em;
  // max-width: 20em;
  word-break: break-all;
  position: absolute;
  z-index: 25;
  cursor: default;
  &::before,
  &::after {
    content: "";
    display: block;
    height: 0;
    width: 0;
    border: 6px solid transparent;
    filter: drop-shadow(0 2px 12px rgba(0, 0, 0, 0.03));
    position: absolute;
  }
  &.top {
    transform: translateY(-100%);
    margin-top: -10px;
    &::before {
      top: 100%;
      border-bottom: none;
      border-top-color: #dbdfeb;
      left: 10px;
    }
    &::after {
      top: calc(100% - 1px);
      border-bottom: none;
      border-top-color: #fff;
      left: 10px;
    }
  }
  &.bottom {
    margin-top: 10px;
    &::before {
      bottom: 100%;
      border-top: none;
      border-bottom-color: #dbdfeb;
      left: 10px;
    }
    &::after {
      bottom: calc(100% - 1px);
      border-top: none;
      border-bottom-color: #fff;
      left: 10px;
    }
  }
  &.left {
    transform: translateX(-100%) translateY(-50%);
    margin-left: -10px;
    &::before {
      left: 100%;
      top: 50%;
      transform: translateY(-50%);
      border-right: none;
      border-left-color: #dbdfeb;
    }
    &::after {
      left: calc(100% - 1px);
      top: 50%;
      transform: translateY(-50%);
      border-right: none;
      border-left-color: #fff;
    }
  }
  &.right {
    margin-left: 10px;
    transform: translateY(-50%);
    &::before {
      right: 100%;
      top: 50%;
      transform: translateY(-50%);
      border-left: none;
      border-right-color: #dbdfeb;
    }
    &::after {
      right: calc(100% - 1px);
      top: 50%;
      transform: translateY(-50%);
      border-left: none;
      border-right-color: #fff;
    }
  }
}
</style>

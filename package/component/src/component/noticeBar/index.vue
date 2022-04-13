<template>
  <div
    class="notice-bar"
    :style="{ background, height: `${height}px` }"
    v-show="!isMode"
  >
    <div class="notice-bar-warp" :style="{ color, fontSize: `${size}px` }">
      <lay-icon
        v-if="leftIcon"
        class="notice-bar-warp-left-icon"
        :type="leftIcon"
      ></lay-icon>
      <div class="notice-bar-warp-text-box" ref="noticeBarWarpRef">
        <div
          class="notice-bar-warp-text"
          ref="noticeBarTextRef"
          v-if="!scrollable"
        >
          {{ text }}
        </div>
      </div>
      <lay-icon
        :type="rightIcon"
        v-if="rightIcon"
        class="notice-bar-warp-right-icon"
        @click="onRightIconClick"
      ></lay-icon>
    </div>
  </div>
</template>

<script lang="ts">
import {
  toRefs,
  reactive,
  defineComponent,
  ref,
  onMounted,
  nextTick,
} from "vue";
export default defineComponent({
  name: "LayNoticeBar",
  props: {
    mode: {
      type: String,
      default: () => "",
    },
    text: {
      type: String,
      default: () => "",
    },
    textlist: {
      type: Array,
      default: [],
    },
    // 通知文本颜色
    color: {
      type: String,
      default: () => "var(--color-warning)",
    },
    // 通知背景色
    background: {
      type: String,
      default: () => "var(--color-warning-light-9)",
    },
    // 字体大小，单位px
    size: {
      type: [Number, String],
      default: () => 14,
    },
    // 通知栏高度，单位px
    height: {
      type: Number,
      default: () => 40,
    },
    // 动画延迟时间 (s)
    delay: {
      type: Number,
      default: () => 1,
    },
    // 滚动速率 (px/s)
    speed: {
      type: Number,
      default: () => 100,
    },
    // 是否开启垂直滚动
    scrollable: {
      type: Boolean,
      default: () => false,
    },
    // 自定义左侧图标
    leftIcon: {
      type: String,
      default: () => "",
    },
    // 自定义右侧图标
    rightIcon: {
      type: String,
      default: () => "",
    },
  },
  setup(props, { emit }) {
    const noticeBarWarpRef = ref();
    const noticeBarTextRef = ref();
    const state = reactive({
      order: 1,
      oneTime: 0,
      twoTime: 0,
      warpOWidth: 0,
      textOWidth: 0,
      isMode: false,
    });
    // 初始化 animation 各项参数
    const initAnimation = () => {
      nextTick(() => {
        state.warpOWidth = noticeBarWarpRef.value.offsetWidth;
        state.textOWidth = noticeBarTextRef.value.offsetWidth;
        document.styleSheets[0].insertRule(
          `@keyframes oneAnimation {0% {left: 0px;} 100% {left: -${state.textOWidth}px;}}`
        );
        document.styleSheets[0].insertRule(
          `@keyframes twoAnimation {0% {left: ${state.warpOWidth}px;} 100% {left: -${state.textOWidth}px;}}`
        );
        computeAnimationTime();
        setTimeout(() => {
          changeAnimation();
        }, props.delay * 1000);
      });
    };
    // 计算 animation 滚动时长
    const computeAnimationTime = () => {
      state.oneTime = state.textOWidth / props.speed;
      state.twoTime = (state.textOWidth + state.warpOWidth) / props.speed;
    };
    // 改变 animation 动画调用
    const changeAnimation = () => {
      if (state.order === 1) {
        noticeBarTextRef.value.style.cssText = `animation: oneAnimation ${state.oneTime}s linear; opactity: 1;}`;
        state.order = 2;
      } else {
        noticeBarTextRef.value.style.cssText = `animation: twoAnimation ${state.twoTime}s linear infinite; opacity: 1;`;
      }
    };
    // 监听 animation 动画的结束
    const listenerAnimationend = () => {
      noticeBarTextRef.value.addEventListener(
        "animationend",
        () => {
          changeAnimation();
        },
        false
      );
    };
    // 右侧 icon 图标点击
    const onRightIconClick = () => {
      if (!props.mode) return false;
      if (props.mode === "closeable") {
        state.isMode = true;
        emit("close");
      } else if (props.mode === "link") {
        emit("link");
      }
    };
    // 页面加载时
    onMounted(() => {
      if (props.scrollable) return false;
      initAnimation();
      listenerAnimationend();
    });
    return {
      noticeBarWarpRef,
      noticeBarTextRef,
      onRightIconClick,
      ...toRefs(state),
    };
  },
});
</script>

<style>
.notice-bar {
  padding: 0 15px;
  width: 100%;
  border-radius: 4px;
}
.notice-bar .notice-bar-warp {
  display: flex;
  align-items: center;
  width: 100%;
  height: inherit;
}
.notice-bar .notice-bar-warp .notice-bar-warp-text-box {
  flex: 1;
  height: inherit;
  display: flex;
  align-items: center;
  overflow: hidden;
  position: relative;
}
.notice-bar .notice-bar-warp .notice-bar-warp-text-box .notice-bar-warp-text {
  white-space: nowrap;
  position: absolute;
  left: 0;
}
.notice-bar .notice-bar-warp .notice-bar-warp-text-box .notice-bar-warp-slot {
  width: 100%;
  white-space: nowrap;
}
.notice-bar
  .notice-bar-warp
  .notice-bar-warp-text-box
  .notice-bar-warp-slot
  ::v-deep(.el-carousel__item) {
  display: flex;
  align-items: center;
}
.notice-bar .notice-bar-warp .notice-bar-warp-left-icon {
  width: 24px;
  font-size: inherit !important;
}
.notice-bar .notice-bar-warp .notice-bar-warp-right-icon {
  width: 24px;
  text-align: right;
  font-size: inherit !important;
}
.notice-bar .notice-bar-warp .notice-bar-warp-right-icon:hover {
  cursor: pointer;
}
</style>
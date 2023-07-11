<script lang="ts" setup>
/**
 * 文本
 * @description 自带主题和常用属性，能根据全局主题和暗黑自动切换，必须放在tmSheet下，获得更好的主题适应能力。
 */
import { computed, inject } from 'vue'
import theme from '../../tool/theme/theme'
import {
  computedClass,
  computedDark,
  computedStyle,
  custom_props,
} from '../../tool/lib/minxs'
import type { colorThemeType, tmVuetify } from '../../tool/lib/interface'
import { useTmpiniaStore } from '../../tool/lib/tmpinia'

// 混淆props共有参数
const props = defineProps({
  ...custom_props,
  label: {
    type: [String, Number],
    default: '',
  },
  fontSize: {
    type: [Number],
    default: 28,
  },
  color: {
    type: String,
    default: '',
  },
  selectable: {
    type: [Boolean],
    default: false,
  },
  unit: {
    type: String,
    default: 'rpx',
  },
  parentClass: {
    type: String,
    default: '',
  },
  lineHeight: {
    type: [Number, String],
    default: 'auto',
  },
})
const emits = defineEmits(['click'])
const store = useTmpiniaStore()
const _parentClass = computed(() => props.parentClass)
// 设置响应式全局组件库配置表。
const tmcfg = computed<tmVuetify>(() => store.tmStore)
// 自定义样式：
const customCSSStyle = computed(() => computedStyle(props))
// 自定类
const customClass = computed(() => computedClass(props))
// 是否暗黑模式。
const isDark = computed(() => computedDark(props, tmcfg.value))
// 计算主题
// const tmcomputed = computed(() => computedTheme(props, isDark.value));
const _label = computed(() => props.label)
const _fontSize = computed(() => Number(props.fontSize) * (store?.tmuiConfig?.themeConfig?.globalFontSizeRatio ?? 1))
// 从父应用组件中获取自动文字色。
const appTextColor = inject(
  'appTextColor',
  computed(() => undefined),
)
const textColor = computed(() => {
  if (props.followTheme && store.tmStore.color)
    return store.tmStore.color
  const isColorHex = theme.isCssColor(props.color)
  // 如果当前是自定义颜色值，直接返回。
  if (isColorHex)
    return props.color

  // 如果定义了颜色，但不是值，而是主题色，则返回对应的主题文本色。
  if (props.color && !isColorHex) {
    const nowcolor: colorThemeType = theme.getColor(props.color)
    return nowcolor.csscolor
  }
  if (!appTextColor) {
    if (isDark.value)
      return 'rgba(252, 252, 252, 1.0)'
    return 'rgba(34, 34, 34, 1.0)'
  }
  if (appTextColor.value)
    return appTextColor.value

  return 'rgba(34, 34, 34, 1.0)'
})
</script>

<template>
  <!-- #ifdef APP-PLUS-NVUE -->
  <text
    :render-whole="true" :selectable="selectable" :user-select="selectable" :class="[_fontSize ? '' : 'text-size-m', customClass]"
    :style="[
      props.lineHeight == 'auto'
        ? { lineHeight: (_fontSize ? _fontSize * 1.3 : 42) + props.unit }
        : {},
      props.lineHeight > 0 ? { lineHeight: props.lineHeight + props.unit } : {},
      {
        color: textColor,
      },
      _fontSize ? { fontSize: _fontSize + props.unit } : '',
      customCSSStyle,
    ]" @click="emits('click', $event)"
  >
    {{ _label }}
  </text>
  <!-- #endif -->
  <!-- #ifndef APP-PLUS-NVUE -->

  <view :render-whole="true" class="flex text-view nv" :class="[_parentClass]">
    <text
      :selectable="selectable" :user-select="selectable" :class="[fontSize ? '' : 'text-size-m', customClass]"
      :style="[
        props.lineHeight == 'auto'
          ? { lineHeight: (_fontSize ? _fontSize * 1.3 : 42) + props.unit }
          : {},
        props.lineHeight > 0 ? { lineHeight: props.lineHeight + props.unit } : {},
        {
          color: textColor,
        },
        _fontSize ? { fontSize: _fontSize + props.unit } : '',
        customCSSStyle,
      ]" @click="emits('click', $event)"
    >
      <slot>{{ _label }}</slot>
    </text>
  </view>
  <!-- #endif -->
</template>

<style scoped>
/* #ifndef APP-NVUE */
.text-view {
  box-sizing: border-box;
}

.nv {
  display: inline-block;
  vertical-align: middle;
}

/* #endif */
</style>

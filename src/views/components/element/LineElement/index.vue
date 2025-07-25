<template>
  <div 
    class="editable-element-shape"
    :class="{ 'lock': elementInfo.lock }"
    :style="{
      top: elementInfo.top + 'px',
      left: elementInfo.left + 'px',
    }"
  >
    <div 
      class="element-content" 
      :style="{ filter: shadowStyle ? `drop-shadow(${shadowStyle})` : '' }"
      @mousedown="$event => handleSelectElement($event)"
      @touchstart.passive="$event => handleSelectElement($event)"
    >
      <svg
        overflow="visible" 
        :width="svgWidth"
        :height="svgHeight"
      >
        <defs>
          <LinePointMarker
            v-if="elementInfo.points[0]"
            :id="elementInfo.id"
            position="start"
            :type="elementInfo.points[0]"
            :color="elementInfo.color"
            :baseSize="elementInfo.width"
          />
          <LinePointMarker
            v-if="elementInfo.points[1]"
            :id="elementInfo.id"
            position="end"
            :type="elementInfo.points[1]"
            :color="elementInfo.color"
            :baseSize="elementInfo.width"
          />
        </defs>
				<path
          class="line-point"
          :d="path" 
          :stroke="elementInfo.color" 
          :stroke-width="elementInfo.width" 
          :stroke-dasharray="lineDashArray"
          fill="none" 
          :marker-start="elementInfo.points[0] ? `url(#${elementInfo.id}-${elementInfo.points[0]}-start)` : ''"
          :marker-end="elementInfo.points[1] ? `url(#${elementInfo.id}-${elementInfo.points[1]}-end)` : ''"
        ></path>
				<path
          class="line-path"
          :d="path" 
          stroke="transparent" 
          stroke-width="20" 
          fill="none" 
          v-contextmenu="contextmenus"
        ></path>
			</svg>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { computed } from 'vue'
import type { PPTLineElement } from '@/types/slides'
import { getLineElementPath } from '@/utils/element'
import type { ContextmenuItem } from '@/components/Contextmenu/types'
import useElementShadow from '@/views/components/element/hooks/useElementShadow'

import LinePointMarker from './LinePointMarker.vue'

const props = defineProps<{
  elementInfo: PPTLineElement
  selectElement: (e: MouseEvent | TouchEvent, element: PPTLineElement, canMove?: boolean) => void
  contextmenus: () => ContextmenuItem[] | null
}>()

const handleSelectElement = (e: MouseEvent | TouchEvent) => {
  if (props.elementInfo.lock) return
  e.stopPropagation()

  props.selectElement(e, props.elementInfo)
}

const shadow = computed(() => props.elementInfo.shadow)
const { shadowStyle } = useElementShadow(shadow)

const svgWidth = computed(() => {
  const width = Math.abs(props.elementInfo.start[0] - props.elementInfo.end[0])
  return width < 24 ? 24 : width
})
const svgHeight = computed(() => {
  const height = Math.abs(props.elementInfo.start[1] - props.elementInfo.end[1])
  return height < 24 ? 24 : height
})

const lineDashArray = computed(() => {
  const size = props.elementInfo.width
  if (props.elementInfo.style === 'dashed') return size <= 8 ? `${size * 5} ${size * 2.5}` : `${size * 5} ${size * 1.5}`
  if (props.elementInfo.style === 'dotted') return size <= 8 ? `${size * 1.8} ${size * 1.6}` : `${size * 1.5} ${size * 1.2}`
  return '0 0'
})

const path = computed(() => {
  return getLineElementPath(props.elementInfo)
})
</script>

<style lang="scss" scoped>
.editable-element-shape {
  position: absolute;
  pointer-events: none;

  &.lock {
    .line-path, .line-point {
      cursor: default;
    }
  }
}

.element-content {
  width: 100%;
  height: 100%;
  position: relative;

  svg {
    transform-origin: 0 0;
    overflow: visible;
  }
}
.line-path, .line-point {
  pointer-events: all;
  cursor: move;
}
</style>

<template>
  <div class="seamless-scroll">
    <div ref="wrapperRef" class="seamless-scroll__wrapper" @mouseover="onMouseOver" @mouseout="onMouseOut">
      <div
        ref="boxRef"
        :class="{ 'seamless-scroll__box': true, 'seamless-scroll__box--odd': isOddChildNodes, 'seamless-scroll__box--even': !isOddChildNodes }">
        <div class="seamless-scroll__box-top" ref="topRef">
          <slot></slot>
        </div>
        <div v-if="showShadowDiv" class="seamless-scroll__box-bottom">
          <slot></slot>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, nextTick, onMounted, ref, watchEffect } from 'vue'
import { useElementSize, useMutationObserver } from '@vueuse/core'
import gsap from 'gsap'

const props = defineProps({
  /**
   * 滑动单位距离需要的时间(s)
   */
  duration: {
    type: Number,
    default: 2
  },
  /**
  * 两次滑动之间的停顿时长(s)
  */
  delay: {
    type: Number,
    default: 2
  }
})

const wrapperRef = ref(null)
const boxRef = ref(null)
const topRef = ref()
const { height: wrapperHeight } = useElementSize(wrapperRef)
const { height: boxHeight } = useElementSize(boxRef)
const { height: topBoxHeight } = useElementSize(topRef)

const timeLine = gsap.timeline()
let scrollingElIndex = 0

const genAnimates = () => {
  if (!topRef.value) {
    return
  }

  const nodeList = topRef.value.childNodes
  const nodeArr = Array.from(nodeList.values()).filter(t => t.nodeType === Node.ELEMENT_NODE)
  const currentScrollingEl = nodeArr[scrollingElIndex]
  scrollingElIndex = (scrollingElIndex + 1) % nodeArr.length
  if (!currentScrollingEl) return

  let rect = currentScrollingEl.getBoundingClientRect()
  const elHeight = rect.height
  const offsetTop = currentScrollingEl.offsetTop
  const scrollTarget = offsetTop + elHeight
  const scrollAreaHeight = topBoxHeight.value
  const willScrollToEdge = scrollTarget >= scrollAreaHeight
  if (!willScrollToEdge) {
    timeLine.to(wrapperRef.value, { scrollTop: scrollTarget, duration: props.duration }, `+=${props.delay}`)
    return
  }
  timeLine.to(wrapperRef.value, { scrollTop: scrollAreaHeight, duration: props.duration }, `+=${props.delay}`)
}

const update = () => {
  timeLine.eventCallback('onComplete', () => {
    if (scrollingElIndex === 0) {
      gsap.to(wrapperRef.value, {
        scrollTop: 0,
        duration: 0,
        onComplete: () => {
          genAnimates()
        }
      })
      return
    }
    genAnimates()
  })
  genAnimates()
}

const showShadowDiv = computed(() => {
  if (!topBoxHeight.value) return false

  if (!wrapperHeight.value) return false

  if (!boxHeight.value) return false

  return boxHeight.value > wrapperHeight.value
})

watchEffect(() => {
  if (showShadowDiv.value) {
    nextTick(() => update())
  }
})

const onMouseOver = () => {
  timeLine.pause()
}

const onMouseOut = () => {
  timeLine.resume
}

const childNodesCount = ref(0)
const isOddChildNodes = computed(() => childNodesCount.value % 2 !== 0)

const refreshChildNodesCount = () => {
  if (!topRef.value) return

  const nodeList = topRef.value.childNodes
  const nodeArr = Array.from(nodeList.values()).filter(t => t.nodeType === Node.ELEMENT_NODE)
  childNodesCount.value = nodeArr.length
}

// 初始化以及增加元素后
useMutationObserver(topRef, (mutations) => {
  refreshChildNodesCount()
}, {
  childList: true
})

onMounted(() => {
  refreshChildNodesCount()
})

</script>

<style lang="scss">
.seamless-scroll {
  &__wrapper {
    width: 100%;
    height: 100%;
    position: relative;
    overflow: hidden;
  }
  &__box {
    padding: 0;
    &-top,
    &-bottom {
      overflow: hidden;
      &>* {
        background-size: 100% 100%;
      }
    }
  }
}
</style>
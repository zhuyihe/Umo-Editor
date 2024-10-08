<template>
  <node-view-wrapper
    ref="containerRef"
    class="umo-node-view"
    :id="node.attrs.id"
    :style="nodeStyle"
  >
    <div
      class="umo-node-container umo-hover-shadow umo-select-outline umo-node-audio"
    >
      <audio ref="audiorRef" :src="node.attrs.src" controls crossorigin></audio>
      <div
        v-if="!node.attrs.uploaded && node.attrs.file !== null"
        class="uploading"
      ></div>
    </div>
  </node-view-wrapper>
</template>

<script setup>
import { nodeViewProps, NodeViewWrapper } from '@tiptap/vue-3'
import { mediaPlayer } from '@/utils/player'

const { node, updateAttributes } = defineProps(nodeViewProps)
const { options } = useStore()

const containerRef = ref(null)
const audiorRef = $ref(null)
let player = $ref(null)
let selected = $ref(false)

const nodeStyle = $computed(() => {
  const { nodeAlign, margin } = node.attrs
  const marginTop =
    margin?.top && margin?.top !== '' ? margin.top + 'px' : undefined
  const marginBottom =
    margin?.bottom && margin?.bottom !== '' ? margin.bottom + 'px' : undefined
  return {
    'justify-content': nodeAlign,
    marginTop,
    marginBottom,
  }
})

onMounted(async () => {
  player = mediaPlayer(audiorRef)
  if (node.attrs.uploaded === false && node.attrs.file) {
    try {
      const { url } = await options.value.onFileUpload(node.attrs.file)
      if (containerRef.value) {
        updateAttributes({ src: url, file: null, uploaded: true })
      }
    } catch (error) {
      useMessage('error', error.message)
    }
  }
})

onBeforeUnmount(() => {
  if (player) {
    player?.destroy()
  }
})

onClickOutside(containerRef, () => (selected = false))
</script>

<style lang="less">
.umo-node-view {
  .umo-node-audio {
    max-width: 100%;
    width: 360px;
    position: relative;
    display: flex;
    border-radius: var(--umo-radius);
    outline: solid 1px var(--umo-border-color);
    audio {
      width: 100%;
      outline: none;
    }
    .uploading {
      position: absolute;
      z-index: 10;
      right: 0;
      top: 0;
      background: rgba(0, 0, 0, 0.2);
      height: 2px;
      top: 0;
      left: 0;
      right: 0;
      border-top-left-radius: var(--umo-radius);
      border-top-right-radius: var(--umo-radius);
      &:after {
        content: '';
        display: block;
        height: 100%;
        background-color: var(--umo-primary-color);
        animation: progress 1s linear infinite;
      }
    }
  }
}
@keyframes progress {
  0% {
    width: 0;
  }
  100% {
    width: 100%;
  }
}
</style>

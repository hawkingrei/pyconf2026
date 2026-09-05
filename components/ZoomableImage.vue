<script setup lang="ts">
import { ref, watch, onUnmounted } from 'vue'

withDefaults(
  defineProps<{
    src: string
    alt?: string
    /** Thumbnail max-height ~170px (also covered by `.temporal-shots img` when nested there) */
    compact?: boolean
    /** Classes on the inner wrapper around the thumbnail img */
    shotClass?: string
    /** Classes on the img (e.g. w-full, w-[140%]) */
    imgClass?: string
    /** Inline style on the thumbnail img */
    imgStyle?: string
    /** Extra classes on the root (layout wrappers like cognitive-slide__diagram-inner) */
    rootClass?: string
  }>(),
  {
    compact: false,
    shotClass: 'deck-zoomable__shot',
  },
)

const open = ref(false)

function close() {
  open.value = false
}

function onKey(e: KeyboardEvent) {
  if (e.key === 'Escape')
    close()
}

watch(open, (v) => {
  if (v) {
    document.addEventListener('keydown', onKey)
    document.body.style.overflow = 'hidden'
  } else {
    document.removeEventListener('keydown', onKey)
    document.body.style.overflow = ''
  }
})

onUnmounted(() => {
  document.removeEventListener('keydown', onKey)
  document.body.style.overflow = ''
})
</script>

<template>
  <div
    class="deck-zoomable"
    :class="[{ 'deck-zoomable--compact': compact }, rootClass]"
  >
    <button
      type="button"
      class="deck-zoomable__hit"
      :aria-label="`放大查看${alt ? `：${alt}` : ''}`"
      @click.stop="open = true"
    >
      <div :class="shotClass">
        <img
          :src="src"
          :alt="alt"
          :class="imgClass"
          :style="imgStyle"
          loading="lazy"
          decoding="async"
        />
      </div>
    </button>
    <Teleport to="body">
      <Transition name="deck-zoom">
        <div
          v-if="open"
          class="deck-zoomable__overlay"
          role="dialog"
          aria-modal="true"
          aria-label="大图预览"
          @click.self="close"
        >
          <button type="button" class="deck-zoomable__close" aria-label="关闭" @click="close">
            ×
          </button>
          <img :src="src" :alt="alt" class="deck-zoomable__full" @click.stop />
        </div>
      </Transition>
    </Teleport>
  </div>
</template>

<style scoped>
.deck-zoomable {
  position: relative;
  min-width: 0;
}
.deck-zoomable__hit {
  all: unset;
  display: block;
  width: 100%;
  cursor: zoom-in;
  position: relative;
  border-radius: 6px;
}
.deck-zoomable__shot {
  display: block;
  width: 100%;
  min-width: 0;
}
.deck-zoomable__shot img {
  display: block;
  width: 100%;
  height: auto;
  object-fit: contain;
  border-radius: 6px;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}
.deck-zoomable__hit:hover .deck-zoomable__shot img {
  box-shadow: 0 6px 28px rgba(28, 25, 23, 0.12);
}
.deck-zoomable--compact .deck-zoomable__shot img {
  max-height: 170px;
}
.deck-zoomable__overlay {
  position: fixed;
  inset: 0;
  z-index: 100000;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: min(4vmin, 2rem);
  background: rgba(28, 25, 23, 0.78);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  cursor: zoom-out;
}
.deck-zoomable__close {
  position: absolute;
  top: 1rem;
  right: 1.1rem;
  width: 2.25rem;
  height: 2.25rem;
  border: none;
  border-radius: 9999px;
  background: rgba(255, 255, 255, 0.12);
  color: rgba(255, 255, 255, 0.92);
  font-size: 1.35rem;
  line-height: 1;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.15s ease;
}
.deck-zoomable__close:hover {
  background: rgba(255, 255, 255, 0.22);
}
.deck-zoomable__full {
  max-width: min(92vw, 1400px);
  max-height: 88vh;
  width: auto;
  height: auto;
  object-fit: contain;
  border-radius: 10px;
  box-shadow: 0 28px 90px rgba(0, 0, 0, 0.45);
  cursor: default;
}
.deck-zoom-enter-active,
.deck-zoom-leave-active {
  transition: opacity 0.22s ease;
}
.deck-zoom-enter-from,
.deck-zoom-leave-to {
  opacity: 0;
}
</style>

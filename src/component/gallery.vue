<template>
  <div
    :id="id"
    class="blueimp-gallery"
    aria-label="image gallery"
    aria-modal="true"
    role="dialog"
  >
    <div
      class="slides"
      aria-live="polite"
    />
    <h3
      class="title"
    />
    <a
      class="prev"
      aria-controls="blueimp-gallery"
      aria-label="previous slide"
      aria-keyshortcuts="ArrowLeft"
    >
      <slot name="prev">‹</slot>
    </a>
    <a
      class="next"
      aria-controls="blueimp-gallery"
      aria-label="next slide"
      aria-keyshortcuts="ArrowRight"
    >
      <slot name="next">›</slot>
    </a>
    <a
      class="close"
      aria-controls="blueimp-gallery"
      aria-label="close"
      aria-keyshortcuts="Escape"
    >
      <slot name="close">×</slot>
    </a>
    <a
      class="play-pause"
      aria-controls="blueimp-gallery"
      aria-label="play slideshow"
      aria-keyshortcuts="Space"
      aria-pressed="false"
      role="button"
    />
    <ol
      class="indicator"
    />
  </div>
</template>

<script>
import 'blueimp-gallery/css/blueimp-gallery.min.css'
import 'blueimp-gallery/js/blueimp-gallery-fullscreen.js'
import 'blueimp-gallery/js/blueimp-gallery-video.js'
import 'blueimp-gallery/js/blueimp-gallery-youtube.js'
import blueimp from 'blueimp-gallery/js/blueimp-gallery.js'
export default {
  props: {
    id: {
      type: String,
      default: 'blueimp-gallery'
    },
    images: {
      type: Array,
      default () {
        return []
      }
    },
    options: {
      type: Object,
      default () {
        return {}
      }
    },
    index: {
      type: Number,
      default: null
    }
  },
  data () {
    return {
      instance: null
    }
  },
  watch: {
    index (value) {
      if (value !== null) {
        this.open(value)
      } else {
        if (this.instance) {
          this.instance.close()
        }
        this.$emit('close')
      }
    }
  },
  destroyed () {
    if (this.instance !== null) {
      this.instance.destroyEventListeners()
      this.instance.close()
      this.instance = null
    }
  },
  methods: {
    open (index = 0) {
      const instance =
        typeof blueimp.Gallery !== 'undefined' ? blueimp.Gallery : blueimp
      const options = Object.assign(
        {
          container: `#${this.id}`,
          index,
          onopen: () => this.$emit('onopen'),
          onopened: () => this.$emit('onopened'),
          onslide: this.onSlideCustom,
          onslideend: (index, slide) =>
            this.$emit('onslideend', { index, slide }),
          onslidecomplete: (index, slide) =>
            this.$emit('onslidecomplete', { index, slide }),
          onclose: () => this.$emit('close'),
          onclosed: () => this.$emit('onclosed')
        },
        this.options
      )
      this.instance = instance(this.images, options)
    },
    onSlideCustom (index, slide) {
      this.$emit('onslide', { index, slide })
      const image = this.images[index]
      if (image !== undefined) {
        const text = image.description
        const node = this.instance.container.find('.description')
        if (text) {
          node.empty()
          node[0].appendChild(document.createTextNode(text))
        }
      }
    }
  }
}
</script>

<style scoped>
.blueimp-gallery > .blueimp-gallery-single > .next, .blueimp-gallery > .blueimp-gallery-single > .prev {
  display: none!important;
}
</style>

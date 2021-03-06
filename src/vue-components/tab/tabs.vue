<template>
  <div class="quasar-tabs row">
    <div v-el:left-scroll class="row items-center justify-center left-scroll">
      <i>chevron_left</i>
    </div>
    <div v-el:scroller class="quasar-tabs-scroller row">
      <slot></slot>
    </div>
    <div v-el:right-scroll class="row items-center justify-center right-scroll">
      <i>chevron_right</i>
    </div>
  </div>
</template>

<script>
import Platform from '../../platform'
import Utils from '../../utils'

const
  scrollNavigationSpeed = 5, // in pixels
  debounceDelay = 50 // in ms

export default {
  methods: {
    redraw () {
      if (!Platform.is.desktop) {
        return
      }
      if (Utils.dom.width(this.$els.scroller) === 0 && this.$els.scroller.scrollWidth === 0) {
        return
      }
      if (Utils.dom.width(this.$els.scroller) + 5 < this.$els.scroller.scrollWidth) {
        this.$el.classList.add('scrollable')
        this.scrollable = true
        this.updateScrollIndicator()
      }
      else {
        this.$el.classList.remove('scrollable')
        this.scrollable = false
      }
    },
    updateScrollIndicator () {
      if (!Platform.is.desktop || !this.scrollable) {
        return
      }

      let action = this.$els.scroller.scrollLeft + Utils.dom.width(this.$els.scroller) + 5 >= this.$els.scroller.scrollWidth ? 'add' : 'remove'

      this.$els.leftScroll.classList[this.$els.scroller.scrollLeft <= 0 ? 'add' : 'remove']('disabled')
      this.$els.rightScroll.classList[action]('disabled')
    },
    scrollToSelectedIfNeeded (tab) {
      if (tab.length === 0 || !this.scrollable) {
        return
      }

      let
        contentRect = this.$els.scroller.getBoundingClientRect(),
        tabRect = tab.getBoundingClientRect(),
        tabWidth = tabRect.width,
        offset = tabRect.left - contentRect.left

      if (offset < 0) {
        this.animScrollTo(this.$els.scroller.scrollLeft + offset)
      }
      else {
        offset += tabWidth - this.$els.scroller.offsetWidth
        /* istanbul ignore else */
        if (offset > 0) {
          this.animScrollTo(this.$els.scroller.scrollLeft + offset)
        }
      }
    },
    animScrollTo (value) {
      if (this.scrollTimer) {
        clearInterval(this.scrollTimer)
      }

      this.scrollTowards(value)
      this.scrollTimer = setInterval(() => {
        if (this.scrollTowards(value)) {
          clearInterval(this.scrollTimer)
        }
      }, 5)
    },
    scrollTowards (value) {
      let
        scrollPosition = this.$els.scroller.scrollLeft,
        direction = value < scrollPosition ? -1 : 1,
        done = false

      scrollPosition += direction * scrollNavigationSpeed

      if (scrollPosition < 0) {
        done = true
        scrollPosition = 0
      }
      else if (direction === -1 && scrollPosition <= value || direction === 1 && scrollPosition >= value) {
        done = true
        scrollPosition = value
      }

      this.$els.scroller.scrollLeft = scrollPosition
      return done
    }
  },
  events: {
    selected (tab) {
      this.$broadcast('tabSelected', tab)

      setTimeout(() => {
        this.scrollToSelectedIfNeeded(tab.$el)
      }, debounceDelay * 4)
    },
    hidden () {
      this.redraw()
    }
  },
  watch: {
    '$children' () {
      this.redraw()
    }
  },
  ready () {
    this.scrollTimer = null
    this.scrollable = !Platform.is.desktop

    // debounce some costly methods;
    // debouncing here because debounce needs to be per instance
    this.redraw = Utils.debounce(this.redraw, debounceDelay)
    this.updateScrollIndicator = Utils.debounce(this.updateScrollIndicator, debounceDelay)

    this.$els.scroller.addEventListener('scroll', this.updateScrollIndicator)
    window.addEventListener('resize', this.redraw)

    // let browser drawing stabilize then
    setTimeout(() => { this.redraw() }, debounceDelay)

    if (Platform.is.desktop) {
      var scrollEvents = {
        start: [],
        stop: []
      }

      scrollEvents.start.push('mousedown')
      scrollEvents.stop.push('mouseup')

      if (Platform.has.touch) {
        scrollEvents.start.push('touchstart')
        scrollEvents.stop.push('touchend')
      }

      scrollEvents.start.forEach(evt => {
        this.$els.leftScroll.addEventListener(evt, () => {
          this.animScrollTo(0)
        })
        this.$els.rightScroll.addEventListener(evt, () => {
          this.animScrollTo(9999)
        })
      })
      scrollEvents.stop.forEach(evt => {
        this.$els.leftScroll.addEventListener(evt, () => {
          clearInterval(this.scrollTimer)
        })
        this.$els.rightScroll.addEventListener(evt, () => {
          clearInterval(this.scrollTimer)
        })
      })
    }
  },
  beforeDestroy () {
    if (this.scrollTimer) {
      clearInterval(this.scrollTimer)
    }
    this.$els.scroller.removeEventListener('scroll', this.updateScrollIndicator)
    window.removeEventListener('resize', this.redraw)
  }
}
</script>

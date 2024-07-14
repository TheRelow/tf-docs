<!-- Отображение всплывающего окна, крепящегося к какому-либо элементу на странице

Пример использования:
<t-btn ref="selectControl" label="TPopup"/>
<t-popup
  :target="() => $refs.selectControl"
  :offset="10"
  width="max-content"
  position="right top"
>
  popup content
</t-popup>
-->

<template>
  <teleport :to="assignTo">
    <transition name="fade">
      <div
          v-if="isOpened"
          :style="popupStyle"
          :class="[
          't-popup',
          `t-popup_variant-${variant}`
        ]"
          :id="`t-popup_${uid}`"
      >
        <div v-if="$slots.header" class="t-popup__header">
          <slot name="header"/>
        </div>
        <div class="t-popup__content">
          <slot>
            {{ content }}
          </slot>
        </div>
        <div v-if="this.$slots.footer" class="t-popup__footer">
          <slot name="footer"/>
        </div>
      </div>
    </transition>
  </teleport>
</template>

<script>
export default {
  name: "TPopup",
  components: {
    Teleport
  },
  props: {
    variant: {
      type: String,
      default: "default" // "default" | "dark"
    },
    // Query-селектор элемента или DOM элемент для которого осуществляется монтирование.
    // Если передаётся ref - его нужно прокидывать анонимной функцией | :target="() => $refs.target"
    target: {
      type: [Function, String],
      default: null
    },
    // Query-селектор элемента или DOM элемент для привязки (tagName или #id или .className)
    assignTo: {
      type: String,
      default: "body"
    },
    // состояние открыт/закрыт для управления извне
    value: {
      type: [Boolean, null],
      default: null
    },
    // Принудительное открытие с нужной стороны (top, left, bottom, right, center и их комбинации)
    position: {
      type: String,
      default: "bottom"
    },
    // Ширина выпадающего блока в px, %, vh, vw и auto - равна родителю
    width: {
      type: [String, null],
      default: "auto"
    },
    // Минимальная ширина выпадающего блока подойдут любые css значения подходящие для min-width
    minWidth: {
      type: [String, null],
      default: null
    },
    // Максимальная ширина выпадающего блока подойдут любые css значения подходящие для max-width
    maxWidth: {
      type: [String, null],
      default: null
    },
    // Высота выпадающего блока в px или vh, auto - равна родителю
    height: {
      type: [String, null],
      default: null
    },
    // Минимальная высота выпадающего блока подойдут любые css значения подходящие для min-height
    minHeight: {
      type: [String, null],
      default: null
    },
    // Максимальная высота выпадающего блока подойдут любые css значения подходящие для max-height
    maxHeight: {
      type: [String, null],
      default: null
    },
    // Отступ попапа от target
    offset: {
      type: Number,
      default: 0
    },
    // Отступ попапа от границ экрана
    screenOffset: {
      type: [Array, null],
      default: () => [60, 20, 20, 20]
    },
    // Закрывать ли при клике вне popup и открытии другого popup
    autoClose: {
      type: Boolean,
      default: true
    },
    // каким действием открывается попап
    openBy: {
      type: String,
      default: "click" // "click" | "contextmenu" | "hover" | "hover target"
    },
    // Попап следует за курсором(работает только с openBy = hover)
    followCursor: {
      type: Boolean,
      default: false
    },
    // Текст тултипа. Отображается если не передан дефолтный слот
    content: {
      type: [String, Number, null],
      default: null
    }
  },
  data() {
    return {
      binding: null,
      alignment: null,
      mouseX: null,
      mouseY: null,
      screenHeight: null,
      screenWidth: null,
      isOpenedState: false,
      targetElement: null,
      targetResizeObserver: null,
      targetBox: null,
      isTargetHovered: false,
      popupElement: null,
      popupResizeObserver: null,
      popupBox: null,
      isPopupHovered: false,
      afterMouseLiveTimeout: null,
      isHovered: {
        target: false,
        popup: false
      }
    }
  },
  computed: {
    isOpened() {
      return this.value ?? this.isOpenedState
    },
    uid() {
      return this._uid
    },
    anchor() {
      if (this.followCursor && this.mouseX && this.mouseY) {
        return { x: this.mouseX, y: this.mouseY }
      }

      if (!this.targetBox) {
        return null
      }

      let x = this.targetBox.left
      let y = this.targetBox.top
      switch (this.binding) {
        case "top":
        case "bottom":
          x += this.alignment === "left" ? 0 : this.alignment === "right" ? this.targetBox.width : this.targetBox.width / 2
          y += this.binding === "top" ? 0 : this.targetBox.height
          break
        case "left":
        case "right":
          x += this.binding === "left" ? 0 : this.targetBox.width
          y += this.alignment === "top" ? 0 : this.alignment === "bottom" ? this.targetBox.height : this.targetBox.height / 2
          break
        default: // "center"
          x += this.targetBox.width / 2
          y += this.alignment === "top" ? 0 : this.alignment === "bottom" ? this.targetBox.height : this.targetBox.height / 2
          break
      }

      return {x, y}
    },
    popupStyle() {
      if (!this.anchor) {
        return null
      }

      const { anchor, screenOffset, screenHeight, screenWidth, binding, alignment, offset } = this

      const elWidth = this.getSize("width")
      const elHeight = this.getSize("height")

      const minTop = screenOffset[0]
      const maxTop = screenHeight - screenOffset[2]
      const minRight = screenOffset[1]
      const maxRight = screenWidth - screenOffset[3]
      const minBottom = screenOffset[2]
      const maxBottom = screenHeight - screenOffset[0]
      const minLeft = screenOffset[3]
      const maxLeft = screenWidth - screenOffset[1]

      let top, right, bottom, left

      const clamp = (value, min, max) => value < min ? min : value > max ? max : value

      switch (binding) {
        case "top":
          bottom = clamp(screenHeight - anchor.y + offset, minBottom, maxBottom - elHeight)
          break
        case "right":
          left = clamp(anchor.x + offset, minLeft, maxLeft - elWidth)
          break
        case "bottom":
          top = clamp(anchor.y + offset, minTop, maxTop - elHeight)
          break
        case "left":
          right = clamp(screenWidth - anchor.x + offset, minRight, maxRight - elWidth)
          break
        default: // center
          left = clamp(anchor.x - elWidth / 2, minLeft, maxLeft - elWidth)
          break
      }
      switch (alignment) {
        case "top":
          top = clamp(anchor.y, minTop, maxTop - elHeight)
          break
        case "right":
          right = clamp(screenWidth - anchor.x, minRight, maxRight - elWidth)
          break
        case "bottom":
          bottom = clamp(screenHeight - anchor.y, minBottom, maxBottom - elHeight)
          break
        case "left":
          left = clamp(anchor.x, minLeft, maxLeft - elWidth)
          break
        default: // center
          if (left !== undefined || right !== undefined) {
            top = clamp(anchor.y - elHeight / 2, minTop, maxTop - elHeight)
            break
          }
          left = clamp(anchor.x - elWidth / 2, minLeft, maxLeft - elWidth)
          break
      }

      return {
        top: typeof top === "number" ? `${top}px` : null,
        right: typeof right === "number" ? `${right}px` : null,
        bottom: typeof bottom === "number" ? `${bottom}px` : null,
        left: typeof left === "number" ? `${left}px` : null,
        // Если высота или ширина не переданы в пропсах - размер блока автоматический, а elWidth и elHeight хранят текущий размер
        width: elWidth && this.width ? `${elWidth}px` : null,
        height: elHeight && this.height ? `${elHeight}px` : null,
        maxWidth: this.maxWidth,
        minWidth: this.minWidth,
        maxHeight: this.maxHeight,
        minHeight: this.minHeight,
        pointerEvents: this.followCursor ? "none" : null
      }
    }
  },
  watch: {
    target() {
      this.setTarget()
    },
    openBy() {
      this.removeTargetOpenByHandlers(this.targetElement)
      this.addTargetOpenByHandlers()
      this.addPopupHoverHandler()
    },
    targetElement(val, oldVal) {
      this.removeTargetOpenByHandlers(oldVal)
      this.addTargetOpenByHandlers()
      this.setTargetBox()
    },
    popupElement() {
      this.checkPopupSizeWatcher()
      this.addPopupHoverHandler()
    },
    isHovered: {
      deep: true,
      handler(val) {
        clearTimeout(this.afterMouseLiveTimeout)
        if (!val.popup && !val.target) {
          this.afterMouseLiveTimeout = setTimeout(() => {
            window.removeEventListener("mousemove", this.mouseMoveHandler)
            this.isOpenedState = false
          }, 50)
          return
        }
        if (this.followCursor) {
          document.addEventListener("mousemove", this.mouseMoveHandler)
        }
        this.isOpenedState = true
      }
    },
    position: {
      immediate: true,
      handler(val) {
        const positionValues = val.split(" ")
        this.binding = positionValues[0]
        this.alignment = positionValues[1] || "center"
      }
    },
    value: {
      immediate: true,
      handler(val) {
        this.isOpenedState = val
      }
    },
    isOpenedState(val) {
      // TODO: fix with open by default
      this.$emit("input", val)
    },
    isOpened(val) {
      this.$emit("toggle", val)
      this.$emit(val ? "open" : "close")

      if (val) {
        this.$nextTick(() => {
          this.popupElement = document.getElementById(`t-popup_${this.uid}`)
          document.addEventListener("click", this.onWindowClick)
        })
        this.startWatching()
        return
      }
      this.popupElement = null
      window.removeEventListener("click", this.onWindowClick)
      this.stopWatching()
    },
    variant() {
      this.$nextTick(this.setPopupBox)
    }
  },
  mounted() {
    this.$nextTick(this.setTarget)
    this.screenWidth = document.documentElement.clientWidth
    this.screenHeight = document.documentElement.clientHeight
  },
  beforeDestroy() {
    this.removeTargetOpenByHandlers(this.targetElement)
    window.removeEventListener("click", this.onWindowClick)
    window.removeEventListener("mousemove", this.mouseMoveHandler)
    this.stopWatching()
  },
  methods: {
    setTarget() {
      const result = typeof this.target === "function" ? this.target() : document.querySelector(this.target)
      this.targetElement = result?.$el ?? result ?? null
      if (!this.targetElement instanceof HTMLElement) {
        console.warn("Не удалось получить target для попапа")
      }
    },
    togglePopup() {
      this.isOpenedState = !this.isOpenedState
    },
    setTargetBox() {
      this.targetBox = this.targetElement.getBoundingClientRect()
    },
    setPopupBox() {
      this.popupBox = this.popupElement.getBoundingClientRect()
    },
    /**
     * Возвращается высота или ширина попапа
     * Если из по пропсам невозможно вычислить - размер берется из DOM
     * @param {string} value - "width" или "height"
     * @returns {number}
     */
    getSize(value) {
      const size = this[value]
      const targetSize = this.targetBox?.[value]

      // Если возможно вычислить размер по пропсам
      if (typeof size === "string" && typeof targetSize === "number") {
        switch (true) {
          case size.endsWith("px"):
            return parseFloat(size)
          case size.endsWith("vh"):
            return (parseFloat(size) / 100) * this.screenHeight
          case size.endsWith("vw"):
            return (parseFloat(size) / 100) * this.screenWidth
          case size.endsWith("%"):
            return (parseFloat(size) / 100) * targetSize
          case size === "auto":
            return targetSize
        }
      }

      // Значение берется из DOM
      return this.popupBox?.[value] ?? 0
    },

    // handlers
    removeTargetOpenByHandlers(target) {
      if (!target) {
        return
      }
      target.removeEventListener("mouseenter", this.mouseEnterTarget)
      target.removeEventListener("mouseleave", this.mouseLeaveTarget)
      target.removeEventListener("click", this.togglePopup)
      target.removeEventListener("contextmenu", this.contextMenuAction)
    },
    addTargetOpenByHandlers() {
      switch (this.openBy) {
        case "hover":
        case "hover target":
          this.targetElement.addEventListener("mouseenter", this.mouseEnterTarget)
          this.targetElement.addEventListener("mouseleave", this.mouseLeaveTarget)
          break
        case "contextmenu":
          this.targetElement.addEventListener("contextmenu", this.contextMenuAction)
          break
        default: // "click"
          this.targetElement.addEventListener("click", this.togglePopup)
          break
      }
    },
    mouseEnterTarget() {
      this.isHovered.target = true
    },
    mouseLeaveTarget() {
      this.isHovered.target = false
    },
    contextMenuAction(e) {
      e.preventDefault()
      this.togglePopup()
    },
    addPopupHoverHandler() {
      // Если попап не выводится при наведении компоненту не нужно знать наведена ли мышь на попап
      if (this.openBy === "hover" && this.popupElement) {
        // Эти обработчики не нужно снимать, тк они автоматически убираются когда попап пропадает
        this.popupElement.addEventListener("mouseenter", this.mouseEnterPopup)
        this.popupElement.addEventListener("mouseleave", this.mouseLeavePopup)
      }
    },
    mouseEnterPopup() {
      this.isHovered.popup = true
    },
    mouseLeavePopup() {
      this.isHovered.popup = false
    },
    onWindowClick(e) {
      if (!this.autoClose) {
        return
      }
      if (!e.composedPath().some(el => el === this.targetElement || el === this.popupElement)) {
        this.isOpened = false
      }
    },
    mouseMoveHandler(e) {
      this.mouseX = e.clientX
      this.mouseY = e.clientY
    },
    checkPopupSizeWatcher() {
      if ((!this.width || !this.height) && this.popupElement) {
        if (this.popupResizeObserver) {
          return
        }
        this.popupResizeObserver = new ResizeObserver(this.setPopupBox)
        this.popupResizeObserver.observe(this.popupElement)
        return
      }
      if (this.popupResizeObserver) {
        this.popupBox = null
        this.popupResizeObserver.disconnect()
        this.popupResizeObserver = null
      }
    },
    startWatching() {
      document.addEventListener("scroll", this.setTargetBox)
      document.addEventListener("resize", this.setTargetBox)
      this.targetResizeObserver = new ResizeObserver(this.setTargetBox)
      this.targetResizeObserver.observe(this.targetElement)
    },
    stopWatching() {
      window.removeEventListener("scroll", this.setTargetBox)
      window.removeEventListener("resize", this.setTargetBox)
      if (this.targetResizeObserver) {
        this.targetResizeObserver.disconnect()
        this.targetResizeObserver = null
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.t-popup {
  position: fixed;
  display: block;
  overflow: auto;
  z-index: 1051;
}

.t-popup_variant {
  &-default {
    padding: 15px 20px 21px;
    border-radius: 8px;
    border: none;
    background-color: #f4f4f4;
    box-shadow: 0 10px 10px -10px rgba(0, 0, 0, 0.25);
  }
  &-dark {
    background-color: rgba(34,34,34,0.8);
    padding: 4px 8px;
    border: 1px solid rgba(34,34,34,1);
    color: #fff;
    border-radius: 6px;
  }
}
</style>

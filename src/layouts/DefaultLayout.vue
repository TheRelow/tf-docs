<template>
  <div class="layout">
    <header class="header">
      <img src="../assets/logo.png" alt="" class="header-logo">
    </header>
    <aside class="aside">
      <nav>
        <ul>
          <li>
            <a href="javascript;">qwe</a>
          </li>
        </ul>
      </nav>
    </aside>
    <main class="main">
      <Breadcrumbs></Breadcrumbs>
      <div class="reading-progress">
        <div class="reading-progress__line" :style="{width: `${readingProgress}%`}"></div>
      </div>
      <section class="content" @scroll="contentScrolled">
        <div class="content__body">
          <slot name="body"></slot>
        </div>
      </section>
      <aside class="right-aside">
        <slot name="right"></slot>
      </aside>
    </main>
  </div>
</template>

<script lang="ts" setup>
import { ref } from "vue"

import Breadcrumbs from "@/components/Breadcrumbs.vue"

const readingProgress = ref(0)

/**
 * Обработчик события прокрутки, вычисляющий прогресс прокрутки в процентах
 * @param {Event} event - Событие прокрутки
 */
const contentScrolled = (event: Event): void => {
  const target = event.target as HTMLElement
  readingProgress.value = (target.scrollTop / (target.scrollHeight - target.clientHeight)) * 100
}
</script>

<style lang="scss" scoped>
.layout {
  display: flex;
  flex-wrap: wrap;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 60px;
  width: 100%;
  padding: 0 15px;
}

.aside {
  width: 340px;
  height: calc(100vh - 60px);
  background-color: $grey-white;
  border-right: 1px solid $grey-light;
}

.main {
  display: flex;
  flex-direction: column;
  flex: 1 1 auto;
  height: calc(100vh - 60px);
  overflow: auto;
}

.reading-progress {
  flex: 0 0 auto;
  width: 100%;
  height: 2px;
}

.reading-progress__line {
  height: 100%;
  width: 30%;
  border-radius: 2px;
  background-color: $blue;
}

.content {
  flex: 1 1 auto;
  width: 100%;
  overflow: auto;
}

.content__body {
  height: 400vh;
  padding: 24px 16px;
}
</style>
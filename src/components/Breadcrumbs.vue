<template>
  <nav class="breadcrumbs">
    <div class="go-back" @click="removeItem"></div>
    <div class="breadcrumbs__wrapper">
      <ul class="breadcrumbs__list" ref="list">
        <li v-for="(item, idx) of items" class="breadcrumbs__list-item">
          {{ item }}
        </li>
      </ul>
    </div>
  </nav>
</template>

<script lang="ts" setup>
import { ref } from "vue";

const qwe = ref(false)
const list = ref()
const items = ref(["qwe", "zxc", "123", "foo", "bar", "baz"])

function removeItem() {
  // const elBox = list.value.children[0].getBoundingClientRect()
  list.value.children[list.value.children.length - 1].classList.add("hidden")
  setTimeout(() => {
    items.value.pop()
  }, 300)
}
</script>

<style lang="scss" scoped>
.breadcrumbs {
  position: relative;
  flex: 0 0 auto;
  display: flex;
  align-items: center;
  gap: 20px;
  width: 100%;
  height: 40px;
  padding: 0 15px;
  border-bottom: 1px solid $grey-light;
}

.go-back {
  position: absolute;
  top: 50%;
  left: 15px;
  width: 26px;
  height: 26px;
  border-radius: 50%;
  background-color: $grey-white;
  border-right: 1px solid $grey-light;
  color: $grey;
  transform: translateY(-50%);
  transition: $slow-transition;
  cursor: pointer;
  z-index: 2;
  &:hover {
    background-color: $blue-white;
    color: $blue;
  }
  &:before, &:after {
    position: absolute;
    top: 50%;
    left: 55%;
    width: 8px;
    height: 2px;
    background-color: currentColor;
    transform-origin: 0 50%;
    transform: translate(-50%, -50%) rotate(45deg);
    content: "";
  }
  &:after {
    transform: translate(-50%, -50%) rotate(-45deg);
  }
}

.breadcrumbs__wrapper {
  padding-left: 33px;
  margin-left: 13px;
}

.breadcrumbs__list {
  left: 0;
  display: flex;
  list-style: none;
  overflow: hidden;
  color: $blue;
}

$radius: 3px;
.breadcrumbs__list-item {
  position: relative;
  padding: 0 12px;
  transition: $slow-transition;

  &:after {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    display: block;
    background-color: $blue;
    opacity: 1;
    clip-path: polygon(0 calc(50% - $radius), 0 calc(50% - $radius), $radius 50%, 0 calc(50% + $radius), 0 calc(50% + $radius));
    transition: 0.15s;
    content: "";
  }

  &.hidden {
    $radius: 10px;
    opacity: 0;
    &:after {
      animation: clip-transition 0.2s forwards;
      //clip-path: polygon(0 calc(50% - $radius), calc(100% - $radius) calc(50% - $radius), 100% 50%, calc(100% - $radius) calc(50% + $radius), 0 calc(50% + $radius));
    }
  }
}

@keyframes clip-transition {
  0% {
    clip-path: polygon(0 calc(50% - $radius), 0 calc(50% - $radius), $radius 50%, 0 calc(50% + $radius), 0 calc(50% + $radius), $radius 50%);
  }
  50% {
    clip-path: polygon(0 calc(50% - $radius), calc(100% - $radius) calc(50% - $radius), 100% 50%, calc(100% - $radius) calc(50% + $radius), 0 calc(50% + $radius), $radius 50%);
  }
  100% {
    clip-path: polygon(calc(100% - $radius) calc(50% - $radius), calc(100% - $radius) calc(50% - $radius), 100% 50%, calc(100% - $radius) calc(50% + $radius), calc(100% - $radius) calc(50% + $radius), 100% 50%);
  }
}
</style>
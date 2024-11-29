<script setup lang="ts">
import List from "@/data.json";
import { computed, onMounted, ref, watch, type Ref } from "vue";
import { onKeyStroke } from '@vueuse/core'
import { useKeyModifier } from '@vueuse/core'
import { useScroll } from '@vueuse/core'

let active = ref(0);
const buttons: Ref<HTMLButtonElement[]> = ref([])
const list = ref(List.sort((a, b) => {
  return a.id - b.id;
}));
const selected = computed(() =>
  list.value.find(element => {
    return element.id === active.value;
  })
);
const shift = useKeyModifier('Shift')
const el = ref<HTMLElement | null>(document.documentElement)
const { y } = useScroll(el, { behavior: "smooth" })
const scrollTo = computed(() => active.value - 4 > 0 ? active.value - 4 : 0)
const showHint = ref(true)

watch(active, () => {
  const button = buttons.value[active.value - 1]
  button.focus({ preventScroll: true })
})

const previous = () => {
  active.value = active.value === 1 ? list.value.length : active.value - 1
  y.value = buttons.value[scrollTo.value].offsetTop
}
const next = () => {
  active.value = active.value === list.value.length ? 1 : active.value + 1
  y.value = buttons.value[scrollTo.value].offsetTop
}

const set = (id: number) => {
  active.value = id
  y.value = buttons.value[scrollTo.value].offsetTop
}

onKeyStroke('ArrowLeft', (e) => {
  previous();
})
onKeyStroke('ArrowRight', (e) => {
  next();
})
onKeyStroke('Tab', (e) => {
  e.preventDefault()
  shift.value ? previous() : next();
})

onMounted(() => {
  buttons.value = Array.from(document.querySelectorAll('button'))
  active.value = 1
  y.value = buttons.value[0].offsetTop
});
</script>

<template>
  <main>
    <div class="grid">
      <button class="word" v-for="element in list" :key="element.id" @click="set(element.id)">
        <p><span v-if="element.id < 10">0</span>{{ element.id }}</p>
        <p>{{ element.name }}</p>
      </button>
    </div>

    <Transition mode="out-in">
      <div v-if="selected" :key="selected!.id" class="selected">
        <div class="header">
          <h2>{{ selected!.name }}</h2>
        </div>
        <p>*</p>
        <p>{{ selected!.description }}</p>
      </div>
    </Transition>
    <div class="info">
      <transition name="appear">
        <div v-if="showHint" class="info-hint">
          <div class="keys">
            <img class="key" src="./assets/tab.svg" alt="">
            <img class="key" src="./assets/shift.svg" alt="">
          </div>
        </div>
      </transition>
      <button class="info-button" @click="(e: MouseEvent) => {showHint = !showHint; (e.target as HTMLElement)?.parentElement?.blur()} " :class="{discreet : !showHint}">
        <transition name="slide" mode="out-in">
          <img v-if="showHint" src="./assets/close.svg" alt="">
          <img v-else src="./assets/open.svg" alt="">
        </transition>
      </button>
      <transition name="appear">
        <div v-if="showHint" class="info-hint">
          <div class="keys">
            <img class="key" src="./assets/left.svg" alt="">
            <p></p>
            <img class="key" src="./assets/right.svg" alt="">
          </div>
        </div>
      </transition>
    </div>
  </main>
</template>

<style lang="scss">
.grid {
  height: fit-content;
  width: 25vw;
  display: flex;
  flex-flow: column;
  padding: 1rem;
}

.word {
  background-color: transparent;
  border: none;
  display: flex;
  align-items: center;
  cursor: pointer;
  overflow: hidden;
  gap: 0.5rem;
  padding: 0;
  height: 3.9rem;

  &:hover,
  &:focus {
    outline: none;

    p {
      &:first-of-type {
        opacity: 1;
        background-color: #000;
        color: #e6be8a;
      }

      &:last-of-type {
        text-decoration: underline;
      }
    }
  }

  p {
    text-align: left;
    font-size: 2rem;
    line-height: 2rem;

    &:first-of-type {
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 0 0.25rem;
      border-radius: 2px;
      font-size: 1.5rem;
      line-height: 1.5rem;
      height: 1.75rem;
      width: 1.75rem;
      opacity: 0.4;
    }

    &:last-of-type {
      text-overflow: ellipsis;
      overflow: hidden;
      white-space: nowrap;
    }

  }
}

.selected {
  position: fixed;
  top: 0;
  left: 25vw;
  height: 100vh;
  width: calc(100vw - 25vw);
  display: flex;
  padding-right: 16%;
  flex-flow: column;
  align-items: center;
  justify-content: center;
  gap: 2rem;

  &>p {
    font-size: 3rem;
    line-height: 3rem;
    max-width: 45rem;
    text-align: center;
    text-justify: distribute;

    &:first-of-type {
      animation: slide 0.25s;
    }

    &:last-of-type {
      animation: slide 0.4s;
    }
  }
}

.info {
  position: fixed;
  top: 3rem;
  right: 4rem;

  left: 25vw;
  width: calc(100vw - 25vw);
  padding-right: 16%;

  z-index: 600;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 2rem;
}

.info-button {
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: transparent;
  border: none;
  border-radius: 100%;
  width: 4rem;
  height: 2rem;
  outline: 2px solid #000;
  overflow: hidden;
  // opacity: 0.4;
  cursor: pointer;
  transition: opacity 0.3s;

  img {
    width: 1.5rem;
  }

  &:hover,
  &:focus {
    opacity: 1;
  }
  }
  
  .discreet {
  opacity: 0.4;
}

.info-hint {
  width: 5rem;
  font-size: 1.75rem;
  line-height: 1;
  // opacity: 0.4;
  height: 2rem;
  display: flex;
  align-items: center;
  padding: 0.5rem;
  gap: 1rem;

  &:first-of-type {
    justify-content: flex-end;
  }

  .keys {
    display: flex;
    gap: 0.25rem;
  }

  .key {
    width: 1.25rem;
  }
}

.text {
  text-align: center;
  font-size: 4rem;
  line-height: 4rem;
  justify-content: flex-start;
  animation: slide 0.4s ease;
}

.header {
  display: flex;
  flex-flow: column;
  align-items: center;
  animation: toggle 0.4s;
  gap: 1rem;

  p {
    font-style: italic;
    line-height: 3rem;
    font-size: 3rem;
    opacity: 0.4;
  }

  h2 {
    font-size: 8.5rem;
    max-width: 60rem;
    line-height: 6.5rem;
    font-weight: 400;
    text-align: center;
  }
}


.v-enter-active,
.v-leave-active {
  transition: opacity 0.3s, transform 0.3s;
}

.v-enter-from {
  opacity: 0;
  transform: translateY(-1rem);
}

.v-leave-to {
  transform: translateY(1rem);
  opacity: 0;
}

.appear-enter-active,
.appear-leave-active {
  transition: opacity 0.3s;
}

.appear-enter-from {
  opacity: 0;
}

.appear-leave-to {
  opacity: 0;
}

.slide-enter-active,
.slide-leave-active {
  transition: transform 0.3s;
}

.slide-enter-from {
  transform: translateY(-1.5rem);
}

.slide-leave-to {
  transform: translateY(1.5rem);
}


@keyframes slide {
  0% {
    transform: translateY(-1rem)
  }

  100% {
    transform: translateY(0);
  }
}
</style>

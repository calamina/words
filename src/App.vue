<script setup lang="ts">
import List from "@/data.json";
import { computed, onMounted, ref, watch, type Ref } from "vue";
import { onKeyStroke } from '@vueuse/core'
import { useKeyModifier } from '@vueuse/core'
import { useScroll } from '@vueuse/core'
import { useMediaQuery } from '@vueuse/core'

const el = ref<HTMLElement | null>(document.documentElement)

const isSmallScreen = useMediaQuery('(max-width: 1024px)')
const shift = useKeyModifier('Shift')
const { y } = useScroll(el, { behavior: "smooth" })

let active = ref(0);

const listButtons: Ref<HTMLButtonElement[]> = ref([])
const list = ref(List.sort((a, b) => {
  return a.id - b.id;
}));
const selected = computed(() =>
  list.value.find(element => {
    return element.id === active.value;
  })
);
const scrollTo = computed(() => active.value - 4 > 0 ? active.value - 4 : 0)
const showHint = ref(true)
const showList = ref(false)

watch(active, () => {
  const button = listButtons.value[active.value - 1]
  button.focus({ preventScroll: true })
})

watch(showList, () => {
  if (showList.value === true) {
    setTimeout(() => {
      const button = listButtons.value[active.value - 1]
      button.focus()
    }, 50);
  }
})

const previous = () => {
  active.value = active.value === 1 ? list.value.length : active.value - 1
  y.value = listButtons.value[scrollTo.value].offsetTop
}
const next = () => {
  active.value = active.value === list.value.length ? 1 : active.value + 1
  y.value = listButtons.value[scrollTo.value].offsetTop
}

const set = (id: number) => {
  active.value = id
  y.value = listButtons.value[scrollTo.value].offsetTop
  showList.value = false
}

onKeyStroke('ArrowLeft', () => {
  previous();
})
onKeyStroke('ArrowRight', () => {
  next();
})
onKeyStroke('Tab', (e) => {
  e.preventDefault()
  shift.value ? previous() : next();
})

onMounted(() => {
  listButtons.value = Array.from(document.querySelectorAll('.word'))
  active.value = 1
  y.value = listButtons.value[0].offsetTop
});
</script>

<template>
  <main>
    <div v-if="isSmallScreen" class="mobile">
      <transition name="appear">
        <button v-if="!showList" class="info-button" @click="previous()">
          <img src="./assets/left.svg" alt="">
        </button>
      </transition>
      <button class="info-button" @click="(showList = !showList) && listButtons[active - 1]?.focus()">
        <img src="./assets/open.svg" alt="">
      </button>
      <transition name="appear">
        <button v-if="!showList" class="info-button" @click="next()">
          <img src="./assets/right.svg" alt="">
        </button>
      </transition>
    </div>
    <Transition>
      <div class="grid" v-show="isSmallScreen ? showList : true">
        <button class="word" v-for="element in list" :key="element.id" @click="set(element.id)">
          <p><span v-if="element.id < 10">0</span>{{ element.id }}</p>
          <p>{{ element.name }}</p>
        </button>
      </div>
    </Transition>

    <Transition mode="out-in">
      <div v-if="selected && isSmallScreen ? !showList : true" :key="selected?.id" class="selected">
        <div class="header">
          <h2>{{ selected?.name }}</h2>
        </div>
        <p>*</p>
        <p>{{ selected?.description }}</p>
      </div>
    </Transition>
    <div class="info">
      <transition name="appear">
        <div class="info-hide" v-if="showHint">
          <!-- <div class="info-hint">
            <div class="keys">
              <img class="key" src="./assets/tab.svg" alt="">
              <img class="key" src="./assets/shift.svg" alt="">
            </div>
          </div> -->
          <button class="info-button" @click="previous()">
            <img src="./assets/left.svg" alt="">
          </button>
        </div>
      </transition>
      <button class="info-button"
        @click="(e: MouseEvent) => { showHint = !showHint; (e.target as HTMLElement)?.parentElement?.blur() }"
        :class="{ discreet: !showHint }">
        <transition name="slide" mode="out-in">
          <img v-if="showHint" src="./assets/close.svg" alt="">
          <img v-else src="./assets/open.svg" alt="">
        </transition>
      </button>
      <transition name="appear">
        <div class="info-hide" v-if="showHint">
          <button class="info-button" @click="next()">
            <img src="./assets/right.svg" alt="">
          </button>
          <!-- <div class="info-hint">
            <div class="keys">
              <img class="key" src="./assets/left.svg" alt="">
              <p></p>
              <img class="key" src="./assets/right.svg" alt="">
            </div>
          </div> -->
        </div>
      </transition>
    </div>
  </main>
</template>

<style lang="scss">
.mobile {
  z-index: 900;
  display: flex;
  gap: 1rem;
  width: 100vw;
  justify-content: center;
  position: fixed;
  bottom: 0;
  left: 0;
  padding-bottom: 3rem;
}

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

.info-hide {
  display: flex;
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
  height: 2rem;
  display: flex;
  align-items: center;
  padding: 0.5rem;
  gap: 1rem;

  &:first-of-type {
    justify-content: flex-end;
  }

  &:last-of-type {
    justify-content: center;
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

@media (max-width: 1024px) {
  html {
    font-size: 0.8rem;
  }

  .info {
    display: none;
  }

  .grid {
    padding-top: 4rem;
    display: grid;
    width: 100%;
    height: calc(100vh - 8rem);
    overflow-y: scroll;
    scrollbar-width: none;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  }

  .word {
    height: 4.5rem;

    & p:last-of-type {
      font-size: 2.25rem;
    }
  }

  .selected {
    left: 0;
    padding-right: 0;
    width: 100vw;
    padding: 1rem 2rem;
  }

  .header h2 {
    font-size: 6rem;
    line-height: 5rem;
  }
}
</style>

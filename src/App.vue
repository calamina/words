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
      <button v-for="element in list" :key="element.id" @click="set(element.id)">
        <p><span v-if="element.id < 10">0</span>{{ element.id }}</p>
        <p>{{ element.name }}</p>
      </button>
    </div>
    <p class="hint" @click="(e: MouseEvent) => (e.target as HTMLElement)?.remove()">Tab & Shift + Tab ||| Arrows to
      navigate</p>
    <Transition mode="out-in">
      <div v-if="selected" :key="selected!.id" class="selected">
        <div class="header">
          <h2>{{ selected!.name }}</h2>
        </div>
        <p>*</p>
        <p>{{ selected!.description }}</p>
      </div>
    </Transition>
  </main>
</template>

<style lang="scss">
.grid {
  height: fit-content;
  width: 25vw;
  display: flex;
  gap: 0.25rem;
  flex-flow: column;
  padding: 1rem;
}

button {
  background-color: transparent;
  border: none;
  display: flex;
  align-items: center;
  cursor: pointer;
  overflow: hidden;
  gap: 0.5rem;
  padding: 0;
  height: 3.5rem;

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

.hint {
  position: fixed;
  right: 1rem;
  z-index: 600;
  bottom: 1rem;
  font-size: 1.5rem;
  line-height: 1;
  opacity: 0.4;
  cursor: pointer;
  display: flex;
  align-items: center;
  font-style: oblique;
  padding: 0.5rem;
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


@keyframes slide {
  0% {
    transform: translateY(-1rem)
  }

  100% {
    transform: translateY(0);
  }
}
</style>

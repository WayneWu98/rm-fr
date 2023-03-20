<script setup lang="ts">
import { ref, nextTick, computed, onMounted, watch } from 'vue'
import { prefix, RM_CMD } from './pseudo-tty'
import raw from '../assets/log.txt?raw'

defineProps<{ actions: string[] }>()
const history = ref<string[]>([])

const tty = ref<HTMLDIVElement>()
const stdin = ref<HTMLParagraphElement>()
const flushing = ref(false)
const cmd = ref(RM_CMD)

let rmed = false

const onInput = () => cmd.value = stdin.value.innerText

let queue = Promise.resolve()
const LIMIT = 1000

const sleep = (duration: number) => new Promise(r => setTimeout(r, duration))

const print = (s: string) => {
  if (history.value.length >= LIMIT) {
    history.value.shift()
  }
  history.value.push(s)
  tty.value.scrollTop = 999999
  return sleep(Math.random() * 3 + 1)
}

watch(
  flushing,
  v => {
    nextTick(() => {
      tty.value.scrollTop = 999999
      if (!flushing.value) focus()
    })
  }
)

const log = (s: string | string[]) => {
  if (typeof s === 'string') s = [s]
  queue = queue.finally(() => flushing.value = true)
  s.forEach(s =>  queue = queue.then(() => print(s)))
  queue = queue.finally(() => flushing.value = false)
}

const focus = () => {
  const s = window.getSelection()
  const r = document.createRange()
  const l = stdin.value.childNodes.length
  r.setStart(stdin.value, l)
  r.setEnd(stdin.value, l)
  s.removeAllRanges()
  s.addRange(r)
}

const runCommand = (s: string) => {
  cmd.value = ''
  if (s.trim() === RM_CMD && !rmed) {
    rmed = true
    return log(raw.split('\n'))
  }
  log(`bash: ${s.split(' ')[0]}: command not found`)
}

onMounted(() => {
  focus()
  stdin.value.registerKeyshort(['Enter'], () => runCommand(stdin.value.innerText))
})

defineExpose({ runCommand })
</script>

<template>
  <div ref="tty" class="tty" @click="focus">
    <p v-for="s in history" :key="s">{{ s }}</p>
    <p v-show="!flushing" class="stdin" key="stdin">
      <span class="prompt">[root@ubuntu /]#</span>
      <span contenteditable ref="stdin" @input="onInput">{{ cmd }}</span>
    </p>
  </div>
</template>

<style scoped>
.tty {
  font-size: 1em;
  line-height: 1em;
  background-color: #000;
  font-family: monospace;
  color: #fff;
  overflow: scroll;
  cursor: text;
}

.tty * {
  margin: 0;
  padding: 0;
  outline: none;
}

.prompt {
  margin-right: 1em;
}
</style>
<script setup lang="ts">
import { ref, onMounted } from 'vue'
import PseudoTty from './components/PseudoTty.vue'
import { RM_CMD } from './components/pseudo-tty'

const tty = ref<typeof PseudoTty>()

onMounted(() => document.body.registerKeyshort(['Enter'], () => tty.value.runCommand(RM_CMD)))

</script>

<template>
  <PseudoTty ref="tty" style="width: 100vw; height: 100vh;" />
</template>

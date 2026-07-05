<template>
  <div class="h-screen bg-black text-slate-400 font-mono select-none overflow-hidden touch-none" @touchstart="ts" @touchmove="tm">
    <div class="absolute top-4 left-4 z-10 text-[10px] bg-zinc-900/80 p-2 rounded border border-zinc-800 backdrop-blur">
      AZI: <span class="text-white">{{ az }}°</span> | ALT: <span class="text-white">{{ al }}°</span>
    </div>
    <svg viewBox="0 0 400 400" class="w-full h-full transform transition-transform duration-75 origin-center" :style="{ transform: `scale(2) translate(${x}px, ${y}px)` }">
      <g stroke="#27272a" stroke-width="0.5" stroke-linecap="round">
        <line x1="150" y1="120" x2="180" y2="100" />
        <line x1="180" y1="100" x2="200" y2="150" />
        <line x1="200" y1="150" x2="160" y2="170" />
        <line x1="160" y1="170" x2="150" y2="120" />
        <line x1="168" y1="135" x2="180" y2="138" />
        <line x1="180" y1="138" x2="192" y2="141" />
        <line x1="280" y1="280" x2="290" y2="330" />
        <line x1="290" y1="330" x2="320" y2="340" />
        <line x1="280" y1="280" x2="310" y2="290" />
      </g>
      <g>
        <circle cx="150" cy="120" r="2.5" fill="#f43f5e" :class="w" />
        <circle cx="200" cy="150" r="2" fill="#38bdf8" :class="w" />
        <circle cx="168" cy="135" r="1" fill="#fff" :class="w" />
        <circle cx="180" cy="138" r="1" fill="#fff" :class="w" />
        <circle cx="192" cy="141" r="1" fill="#fff" :class="w" />
        <circle cx="280" cy="280" r="2.2" fill="#38bdf8" :class="w" />
        <circle cx="290" cy="330" r="1.8" fill="#bae6fd" :class="w" />
        <circle v-for="(s, i) in b" :key="i" :cx="s.x" :cy="s.y" :r="s.r" fill="#fff" :class="w" :style="{ animationDelay: s.d }" />
      </g>
    </svg>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
const x = ref(0), y = ref(0), az = ref(0), al = ref(0), b = ref([]), w = 'animate-pulse'
let sx = 0, sy = 0
const ts = (e) => { sx = e.touches[0].clientX - x.value; sy = e.touches[0].clientY - y.value }
const tm = (e) => {
  x.value = e.touches[0].clientX - sx; y.value = e.touches[0].clientY - sy
  az.value = Math.round((x.value * 0.5) % 360); al.value = Math.round(Math.max(-90, Math.min(90, y.value * 0.3)))
}
onMounted(() => {
  for (let i = 0; i < 150; i++) b.value.push({ x: Math.random() * 400, y: Math.random() * 400, r: Math.random() * 0.8 + 0.2, d: `${Math.random() * 2}s` })
})
</script>

<style scoped>
.animate-pulse { animation: p 1.5s infinite ease-in-out; }
@keyframes p { 0%, 100% { opacity: 0.3; } 50% { opacity: 1; } }
</style>

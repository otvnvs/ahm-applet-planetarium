<template>
  <div class="H flex flex-col h-screen bg-slate-950 text-slate-200 select-none font-mono">
    <!-- Top Bar -->
    <header class="p-3 bg-slate-900 border-b border-slate-800 flex justify-between items-center text-xs">
      <span class="font-bold tracking-widest text-cyan-400">SKY_MD</span>
      <span class="text-slate-500 flex items-center gap-1">
        <span class="w-1.5 h-1.5 rounded-full bg-emerald-500 animate-pulse"></span>
        ZA // RET_OK
      </span>
    </header>

    <!-- App Content Layout -->
    <main class="flex-1 overflow-y-auto p-4 space-y-4">
      <!-- Radar Core View -->
      <div v-if="t == 's'" class="flex flex-col items-center">
        <!-- Coordinate HUD -->
        <div class="w-full grid grid-cols-2 gap-2 text-[10px] bg-slate-900 p-2.5 rounded border border-slate-800">
          <div>RA: <span class="text-cyan-400">{{ c.ra }}</span></div>
          <div>DEC: <span class="text-indigo-400">{{ c.dec }}</span></div>
        </div>

        <!-- Radar Ring Array -->
        <div class="relative w-64 h-64 bg-slate-900/50 rounded-full border border-slate-800 mt-6 flex items-center justify-center p-4">
          <svg viewBox="0 0 100 100" class="w-full h-full transform transition-transform duration-500" :style="{ transform: `rotate(${r}deg)` }">
            <circle cx="50" cy="50" r="45" fill="none" stroke="#1e293b" stroke-dasharray="2 2" />
            <circle cx="50" cy="50" r="25" fill="none" stroke="#1e293b" />
            <path d="M50 0v100M0 50h100" stroke="#1e293b" stroke-width="0.5" />
            <!-- Star Nodes -->
            <circle cx="50" cy="30" r="2" fill="#38bdf8" class="cursor-pointer" @click="s('Acrux')" />
            <circle cx="30" cy="60" r="2.5" fill="#f43f5e" class="cursor-pointer animate-pulse" @click="s('Betel')" />
            <circle cx="70" cy="45" r="1.5" fill="#ffffff" class="cursor-pointer" @click="s('Rigel')" />
          </svg>
          <div class="absolute inset-0 border border-cyan-500/10 rounded-full pointer-events-none flex items-center justify-center">
            <div class="w-8 h-8 border border-dashed border-cyan-500/30 rounded-full animate-ping"></div>
          </div>
        </div>

        <!-- Telemetry Data Readout -->
        <div class="w-full mt-6 bg-slate-900 p-3 rounded border border-slate-800 min-h-[60px] text-xs">
          <div v-if="o">
            <div class="flex justify-between font-bold text-white mb-1">
              <span>{{ o.n }}</span>
              <span class="text-cyan-400 text-[10px]">{{ o.m }}</span>
            </div>
            <p class="text-slate-400 text-[11px]">{{ o.d }}</p>
          </div>
          <p v-else class="text-slate-600 text-center text-[11px] pt-2">Select node targeting matrix.</p>
        </div>
      </div>

      <!-- Directory View -->
      <div v-if="t == 'd'" class="space-y-1.5 text-xs">
        <div v-for="i in db" :key="i.n" @click="f(i)" class="p-2.5 bg-slate-900 border border-slate-800 rounded flex justify-between items-center active:bg-slate-800">
          <div>
            <div class="font-bold text-white">{{ i.n }}</div>
            <div class="text-[10px] text-slate-500">Brightest Node: {{ i.b }}</div>
          </div>
          <span class="text-cyan-400 font-bold">{{ i.v }}</span>
        </div>
      </div>
    </main>

    <!-- Ultra-Compact Navigation Footer -->
    <nav class="bg-slate-900 border-t border-slate-800 flex justify-around text-[10px] py-2 font-bold">
      <button @click="t = 's'" :class="t == 's' ? 'text-cyan-400' : 'text-slate-500'">[ SKY ]</button>
      <button @click="t = 'd'" :class="t == 'd' ? 'text-cyan-400' : 'text-slate-500'">[ DIR ]</button>
    </nav>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const t = ref('s') // tab layout state switch
const r = ref(0)   // rotation layout vector tracking degree
const o = ref(null)// focused astronomical coordinate object cache
const c = ref({ ra: '12h24m', dec: '-63°05' }) // coordinate telemetric readouts

const db = [
  { n: 'Crux', b: 'Acrux', v: '98%', d: 'Crux constellation star matrix layout mapping.', m: '0.77' },
  { n: 'Orion', b: 'Rigel', v: '41%', d: 'Orion hunter sector deep star coordinates.', m: '0.13' }
]

const s = (k) => {
  const match = db.find(x => x.b === k || x.n === k)
  o.value = match ? { n: match.n, m: `MAG ${match.m}`, d: match.d } : { n: k, m: 'MAG N/A', d: 'Ambient system sky node map coordinate.' }
}

const f = (i) => {
  o.value = { n: i.n, m: `MAG ${i.m}`, d: i.d }
  r.value = 90
  t.value = 's'
}

let tm = null
onMounted(() => {
  tm = setInterval(() => {
    r.value = (r.value + 1) % 360
    c.value.ra = `12h${Math.floor(Math.random() * 60)}m`
  }, 1200)
})
onUnmounted(() => clearInterval(tm))
</script>

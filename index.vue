<template>
  <div class="fixed inset-0 bg-black overflow-hidden select-none touch-none m-0 p-0 overscroll-none">
    <canvas ref="cv" class="w-full h-full block bg-black m-0 p-0 touch-none"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const cv = ref(null)
const az = ref(0)
const al = ref(0)
const fov = ref(60)

let w, h, ctx, rId, stars = [], lines = [], nebs = []
let pAz = 0, pAl = 0, sx = 0, sy = 0, sd = 0, sf = 60, ip = false, grid = false

const ortho = (ra, dec, az, al, f, w, h) => {
  const sd = Math.sin(dec), cd = Math.cos(dec)
  const sr = Math.sin(ra), cr = Math.cos(ra)
  const sa = Math.sin(al), ca = Math.cos(al)
  const sz = Math.sin(az), cz = Math.cos(az)
  let x3 = cd * cr, y3 = cd * sr, z3 = sd
  let x = x3 * sz - y3 * cz
  let y = x3 * cz * sa + y3 * sz * sa - z3 * ca
  let z = x3 * cz * ca + y3 * sz * ca + z3 * sa
  if (z <= 0) return null
  const d = Math.min(w, h) / Math.tan((f * Math.PI) / 360)
  return { x: w / 2 + (x * d) / z, y: h / 2 - (y * d) / z, z }
}

const ts = (e) => {
  e.preventDefault()
  const t = e.touches
  if (t.length === 2) {
    ip = true
    sd = Math.hypot(t[0].clientX - t[1].clientX, t[0].clientY - t[1].clientY)
    sf = fov.value
  } else if (!ip) {
    sx = t[0].clientX
    sy = t[0].clientY
    if (sx > w - 100 && sx < w - 16 && sy > 16 && sy < 40) {
      grid = !grid
      return
    }
    pAz = az.value
    pAl = al.value
  }
}

const tm = (e) => {
  e.preventDefault()
  const t = e.touches
  if (t.length === 2 && ip) {
    const d = Math.hypot(t[0].clientX - t[1].clientX, t[0].clientY - t[1].clientY)
    if (d > 0) fov.value = Math.max(1, Math.min(160, sf * (sd / d)))
  } else if (t.length === 1 && !ip) {
    az.value = pAz - ((t[0].clientX - sx) * 0.005)
    al.value = Math.max(-Math.PI / 2, Math.min(Math.PI / 2, pAl + (t[0].clientY - sy) * 0.005))
  }
}

const te = (e) => { e.preventDefault(); if (e.touches.length === 0) ip = false }

const res = () => {
  if (!cv.value) return
  w = cv.value.width = window.innerWidth
  h = cv.value.height = window.innerHeight
}

const draw = () => {
  ctx.fillStyle = '#000000'
  ctx.fillRect(0, 0, w, h)
  const t = Date.now() * 0.0005
  const scl = 60 / fov.value

  nebs.forEach(n => {
    const p = ortho(n.ra, n.dec, az.value, al.value, fov.value, w, h)
    if (p) {
      ctx.save()
      ctx.globalCompositeOperation = 'screen'
      ctx.translate(p.x, p.y)
      ctx.rotate(n.rot)
      
      n.puffs.forEach(pf => {
        const cx = pf.x * scl
        const cy = pf.y * scl
        const rRad = pf.r * scl
        
        if (rRad > 2) {
          const g = ctx.createRadialGradient(cx, cy, 0, cx, cy, rRad)
          g.addColorStop(0, pf.c)
          g.addColorStop(0.3, pf.c.replace('0.1', '0.04'))
          g.addColorStop(0.7, pf.c.replace('0.1', '0.01'))
          g.addColorStop(1, 'rgba(0,0,0,0)')
          
          ctx.fillStyle = g
          ctx.beginPath()
          ctx.arc(cx, cy, rRad, 0, Math.PI * 2)
          ctx.fill()
        }
      })
      ctx.restore()
    }
  })

  if (grid) {
    ctx.strokeStyle = 'rgba(113,113,122,0.15)'
    ctx.lineWidth = 0.5
    for (let d = -75; d <= 75; d += 15) {
      ctx.beginPath()
      let f = true
      for (let r = 0; r <= 365; r += 5) {
        const p = ortho((r * Math.PI) / 180, (d * Math.PI) / 180, az.value, al.value, fov.value, w, h)
        if (p) {
          f ? ctx.moveTo(p.x, p.y) : ctx.lineTo(p.x, p.y)
          f = false
        } else f = true
      }
      ctx.stroke()
    }
    for (let r = 0; r < 360; r += 30) {
      ctx.beginPath()
      let f = true
      for (let d = -90; d <= 90; d += 5) {
        const p = ortho((r * Math.PI) / 180, (d * Math.PI) / 180, az.value, al.value, fov.value, w, h)
        if (p) {
          f ? ctx.moveTo(p.x, p.y) : ctx.lineTo(p.x, p.y)
          f = false
        } else f = true
      }
      ctx.stroke()
    }
  }

  ctx.strokeStyle = 'rgba(63,63,70,0.5)'
  ctx.lineWidth = 0.6
  lines.forEach(l => {
    const p1 = ortho(l.a.ra, l.a.dec, az.value, al.value, fov.value, w, h)
    const p2 = ortho(l.b.ra, l.b.dec, az.value, al.value, fov.value, w, h)
    if (p1 && p2) {
      ctx.beginPath()
      ctx.moveTo(p1.x, p1.y)
      ctx.lineTo(p2.x, p2.y)
      ctx.stroke()
    }
  })

  stars.forEach(s => {
    const p = ortho(s.ra, s.dec, az.value, al.value, fov.value, w, h)
    if (p) {
      const o = 0.7 + 0.3 * Math.sin(t + s.o)
      const rRad = s.r * scl
      
      if (s.g && rRad > 0.8) {
        ctx.save()
        ctx.globalCompositeOperation = 'screen'
        const g = ctx.createRadialGradient(p.x, p.y, 0, p.x, p.y, rRad * 8)
        g.addColorStop(0, s.gc.replace('G', (0.45 * o).toFixed(2)))
        g.addColorStop(0.2, s.gc.replace('G', (0.15 * o).toFixed(2)))
        g.addColorStop(0.5, s.gc.replace('G', (0.04 * o).toFixed(2)))
        g.addColorStop(1, 'rgba(0,0,0,0)')
        ctx.fillStyle = g
        ctx.beginPath()
        ctx.arc(p.x, p.y, rRad * 8, 0, Math.PI * 2)
        ctx.fill()
        ctx.restore()
      }
      
      ctx.fillStyle = s.c.replace('O', o.toFixed(2))
      ctx.beginPath()
      ctx.arc(p.x, p.y, Math.max(0.4, rRad), 0, Math.PI * 2)
      ctx.fill()
    }
  })

  ctx.fillStyle = '#52525b'
  ctx.font = '10px monospace'
  ctx.fillText(`AZI: ${Math.round((az.value * 180) / Math.PI)}°`, 16, 24)
  ctx.fillText(`ALT: ${Math.round((al.value * 180) / Math.PI)}°`, 16, 36)
  ctx.fillText(`FOV: ${Math.round(fov.value)}°`, 16, 48)

  ctx.fillStyle = grid ? '#06b6d4' : '#27272a'
  ctx.fillRect(w - 100, 16, 84, 24)
  ctx.fillStyle = grid ? '#000000' : '#a1a1aa'
  ctx.font = 'bold 9px monospace'
  ctx.textAlign = 'center'
  ctx.fillText('GRID LNS', w - 58, 31)
  ctx.textAlign = 'left'

  rId = requestAnimationFrame(draw)
}

const genNeb = (ra, dec, baseColor, numPuffs, radius) => {
  const puffs = []
  for (let i = 0; i < numPuffs; i++) {
    const ang = Math.random() * Math.PI * 2
    const dist = Math.random() * radius * 0.6
    puffs.push({
      x: Math.cos(ang) * dist,
      y: Math.sin(ang) * dist,
      r: radius * 0.4 + Math.random() * radius * 0.5,
      c: baseColor
    })
  }
  return { ra, dec, rot: Math.random() * Math.PI, puffs, s: radius * 1.5 }
}

onMounted(() => {
  ctx = cv.value.getContext('2d')
  window.addEventListener('resize', res)
  cv.value.addEventListener('touchstart', ts, { passive: false })
  cv.value.addEventListener('touchmove', tm, { passive: false })
  cv.value.addEventListener('touchend', te, { passive: false })
  document.body.style.overflow = 'hidden'
  res()

  const cls = ['rgba(255,255,255,O)', 'rgba(235,245,255,O)', 'rgba(215,230,255,O)', 'rgba(255,230,230,O)', 'rgba(255,240,215,O)']
  
  const oStar = [
    { ra: 1.48, dec: -0.14, r: 3.5, c: 'rgba(244,63,94,O)', g: true, gc: 'rgba(244,63,94,G)' },
    { ra: 1.41, dec: 0.13, r: 3.2, c: 'rgba(56,189,248,O)', g: true, gc: 'rgba(56,189,248,G)' },
    { ra: 1.52, dec: -0.16, r: 2.2, c: 'rgba(255,255,255,O)', g: false },
    { ra: 1.51, dec: -0.03, r: 2.4, c: 'rgba(255,255,255,O)', g: false },
    { ra: 1.50, dec: 0.10, r: 2.3, c: 'rgba(255,255,255,O)', g: false },
    { ra: 1.62, dec: -0.22, r: 3.4, c: 'rgba(129,140,248,O)', g: true, gc: 'rgba(129,140,248,G)' }
  ].map(s => ({ ...s, o: Math.random() * 100 }))

  const bStars = []
  for (let i = 0; i < 800; i++) {
    const isCluster = i > 720
    const ra = isCluster ? 1.46 + (Math.random() - 0.5) * 0.14 : Math.random() * Math.PI * 2
    const dec = isCluster ? -0.05 + (Math.random() - 0.5) * 0.14 : Math.acos(2 * Math.random() - 1) - Math.PI / 2
    bStars.push({
      ra, dec,
      r: Math.random() * Math.random() * 2.2 + 0.6,
      c: Math.random() > 0.2 ? cls[0] : cls[Math.floor(Math.random() * cls.length)],
      o: Math.random() * 100,
      g: isCluster && Math.random() > 0.5,
      gc: 'rgba(165,180,252,G)'
    })
  }

  stars = [...oStar, ...bStars]
  
  lines = [
    { a: oStar[0], b: oStar[2] },
    { a: oStar[2], b: oStar[3] },
    { a: oStar[3], b: oStar[4] },
    { a: oStar[4], b: oStar[1] },
    { a: oStar[3], b: oStar[5] }
  ]

  nebs = [
    genNeb(1.45, -0.05, 'rgba(139,92,246,0.12)', 14, 55),
    genNeb(1.48, -0.03, 'rgba(236,72,153,0.10)', 10, 40),
    genNeb(1.60, -0.20, 'rgba(6,182,212,0.08)', 12, 50),
    genNeb(3.20, 0.45, 'rgba(234,179,8,0.06)', 16, 75)
  ]

  draw()
})

onUnmounted(() => {
  window.removeEventListener('resize', res)
  if (cv.value) {
    cv.value.removeEventListener('touchstart', ts)
    cv.value.removeEventListener('touchmove', tm)
    cv.value.removeEventListener('touchend', te)
  }
  document.body.style.overflow = ''
  cancelAnimationFrame(rId)
})
</script>

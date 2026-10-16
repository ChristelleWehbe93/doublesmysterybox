<script setup>
import { computed, ref } from 'vue'
import Background from './Background.vue'

// item: [prompt, answer, options, why, a, b]
const levels = [
  { name: 'Skip Count', emoji: '🚀', mascot: '🐰', desc: 'Jump by 5s and 10s!', items: [
    ['5, 10, 15, __', 20, [20, 25, 30], 'Add 5 to 15.'],
    ['10, 20, __, 40', 30, [25, 30, 35], 'Add 10 to 20.'],
    ['0, 5, 10, __', 15, [15, 20, 25], 'Count five more.'],
    ['20, 30, __, 50', 40, [35, 45, 40], 'Add 10 to 30.'],
    ['5, 10, 15, 20, __', 25, [30, 25, 35], 'Add 5 to 20.'],
    ['0, 10, __, 30', 20, [20, 15, 25], 'Count by tens.'],
  ] },
  { name: 'Make 10', emoji: '🧩', mascot: '🦊', desc: 'Fill a ten-frame first!', items: [
    ['8 + 5 = 10 + __', 3, [3, 4, 5], '8 needs 2 to make 10. There are 3 left.', 8, 5],
    ['9 + 6 = 10 + __', 5, [6, 5, 4], '9 needs 1. There are 5 left.', 9, 6],
    ['7 + 8 = 10 + __', 5, [5, 6, 4], '7 needs 3. There are 5 left.', 7, 8],
    ['6 + 7 = 10 + __', 3, [2, 4, 3], '6 needs 4. There are 3 left.', 6, 7],
    ['8 + 6 = __', 14, [13, 14, 15], '8 + 2 + 4 = 10 + 4 = 14.', 8, 6],
    ['9 + 4 = __', 13, [12, 14, 13], '9 + 1 + 3 = 10 + 3 = 13.', 9, 4],
  ] },
  { name: 'Rewrite Sums', emoji: '✏️', mascot: '🐼', desc: 'Find the matching addition!', items: [
    ['9 + 4 = 10 + __', 3, [3, 4, 2], '9 + 1 + 3 = 10 + 3.'],
    ['6 + 8 = 10 + __', 4, [4, 6, 2], '6 + 4 + 4 = 10 + 4.'],
    ['7 + 5 = 7 + 3 + __', 2, [2, 3, 4], '5 is the same as 3 + 2.'],
    ['9 + 8 = 10 + __', 7, [8, 6, 7], '9 + 1 + 7 = 10 + 7.'],
    ['8 + 7 = 8 + 2 + __', 5, [6, 5, 4], '7 can be split into 2 and 5.'],
    ['6 + 9 = 10 + __', 5, [5, 6, 4], '6 + 4 + 5 = 10 + 5.'],
  ] },
  { name: 'Odd or Even', emoji: '🍎', mascot: '🐸', desc: 'Pair the apples! Any leftovers?', items: [
    ['8 apples: odd or even?', 'Even', ['Odd', 'Even', 'Neither'], '8 makes 4 complete pairs.', 8],
    ['7 apples: odd or even?', 'Odd', ['Even', 'Neither', 'Odd'], '7 makes 3 pairs and 1 leftover.', 7],
    ['11 apples: odd or even?', 'Odd', ['Odd', 'Even', 'Neither'], '11 makes 5 pairs and 1 leftover.', 11],
    ['12 apples: odd or even?', 'Even', ['Odd', 'Even', 'Neither'], '12 makes 6 complete pairs.', 12],
    ['15 apples: odd or even?', 'Odd', ['Even', 'Odd', 'Neither'], '15 makes 7 pairs and 1 leftover.', 15],
    ['16 apples: odd or even?', 'Even', ['Even', 'Odd', 'Neither'], '16 makes 8 complete pairs.', 16],
  ] },
]
const total = levels.reduce((s, l) => s + l.items.length, 0)

const shuffle = (a) => {
  const b = [...a]
  for (let i = b.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[b[i], b[j]] = [b[j], b[i]]
  }
  return b
}

const level = ref(0)
const idx = ref(0)
const score = ref(0)
const started = ref(false)
const finished = ref(false)
const picked = ref(null)
const soundOn = ref(true)
const motionOn = ref(true)
const confetti = ref([])
const order = ref([])
const options = ref([])

const l = computed(() => levels[level.value])
const q = computed(() => l.value.items[order.value[idx.value]])
const done = computed(() => levels.slice(0, level.value).reduce((s, x) => s + x.items.length, 0) + idx.value)
const step = computed(() => (/^(10, 20|20, 30|0, 10)/.test(q.value[0]) ? 10 : 5))
const last = computed(() => idx.value === l.value.items.length - 1)

function newRound() {
  order.value = shuffle(l.value.items.map((_, i) => i))
  options.value = shuffle(q.value[2])
}

let audio
function beep(ok) {
  if (!soundOn.value) return
  try {
    audio ??= new AudioContext()
    const o = audio.createOscillator()
    const g = audio.createGain()
    o.frequency.value = ok ? 660 : 240
    g.gain.setValueAtTime(0.08, audio.currentTime)
    g.gain.exponentialRampToValueAtTime(0.001, audio.currentTime + 0.23)
    o.connect(g).connect(audio.destination)
    o.start()
    o.stop(audio.currentTime + 0.23)
  } catch {}
}

function burst() {
  if (!motionOn.value) return
  const batch = Array.from({ length: 22 }, (_, i) => ({
    id: `${Date.now()}-${i}`,
    ch: ['⭐', '✨', '🎉', '💖', '🌟'][i % 5],
    left: Math.random() * 100,
    delay: Math.random() * 0.8,
  }))
  confetti.value.push(...batch)
  setTimeout(() => (confetti.value = confetti.value.filter((c) => !batch.includes(c))), 4500)
}

function start() {
  level.value = idx.value = score.value = 0
  finished.value = false
  started.value = true
  picked.value = null
  newRound()
}

function answer(i) {
  if (picked.value !== null) return
  picked.value = i
  const ok = options.value[i] === q.value[1]
  if (ok) score.value++
  beep(ok)
  if (ok) burst()
}

function next() {
  picked.value = null
  if (++idx.value >= l.value.items.length) {
    idx.value = 0
    if (++level.value === levels.length) {
      level.value--
      finished.value = true
      burst()
      return
    }
    newRound()
    return
  }
  options.value = shuffle(q.value[2])
}
</script>

<template>
  <div class="adv" :class="{ paused: !motionOn }">
    <Background />
    <span
      v-for="c in confetti"
      :key="c.id"
      class="confetti"
      :style="{ left: c.left + 'vw', animationDelay: c.delay + 's' }"
      >{{ c.ch }}</span
    >

    <div class="wrap">
      <header class="top">
        <div class="brand">🌈 The Road to 20!</div>
        <div class="score">⭐ {{ score }} stars</div>
      </header>
      <div class="scene" aria-hidden="true">
        <div class="sun">☀️</div>
        <div class="rainbow">🌈</div>
        <div class="cloud one">☁️</div>
        <div class="cloud two">☁️</div>
        <div class="hills"></div>
        <div class="runner">{{ l.mascot }}</div>
      </div>
      <div class="path">
        <div
          v-for="(x, i) in levels"
          :key="x.name"
          class="stop"
          :class="i === level && !finished ? 'active' : i < level || finished ? 'done' : ''"
        >
          <span class="em">{{ x.emoji }}</span
          >{{ x.name }}
        </div>
      </div>
      <div class="meter">
        <div class="bar" :style="{ width: (finished ? 100 : (100 * done) / total) + '%' }"></div>
      </div>

      <section :key="`${started}${finished}${level}${idx}`" class="panel" aria-live="polite">
        <template v-if="!started">
          <div class="hop-big">🦄</div>
          <h1>Welcome to Rainbow Quest!</h1>
          <p class="sub">Four magical math worlds • {{ total }} fun questions • collect stars!</p>
          <p style="font-size: 36px">🚀 🧩 ✏️ 🍎</p>
          <button class="btn" @click="start">🎮 Let's Play!</button>
        </template>

        <template v-else-if="finished">
          <div class="hop-big">🏆</div>
          <h1>YOU DID IT!</h1>
          <h2>⭐ {{ score }} / {{ total }} stars ⭐</h2>
          <p class="sub">You're a Math Adventure Champion!</p>
          <button class="btn" @click="start">🔄 Play Again</button>
        </template>

        <template v-else>
          <div class="kicker">WORLD {{ level + 1 }} OF {{ levels.length }} • QUESTION {{ idx + 1 }} OF {{ l.items.length }}</div>
          <h2>{{ l.emoji }} {{ l.name }}</h2>
          <div class="sub">{{ l.desc }}</div>
          <div class="art">
            <template v-if="level === 0">
              <div class="giant">🚀</div>
              <span class="number">+{{ step }}</span>
              <span class="arrow">➜</span>
              <span class="number">❓</span>
              <div class="giant">🌟</div>
            </template>
            <template v-else-if="level === 1">
              <template v-for="(n, k) in [q[4], q[5]]" :key="k">
                <span v-if="k" class="arrow">➕</span>
                <div class="ten" :aria-label="`${n} counters`">
                  <div
                    v-for="i in 10"
                    :key="i"
                    class="cell"
                    :class="i <= n && ['on', k && 'alt']"
                    :style="{ animationDelay: (i - 1) * 0.045 + 's' }"
                  ></div>
                </div>
              </template>
            </template>
            <template v-else-if="level === 2">
              <div class="giant">🧠</div>
              <div class="number">🔟</div>
              <div class="giant">🪄</div>
            </template>
            <template v-else>
              <div v-for="i in Math.floor(q[4] / 2)" :key="i" class="pair" :style="{ animationDelay: (i - 1) * 0.06 + 's' }">🍎🍎</div>
              <div v-if="q[4] % 2" class="pair single">🍎</div>
            </template>
          </div>
          <h1>{{ q[0] }}</h1>
          <div class="answers">
            <button
              v-for="(a, i) in options"
              :key="a"
              class="answer"
              :disabled="picked !== null"
              :class="picked !== null && (a === q[1] ? 'correct' : i === picked && 'wrong')"
              @click="answer(i)"
            >
              {{ a }}
            </button>
          </div>
          <template v-if="picked !== null">
            <div class="feedback">
              {{ options[picked] === q[1] ? '🌟 Fantastic! You earned a star!' : '💡 Good try!' }}
              <div style="font-size: 16px; margin-top: 5px">{{ q[3] }}</div>
            </div>
            <button class="btn" @click="next">
              {{ last ? (level === levels.length - 1 ? '🏆 See My Prize' : '🔓 Next World') : '➡️ Next Question' }}
            </button>
          </template>
        </template>
      </section>

      <div class="settings">
        <button class="smallbtn" @click="soundOn = !soundOn">{{ soundOn ? '🔊 Sound: On' : '🔇 Sound: Off' }}</button>
        <button class="smallbtn" @click="motionOn = !motionOn">{{ motionOn ? '🎞️ Motion: On' : '⏸️ Motion: Off' }}</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.adv {
  color: #26325b;
  font-family: 'Trebuchet MS', system-ui, sans-serif;
  min-height: 100vh;
  margin: -0.75rem;
  overflow-x: hidden;
}
.adv.paused,
.adv.paused :deep(*) {
  animation-play-state: paused;
}
button { font: inherit; cursor: pointer }
button:focus-visible { outline: 4px solid #ffad00; outline-offset: 4px }
.cloud { position: absolute; font-size: clamp(55px, 10vw, 115px); opacity: 0.58; animation: drift linear infinite }
.cloud.one { top: 8%; left: -150px; animation-duration: 37s }
.cloud.two { top: 35%; left: -200px; animation-duration: 53s; animation-delay: -20s }
@keyframes drift { to { transform: translateX(calc(100vw + 360px)) } }
.wrap { position: relative; z-index: 1; max-width: 990px; margin: auto; padding: 18px 14px 45px }
.top { display: flex; align-items: center; justify-content: space-between; gap: 10px; flex-wrap: wrap }
.brand { font-size: clamp(22px, 4vw, 37px); font-weight: 900; text-shadow: 0 2px white }
.score { padding: 12px 18px; background: #fff; border: 3px solid #ffe28d; border-radius: 28px; font-weight: 900; box-shadow: 0 5px 0 #eacb79 }
.scene { margin: 18px 0; background: linear-gradient(#9ce2ff, #e9f9ff 72%); height: 195px; border: 5px solid #fff; border-radius: 28px; overflow: hidden; position: relative; box-shadow: 0 9px 0 #bdd1ef }
.sun { position: absolute; right: 12%; top: 10px; font-size: 65px; animation: sunspin 20s linear infinite }
@keyframes sunspin { to { transform: rotate(360deg) } }
.rainbow { position: absolute; left: 6%; top: 5px; font-size: 80px; animation: bob 3s ease-in-out infinite }
.hills { position: absolute; bottom: -62px; left: -10%; width: 120%; height: 135px; border-radius: 50%; background: #65d59c; box-shadow: 0 -12px 0 #91e6ad inset }
.runner { position: absolute; bottom: 22px; left: 44%; font-size: 88px; animation: hop 1.4s ease-in-out infinite; filter: drop-shadow(0 8px 2px #6bb98b) }
@keyframes hop { 50% { transform: translateY(-19px) rotate(-6deg) } }
@keyframes bob { 50% { transform: translateY(12px) } }
.hop-big { font-size: 85px; animation: hop 2s infinite }
.scene .cloud { font-size: 45px; opacity: 0.9 }
.scene .cloud.one { top: 15px }
.scene .cloud.two { top: 62px }
.path { display: grid; grid-template-columns: repeat(4, 1fr); gap: 8px; margin: 15px 0 }
.stop { border: 3px solid #fff; border-radius: 18px; background: #ffffffc9; text-align: center; padding: 10px 4px; font-weight: 900; font-size: clamp(10px, 2.3vw, 15px); box-shadow: 0 4px 0 #c7d6eb }
.stop .em { font-size: clamp(23px, 4vw, 38px); display: block; animation: bob 2.5s infinite }
.stop.active { background: #fff1b2; border-color: #ffc63e }
.stop.done { background: #d7ffeb; border-color: #52cf9e }
.meter { height: 15px; background: #fff; border-radius: 30px; overflow: hidden; border: 2px solid white; margin-bottom: 17px }
.bar { height: 100%; background: linear-gradient(90deg, #20d6a5, #58a1ff, #c17bff); transition: width 0.5s }
.panel { background: #fffefa; border: 4px solid white; border-radius: 32px; box-shadow: 0 10px 0 #b9cce6; padding: clamp(18px, 4vw, 32px); text-align: center; min-height: 360px; animation: appear 0.45s both }
@keyframes appear { from { opacity: 0; transform: translateY(16px) scale(0.98) } to { opacity: 1; transform: none } }
h1 { font-size: clamp(26px, 5vw, 45px); font-weight: 700; margin: 12px 0 }
h2 { font-size: clamp(23px, 4vw, 33px); font-weight: 700; margin: 12px 0 }
.kicker { font-size: 14px; font-weight: 900; color: #66789d }
.sub { font-weight: 700; color: #65728b }
.art { min-height: 135px; display: flex; flex-wrap: wrap; align-items: center; justify-content: center; gap: 12px; margin: 16px auto; padding: 15px; background: linear-gradient(130deg, #eaf8ff, #fff2f8); border: 3px dashed #bddaff; border-radius: 25px; max-width: 750px }
.art .giant { font-size: clamp(45px, 9vw, 90px); animation: bob 2s infinite }
.number { font-size: clamp(24px, 5vw, 43px); font-weight: 900; padding: 8px 14px; border-radius: 18px; background: white; box-shadow: 0 4px 0 #d9e7fa }
.arrow { font-size: 35px; color: #ff6d9e }
.ten { display: grid; grid-template-columns: repeat(5, 1fr); gap: 5px; padding: 8px; background: white; border: 3px solid #a9c8f4; border-radius: 14px; width: 190px }
.cell { aspect-ratio: 1; border: 2px solid #e0e9f8; border-radius: 50%; background: #f7f9ff }
.cell.on { background: #ff9b59; border-color: #f17736; box-shadow: inset -4px -4px 0 #ed763f; animation: pop 0.4s backwards }
.cell.on.alt { background: #6dd9ba; border-color: #2ab79b; box-shadow: inset -4px -4px 0 #3cb79b }
@keyframes pop { from { transform: scale(0.3) } to { transform: scale(1) } }
.pair { display: inline-flex; gap: 3px; padding: 7px; border: 3px solid #9cddcb; border-radius: 16px; background: white; font-size: 29px; animation: pop 0.45s backwards }
.single { border-color: #ff95a8 }
.answers { display: grid; grid-template-columns: repeat(3, 1fr); gap: 13px; margin: 16px auto; max-width: 760px }
.answer { border: 3px solid #c5d6fb; background: #f4f8ff; border-radius: 22px; padding: 19px 8px; font-size: clamp(21px, 4vw, 32px); font-weight: 900; box-shadow: 0 6px 0 #c5d6fb; transition: transform 0.15s }
.answer:hover:not(:disabled) { transform: translateY(-5px); background: #e7f3ff }
.answer:disabled { cursor: default }
.answer.correct { background: #d4ffe9; border-color: #26bc85; box-shadow: 0 6px 0 #26bc85 }
.answer.wrong { background: #ffe2e7; border-color: #ff6c87; box-shadow: 0 6px 0 #ff6c87 }
.feedback { background: #fff3cb; border-radius: 18px; padding: 13px; font-size: 18px; font-weight: 800; margin: 14px auto; max-width: 760px }
.btn { border: 0; border-radius: 25px; background: linear-gradient(120deg, #ff7a9c, #f54f88); color: white; font-size: clamp(19px, 3vw, 25px); font-weight: 900; padding: 17px 32px; box-shadow: 0 7px 0 #c5366c; transition: transform 0.15s }
.btn:hover { transform: translateY(-3px) }
.btn:active { transform: translateY(4px); box-shadow: 0 2px 0 #c5366c }
.settings { display: flex; justify-content: center; gap: 15px; align-items: center; flex-wrap: wrap; margin: 16px }
.smallbtn { border: 2px solid #a9c6ef; background: white; padding: 8px 14px; border-radius: 14px; font-weight: 900 }
.confetti { position: fixed; z-index: 5; top: -30px; pointer-events: none; animation: fall 3.5s linear forwards; font-size: 26px }
@keyframes fall { to { transform: translateY(110vh) rotate(720deg); opacity: 0 } }
@media (max-width: 520px) {
  .answers { gap: 6px }
  .answer { padding: 14px 3px }
  .scene { height: 160px }
  .runner { font-size: 67px }
  .ten { width: 142px; gap: 3px }
  .pair { font-size: 22px; padding: 4px }
  .art { gap: 7px }
}
@media (prefers-reduced-motion: reduce) {
  * { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important }
}
</style>

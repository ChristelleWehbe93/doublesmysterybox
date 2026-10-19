<script setup>
import { computed, ref } from 'vue'
import Background from './Background.vue'

const levels = {
  beginner: { label: '🌱 Beginner', range: [1, 10], on: 'bg-emerald-500 shadow-[0_5px_#18734d]' },
  medium: { label: '🚀 Medium', range: [10, 20], on: 'bg-amber-500 shadow-[0_5px_#a26612]' },
  challenge: { label: '🔥 Challenge', range: [20, 50], on: 'bg-rose-500 shadow-[0_5px_#a42c47]' },
}
const titles = ['🐊 Feed the Greater Sum!', '⚖️ Balance the Sums!', '🌟 Compare and Win!']
const answerBg = ['bg-violet-500 shadow-[0_5px_#52359b]', 'bg-pink-500 shadow-[0_5px_#a43c69]', 'bg-teal-500 shadow-[0_5px_#127b68]']
const TOTAL = 12

const rand = (min, max) => Math.floor(Math.random() * (max - min + 1)) + min
const sum = (a) => a.reduce((x, y) => x + y, 0)
const str = (a) => a.join(' + ')
const shuffle = (a) => {
  const b = [...a]
  for (let i = b.length - 1; i > 0; i--) {
    const j = rand(0, i)
    ;[b[i], b[j]] = [b[j], b[i]]
  }
  return b
}

// a number, or a split into two addends
function expressionFor(total) {
  if (total <= 1 || Math.random() < 0.13) return [total]
  let low = Math.max(1, total - 25)
  let high = Math.min(total - 1, 25)
  if (low > high) [low, high] = [1, total - 1]
  const first = rand(low, high)
  return [first, total - first]
}

// even mix of equal, greater and less comparisons, no duplicates
function generateQuestions(difficulty) {
  const [lo, hi] = levels[difficulty].range
  const seen = new Set()
  const out = []
  for (let tries = 0; out.length < TOTAL && tries < 2000; tries++) {
    const target = out.length % 3
    let x = rand(lo, hi)
    let y = target === 0 ? x : rand(lo, hi)
    if (target === 1 && x <= y) [x, y] = [Math.max(x, y), Math.min(x, y)]
    if (target === 2 && x >= y) [x, y] = [Math.min(x, y), Math.max(x, y)]
    if (target !== 0 && x === y) continue
    const key = `${x}:${y}`
    if (seen.has(key)) continue
    seen.add(key)
    out.push([expressionFor(x), expressionFor(y)])
  }
  return shuffle(out)
}

const difficulty = ref('beginner')
const scores = ref([0, 0])
const team = ref(0)
const idx = ref(0)
const questions = ref([])
const picked = ref(null)
const points = ref(0)
const finished = ref(false)
const showPics = ref(false)
const soundOn = ref(true)
const confetti = ref([])

const teamName = computed(() => (team.value === 0 ? 'Red' : 'Yellow'))
const cur = computed(() => {
  const [a, b] = questions.value[idx.value]
  const av = sum(a)
  const bv = sum(b)
  return { a, b, av, bv, correct: av > bv ? '>' : av < bv ? '<' : '=' }
})
const ok = computed(() => picked.value === cur.value.correct)
const verdict = computed(() => ({ '=': 'Both sides are equal!', '>': 'The left side is greater!', '<': 'The left side is less!' })[cur.value.correct])
const winner = computed(() => {
  const [r, y] = scores.value
  return r === y ? '🤝 It’s a Tie!' : r > y ? '🔴 TEAM RED WINS!' : '🟡 TEAM YELLOW WINS!'
})

let audio
function beep(good) {
  if (!soundOn.value) return
  try {
    audio ??= new AudioContext()
    const o = audio.createOscillator()
    const g = audio.createGain()
    const t = audio.currentTime
    o.frequency.setValueAtTime(good ? 660 : 260, t)
    o.frequency.exponentialRampToValueAtTime(good ? 950 : 180, t + 0.2)
    g.gain.setValueAtTime(0.13, t)
    g.gain.exponentialRampToValueAtTime(0.001, t + 0.27)
    o.connect(g).connect(audio.destination)
    o.start()
    o.stop(t + 0.28)
  } catch {}
}

function burst() {
  const batch = Array.from({ length: 35 }, (_, i) => ({
    id: `${Date.now()}-${i}`,
    ch: ['⭐', '🎉', '✨', '🌟'][i % 4],
    left: Math.random() * 100,
    size: 15 + Math.random() * 25,
    delay: Math.random() * 0.6,
  }))
  confetti.value.push(...batch)
  setTimeout(() => (confetti.value = confetti.value.filter((c) => !batch.includes(c))), 3400)
}

function start(level = difficulty.value) {
  difficulty.value = level
  scores.value = [0, 0]
  team.value = idx.value = 0
  picked.value = null
  finished.value = false
  questions.value = generateQuestions(level)
}

function answer(choice) {
  if (picked.value) return
  picked.value = choice
  if (ok.value) {
    points.value = [10, 15, 20, 25][rand(0, 3)]
    scores.value[team.value] += points.value
    burst()
  }
  beep(ok.value)
}

function next() {
  picked.value = null
  team.value = 1 - team.value
  if (++idx.value === TOTAL) {
    finished.value = true
    burst()
  }
}

const reset = () => confirm('Start a new game and reset scores?') && start()

start()
</script>

<template>
  <div class="relative">
    <Background />
    <span
      v-for="c in confetti"
      :key="c.id"
      class="sb-fall pointer-events-none fixed top-[-20px] z-[100]"
      :style="{ left: c.left + 'vw', fontSize: c.size + 'px', animationDelay: c.delay + 's' }"
      >{{ c.ch }}</span
    >

    <div class="relative z-10 text-[#183452]">
      <header class="text-center">
        <h1 class="m-1 text-4xl font-black text-white [text-shadow:0_5px_#2574b2,0_8px_18px_#22628a] sm:text-6xl">🐊 SUM BATTLE! ⭐</h1>
        <p class="m-1 font-bold text-[#205176]">The Great Comparison Team Challenge • Grade 2</p>
      </header>

      <section class="mt-4 grid grid-cols-2 gap-3">
        <div
          v-for="(t, i) in [
            { n: 'RED', e: '🔴', c: 'bg-[#ffe0e0] text-[#aa2031]' },
            { n: 'YELLOW', e: '🟡', c: 'bg-[#fff3b0] text-[#795400]' },
          ]"
          :key="t.n"
          class="rounded-3xl border-[5px] p-3 text-center text-base font-black transition sm:text-2xl"
          :class="[t.c, team === i && !finished ? 'scale-[1.025] border-[#ffb600] ring-4 ring-white' : 'border-transparent']"
        >
          {{ t.e }} TEAM {{ t.n }}
          <div class="text-3xl sm:text-5xl">{{ scores[i] }} ⭐</div>
        </div>
      </section>

      <section class="my-4 rounded-[28px] border-[5px] border-white bg-white/95 p-3 shadow-[0_12px_0_#357a9a44,0_18px_30px_#15517a33] sm:p-5">
        <div class="mb-3 rounded-3xl border-[3px] border-dashed border-[#8dd4f3] bg-[#f2fbff] p-3 text-center">
          <strong>🎯 Choose your level:</strong>
          <div class="my-3 flex flex-wrap justify-center gap-2">
            <button
              v-for="(lv, k) in levels"
              :key="k"
              class="min-w-24 cursor-pointer rounded-2xl px-2 py-2.5 text-[15px] leading-tight font-black text-white transition active:translate-y-1 active:shadow-none sm:min-w-36 sm:px-5 sm:text-xl"
              :class="difficulty === k ? [lv.on, 'outline-4 -outline-offset-6 outline-white -translate-y-0.5'] : 'bg-[#7c9ab2] shadow-[0_5px_#47677e]'"
              @click="start(k)"
            >
              {{ lv.label }}<br /><small class="text-sm">{{ lv.range[0] }}–{{ lv.range[1] }}</small>
            </button>
          </div>
          <div class="text-[13px] text-[#55758b]">Choose a level to start a fresh game with new equations!</div>
        </div>

        <div v-if="finished" class="text-center">
          <div class="sb-bob text-8xl">🏆🎊</div>
          <h2 class="text-2xl font-bold">MISSION COMPLETE!</h2>
          <div class="text-3xl font-black text-[#da5b8b] sm:text-5xl">{{ winner }}</div>
          <p class="my-3">Team Red: {{ scores[0] }} ⭐ • Team Yellow: {{ scores[1] }} ⭐</p>
          <button class="cursor-pointer rounded-2xl bg-[#f4a900] px-5 py-3 text-xl font-black text-[#452d00] shadow-[0_5px_#ab6b00] active:translate-y-1" @click="start()">Play Again 🎮</button>
        </div>

        <div v-else class="text-center">
          <div class="mx-auto mt-1.5 mb-4 max-w-2xl rounded-2xl border-[3px] border-[#ffcd55] bg-[#fff0b5] p-3 text-xl font-bold text-[#426a87] sm:text-3xl">
            {{ team === 0 ? '🔴 TEAM RED’S TURN' : '🟡 TEAM YELLOW’S TURN' }} • {{ difficulty.toUpperCase() }} • Challenge {{ idx + 1 }} of {{ TOTAL }}
          </div>
          <h2 class="text-2xl font-bold">{{ titles[idx % 3] }}</h2>

          <div class="my-5 flex flex-wrap items-center justify-center gap-3 text-3xl font-black sm:text-5xl">
            <span class="min-w-30 rounded-2xl bg-[#e8f4ff] px-5 py-3">{{ str(cur.a) }}</span>
            <span class="text-5xl text-[#ee5a7c]">?</span>
            <span class="min-w-30 rounded-2xl bg-[#e8f4ff] px-5 py-3">{{ str(cur.b) }}</span>
          </div>

          <div v-if="showPics" class="my-2 flex flex-wrap items-center justify-around gap-2">
            <div v-for="(n, s) in [cur.av, cur.bv]" :key="s">
              <div class="mb-1 text-sm font-bold">{{ s ? 'Right' : 'Left' }} side</div>
              <div class="flex max-w-[270px] flex-wrap justify-center gap-1">
                <i
                  v-for="d in n"
                  :key="d"
                  class="size-[17px] rounded-full border-2"
                  :class="s ? 'border-[#259f82] bg-[#60cfa5]' : 'border-[#e47a32] bg-[#ffae37]'"
                ></i>
              </div>
            </div>
          </div>

          <p>Choose the correct comparison symbol:</p>
          <div class="flex flex-wrap justify-center gap-4">
            <button
              v-for="(s, i) in ['>', '<', '=']"
              :key="s"
              :disabled="!!picked"
              class="min-w-20 cursor-pointer rounded-2xl px-5 py-3 text-4xl font-black text-white transition active:translate-y-1 active:shadow-none sm:min-w-26 sm:text-5xl"
              :class="[
                answerBg[i],
                picked && s === cur.correct && 'outline-[5px] outline-[#3ac681]',
                picked === s && !ok && 'outline-[5px] outline-[#f26a6a]',
                picked && s !== cur.correct && picked !== s && 'opacity-50',
              ]"
              @click="answer(s)"
            >
              {{ s }}
            </button>
          </div>

          <template v-if="picked">
            <div class="m-4 text-2xl font-black" aria-live="polite">
              {{ ok ? `🎉 Correct! +${points} points for Team ${teamName}!` : `💪 Nice try! The correct sign is ${cur.correct}.` }}
            </div>
            <div class="mx-auto my-4 max-w-2xl rounded-2xl bg-[#fff5c9] p-4 text-xl leading-normal">
              <strong>{{ str(cur.a) }} = {{ cur.av }}</strong> and <strong>{{ str(cur.b) }} = {{ cur.bv }}</strong><br />
              {{ cur.av }} {{ cur.correct }} {{ cur.bv }} — {{ verdict }}
            </div>
            <button class="cursor-pointer rounded-2xl bg-[#f4a900] px-5 py-3 text-xl font-black text-[#452d00] shadow-[0_5px_#ab6b00] active:translate-y-1" @click="next">
              {{ idx === TOTAL - 1 ? '🏆 See the Winner' : 'Next Challenge ➜' }}
            </button>
          </template>
        </div>

        <div class="my-3 flex flex-wrap items-center justify-center gap-2.5">
          <label class="font-bold"><input v-model="showPics" type="checkbox" class="mr-1 size-5 align-middle" /> Show counting pictures</label>
          <button class="cursor-pointer rounded-2xl bg-[#3678d4] px-3 py-2.5 text-sm font-black text-white shadow-[0_5px_#1d4b8b] active:translate-y-1" @click="soundOn = !soundOn">
            {{ soundOn ? '🔊 Sound On' : '🔇 Sound Off' }}
          </button>
          <button class="cursor-pointer rounded-2xl bg-[#e75e98] px-3 py-2.5 text-sm font-black text-white shadow-[0_5px_#a53665] active:translate-y-1" @click="reset">↻ New Game</button>
        </div>
      </section>
    </div>
  </div>
</template>

<style scoped>
.sb-fall { animation: sb-fall 2.6s linear forwards; }
.sb-bob { animation: sb-bob 2s ease-in-out infinite; }
@keyframes sb-fall { to { transform: translateY(110vh) rotate(760deg); opacity: 0; } }
@keyframes sb-bob { 50% { transform: translateY(-12px) rotate(10deg); } }
</style>

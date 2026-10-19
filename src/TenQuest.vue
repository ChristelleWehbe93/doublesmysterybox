<script setup>
import { computed, ref } from 'vue'

const levels = {
  beginner: { label: '🌱 Beginner', sub: 'Within 10', max: 10 },
  medium: { label: '⭐ Medium', sub: 'Within 20', max: 20 },
  challenge: { label: '🔥 Challenge', sub: 'Within 50', max: 50 },
}
const TOTAL = 12
const planetBg = [
  'bg-[radial-gradient(circle_at_35%_25%,#c7ffd9,#48c5aa_48%,#237cb2)]',
  'bg-[radial-gradient(circle_at_35%_25%,#a9edff,#6179dc_45%,#624ab1_80%)]',
  'bg-[radial-gradient(circle_at_35%_25%,#ffebb6,#f6a15e_48%,#d34b8b)]',
]

const rand = (a, b) => Math.floor(Math.random() * (b - a + 1)) + a
const shuffle = (a) => {
  for (let i = a.length - 1; i > 0; i--) {
    const j = rand(0, i)
    ;[a[i], a[j]] = [a[j], a[i]]
  }
  return a
}

function makeQuestion(level, used) {
  let a, b, key
  for (let tries = 0; tries < 250; tries++) {
    if (level === 'beginner') {
      a = rand(4, 10)
      b = rand(1, a)
    } else if (level === 'medium') {
      a = rand(11, 20)
      b = Math.random() < 0.78 ? rand(a - 9, Math.min(a, 12)) : rand(2, Math.min(a, 12))
    } else {
      a = rand(21, 50)
      b = Math.random() < 0.22 ? rand(1, Math.min(a, 20)) : rand((a % 10) + 1, Math.min(a, 20))
    }
    key = `${a}-${b}`
    if (!used.has(key)) break
  }
  used.add(key)
  const result = a - b
  const options = new Set([result])
  while (options.size < 3) {
    const n = result + rand(1, 6) * (Math.random() < 0.5 ? -1 : 1)
    if (n >= 0 && n <= levels[level].max) options.add(n)
  }
  return { a, b, result, options: shuffle([...options]) }
}

const level = ref('medium')
const questions = ref([])
const done = ref(new Set())
const turn = ref(0)
const scores = ref([0, 0])
const current = ref(null)
const picked = ref(null)
const showHint = ref(false)
const soundOn = ref(true)
const pop = ref(false)

const q = computed(() => questions.value[current.value])
const ok = computed(() => picked.value === q.value.result)
const finished = computed(() => done.value.size === TOTAL)
const teamLabel = computed(() => (turn.value % 2 ? '🔵 TEAM BLUE' : '🔴 TEAM RED'))
const status = computed(() =>
  finished.value ? '🌟 MISSION COMPLETE!' : `${teamLabel.value}'S TURN — ${current.value === null ? 'Choose a planet!' : 'Solve the equation!'}`,
)
const winner = computed(() => {
  const [r, b] = scores.value
  return `${r === b ? '🤝 IT’S A TIE!' : r > b ? '🏆 TEAM RED WINS! 🔴' : '🏆 TEAM BLUE WINS! 🔵'}  ${r} – ${b}`
})
const hint = computed(() => {
  const { a, b } = q.value
  const tens = Math.floor(a / 10) * 10
  const ones = a - tens
  const remaining = b - Math.min(b, ones)
  const text =
    level.value === 'beginner'
      ? `Start with ${a}. Take away ${b}. You can count back or use counters.`
      : remaining === 0
        ? `Start at ${a}. Subtract ${b}. Try separating the tens and ones.`
        : `Step 1: ${a} − ${ones} = ${tens} (reach a ten). Step 2: You still need to subtract ${remaining}. Think: ${tens} − ${remaining} = ?`
  return `💡 USING 10: ${text}`
})

let audio
function beep(good) {
  if (!soundOn.value) return
  try {
    audio ??= new AudioContext()
    const o = audio.createOscillator()
    const g = audio.createGain()
    const t = audio.currentTime
    o.frequency.setValueAtTime(good ? 640 : 240, t)
    if (good) o.frequency.exponentialRampToValueAtTime(950, t + 0.16)
    g.gain.setValueAtTime(0.07, t)
    g.gain.exponentialRampToValueAtTime(0.001, t + 0.25)
    o.connect(g).connect(audio.destination)
    o.start()
    o.stop(t + 0.26)
  } catch {}
}

function newGame(lv = level.value) {
  level.value = lv
  const used = new Set()
  questions.value = Array.from({ length: TOTAL }, () => makeQuestion(lv, used))
  done.value = new Set()
  turn.value = 0
  scores.value = [0, 0]
  current.value = picked.value = null
}

function open(i) {
  if (current.value !== null || done.value.has(i)) return
  current.value = i
  picked.value = null
  showHint.value = false
}

function choose(n) {
  if (picked.value !== null) return
  picked.value = n
  if (ok.value) {
    scores.value[turn.value % 2] += 10
    pop.value = true
    setTimeout(() => (pop.value = false), 500)
  }
  beep(ok.value)
}

function next() {
  done.value.add(current.value)
  current.value = null
  turn.value++
}

newGame()

const btn = 'cursor-pointer rounded-2xl px-5 py-3 text-lg font-black text-[#1d2b57] focus-visible:outline-4 focus-visible:outline-offset-2 focus-visible:outline-[#ffea75] disabled:cursor-default disabled:opacity-40'
const panel = 'rounded-3xl border-2 border-[#607bc2] bg-[#152454]/90 p-3 shadow-[0_14px_35px_#050c2999] backdrop-blur-sm sm:p-5'
</script>

<template>
  <div class="relative text-white">
    <div class="fixed inset-0 bg-[radial-gradient(ellipse_at_50%_90%,#253c82,#101744_55%,#090d2b)]"></div>
    <div class="tq-stars pointer-events-none fixed inset-0 opacity-60"></div>
    <div class="tq-float pointer-events-none fixed top-[8%] -left-[85px] size-[180px] rounded-full bg-[radial-gradient(circle_at_30%_25%,#a5edff,#3766c8_60%,#25346c)] drop-shadow-[0_0_30px_#86a5ff55]"></div>
    <div class="tq-float pointer-events-none fixed -right-[45px] bottom-[10%] size-[120px] rounded-full bg-[radial-gradient(circle_at_25%_25%,#ffc3a4,#e85a88_70%,#a33d8a)] drop-shadow-[0_0_30px_#86a5ff55] [animation-delay:-3s]"></div>

    <div class="relative z-10 pb-10">
      <header class="text-center">
        <div class="text-xs font-black tracking-[3px] text-[#b8d5ff]">GRADE 2 • TEAM MATH ADVENTURE</div>
        <h1 class="my-1 text-5xl font-black text-[#ffeb80] [text-shadow:0_5px_#3d3d90] sm:text-7xl">🚀 TEN QUEST!</h1>
        <div class="font-bold text-[#dce9ff]">The Subtraction Space Race <span class="tq-ship inline-block text-4xl">🛸</span></div>
      </header>

      <div class="my-4 flex flex-wrap items-center justify-center gap-2.5">
        <button
          v-for="(lv, k) in levels"
          :key="k"
          class="cursor-pointer rounded-2xl border-2 px-4 py-3 text-[15px] font-black"
          :class="level === k ? 'border-white bg-[#f6cb4d] text-[#202b59] shadow-[0_0_18px_#f6cb4d88]' : 'border-[#8da6ef] bg-[#283b79]'"
          @click="newGame(k)"
        >
          {{ lv.label }}<br />{{ lv.sub }}
        </button>
      </div>

      <div class="my-3 grid grid-cols-2 gap-3">
        <div
          v-for="(t, i) in [
            { n: '🔴 TEAM RED', c: 'from-[#b82346] to-[#f0525d]' },
            { n: '🔵 TEAM BLUE', c: 'from-[#1456b7] to-[#29a8e9]' },
          ]"
          :key="i"
          class="rounded-3xl border-[3px] bg-gradient-to-br p-3 text-center transition"
          :class="[t.c, !finished && turn % 2 === i ? 'scale-[1.02] border-[#fff2a6] shadow-[0_0_24px_#ffed8a88]' : 'border-transparent opacity-80']"
        >
          <div class="text-lg font-black sm:text-2xl">{{ t.n }}</div>
          <div class="text-4xl font-black">{{ scores[i] }}</div>
        </div>
      </div>

      <div class="mb-4 rounded-2xl bg-[#fff2b8] p-3 text-center text-lg font-black text-[#25305b]">{{ status }}</div>

      <section v-if="finished" :class="panel" class="text-center">
        <div class="text-3xl font-black text-[#ffe580] sm:text-5xl">{{ winner }}</div>
        <p class="my-3 text-sm text-[#cbdcff]">All planets explored! Amazing teamwork!</p>
        <button :class="btn" class="bg-[#ffd65a]" @click="newGame()">🚀 Play Again with New Equations</button>
      </section>

      <section v-else-if="current === null" :class="panel">
        <div class="text-center text-sm text-[#cbdcff]">{{ done.size }} of {{ TOTAL }} planets explored • Choose a planet to reveal a question</div>
        <div class="my-5 grid grid-cols-4 gap-2 sm:grid-cols-6 sm:gap-3">
          <button
            v-for="i in TOTAL"
            :key="i"
            :disabled="done.has(i - 1)"
            :aria-label="`Explore planet ${i}`"
            class="tq-bob aspect-square min-w-0 cursor-pointer rounded-full border-2 border-[#9fe4ff] text-2xl font-black text-white shadow-[inset_-8px_-10px_15px_#1d226977,0_7px_0_#101b4c,0_0_17px_#9edcff66] transition hover:not-disabled:scale-110 disabled:cursor-default disabled:opacity-35 disabled:grayscale sm:text-4xl"
            :class="planetBg[i % 3]"
            @click="open(i - 1)"
          >
            {{ i }}
          </button>
        </div>
        <div class="flex flex-wrap justify-center gap-2.5">
          <button :class="btn" class="bg-[#c4d8ff]" @click="newGame()">🔄 New Game • New Equations</button>
          <button :class="btn" class="bg-[#c4d8ff]" @click="soundOn = !soundOn">{{ soundOn ? '🔊 Sound On' : '🔇 Sound Off' }}</button>
        </div>
      </section>

      <section v-else :class="[panel, pop && 'tq-pop']" class="text-center">
        <div class="text-xs font-black tracking-[3px] text-[#b8d5ff]">🪐 PLANET {{ current + 1 }} • {{ level.toUpperCase() }}</div>
        <div class="text-sm text-[#cbdcff]">Solve the subtraction. Choose the correct answer!</div>
        <div class="my-3 text-5xl font-black text-[#fff1a3] [text-shadow:0_5px_#293872] sm:text-7xl">{{ q.a }} − {{ q.b }} = ?</div>
        <div class="mt-4 grid grid-cols-3 gap-2 sm:gap-3">
          <button
            v-for="n in q.options"
            :key="n"
            :disabled="picked !== null"
            class="cursor-pointer rounded-2xl border-[3px] border-[#a8cbff] py-4 text-3xl font-black text-[#25336b] transition focus-visible:outline-4 focus-visible:outline-offset-2 focus-visible:outline-[#ffea75] enabled:hover:-translate-y-0.5 enabled:hover:bg-[#fff2a4] disabled:opacity-70 sm:text-4xl"
            :class="picked === null ? 'bg-[#f2f7ff]' : n === q.result ? 'bg-[#b6f9c7]' : n === picked ? 'bg-[#ffd0d0]' : 'bg-[#f2f7ff]'"
            @click="choose(n)"
          >
            {{ n }}
          </button>
        </div>
        <div class="my-4 min-h-8 text-lg font-black" :class="ok ? 'text-[#a9ffcb]' : 'text-[#ffd6a3]'" aria-live="polite">
          <template v-if="picked !== null">{{ ok ? '🌟 Correct! +10 points!' : `💪 Nice try! The correct answer is ${q.result}.` }}</template>
        </div>
        <div class="flex flex-wrap justify-center gap-2.5">
          <button :class="btn" class="bg-[#c4d8ff]" @click="showHint = !showHint">💡 Show a Using-10 Hint</button>
          <button v-if="picked !== null" :class="btn" class="bg-[#ffd65a]" @click="next">{{ done.size === TOTAL - 1 ? '🏆 See Results' : '➡️ Next Turn' }}</button>
        </div>
        <div v-if="showHint" class="mt-3 rounded-2xl border border-[#9bcaff] bg-[#243c76] p-3 text-left leading-relaxed">{{ hint }}</div>
      </section>

      <div class="mt-4 text-center text-xs text-[#d1dcff]">No answers are shown until a team makes a choice. Hints are optional. Questions change every new game.</div>
    </div>
  </div>
</template>

<style scoped>
.tq-stars {
  background-image: radial-gradient(2px 2px at 10% 15%, white 90%, transparent), radial-gradient(2px 2px at 75% 30%, white 90%, transparent),
    radial-gradient(3px 3px at 40% 65%, #b5edff 90%, transparent), radial-gradient(2px 2px at 90% 80%, white 90%, transparent);
  background-size: 230px 190px;
  animation: tq-drift 35s linear infinite;
}
.tq-float { animation: tq-float 8s ease-in-out infinite; }
.tq-bob { animation: tq-bob 3s ease-in-out infinite; }
.tq-bob:nth-child(even) { animation-delay: -1.4s; }
.tq-ship { animation: tq-ship 2.5s ease-in-out infinite; }
.tq-pop { animation: tq-pop 0.45s ease; }
@keyframes tq-drift { to { background-position: 460px 190px; } }
@keyframes tq-float { 50% { transform: translateY(-20px) rotate(8deg); } }
@keyframes tq-bob { 50% { translate: 0 -7px; } }
@keyframes tq-ship { 50% { transform: translateY(-9px) rotate(-7deg); } }
@keyframes tq-pop { 50% { transform: scale(1.06); } }
</style>

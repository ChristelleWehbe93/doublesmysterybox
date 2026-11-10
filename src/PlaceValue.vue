<script setup>
import { computed, ref } from 'vue'

const KEY = 'pv-adventure-used-v2'
const BEST = 'pv-adventure-best-v2'
const TOTAL = 900 // numbers 100–999
const defaultTeams = ['Blue Rockets', 'Pink Stars', 'Green Dragons', 'Golden Lions']
const scoreBg = ['bg-[#448bd8]', 'bg-[#df58a5]', 'bg-[#1bad8d]', 'bg-[#e89c2d]']

const read = (k, d) => {
  try {
    return JSON.parse(localStorage.getItem(k)) ?? d
  } catch {
    return d
  }
}
const save = (k, v) => {
  try {
    localStorage.setItem(k, JSON.stringify(v))
  } catch {}
}
const rand = (n) => Math.floor(Math.random() * n)
const shuffle = (a) => {
  for (let i = a.length - 1; i > 0; i--) {
    const j = rand(i + 1)
    ;[a[i], a[j]] = [a[j], a[i]]
  }
  return a
}

function makeQuestion(n) {
  const h = Math.floor(n / 100), t = Math.floor(n / 10) % 10, o = n % 10
  const type = rand(5)
  const swap1 = h * 100 + o * 10 + t, swap2 = t * 100 + h * 10 + o
  let prompt, correct, wrong
  if (type === 0) {
    prompt = '🔎 Count the base-ten blocks!'
    correct = String(n)
    wrong = [swap1, swap2, n + 10, n - 10]
  } else if (type === 1) {
    prompt = `🧩 What is the expanded form of ${n}?`
    correct = `${h * 100} + ${t * 10} + ${o}`
    wrong = [`${h * 100} + ${o * 10} + ${t}`, `${h * 100} + ${t} + ${o * 10}`, `${h * 100} + ${t * 10} + ${o + 1}`]
  } else if (type === 2) {
    const p = rand(3), v = [h, t, o][p], mul = [100, 10, 1][p]
    prompt = `🕵️ What is the value of the digit ${v} in ${n}?`
    correct = String(v * mul)
    wrong = [v, v * 10, v * 100, (v + 1) * mul]
  } else if (type === 3) {
    prompt = `🏗️ Build a number: ${h} hundreds, ${t} tens, and ${o} ones`
    correct = String(n)
    wrong = [swap1, swap2, n + 100, n - 10]
  } else {
    prompt = `💎 Which number matches ${h * 100} + ${t * 10} + ${o}?`
    correct = String(n)
    wrong = [swap1, n + 10, n - 10, n + 100]
  }
  const opts = [...new Set([correct, ...wrong.map(String)])]
  for (let k = 1; opts.length < 3; k++) if (!opts.includes(String(n + k * 7))) opts.push(String(n + k * 7))
  // always keep the correct answer, then trim to 3 options
  const options = shuffle([correct, ...shuffle(opts.filter((x) => x !== correct)).slice(0, 2)])
  return { n, h, t, o, type, prompt, correct, options }
}

const screen = ref('home')
const mode = ref('solo')
const used = ref(read(KEY, []))
const teamCount = ref(2)
const teamNames = ref([...defaultTeams])
const playerName = ref('')
const length = ref(10)
const queue = ref([])
const pos = ref(0)
const names = ref([])
const scores = ref([])
const picked = ref(null)
const bursts = ref([])

const left = computed(() => TOTAL - used.value.length)
const q = computed(() => queue.value[pos.value])
const turn = computed(() => pos.value % names.value.length)
const ok = computed(() => picked.value === q.value.correct)
const high = computed(() => Math.max(...scores.value))
const winners = computed(() => names.value.filter((_, i) => scores.value[i] === high.value))
const title = computed(() =>
  mode.value === 'solo' ? '🎊 Quest Complete!' : winners.value.length > 1 ? '🎊 It’s a Tie!' : `🎊 ${winners.value[0]} Wins!`,
)
const blocks = computed(() => [
  ['HUNDREDS', q.value.h, 'size-11 border-2 border-[#2c76c8] bg-[#7fbfff] bg-[linear-gradient(to_right,#2c76c8_1px,transparent_1px),linear-gradient(to_bottom,#2c76c8_1px,transparent_1px)] bg-[length:10%_10%]'],
  ['TENS', q.value.t, 'h-[90px] w-3.5 border-2 border-[#19845a] bg-[repeating-linear-gradient(to_bottom,#83e5b2_0_8px,#19845a_8px_9px)]'],
  ['ONES', q.value.o, 'size-5 rounded-sm border-2 border-[#c88313] bg-[#ffd269]'],
])

const btn = 'cursor-pointer rounded-2xl px-5 py-3 text-lg font-black text-[#51340b] bg-[#ffb62f] shadow-[0_5px_0_#d28b13] transition hover:-translate-y-0.5 focus-visible:outline-4 focus-visible:outline-offset-2 focus-visible:outline-blue-600'
const btn2 = 'cursor-pointer rounded-2xl px-5 py-3 text-lg font-black text-[#29477c] bg-[#e9f0ff] shadow-[0_5px_0_#b7c9e6] transition hover:-translate-y-0.5'
const panel = 'rounded-3xl border-4 border-white bg-[#fffdf5] p-4 shadow-[0_9px_0_#8bc8d6] sm:p-6'
const input = 'w-full max-w-xs rounded-xl border-[3px] border-[#b9d5ee] bg-white p-2.5'

function setup(m) {
  mode.value = m
  screen.value = 'setup'
}

function start() {
  const usedSet = new Set(used.value)
  const nums = []
  for (let n = 100; n < 1000; n++) if (!usedSet.has(n)) nums.push(n)
  if (!nums.length) return (screen.value = 'empty')
  queue.value = shuffle(nums).slice(0, Math.min(length.value, nums.length)).map(makeQuestion)
  used.value = [...used.value, ...queue.value.map((x) => x.n)]
  save(KEY, used.value)
  pos.value = 0
  picked.value = null
  names.value =
    mode.value === 'solo'
      ? [playerName.value.trim() || 'Explorer']
      : teamNames.value.slice(0, teamCount.value).map((x, i) => x.trim() || `Team ${i + 1}`)
  scores.value = names.value.map(() => 0)
  screen.value = 'game'
}

function pick(v) {
  if (picked.value !== null) return
  picked.value = v
  if (ok.value) {
    scores.value[turn.value] += 10
    confetti()
  }
}

function advance() {
  picked.value = null
  if (++pos.value < queue.value.length) return
  if (mode.value === 'solo' && scores.value[0] > read(BEST, 0)) save(BEST, scores.value[0])
  screen.value = 'end'
  confetti()
}

function resetBank() {
  used.value = []
  save(KEY, [])
  screen.value = 'home'
}

function confetti() {
  const items = Array.from({ length: 22 }, () => ({
    id: Math.random(),
    e: ['⭐', '🎉', '✨', '💛', '💎'][rand(5)],
    left: Math.random() * 100,
    delay: Math.random() * 0.4,
  }))
  bursts.value.push(...items)
  setTimeout(() => (bursts.value = bursts.value.filter((b) => !items.includes(b))), 2500)
}
</script>

<template>
  <div class="relative min-h-screen bg-[linear-gradient(155deg,#78d7ff,#d4f7ff_48%,#a4e9aa)] pb-8 text-[#20325a]">
    <div class="pointer-events-none fixed inset-0 overflow-hidden">
      <span v-for="b in bursts" :key="b.id" class="pv-fall absolute -top-8 text-2xl" :style="{ left: b.left + '%', animationDelay: b.delay + 's' }">{{ b.e }}</span>
    </div>

    <header class="rounded-3xl border-4 border-white bg-gradient-to-br from-[#6038bd] to-[#159fe4] p-4 text-center text-white shadow-[0_10px_0_#3452a3]">
      <div class="text-3xl">⭐ 🚀 🏰 💎 ⭐</div>
      <h1 class="my-1 text-3xl font-black [text-shadow:3px_4px_#2e277f] sm:text-5xl">PLACE VALUE ADVENTURE!</h1>
      <p>🔷 Hundreds &nbsp; 🟩 Tens &nbsp; 🟨 Ones</p>
    </header>

    <section v-if="screen === 'home'" :class="panel" class="my-6 text-center">
      <h2 class="mb-4 text-2xl font-black">🎮 Choose Your Adventure!</h2>
      <div class="grid gap-5 sm:grid-cols-2">
        <button class="cursor-pointer rounded-3xl border-4 border-white bg-gradient-to-br from-[#f660a5] to-[#9c44d8] p-6 text-2xl font-black text-white shadow-[0_8px_0_#5556] transition hover:-translate-y-1" @click="setup('team')">
          🏆<br />TEAM BATTLE<small class="mt-2 block text-sm">2–4 teams • Classroom challenge</small>
        </button>
        <button class="cursor-pointer rounded-3xl border-4 border-white bg-gradient-to-br from-[#39bbff] to-[#3168da] p-6 text-2xl font-black text-white shadow-[0_8px_0_#5556] transition hover:-translate-y-1" @click="setup('solo')">
          🧑‍🚀<br />SOLO QUEST<small class="mt-2 block text-sm">Play at home • Beat your best score</small>
        </button>
      </div>
      <p class="mt-4 text-sm text-slate-500">✨ {{ left }} fresh number challenges remaining in this browser.</p>
    </section>

    <section v-else-if="screen === 'setup'" :class="panel" class="my-6">
      <h2 class="text-2xl font-black">{{ mode === 'team' ? '🏆 Prepare Your Teams!' : '🚀 Prepare Your Solo Quest!' }}</h2>
      <template v-if="mode === 'team'">
        <label class="mt-3 block font-bold">Number of teams</label>
        <select v-model.number="teamCount" :class="input">
          <option v-for="n in [2, 3, 4]" :key="n" :value="n">{{ n }} teams</option>
        </select>
        <div class="mt-3 flex flex-wrap gap-3">
          <input v-for="i in teamCount" :key="i" v-model="teamNames[i - 1]" :placeholder="`Team ${i}`" maxlength="25" aria-label="Team name" :class="input" class="!w-48" />
        </div>
      </template>
      <template v-else>
        <label class="mt-3 block font-bold">Player name</label>
        <input v-model="playerName" placeholder="Your name" maxlength="25" :class="input" />
        <p class="mt-2 text-sm text-slate-500">🏅 Personal best: {{ read(BEST, 0) }} points</p>
      </template>
      <label class="mt-3 block font-bold">Questions this round</label>
      <select v-model.number="length" :class="input">
        <option v-for="n in [10, 15, 20]" :key="n" :value="n">{{ n }} questions</option>
      </select>
      <p class="my-2 text-sm text-slate-500">{{ left }} unused numbers available. Progress is saved in this browser.</p>
      <button :class="btn" class="mr-2" @click="start">🚀 START ADVENTURE!</button>
      <button :class="btn2" @click="screen = 'home'">Back</button>
    </section>

    <section v-else-if="screen === 'game'" :class="panel" class="my-6">
      <div class="grid grid-cols-2 gap-2.5 sm:grid-cols-4" :class="names.length === 1 && '!grid-cols-1'">
        <div
          v-for="(n, i) in names"
          :key="i"
          class="rounded-2xl p-3 text-center text-lg font-bold text-white"
          :class="[scoreBg[i], turn === i && 'outline-[5px] outline-[#ffdc53]']"
        >
          {{ n }}<strong class="block text-3xl">⭐ {{ scores[i] }}</strong>{{ turn === i ? '🎯 YOUR TURN' : '' }}
        </div>
      </div>
      <p class="mt-3 text-center">🌟 Challenge {{ pos + 1 }} of {{ queue.length }} • +10 points</p>
      <div class="mt-2 h-4 overflow-hidden rounded-full bg-[#d7e4f6]">
        <div class="h-full bg-gradient-to-r from-[#f5a830] to-[#ffdf5b] transition-all" :style="{ width: (pos / queue.length) * 100 + '%' }"></div>
      </div>
      <h2 class="my-5 text-center text-2xl font-black sm:text-3xl">{{ q.prompt }}</h2>

      <div v-if="q.type === 0" class="mx-auto my-4 grid max-w-xl grid-cols-3 gap-1.5">
        <div v-for="[label, count, cls] in blocks" :key="label" class="min-h-40 rounded-2xl bg-[#eaf4ff] px-1 py-3 text-center">
          <b class="mb-3 block text-xs">{{ label }}</b>
          <div class="flex flex-wrap items-center justify-center gap-1.5">
            <div v-for="i in count" :key="i" :class="cls"></div>
            <span v-if="!count">0 blocks</span>
          </div>
        </div>
      </div>
      <div v-else class="text-center text-7xl">{{ ['', '🧮', '🔍', '🏗️', '💎'][q.type] }}</div>

      <div class="mt-4 grid gap-3 sm:grid-cols-3">
        <button
          v-for="(v, i) in q.options"
          :key="v"
          :disabled="picked !== null"
          class="min-h-16 cursor-pointer rounded-2xl p-3 text-xl font-black text-[#1b3d72] shadow-[0_5px_0_#b6cde9] disabled:cursor-default"
          :class="picked !== null && v === q.correct ? 'bg-[#79e5a7]' : v === picked ? 'bg-[#ff9eaa]' : 'bg-[#edf5ff]'"
          @click="pick(v)"
        >
          {{ 'ABC'[i] }}. {{ v }}
        </button>
      </div>
      <p class="my-3 min-h-9 text-center text-xl font-black" aria-live="polite">
        <template v-if="picked !== null">{{ ok ? '🎉 AWESOME! +10 POINTS!' : `💪 GOOD TRY! The answer is ${q.correct}` }}</template>
      </p>
      <div v-if="picked !== null" class="text-center">
        <button :class="btn" @click="advance">{{ pos === queue.length - 1 ? '🏆 SEE RESULTS' : 'NEXT CHALLENGE ➜' }}</button>
      </div>
    </section>

    <section v-else-if="screen === 'end'" :class="panel" class="my-6 text-center">
      <div class="text-7xl">🏆</div>
      <h2 class="text-2xl font-black">{{ title }}</h2>
      <p v-for="(n, i) in names" :key="i" class="text-2xl">{{ scores[i] === high ? '🏆' : '⭐' }} {{ n }}: <b>{{ scores[i] }} points</b></p>
      <p class="my-3 text-sm text-slate-500">{{ left }} unused number challenges remain in this browser.</p>
      <button :class="btn" class="mr-2" @click="setup(mode)">🎮 PLAY NEW GAME</button>
      <button :class="btn2" @click="screen = 'home'">🏠 HOME</button>
    </section>

    <section v-else :class="panel" class="my-6 text-center">
      <h2 class="text-2xl font-black">🎉 You've completed the question bank!</h2>
      <p class="my-3">Every question has been used. Reset to start a fresh cycle.</p>
      <button :class="btn" @click="resetBank">RESET QUESTION BANK</button>
    </section>
  </div>
</template>

<style scoped>
.pv-fall { animation: pv-fall 2s linear forwards; }
@keyframes pv-fall { to { transform: translateY(110vh) rotate(500deg); opacity: 0; } }
</style>

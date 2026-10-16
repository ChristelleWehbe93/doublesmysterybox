<script setup>
import { computed, ref } from 'vue'

const bank = [
  { name: 'Skip Count', emoji: '🚀', desc: 'Count forward by 5s and 10s.', q: [
    ['5, 10, 15, ___', ['20', '25', '30'], 0, 'Add 5 to 15 to get 20.'],
    ['10, ___, 30, 40', ['15', '20', '25'], 1, 'Count by 10s: 10, 20, 30, 40.'],
    ['5, 10, ___, 20', ['12', '25', '15'], 2, 'Each number increases by 5.'],
    ['0, 10, 20, ___', ['30', '25', '40'], 0, '20 + 10 = 30.'],
    ['10, 15, 20, ___', ['30', '25', '35'], 1, 'Add 5 to 20 to get 25.'],
    ['20, 30, ___, 50', ['35', '45', '40'], 2, 'Add 10 to 30 to get 40.'],
    ['25, 30, ___, 40', ['35', '45', '50'], 0, 'Add 5 to 30 to get 35.'],
  ] },
  { name: 'Make 10', emoji: '🧩', desc: 'Break apart numbers to make 10 first.', q: [
    ['8 + 5 = 8 + 2 + ___', ['2', '3', '5'], 1, '8 + 2 = 10, then 10 + 3 = 13.'],
    ['9 + 6 = 10 + ___', ['5', '6', '4'], 0, 'Take 1 from 6 to make 10 with 9. Five remain.'],
    ['7 + 8 = 7 + 3 + ___', ['4', '5', '6'], 1, '7 + 3 = 10, then add the remaining 5.'],
    ['6 + 7 = 10 + ___', ['2', '4', '3'], 2, '6 needs 4 to make 10, leaving 3.'],
    ['8 + 6 = ?', ['13', '14', '15'], 1, '8 + 2 + 4 = 10 + 4 = 14.'],
    ['9 + 7 = 9 + 1 + ___', ['6', '7', '8'], 0, '9 + 1 = 10; six remain.'],
    ['7 + 6 = 10 + ___', ['4', '3', '2'], 1, '7 + 3 = 10; three remain.'],
    ['8 + 9 = 10 + ___', ['8', '6', '7'], 2, '8 + 2 + 7 = 17.'],
  ] },
  { name: 'Rewrite the Sum', emoji: '✏️', desc: 'Find an equivalent addition expression.', q: [
    ['Which equals 9 + 4?', ['10 + 3', '10 + 4', '9 + 3'], 0, '9 + 1 + 3 = 10 + 3 = 13.'],
    ['Which equals 6 + 8?', ['10 + 2', '10 + 4', '10 + 6'], 1, '6 + 4 + 4 = 10 + 4 = 14.'],
    ['7 + 5 = 7 + 3 + ___', ['3', '5', '2'], 2, 'Split 5 into 3 and 2.'],
    ['Which equals 9 + 8?', ['10 + 7', '10 + 8', '10 + 6'], 0, '9 + 1 + 7 = 17.'],
    ['Which equals 8 + 7?', ['10 + 4', '10 + 5', '10 + 6'], 1, '8 + 2 + 5 = 15.'],
    ['6 + 9 = 6 + 4 + ___', ['4', '6', '5'], 2, 'Split 9 into 4 and 5.'],
    ['Which equals 7 + 7?', ['10 + 4', '10 + 3', '10 + 5'], 0, '7 + 3 + 4 = 14.'],
  ] },
  { name: 'Odd or Even', emoji: '🟣', desc: 'Pair objects and look for leftovers.', q: [
    ['● ● ● ● ● ● ● ● — Odd or even?', ['Odd', 'Even', 'Neither'], 1, 'All 8 objects can be paired. No leftovers.'],
    ['● ● ● ● ● ● ● — Odd or even?', ['Even', 'Neither', 'Odd'], 2, 'Seven objects make 3 pairs with 1 leftover.'],
    ['11 objects make 5 pairs. How many are left?', ['1', '0', '2'], 0, '5 pairs use 10 objects, leaving 1.'],
    ['Which quantity can be paired with no leftovers?', ['13', '16', '19'], 1, '16 makes 8 complete pairs.'],
    ['● ● ● ● ● ● ● ● ● ● — Odd or even?', ['Even', 'Odd', 'Neither'], 0, '10 objects make 5 pairs.'],
    ['15 objects make 7 pairs. How many are left?', ['0', '1', '2'], 1, '7 pairs use 14 objects, leaving 1.'],
    ['Which number is odd?', ['12', '18', '17'], 2, '17 makes 8 pairs with 1 leftover.'],
  ] },
]

const total = bank.length * 4
const level = ref(0)
const index = ref(0)
const score = ref(0)
const started = ref(false)
const finished = ref(false)
const picked = ref(null) // index of chosen option in the shuffled list
const questions = ref([])

const shuffle = (a) => {
  const b = [...a]
  for (let i = b.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[b[i], b[j]] = [b[j], b[i]]
  }
  return b
}

function start() {
  questions.value = bank.map((b) =>
    shuffle(b.q).slice(0, 4).map(([prompt, opts, correct, why]) => ({
      prompt,
      why,
      opts: shuffle(opts.map((v, i) => ({ v, ok: i === correct }))),
    })),
  )
  level.value = index.value = score.value = 0
  picked.value = null
  started.value = true
  finished.value = false
}

const q = computed(() => questions.value[level.value][index.value])
const done = computed(() => level.value * 4 + index.value)
const correct = computed(() => q.value.opts[picked.value].ok)

function choose(i) {
  picked.value = i
  if (q.value.opts[i].ok) score.value++
}

function next() {
  picked.value = null
  if (index.value < 3) index.value++
  else if (level.value < bank.length - 1) {
    level.value++
    index.value = 0
  } else finished.value = true
}

const nextLabel = computed(() =>
  index.value < 3 ? 'Next Question →' : level.value < bank.length - 1 ? 'Unlock Next Challenge →' : 'See My Results →',
)

const panel = 'my-3.5 rounded-3xl bg-white p-4 shadow-lg shadow-blue-900/10 sm:p-5.5'
const btn = 'mt-4 block w-full cursor-pointer rounded-xl bg-[#0071ce] px-3.5 py-4 text-lg font-bold text-white hover:brightness-95 focus-visible:outline-3 focus-visible:outline-offset-2 focus-visible:outline-amber-500'
</script>

<template>
  <section :class="panel">
    <div class="flex items-center justify-between gap-3">
      <div class="text-xs font-extrabold tracking-widest text-slate-500">GRADE 2 • MATH ADVENTURE</div>
      <div class="rounded-full bg-amber-100 px-4 py-2.5 font-extrabold">⭐ {{ score }} stars</div>
    </div>
    <h1 class="my-1.5 text-[clamp(28px,5vw,42px)] font-bold">The Road to 20! 🚀</h1>
    <div class="mt-5 grid grid-cols-4 gap-1.5 sm:gap-2">
      <div
        v-for="(b, i) in bank"
        :key="b.name"
        class="rounded-2xl p-2 text-center text-[10px] font-bold sm:p-3 sm:text-xs"
        :class="
          finished || (started && i < level)
            ? 'bg-emerald-100'
            : started && i === level
              ? 'bg-blue-100 outline-2 outline-blue-500'
              : 'bg-slate-100'
        "
      >
        <span class="mb-1 block text-2xl">{{ b.emoji }}</span>{{ b.name }}
      </div>
    </div>
  </section>

  <section :class="panel" aria-live="polite">
    <div v-if="!started" class="text-center">
      <h2 class="text-2xl font-bold">Ready for your math mission?</h2>
      <p class="mt-2">Answer questions, earn stars, and unlock all four challenges. Each question has three choices!</p>
      <button :class="btn" @click="start">Start Adventure →</button>
    </div>

    <div v-else-if="finished" class="text-center">
      <h2 class="text-2xl font-bold">🏆 Math Champion!</h2>
      <div class="my-3 text-4xl font-bold">{{ score }} / {{ total }} stars</div>
      <p>{{ score === total ? 'Perfect score! Amazing math skills!' : 'Great effort! Play again for a new mix of questions.' }}</p>
      <button :class="btn" @click="start">🔄 Play Again</button>
    </div>

    <template v-else>
      <div class="flex justify-between text-sm text-slate-500">
        <span>Question {{ done + 1 }} of {{ total }}</span>
        <span>Challenge {{ level + 1 }} of {{ bank.length }}</span>
      </div>
      <div class="mb-5 mt-3 h-2.5 overflow-hidden rounded-full bg-slate-200">
        <div class="h-full bg-blue-600 transition-[width]" :style="{ width: `${(done / total) * 100}%` }"></div>
      </div>
      <div class="text-center">
        <h2 class="text-[clamp(24px,4vw,34px)] font-bold">{{ bank[level].emoji }} {{ bank[level].name }}</h2>
        <p class="text-sm text-slate-500">{{ bank[level].desc }}</p>
        <div class="mx-auto my-7 text-[clamp(23px,4vw,34px)] font-extrabold leading-snug">{{ q.prompt }}</div>
      </div>
      <div class="grid gap-3">
        <button
          v-for="(o, i) in q.opts"
          :key="o.v"
          :disabled="picked !== null"
          class="flex cursor-pointer items-center gap-4 rounded-2xl border-2 p-4 text-left text-2xl font-bold disabled:cursor-default"
          :class="
            picked === null
              ? 'border-slate-300 bg-white hover:border-blue-500 hover:bg-blue-50'
              : o.ok
                ? 'border-emerald-500 bg-emerald-50'
                : i === picked
                  ? 'border-red-400 bg-red-50'
                  : 'border-slate-300 bg-white'
          "
          @click="choose(i)"
        >
          <span class="rounded-lg bg-slate-200 px-3 py-2 text-base">{{ 'ABC'[i] }}</span>
          <span>{{ o.v }}</span>
        </button>
      </div>
      <template v-if="picked !== null">
        <div class="my-5 rounded-2xl bg-blue-50 p-4 leading-relaxed">
          <strong>{{ correct ? '🌟 Correct! You earned a star!' : '💡 Good try! Here is how to solve it:' }}</strong><br />{{ q.why }}
        </div>
        <button :class="btn" @click="next">{{ nextLabel }}</button>
      </template>
    </template>
  </section>

  <p class="mt-3 text-center text-sm text-slate-500">
    Practice skip counting, making 10, equivalent sums, and odd &amp; even numbers.
  </p>
</template>

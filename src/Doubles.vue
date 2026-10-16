<script setup>
  import { computed, ref, watch } from 'vue'

  const limits = { easy: 10, medium: 12, challenge: 20 }
  const mode = ref('teams')
  const difficulty = ref('medium')
  const round = ref(0)
  const game = ref(null)

  const solo = computed(() => game.value.mode === 'solo')
  const fact = computed(() => game.value.facts[game.value.selected])

  function shuffle(a) {
    for (let i = a.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1))
      ;[a[i], a[j]] = [a[j], a[i]]
    }
    return a
  }

  const range = (limit) => Array.from({ length: limit + 1 }, (_, i) => i)

  function makeChoices(n, limit) {
    const wrong = shuffle(range(limit).map((i) => i * 2).filter((v) => v !== n * 2))
    return shuffle([n * 2, ...wrong.slice(0, 2)])
  }

  function newGame() {
    round.value++
    const limit = limits[difficulty.value]
    const nums = shuffle(range(limit))
    while (nums.length < 12) nums.push(...shuffle(range(limit)).slice(0, 12 - nums.length))
    game.value = {
      mode: mode.value,
      facts: nums.slice(0, 12).map((n) => ({ n, points: (Math.floor(Math.random() * 5) + 1) * 10 })),
      opened: [],
      turn: 0,
      score: [0, 0],
      selected: null,
      stage: 'boxes',
      choices: [],
      picked: null,
    }
  }

  function openBox(i) {
    game.value.selected = i
    game.value.choices = makeChoices(game.value.facts[i].n, limits[difficulty.value])
    game.value.stage = 'question'
  }

  function check(value) {
    const g = game.value
    g.picked = value
    if (value === fact.value.n * 2) g.score[solo.value ? 0 : g.turn] += fact.value.points
    g.opened.push(g.selected)
    g.stage = 'result'
  }

  function next() {
    const g = game.value
    g.selected = null
    if (!solo.value) g.turn = 1 - g.turn
    g.stage = g.opened.length === 12 ? 'finished' : 'boxes'
  }

  const turnText = computed(() => {
    if (game.value.stage === 'finished') return '🏆 Round Complete!'
    if (solo.value) return '⭐ Your Turn!'
    return game.value.turn === 0 ? '🔵 Team Blue — Your Turn!' : '🟢 Team Green — Your Turn!'
  })

  const winner = computed(() => {
    const [a, b] = game.value.score
    if (solo.value) return `⭐ You earned ${a} points!`
    return a === b ? '🤝 It’s a tie!' : a > b ? '🔵 Team Blue Wins!' : '🟢 Team Green Wins!'
  })

  watch([mode, difficulty], newGame)
  newGame()

  const panel = 'my-3.5 rounded-3xl bg-white p-4 shadow-lg shadow-blue-900/10 sm:p-5.5'
  const btn = 'cursor-pointer rounded-xl bg-[#0071ce] px-3.5 py-2.5 font-bold text-white hover:brightness-95 focus-visible:outline-3 focus-visible:outline-offset-2 focus-visible:outline-amber-500 disabled:cursor-default disabled:opacity-60'
  const field = 'rounded-xl border-2 border-slate-300 bg-white px-3.5 py-2.5'
</script>

<template>
  <section :class="panel">
    <h1 class="text-center text-[clamp(28px,5vw,42px)] font-bold">🎁 Mystery Doubles!</h1>
    <p class="mb-5 mt-1 text-center text-slate-500">A new mystery every round • Grade 2 Math</p>
    <div class="flex flex-wrap items-center justify-center gap-2.5">
      <label for="mode" class="font-bold">Mode</label>
      <select id="mode" v-model="mode" :class="field">
        <option value="teams">🔵🟢 Two Teams</option>
        <option value="solo">⭐ Solo Player</option>
      </select>
      <label for="difficulty" class="font-bold">Difficulty</label>
      <select id="difficulty" v-model="difficulty" :class="field">
        <option value="easy">Easy (0–10)</option>
        <option value="medium">Medium (0–12)</option>
        <option value="challenge">Challenge (0–20)</option>
      </select>
    </div>
    <div class="mt-5 flex flex-wrap justify-center gap-2.5">
      <div v-if="solo" class="min-w-35 flex-1 rounded-2xl bg-amber-100 p-3.5 text-center font-bold text-amber-800">
        ⭐ Your Points<strong class="block text-4xl">{{ game.score[0] }}</strong>
      </div>
      <template v-else>
        <div class="min-w-35 flex-1 rounded-2xl bg-blue-100 p-3.5 text-center font-bold text-[#0071ce]">
          🔵 Team Blue<strong class="block text-4xl">{{ game.score[0] }}</strong>
        </div>
        <div class="min-w-35 flex-1 rounded-2xl bg-emerald-100 p-3.5 text-center font-bold text-emerald-700">
          🟢 Team Green<strong class="block text-4xl">{{ game.score[1] }}</strong>
        </div>
      </template>
    </div>
  </section>

  <section :class="panel">
    <div class="text-center text-xl font-extrabold">{{ turnText }}</div>
    <div class="mb-4 mt-2 text-center text-slate-500">
      {{ game.opened.length }} of 12 boxes opened · Round {{ round }}
    </div>

    <div v-if="game.stage === 'boxes'" class="grid grid-cols-3 gap-2 sm:gap-3">
      <button
        v-for="(f, i) in game.facts"
        :key="i"
        :disabled="game.opened.includes(i)"
        class="flex min-h-21 cursor-pointer flex-col items-center justify-center gap-1 rounded-xl border-2 border-amber-300 bg-amber-50 font-bold text-amber-900 hover:brightness-95 disabled:cursor-default disabled:border-slate-200 disabled:bg-slate-100 disabled:text-slate-500 sm:min-h-25"
        @click="openBox(i)"
      >
        <span class="text-3xl">{{ game.opened.includes(i) ? '✅' : '🎁' }}</span>
        <span>{{ game.opened.includes(i) ? 'Opened' : `Box ${i + 1}` }}</span>
      </button>
    </div>

    <div v-else-if="game.stage === 'question'" class="text-center">
      <div class="text-slate-500">🎁 MYSTERY BOX {{ game.selected + 1 }}</div>
      <div class="my-5 text-[clamp(38px,8vw,64px)] font-extrabold">{{ fact.n }} + {{ fact.n }} = ?</div>
      <p class="text-slate-500">
        {{ solo ? 'Say the doubles fact aloud before answering!' : '🤫 Let the whole team whisper the doubles fact together!' }}
      </p>
      <p class="text-slate-500">Choose the correct answer:</p>
      <div class="mx-auto my-5 grid max-w-130 grid-cols-3 gap-3" role="group" aria-label="Answer choices">
        <button
          v-for="v in game.choices"
          :key="v"
          class="cursor-pointer rounded-xl border-2 border-sky-200 bg-sky-50 px-2 py-5 text-[clamp(22px,5vw,32px)] font-bold text-sky-800 hover:bg-sky-100"
          @click="check(v)"
        >
          {{ v }}
        </button>
      </div>
    </div>

    <div v-else-if="game.stage === 'result'" class="text-center">
      <div class="my-5 text-[clamp(38px,8vw,64px)] font-extrabold">{{ fact.n }} + {{ fact.n }} = {{ fact.n * 2 }}</div>
      <div class="m-4 text-2xl font-extrabold" aria-live="polite">
        {{
          game.picked === fact.n * 2
            ? `🎉 Correct! +${fact.points} mystery points!`
            : `You chose ${game.picked}. The correct answer is ${fact.n * 2}. No points this turn.`
        }}
      </div>
      <button :class="btn" @click="next">Next Turn →</button>
    </div>

    <div v-else class="text-center">
      <div class="text-2xl font-extrabold">{{ winner }}</div>
      <p>Great job practicing doubles facts!</p>
      <button :class="btn" @click="newGame">🎲 Play a New Round</button>
    </div>
  </section>

  <div class="flex justify-center">
    <button class="cursor-pointer rounded-xl bg-slate-200 px-3.5 py-2.5 font-bold text-slate-700" @click="newGame">
      ↻ Restart with New Questions
    </button>
  </div>
  <p class="mt-3 text-center text-sm text-slate-500">
    All points are hidden until an answer is checked. Each round shuffles questions, answer choices, and points.
  </p>
</template>

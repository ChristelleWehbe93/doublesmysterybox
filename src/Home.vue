<script setup>
  const games = [
    { 
      id: 'mystery_doubles', 
      emoji: '🎁', 
      title: 'Mystery Doubles', 
      desc: 'Grade 2 Math',
      type: 'progress',
      buttonText: 'Start',
      buttonBg: 'from-rose-700 to-rose-600 hover:from-rose-600 hover:to-rose-500 shadow-[0_4px_14px_rgba(225,29,72,0.4)]'
    },
    { 
      id: 'road_to_20', 
      emoji: '🚀', 
      title: 'The Road to 20', 
      desc: 'Grade 2 Math Adventure',
      type: 'track',
      buttonText: 'Start',
      buttonBg: 'from-amber-600 to-amber-500 hover:from-amber-500 hover:to-amber-400 shadow-[0_4px_14px_rgba(217,119,6,0.4)]'
    },
  ]
</script>

<template>
  <div class="min-h-screen   bg-gradient-to-br   from-blue-600 via-sky-500 to-emerald-500 p-6 flex flex-col items-center justify-center font-sans text-white relative overflow-hidden">
    
      <!-- Playful blobs -->
      <div class="absolute top-12 left-1/4 w-72 h-72 bg-yellow-300/30 rounded-full blur-3xl pointer-events-none"></div>
      <div class="absolute bottom-10 right-1/4 w-96 h-96 bg-cyan-200/30 rounded-full blur-3xl pointer-events-none"></div>

    <!-- Header -->
    <div class="flex items-center gap-3 mb-10 z-10">
      <span class="text-3xl drop-shadow-[0_0_10px_rgba(234,179,8,0.5)]">🏆</span>
      <h1 class="text-3xl sm:text-4xl font-extrabold tracking-wide bg-gradient-to-r from-yellow-200 via-amber-300 to-yellow-200 bg-clip-text text-transparent drop-shadow-md comic-relief-bold">
        Math Champions
      </h1>
    </div>

    <!-- Cards Grid -->
    <div class="grid grid-cols-1 md:grid-cols-2 gap-8 max-w-4xl w-full z-10 px-4">
      <a
        v-for="g in games"
        :key="g.id"
        :href="`/${g.id}`"
        class="bg-white/95 backdrop-blur-md rounded-3xl p-6 text-slate-800 shadow-[0_10px_30px_rgba(0,0,0,0.3)] border border-white/20 flex flex-col justify-between transform transition hover:scale-[1.02] group"
      >
        <div>
          <!-- Emoji Icon Area -->
          <div class="flex justify-center mb-6 relative">
            <div class="absolute inset-0 bg-amber-200/30 rounded-full blur-xl group-hover:scale-125 transition"></div>
            <span class="text-6xl relative drop-shadow-md">{{ g.emoji }}</span>
          </div>
          
          <h2 class="comic-relief-bold text-2xl font-bold text-center text-slate-900 mb-1">{{ g.title }}</h2>
          <p class="comic-relief-regular text-sm font-medium text-center text-slate-500 mb-6">{{ g.desc }}</p>
          
          <!-- Card 1 specific: Progress Bar -->
          <div v-if="g.type === 'progress'" class="flex items-center gap-3 mb-6 bg-slate-50 p-3 rounded-2xl border border-slate-100">
            <span class="text-lg">🪙</span>
            <div class="flex-1 bg-slate-200 h-3 rounded-full overflow-hidden">
              <div class="bg-gradient-to-r from-amber-400 to-indigo-500 h-full w-2/5 rounded-full"></div>
            </div>
            <span class="text-lg">💎</span>
          </div>

          <!-- Card 2 specific: Roadmap Track -->
          <div v-if="g.type === 'track'" class="flex items-center justify-between mb-6 bg-slate-50 py-3 px-4 rounded-2xl border border-slate-100 text-xs font-bold text-slate-600">
            <div class="w-7 h-7 rounded-full bg-amber-200 border border-amber-400 flex items-center justify-center shadow-sm">1</div>
            <div class="flex-1 h-0.5 bg-slate-300 mx-2 relative">
              <div class="absolute -top-2 left-1/2 -translate-x-1/2 bg-amber-500 text-white text-[10px] px-1.5 py-0.5 rounded-full shadow">10</div>
            </div>
            <div class="w-7 h-7 rounded-full bg-amber-300 border border-amber-500 flex items-center justify-center shadow-sm">20</div>
            <span class="text-lg ml-2">🪐</span>
          </div>
        </div>

        <!-- Action Button -->
        <div class="w-full py-3 px-6 text-center text-white font-bold rounded-2xl transition active:scale-95 bg-gradient-to-r" :class="g.buttonBg">
          {{ g.buttonText }}
        </div>
      </a>
    </div>

    <!-- Optional Coming Soon Preview Banner -->
    <!-- <div class="mt-8 max-w-sm w-full bg-slate-900/60 backdrop-blur-md rounded-2xl p-4 border border-slate-700/50 flex items-center justify-between z-10 shadow-lg">
      <div class="flex items-center gap-3">
        <div class="w-10 h-10 rounded-xl bg-amber-500/20 flex items-center justify-center text-amber-400 text-lg">⭐</div>
        <div>
          <h3 class="text-sm font-bold text-slate-200">Star Collector</h3>
          <p class="text-xs text-slate-400">Unlock more challenges</p>
        </div>
      </div>
      <span class="text-xs font-semibold px-3 py-1 bg-slate-800 text-slate-300 rounded-full border border-slate-700">Coming Soon</span>
    </div> -->

  </div>
</template>
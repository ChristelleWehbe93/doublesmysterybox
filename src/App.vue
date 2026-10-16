<script setup>
  import { ref, watchEffect } from 'vue'
  import Home from './Home.vue'
  import Doubles from './Doubles.vue'
  import RoadTo20 from './RoadTo20.vue'

  // ponytail: History API routing, swap for vue-router if you need params/nesting
  const games = { mystery_doubles: Doubles, road_to_20: RoadTo20 }
  const titles = { mystery_doubles: 'Mystery Doubles', road_to_20: 'The Road to 20' }
  const hash = ref(location.pathname.slice(1))
  addEventListener('popstate', () => (hash.value = location.pathname.slice(1)))
  // client-side navigation for same-origin links, no page reload
  addEventListener('click', (e) => {
    const a = e.target.closest('a')
    if (!a || a.origin !== location.origin || e.metaKey || e.ctrlKey || e.shiftKey) return
    e.preventDefault()
    history.pushState(null, '', a.pathname)
    hash.value = a.pathname.slice(1)
  })
  watchEffect(() => (document.title = titles[hash.value] || 'Math Champions'))
</script>

<template>
  <div class="min-h-screen bg-[#eef4fb] text-[#203047]" :class="{ 'p-3 sm:p-5': games[hash] }">
    <main :class="{ 'mx-auto max-w-3xl': games[hash] }">
      <a v-if="games[hash]" href="/" class="relative z-10 mb-2 inline-block font-bold text-blue-700">← All games</a>
      <component :is="games[hash] || Home" />
    </main>
  </div>
</template>

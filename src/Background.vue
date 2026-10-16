<template>
  <div class="sky" aria-hidden="true">
    <div class="orb a"></div>
    <div class="orb b"></div>
    <div class="orb c"></div>
    <div class="cloud one">☁️</div>
    <div class="cloud two">☁️</div>
    <div class="cloud three">☁️</div>
    <div class="spark" style="top:20%;left:20%">✨</div>
    <div class="spark" style="top:60%;right:16%;animation-delay:1s">⭐</div>
    <div class="spark" style="top:80%;left:12%;animation-delay:2s">✨</div>
  </div>
</template>

<style scoped>
.sky {
  position: fixed;
  inset: 0;
  pointer-events: none;
  overflow: hidden;
  z-index: 0;
  background: linear-gradient(120deg, #b7eaff, #f6d9ff, #ffe9bb, #bcefe3);
  background-size: 350% 350%;
  animation: skyshift 19s ease infinite;
}
@keyframes skyshift { 0%, 100% { background-position: 0% 50% } 50% { background-position: 100% 50% } }
.orb { position: absolute; border-radius: 50%; filter: blur(3px); opacity: 0.33; animation: float 14s ease-in-out infinite alternate }
.orb.a { width: 280px; height: 280px; background: #fff; top: 4%; left: -7% }
.orb.b { width: 230px; height: 230px; background: #ff8ac0; bottom: 2%; right: -4%; animation-delay: -5s }
.orb.c { width: 160px; height: 160px; background: #7bffdc; top: 35%; right: 7%; animation-delay: -9s }
@keyframes float { to { transform: translate(70px, -65px) scale(1.15) } }
.cloud { position: absolute; font-size: clamp(55px, 10vw, 115px); opacity: 0.58; animation: drift linear infinite }
.cloud.one { top: 8%; left: -150px; animation-duration: 37s }
.cloud.two { top: 35%; left: -200px; animation-duration: 53s; animation-delay: -20s }
.cloud.three { top: 69%; left: -120px; animation-duration: 44s; animation-delay: -13s }
@keyframes drift { to { transform: translateX(calc(100vw + 360px)) } }
.spark { position: absolute; animation: twinkle 3s ease-in-out infinite alternate; font-size: 23px }
@keyframes twinkle { to { transform: translateY(-15px) rotate(35deg) scale(1.4); opacity: 0.3 } }
@media (prefers-reduced-motion: reduce) {
  * { animation-duration: 0.01ms !important }
}
</style>

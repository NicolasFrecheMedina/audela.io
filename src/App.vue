<template>
  <main ref="surface" class="surface" :data-theme="theme" @pointermove="move" @pointerleave="settle">
    <div class="ambient ambient-left" aria-hidden="true"></div>
    <div class="ambient ambient-right" aria-hidden="true"></div>
    <div class="grain" aria-hidden="true"></div>

    <header class="masthead enter enter-meta">
      <button
        class="direction"
        type="button"
        :aria-label="theme === 'air' ? 'Activer le mode profond' : 'Activer le mode aérien'"
        :aria-pressed="theme === 'deep'"
        @click="toggleTheme"
      >
        <span :class="{ active: theme === 'air' }">A</span>
        <i aria-hidden="true"><b></b></i>
        <span :class="{ active: theme === 'deep' }">D</span>
      </button>
    </header>

    <section class="hero" aria-labelledby="hero-title">
      <div class="memory-field" aria-hidden="true">
        <div class="field-aura"></div>
        <div class="field-shadow"></div>
        <div class="orbit orbit-outer"><i></i></div>
        <div class="orbit orbit-middle"><i></i></div>
        <div class="orbit orbit-inner"><i></i></div>
        <div class="memory-core">
          <div class="membrane membrane-one"></div>
          <div class="membrane membrane-two"></div>
          <div class="membrane membrane-three"></div>
          <div class="membrane membrane-four"></div>
          <div class="refraction"></div>
          <div class="core-light"></div>
          <div class="core-seed"></div>
          <span v-for="number in 4" :key="number" class="fragment" :class="`fragment-${number}`"></span>
        </div>
        <div class="coordinates"></div>
      </div>

      <button
        v-for="(axis, index) in axes"
        :key="axis.id"
        type="button"
        class="axis enter enter-axes"
        :class="`axis-${axis.id}`"
        :aria-label="content[theme].axes[index]"
        @pointerenter="charge(axis.id)"
        @pointerleave="release"
        @focus="charge(axis.id)"
        @blur="release"
      >
        <span class="axis-dot"></span>
        <span class="axis-labels" aria-hidden="true">
          <span class="content-layer" :class="{ active: theme === 'air' }">{{ content.air.axes[index] }}</span>
          <span class="content-layer" :class="{ active: theme === 'deep' }">{{ content.deep.axes[index] }}</span>
        </span>
      </button>

      <div class="hero-copy">
        <p class="overline enter enter-title">UNE PRÉSENCE, AUTREMENT</p>
        <h1 id="hero-title" class="title enter enter-title">AU-DELÀ</h1>
        <p class="question text-stack enter enter-copy">
          <span class="content-layer" :class="{ active: theme === 'air' }" :aria-hidden="theme !== 'air'">Et si notre présence pouvait <br class="mobile-break">dépasser le temps&nbsp;?</span>
          <span class="content-layer" :class="{ active: theme === 'deep' }" :aria-hidden="theme !== 'deep'">Et si ce qui reste pouvait <br class="mobile-break">continuer à évoluer&nbsp;?</span>
        </p>
        <p class="description text-stack enter enter-copy">
          <span class="content-layer" :class="{ active: theme === 'air' }" :aria-hidden="theme !== 'air'">Une exploration autour de l’identité, de la mémoire<br class="desktop-break"> et de la continuité numérique.</span>
          <span class="content-layer" :class="{ active: theme === 'deep' }" :aria-hidden="theme !== 'deep'">Une exploration de la trace, de la persistance<br class="desktop-break"> et de ce que nous laissons derrière nous.</span>
        </p>
      </div>
    </section>

    <footer class="footer enter enter-footer">
      <span class="signal" aria-hidden="true"><i></i><i></i><i></i></span>
      <a class="studio-link" href="https://nacodev.com" target="_blank" rel="noopener noreferrer">
        <span>NacoDev</span><span class="arrow" aria-hidden="true">↗</span>
      </a>
    </footer>
    <div class="edge edge-top" aria-hidden="true"></div>
    <div class="edge edge-bottom" aria-hidden="true"></div>
  </main>
</template>

<script>
import { onBeforeUnmount, onMounted, ref } from 'vue'

const THEME_KEY = 'audela-art-direction'

export default {
  name: 'App',
  setup() {
    const surface = ref(null)
    const theme = ref('air')
    const axes = [{ id: 'identity' }, { id: 'memory' }, { id: 'time' }]
    const content = {
      air: {
        axes: ['IDENTITÉ', 'MÉMOIRE', 'TEMPS']
      },
      deep: {
        axes: ['TRACE', 'ABSENCE', 'PERSISTANCE']
      }
    }
    let frame = 0
    let targetX = 0
    let targetY = 0
    let currentX = 0
    let currentY = 0
    let reducedMotion

    const updateThemeColor = () => {
      document.querySelector('meta[name="theme-color"]')?.setAttribute('content', theme.value === 'deep' ? '#090c13' : '#f3f5f5')
    }

    const toggleTheme = () => {
      theme.value = theme.value === 'air' ? 'deep' : 'air'
      try {
        window.localStorage.setItem(THEME_KEY, theme.value)
      } catch (error) {
        // The visual switch remains functional when storage is unavailable.
      }
      updateThemeColor()
    }

    const render = () => {
      currentX += (targetX - currentX) * 0.075
      currentY += (targetY - currentY) * 0.075
      if (surface.value) {
        surface.value.style.setProperty('--x', currentX.toFixed(3))
        surface.value.style.setProperty('--y', currentY.toFixed(3))
        surface.value.style.setProperty('--light-x', `${50 + currentX * 11}%`)
        surface.value.style.setProperty('--light-y', `${43 + currentY * 9}%`)
      }
      const moving = Math.abs(targetX - currentX) > 0.002 || Math.abs(targetY - currentY) > 0.002
      frame = moving ? requestAnimationFrame(render) : 0
    }

    const requestRender = () => {
      if (!frame) frame = requestAnimationFrame(render)
    }

    const move = (event) => {
      if (reducedMotion?.matches || event.pointerType === 'touch') return
      targetX = (event.clientX / window.innerWidth - 0.5) * 2
      targetY = (event.clientY / window.innerHeight - 0.5) * 2
      requestRender()
    }

    const settle = () => {
      targetX = 0
      targetY = 0
      requestRender()
    }

    const charge = (axis) => {
      if (!surface.value) return
      surface.value.dataset.axis = axis
      surface.value.style.setProperty('--charge', '1')
    }

    const release = () => {
      if (!surface.value) return
      delete surface.value.dataset.axis
      surface.value.style.setProperty('--charge', '0')
    }

    onMounted(() => {
      reducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)')
      try {
        const saved = window.localStorage.getItem(THEME_KEY)
        if (saved === 'air' || saved === 'deep') theme.value = saved
      } catch (error) {
        // Default to the airy mode when storage cannot be read.
      }
      updateThemeColor()
    })

    onBeforeUnmount(() => {
      if (frame) cancelAnimationFrame(frame)
    })

    return { axes, content, surface, theme, toggleTheme, move, settle, charge, release }
  }
}
</script>

<style>
:root { color-scheme: light dark; }
* { box-sizing: border-box; }
html, body, #app { width: 100%; height: 100%; margin: 0; overflow: hidden; }
body {
  background: #f3f5f5;
  font-family: "Avenir Next", Avenir, Montserrat, "Helvetica Neue", Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  text-rendering: geometricPrecision;
}
button, a { color: inherit; font: inherit; }
button { border: 0; }

.surface {
  --x: 0; --y: 0; --light-x: 50%; --light-y: 43%; --charge: 0;
  --surface: #f3f5f5; --surface-alt: #fafbfb; --ink: #252b33; --body: #66717b; --muted: #89949d;
  --line: rgba(120, 148, 166, .28); --bright: rgba(255, 255, 255, .86);
  --ice: rgba(158, 207, 228, .66); --violet: rgba(204, 190, 231, .52);
  --glow: rgba(255, 255, 255, .9); --shadow: rgba(77, 98, 112, .18); --focus: #587c94;
  position: relative; isolation: isolate; width: 100%; height: 100dvh; min-height: 520px; overflow: hidden;
  color: var(--ink);
  background: radial-gradient(circle at var(--light-x) var(--light-y), rgba(255,255,255,.98), rgba(255,255,255,.52) 28%, transparent 57%), linear-gradient(145deg, var(--surface-alt), var(--surface) 54%, #f8f9fa);
  transition: color .9s ease, background 1.1s cubic-bezier(.2,.75,.2,1);
}
.surface[data-theme="deep"] {
  --surface: #090c13; --surface-alt: #111622; --ink: #edf5f7; --body: #a9b9c2; --muted: #768994;
  --line: rgba(119,178,205,.26); --bright: rgba(203,235,245,.2); --ice: rgba(38,118,151,.48);
  --violet: rgba(103,78,150,.42); --glow: rgba(157,222,239,.42); --shadow: rgba(0,0,0,.58); --focus: #9ed8e8;
  background: radial-gradient(circle at var(--light-x) var(--light-y), rgba(32,63,78,.52), rgba(13,25,36,.34) 29%, transparent 58%), radial-gradient(circle at 84% 88%, rgba(49,30,80,.34), transparent 38%), linear-gradient(145deg, #111722, var(--surface) 55%, #070910);
}
.surface::before {
  content: ""; position: absolute; z-index: -4; inset: 16px; border: 1px solid var(--bright); border-radius: 28px;
  box-shadow: inset 0 0 0 1px var(--line), inset 22px 22px 54px var(--shadow), inset -20px -18px 48px var(--glow), 0 18px 50px var(--shadow);
  opacity: 0; animation: reveal 1s .08s ease-out forwards; transition: border-color .9s ease, box-shadow .9s ease;
}
.ambient { position: absolute; z-index: -3; width: min(42vw,600px); aspect-ratio: 1; border-radius: 50%; filter: blur(18px); pointer-events: none; transition: background 1s ease; will-change: transform; }
.ambient-left { left: -16vw; top: 13%; background: radial-gradient(circle,var(--ice),transparent 69%); transform: translate3d(calc(var(--x) * -12px),calc(var(--y) * -8px),0); }
.ambient-right { right: -11vw; bottom: -20%; background: radial-gradient(circle,var(--violet),transparent 69%); transform: translate3d(calc(var(--x) * 14px),calc(var(--y) * 10px),0); }
.grain { position: absolute; z-index: 20; inset: 0; opacity: .13; pointer-events: none; mix-blend-mode: multiply; background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 180 180' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='.85' numOctaves='3' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='.055'/%3E%3C/svg%3E"); }
[data-theme="deep"] .grain { opacity: .2; mix-blend-mode: screen; }

.masthead, .footer { position: absolute; z-index: 15; left: clamp(34px,4.5vw,76px); right: clamp(34px,4.5vw,76px); display: flex; align-items: center; }
.masthead { top: clamp(31px,5vh,58px); }
.direction { display: inline-flex; align-items: center; gap: 8px; padding: 8px 9px 8px 0; color: var(--muted); background: transparent; cursor: pointer; font-size: 9px; font-weight: 600; letter-spacing: .12em; }
.direction span { opacity: .45; transition: opacity .4s, color .4s, transform .4s; }
.direction span.active { color: var(--ink); opacity: 1; transform: translateY(-1px); }
.direction i { position: relative; width: 24px; height: 1px; background: var(--line); transform: rotate(-48deg); }
.direction i b { position: absolute; top: -2px; left: 1px; width: 5px; height: 5px; border-radius: 50%; background: var(--ink); box-shadow: 0 0 8px var(--glow); transition: transform .6s cubic-bezier(.2,.9,.2,1), background-color .5s; }
[data-theme="deep"] .direction i b { transform: translateX(17px); }
.direction:hover span:not(.active) { opacity: .8; }

.hero { position: absolute; inset: 0; display: grid; place-items: center; }
.memory-field { position: absolute; z-index: 2; top: 45%; left: 50%; width: clamp(320px,34vw,520px); aspect-ratio: 1; transform: translate(-50%,-50%); animation: form-core 1.25s .25s cubic-bezier(.22,.8,.25,1) backwards; will-change: transform; }
.field-aura { position: absolute; inset: 6%; border-radius: 50%; background: radial-gradient(circle,var(--glow),var(--ice) 34%,transparent 70%); filter: blur(calc(25px - var(--charge) * 6px)); opacity: calc(.38 + var(--charge) * .18); transform: translate3d(calc(var(--x) * -14px),calc(var(--y) * -10px),0) scale(calc(1 + var(--charge) * .06)); transition: .7s ease; }
.field-shadow { position: absolute; inset: 15%; border-radius: 48% 52% 56% 44%/51% 43% 57% 49%; background: radial-gradient(circle at 35% 28%,var(--glow),var(--ice) 48%,var(--violet) 72%,transparent 74%); filter: blur(14px); box-shadow: 25px 32px 55px var(--shadow),-20px -24px 42px var(--glow); transform: translate3d(calc(var(--x) * -11px),calc(var(--y) * -8px),0) scale(calc(1 + var(--charge) * .04)); transition: background .9s, box-shadow .9s, transform .7s; }
.orbit { position: absolute; border: 1px solid var(--line); border-radius: 50%; box-shadow: inset 7px 9px 18px rgba(120,160,182,.06),0 0 16px var(--glow); animation: orbit 10s ease-in-out infinite alternate; transition: border-color .6s; will-change: transform; }
.orbit i { position: absolute; width: 4px; height: 4px; border-radius: 50%; background: var(--ink); box-shadow: 0 0 0 5px var(--ice),0 0 12px var(--glow); opacity: .45; }
.orbit-outer { inset: 0 -4% -1% 3%; border-left-color: transparent; transform: translate3d(calc(var(--x) * -9px),calc(var(--y) * -6px),0) rotate(-14deg) scaleX(calc(.9 + var(--charge) * .03)); animation-duration: 12s; }
.orbit-outer i { top: 12%; right: 18%; }
.orbit-middle { inset: 7% 12% 13% 5%; border-right-color: transparent; transform: translate3d(calc(var(--x) * 11px),calc(var(--y) * 8px),0) rotate(18deg) scaleY(calc(.92 + var(--charge) * .04)); animation-duration: 9s; animation-delay: -3s; }
.orbit-middle i { bottom: 7%; left: 27%; width: 3px; height: 3px; }
.orbit-inner { inset: 20% 17% 18% 24%; border-top-color: transparent; transform: translate3d(calc(var(--x) * -7px),calc(var(--y) * 10px),0) rotate(42deg) scaleX(calc(1.06 + var(--charge) * .05)); animation-duration: 7s; animation-delay: -5s; }
.orbit-inner i { top: 23%; left: 2%; }
.memory-core { position: absolute; inset: 22%; border-radius: 43% 57% 52% 48%/49% 44% 56% 51%; filter: drop-shadow(13px 17px 20px var(--shadow)) drop-shadow(-10px -11px 17px var(--glow)); transform: translate3d(calc(var(--x) * 5px),calc(var(--y) * 4px),0) scale(calc(1 + var(--charge) * .035)); transition: filter .9s,transform .7s cubic-bezier(.2,.8,.2,1); }
.membrane { position: absolute; border: 1px solid var(--bright); backdrop-filter: blur(3px) saturate(1.15); transition: background .9s,border-color .9s,box-shadow .9s; will-change: transform; }
.membrane-one { inset: 0; border-radius: 43% 57% 52% 48%/49% 44% 56% 51%; background: linear-gradient(138deg,var(--glow),var(--ice) 57%,var(--violet)); box-shadow: inset 11px 10px 20px var(--glow),inset -13px -10px 24px var(--shadow); animation: membrane-one 8s ease-in-out infinite alternate; }
.membrane-two { inset: 12% 10% 11% 14%; border-radius: 56% 44% 43% 57%/45% 58% 42% 55%; background: linear-gradient(315deg,var(--glow),var(--ice)); box-shadow: inset -8px -9px 17px var(--shadow),6px 8px 15px var(--shadow); transform: translate3d(calc(var(--x) * -8px),calc(var(--y) * -6px),0) rotate(-8deg); animation: membrane-two 6.5s -1s ease-in-out infinite alternate; }
.membrane-three { inset: 27% 24% 23% 28%; border-radius: 46% 54% 60% 40%/59% 43% 57% 41%; background: var(--bright); box-shadow: inset 4px 5px 10px var(--glow),inset -5px -4px 11px var(--shadow); transform: translate3d(calc(var(--x) * 10px),calc(var(--y) * 8px),0) rotate(11deg); animation: membrane-three 5s -2s ease-in-out infinite alternate; }
.membrane-four { inset: 38% 37% 35% 39%; border-radius: 60% 40% 48% 52%/44% 58% 42% 56%; background: radial-gradient(circle at 35% 30%,var(--glow),var(--ice)); box-shadow: 0 0 20px var(--ice); animation: membrane-four 4s -1.5s ease-in-out infinite alternate; }
.refraction { position: absolute; inset: 6%; border-radius: 49% 51% 44% 56%/53% 46% 54% 47%; border-top: 1px solid var(--bright); filter: blur(.2px); opacity: .75; transform: translate3d(calc(var(--x) * 14px),calc(var(--y) * 11px),0) rotate(23deg); }
.core-light { position: absolute; inset: 16%; border-radius: 50%; background: radial-gradient(circle at 38% 33%,var(--glow),var(--ice) 55%,transparent 72%); filter: blur(4px); transform: translate3d(calc(var(--x) * 14px),calc(var(--y) * 11px),0); }
.core-seed { position: absolute; top: 47%; left: 48%; width: 8%; aspect-ratio: 1; border-radius: 50%; background: var(--ink); box-shadow: 0 0 5px var(--glow),0 0 22px 9px var(--ice); opacity: .7; transform: translate(-50%,-50%) translate3d(calc(var(--x) * 5px),calc(var(--y) * 5px),0); }
.fragment { position: absolute; border-radius: 50%; background: var(--bright); box-shadow: 2px 3px 8px var(--shadow),0 0 6px var(--glow); }
.fragment-1 { width: 7px; height: 7px; top: 17%; left: 27%; transform: translate3d(calc(var(--x) * -12px),calc(var(--y) * -9px),0); }
.fragment-2 { width: 5px; height: 5px; top: 32%; right: 13%; transform: translate3d(calc(var(--x) * 15px),calc(var(--y) * 11px),0); }
.fragment-3 { width: 4px; height: 4px; bottom: 15%; left: 38%; transform: translate3d(calc(var(--x) * -10px),calc(var(--y) * 13px),0); }
.fragment-4 { width: 3px; height: 3px; bottom: 31%; right: 19%; transform: translate3d(calc(var(--x) * 11px),calc(var(--y) * -12px),0); }
.coordinates { position: absolute; inset: 0; opacity: calc(.28 + var(--charge) * .15); background: linear-gradient(90deg,transparent calc(50% - .5px),var(--line) 50%,transparent calc(50% + .5px)),linear-gradient(0deg,transparent calc(50% - .5px),var(--line) 50%,transparent calc(50% + .5px)); mask-image: radial-gradient(circle,#000,transparent 70%); transform: translate3d(calc(var(--x) * -3px),calc(var(--y) * -3px),0); }

.hero-copy { position: absolute; z-index: 4; top: 45%; left: 50%; width: min(94vw,1040px); text-align: center; transform: translate(-50%,-50%); pointer-events: none; }
.overline { margin: 0 0 clamp(16px,2.4vh,25px); color: var(--muted); font-size: clamp(7px,.56vw,9px); font-weight: 600; letter-spacing: .46em; }
.title { margin: 0; color: var(--ink); font-family: "Sitka Display","Iowan Old Style","Palatino Linotype",Palatino,serif; font-size: clamp(68px,10.4vw,166px); font-weight: 500; font-kerning: normal; font-optical-sizing: auto; line-height: .76; letter-spacing: calc(.035em + var(--x) * .008em); text-indent: .035em; text-shadow: 0 1px 1px var(--glow),0 15px 30px var(--shadow); transform: translate3d(calc(var(--x) * 2.5px),calc(var(--y) * 1.7px),0) skewX(calc(var(--x) * -.22deg)); will-change: transform,letter-spacing; }
.question { margin: clamp(29px,4.2vh,45px) 0 0; color: var(--body); font-family: Didot,"Bodoni 72",Georgia,serif; font-size: clamp(17px,1.45vw,23px); font-style: italic; font-weight: 400; letter-spacing: .018em; }
.description { margin: 12px auto 0; color: var(--muted); font-size: clamp(9px,.72vw,11px); font-weight: 500; line-height: 1.75; letter-spacing: .09em; }
.overline,.title,.question,.description { transition: color .8s,text-shadow .8s; }
.mobile-break { display: none; }
.text-stack,.axis-labels { display: grid; }
.text-stack > .content-layer,.axis-labels > .content-layer { grid-area: 1/1; }
.content-layer { opacity: 0; filter: blur(5px); transform: translateY(4px); transition: opacity .72s cubic-bezier(.2,.75,.2,1),filter .72s cubic-bezier(.2,.75,.2,1),transform .72s cubic-bezier(.2,.75,.2,1); will-change: opacity,filter,transform; }
.content-layer.active { opacity: 1; filter: blur(0); transform: translateY(0); transition-delay: .08s; }

.axis { position: absolute; z-index: 8; display: flex; align-items: center; gap: 9px; padding: 10px; color: var(--muted); background: transparent; cursor: pointer; font-size: 8px; font-weight: 600; letter-spacing: .28em; transition: color .45s,transform .65s cubic-bezier(.2,.8,.2,1); }
.axis-labels { text-align: left; }
.axis-memory .axis-labels { text-align: right; }
.axis-dot { width: 4px; height: 4px; border: 1px solid var(--muted); border-radius: 50%; box-shadow: 0 0 0 4px var(--ice); transition: .45s ease; }
.axis:hover,.axis:focus-visible { color: var(--ink); }
.axis:hover .axis-dot,.axis:focus-visible .axis-dot { background: var(--ink); box-shadow: 0 0 0 7px var(--ice),0 0 14px var(--glow); transform: scale(.8); }
.axis-identity { top: 28%; left: clamp(7%,16vw,24%); transform: translate3d(calc(var(--x) * -6px),calc(var(--y) * -4px),0); }
.axis-memory { top: 30%; right: clamp(7%,16vw,24%); flex-direction: row-reverse; transform: translate3d(calc(var(--x) * 7px),calc(var(--y) * -5px),0); }
.axis-time { top: 68%; left: 50%; transform: translateX(-50%) translate3d(calc(var(--x) * 3px),calc(var(--y) * 7px),0); }
[data-axis="identity"] .orbit-outer,[data-axis="memory"] .orbit-middle,[data-axis="time"] .orbit-inner { border-color: var(--focus); }

.footer { bottom: clamp(31px,5vh,58px); justify-content: flex-end; }
.signal { position: absolute; left: 50%; display: flex; gap: 4px; transform: translateX(-50%); }
.signal i { width: 2px; height: 2px; border-radius: 50%; background: var(--muted); animation: signal 2.4s ease-in-out infinite; }
.signal i:nth-child(2) { animation-delay: .3s; } .signal i:nth-child(3) { animation-delay: .6s; }
.studio-link { display: flex; align-items: center; gap: 9px; padding: 12px 2px 12px 14px; color: var(--body); font-size: 9px; font-weight: 600; letter-spacing: .16em; text-decoration: none; transition: color .3s; }
.studio-link::before { content: ""; width: 25px; height: 1px; background: var(--line); transition: width .35s,background-color .35s; }
.studio-link:hover,.studio-link:focus-visible { color: var(--ink); }
.studio-link:hover::before,.studio-link:focus-visible::before { width: 39px; background: var(--focus); }
.arrow { font-size: 13px; font-weight: 400; transition: transform .3s; }
.studio-link:hover .arrow,.studio-link:focus-visible .arrow { transform: translate(3px,-3px); }
button:focus-visible,a:focus-visible { outline: 1px solid var(--focus); outline-offset: 5px; border-radius: 3px; }
.edge { position: absolute; z-index: 5; left: 50%; width: 1px; height: 22px; background: linear-gradient(var(--muted),transparent); opacity: .5; }
.edge-top { top: 0; } .edge-bottom { bottom: 0; transform: rotate(180deg); }

.enter { opacity: 0; animation: enter .75s cubic-bezier(.22,.8,.25,1) forwards; }
.enter-meta { animation-delay: 1.35s; } .enter-title { animation-delay: .78s; } .enter-copy { animation-delay: 1.02s; } .enter-axes { animation-delay: 1.2s; } .enter-footer { animation-delay: 1.4s; }
@keyframes reveal { to { opacity: 1; } }
@keyframes enter { from { opacity: 0; filter: blur(6px); } to { opacity: 1; filter: blur(0); } }
@keyframes form-core { from { opacity: .25; transform: translate(-50%,-50%) scale(.82) rotate(-4deg); filter: blur(14px); } }
@keyframes orbit { from { border-radius: 47% 53% 44% 56%/58% 43% 57% 42%; } to { border-radius: 55% 45% 52% 48%/44% 57% 43% 56%; } }
@keyframes membrane-one { to { border-radius: 51% 49% 43% 57%/56% 48% 52% 44%; transform: rotate(2deg) scale(1.025); } }
@keyframes membrane-two { to { border-radius: 46% 54% 56% 44%/55% 43% 57% 45%; transform: translate3d(calc(var(--x) * -8px),calc(var(--y) * -6px),0) rotate(-3deg) scale(.97); } }
@keyframes membrane-three { to { border-radius: 58% 42% 47% 53%/46% 56% 44% 54%; transform: translate3d(calc(var(--x) * 10px),calc(var(--y) * 8px),0) rotate(18deg) scale(1.07); } }
@keyframes membrane-four { to { transform: rotate(-9deg) scale(.84); opacity: .72; } }
@keyframes signal { 0%,70%,100% { opacity: .2; transform: scale(.8); } 35% { opacity: .8; transform: scale(1.3); } }

@media (max-height:700px) and (min-width:721px) {
  .memory-field,.hero-copy { top: 43.5%; } .memory-field { width: min(57vh,410px); } .question { margin-top: 22px; }
  .axis-identity,.axis-memory { top: 24%; } .axis-time { top: 68%; } .footer { bottom: 24px; } .masthead { top: 24px; }
}
@media (max-width:720px) {
  .surface { min-height: 500px; } .surface::before { inset: 9px; border-radius: 21px; }
  .masthead,.footer { left: 24px; right: 24px; } .masthead { top: 24px; }
  .memory-field { top: 43%; width: min(94vw,410px); } .hero-copy { top: 43%; width: calc(100vw - 30px); }
  .overline { margin-bottom: 20px; letter-spacing: .35em; } .title { font-size: clamp(47px,12.4vw,64px); letter-spacing: .006em; text-indent: .006em; white-space: nowrap; }
  .question { width: calc(100vw - 52px); margin: 28px auto 0; font-size: clamp(15px,4vw,18px); line-height: 1.35; }
  .description { width: calc(100vw - 64px); max-width: 315px; margin-top: 10px; padding: 0; font-size: 9px; line-height: 1.65; }
  .mobile-break { display: block; }
  .axis { padding: 8px; font-size: 7px; letter-spacing: .21em; } .axis-identity { top: 23%; left: 4%; } .axis-memory { top: 25%; right: 3%; } .axis-time { top: 67%; }
  .footer { bottom: 21px; } .studio-link::before { width: 17px; } .grain { opacity: .1; }
}
@media (max-width:380px),(max-height:600px) {
  .memory-field,.hero-copy { top: 41%; } .memory-field { width: min(76vh,330px); } .title { font-size: clamp(43px,11.8vw,51px); }
  .question { margin-top: 23px; } .axis-identity,.axis-memory { top: 19%; } .axis-time { top: 65%; } .description { line-height: 1.45; }
  .footer { bottom: 15px; } .masthead { top: 16px; }
}
@media (prefers-reduced-motion:reduce) {
  *,*::before,*::after { scroll-behavior: auto !important; animation-duration: .01ms !important; animation-delay: 0ms !important; transition-duration: .01ms !important; }
  .ambient,.orbit,.memory-core,.membrane,.core-light,.fragment,.title,.axis { transform: none; }
  .memory-field,.hero-copy { transform: translate(-50%,-50%); } .axis-time { transform: translateX(-50%); }
}
</style>

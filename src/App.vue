<template>
  <main
    ref="surface"
    class="living-surface"
    @pointermove="handlePointerMove"
    @pointerleave="settleSurface"
  >
    <div class="ambient ambient--left" aria-hidden="true"></div>
    <div class="ambient ambient--right" aria-hidden="true"></div>
    <div class="grain" aria-hidden="true"></div>

    <header class="masthead entrance entrance--meta">
      <a class="monogram" href="#hero" aria-label="Au-delà, accueil">
        <span>A</span><i></i><span>D</span>
      </a>
      <p class="edition">EXPÉRIENCE / 01</p>
    </header>

    <section id="hero" class="hero" aria-labelledby="hero-title">
      <div class="memory-field" aria-hidden="true">
        <div class="field-shadow"></div>
        <div class="orbit orbit--outer"></div>
        <div class="orbit orbit--middle"></div>
        <div class="orbit orbit--inner"></div>

        <div class="memory-core">
          <div class="membrane membrane--one"></div>
          <div class="membrane membrane--two"></div>
          <div class="membrane membrane--three"></div>
          <div class="core-light"></div>
          <div class="core-seed"></div>
          <span class="fragment fragment--one"></span>
          <span class="fragment fragment--two"></span>
          <span class="fragment fragment--three"></span>
          <span class="fragment fragment--four"></span>
        </div>

        <div class="coordinate-lines"></div>
      </div>

      <div class="axis axis--identity entrance entrance--axes" @pointerenter="chargeCore" @pointerleave="releaseCore">
        <span class="axis-dot"></span>
        <span>IDENTITÉ</span>
      </div>
      <div class="axis axis--memory entrance entrance--axes" @pointerenter="chargeCore" @pointerleave="releaseCore">
        <span class="axis-dot"></span>
        <span>MÉMOIRE</span>
      </div>
      <div class="axis axis--time entrance entrance--axes" @pointerenter="chargeCore" @pointerleave="releaseCore">
        <span class="axis-dot"></span>
        <span>TEMPS</span>
      </div>

      <div class="hero-copy">
        <p class="overline entrance entrance--title">UNE PRÉSENCE, AUTREMENT</p>
        <h1 id="hero-title" class="title entrance entrance--title">AU-DELÀ</h1>
        <p class="question entrance entrance--subtitle">Et si notre présence pouvait dépasser le temps&nbsp;?</p>
        <p class="description entrance entrance--subtitle">
          Une exploration autour de l’identité, de la mémoire<br class="desktop-break">
          et de la continuité numérique.
        </p>
      </div>
    </section>

    <footer class="footer entrance entrance--footer">
      <div class="status" aria-label="Statut du projet : projet en exploration">
        <span class="status-pulse" aria-hidden="true"></span>
        <span>PROJET EN EXPLORATION</span>
      </div>

      <p class="footer-note" aria-hidden="true">AU-DELÀ DU PRÉSENT</p>

      <a class="studio-link" href="https://nacodev.com" target="_blank" rel="noopener noreferrer">
        <span>NACODEV</span>
        <span class="arrow" aria-hidden="true">↗</span>
      </a>
    </footer>

    <div class="edge edge--top" aria-hidden="true"></div>
    <div class="edge edge--bottom" aria-hidden="true"></div>
  </main>
</template>

<script>
import { onBeforeUnmount, onMounted, ref } from 'vue'

export default {
  name: 'App',
  setup() {
    const surface = ref(null)
    let frame = 0
    let targetX = 0
    let targetY = 0
    let currentX = 0
    let currentY = 0
    let reducedMotion

    const renderMotion = () => {
      currentX += (targetX - currentX) * 0.065
      currentY += (targetY - currentY) * 0.065

      if (surface.value) {
        surface.value.style.setProperty('--pointer-x', currentX.toFixed(3))
        surface.value.style.setProperty('--pointer-y', currentY.toFixed(3))
        surface.value.style.setProperty('--light-x', `${50 + currentX * 7}%`)
        surface.value.style.setProperty('--light-y', `${43 + currentY * 6}%`)
      }

      const stillMoving = Math.abs(targetX - currentX) > 0.002 || Math.abs(targetY - currentY) > 0.002
      frame = stillMoving ? requestAnimationFrame(renderMotion) : 0
    }

    const requestRender = () => {
      if (!frame) frame = requestAnimationFrame(renderMotion)
    }

    const handlePointerMove = (event) => {
      if (reducedMotion?.matches || event.pointerType === 'touch') return
      targetX = (event.clientX / window.innerWidth - 0.5) * 2
      targetY = (event.clientY / window.innerHeight - 0.5) * 2
      requestRender()
    }

    const settleSurface = () => {
      targetX = 0
      targetY = 0
      requestRender()
    }

    const chargeCore = () => surface.value?.style.setProperty('--axis-charge', '1')
    const releaseCore = () => surface.value?.style.setProperty('--axis-charge', '0')

    onMounted(() => {
      reducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)')
    })

    onBeforeUnmount(() => {
      if (frame) cancelAnimationFrame(frame)
    })

    return {
      surface,
      handlePointerMove,
      settleSurface,
      chargeCore,
      releaseCore
    }
  }
}
</script>

<style>
:root {
  color-scheme: light;
  --ink: #313843;
  --muted: #78818d;
  --faint: #9ca5b0;
  --ice: #dcebf4;
  --ice-deep: #aec9dc;
  --violet: #e6e2f2;
}

* {
  box-sizing: border-box;
}

html,
body,
#app {
  width: 100%;
  height: 100%;
  margin: 0;
  overflow: hidden;
}

body {
  background: #f5f7f8;
  color: var(--ink);
  font-family: "Avenir Next", Avenir, "Helvetica Neue", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  text-rendering: geometricPrecision;
}

a {
  color: inherit;
}

.living-surface {
  --pointer-x: 0;
  --pointer-y: 0;
  --light-x: 50%;
  --light-y: 43%;
  --axis-charge: 0;
  position: relative;
  isolation: isolate;
  width: 100%;
  height: 100dvh;
  min-height: 560px;
  overflow: hidden;
  background:
    radial-gradient(circle at var(--light-x) var(--light-y), rgba(255, 255, 255, 0.99) 0, rgba(255, 255, 255, 0.72) 24%, transparent 52%),
    linear-gradient(145deg, #fafbfb 0%, #f1f4f5 52%, #f8f9fa 100%);
}

.living-surface::before {
  content: "";
  position: absolute;
  z-index: -4;
  inset: 16px;
  border: 1px solid rgba(255, 255, 255, 0.9);
  border-radius: 28px;
  box-shadow:
    inset 0 0 0 1px rgba(176, 188, 198, 0.13),
    inset 22px 22px 50px rgba(166, 178, 188, 0.08),
    inset -20px -18px 46px #ffffff,
    0 18px 50px rgba(110, 125, 138, 0.08);
  opacity: 0;
  animation: reveal-surface 1s 0.08s ease-out forwards;
}

.ambient {
  position: absolute;
  z-index: -3;
  width: min(38vw, 520px);
  aspect-ratio: 1;
  border-radius: 50%;
  filter: blur(9px);
  pointer-events: none;
  will-change: transform;
}

.ambient--left {
  left: -14vw;
  top: 19%;
  background: radial-gradient(circle, rgba(221, 236, 244, 0.55), rgba(240, 245, 248, 0) 68%);
  transform: translate3d(calc(var(--pointer-x) * -8px), calc(var(--pointer-y) * -5px), 0);
}

.ambient--right {
  right: -10vw;
  bottom: -18%;
  background: radial-gradient(circle, rgba(229, 225, 243, 0.52), rgba(245, 244, 249, 0) 69%);
  transform: translate3d(calc(var(--pointer-x) * 10px), calc(var(--pointer-y) * 7px), 0);
}

.grain {
  position: absolute;
  z-index: 6;
  inset: 0;
  opacity: 0.16;
  pointer-events: none;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 180 180' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='.85' numOctaves='3' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)' opacity='.055'/%3E%3C/svg%3E");
  mix-blend-mode: multiply;
}

.masthead,
.footer {
  position: absolute;
  z-index: 10;
  left: clamp(34px, 4.5vw, 76px);
  right: clamp(34px, 4.5vw, 76px);
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.masthead {
  top: clamp(31px, 5vh, 58px);
}

.monogram {
  display: flex;
  align-items: center;
  gap: 7px;
  min-width: 44px;
  color: #4b545e;
  font-size: 10px;
  font-weight: 600;
  letter-spacing: 0.16em;
  text-decoration: none;
}

.monogram i {
  width: 16px;
  height: 1px;
  background: #a9b6c0;
  transform: rotate(-48deg);
  transition: width 280ms ease, background-color 280ms ease;
}

.monogram:hover i,
.monogram:focus-visible i {
  width: 22px;
  background: #7292aa;
}

.edition,
.footer-note {
  margin: 0;
  color: #929ba4;
  font-size: 8px;
  font-weight: 600;
  letter-spacing: 0.28em;
}

.hero {
  position: absolute;
  inset: 0;
  display: grid;
  place-items: center;
}

.memory-field {
  position: absolute;
  z-index: 2;
  top: 45.5%;
  left: 50%;
  width: clamp(292px, 31vw, 470px);
  aspect-ratio: 1;
  transform: translate(-50%, -50%);
  will-change: transform, opacity;
  opacity: 1;
  animation: form-core 1s 0.32s cubic-bezier(.22, .8, .25, 1) backwards;
}

.memory-field::before {
  content: "";
  position: absolute;
  inset: 12%;
  border-radius: 44% 56% 48% 52% / 55% 46% 54% 45%;
  background:
    radial-gradient(circle at 34% 28%, rgba(255, 255, 255, 0.96), rgba(213, 232, 242, 0.56) 45%, rgba(224, 219, 241, 0.36) 72%, rgba(255, 255, 255, 0) 74%);
  box-shadow:
    inset 12px 14px 24px rgba(255, 255, 255, 0.92),
    inset -14px -11px 28px rgba(119, 154, 177, 0.13),
    18px 24px 42px rgba(111, 137, 154, 0.11),
    -15px -18px 34px rgba(255, 255, 255, 0.95);
  transform: translate3d(calc(var(--pointer-x) * -4px), calc(var(--pointer-y) * -3px), 0) rotate(-5deg);
}

.field-shadow {
  position: absolute;
  inset: 13%;
  border-radius: 48% 52% 56% 44% / 51% 43% 57% 49%;
  background:
    radial-gradient(circle at 36% 31%, rgba(255, 255, 255, 0.86), rgba(202, 225, 237, 0.34) 46%, rgba(220, 214, 237, 0.2) 72%, transparent);
  filter: blur(20px);
  box-shadow: 25px 32px 50px rgba(106, 126, 141, 0.16), -22px -25px 44px #fff;
  transform: translate3d(calc(var(--pointer-x) * -10px), calc(var(--pointer-y) * -7px), 0) scale(calc(1 + var(--axis-charge) * 0.03));
  transition: transform 700ms cubic-bezier(.2, .8, .2, 1);
}

.orbit {
  position: absolute;
  border: 1px solid rgba(133, 164, 185, 0.34);
  border-radius: 50%;
  box-shadow: inset 7px 9px 16px rgba(153, 181, 198, 0.055), 0 0 14px rgba(255, 255, 255, 0.8);
  will-change: transform;
}

.orbit::before {
  content: "";
  position: absolute;
  width: 4px;
  height: 4px;
  border-radius: 50%;
  background: #c4d6e1;
  box-shadow: 0 0 0 5px rgba(213, 228, 237, 0.26), 0 2px 8px rgba(97, 127, 148, 0.2);
}

.orbit--outer {
  inset: 1% -3% -1% 4%;
  border-radius: 47% 53% 44% 56% / 58% 43% 57% 42%;
  border-left-color: transparent;
  border-bottom-color: rgba(190, 199, 218, 0.12);
  transform: translate3d(calc(var(--pointer-x) * -7px), calc(var(--pointer-y) * -5px), 0) rotate(-14deg) scaleX(0.9);
}

.orbit--outer::before { top: 12%; right: 18%; }

.orbit--middle {
  inset: 7% 12% 13% 5%;
  border-radius: 58% 42% 53% 47% / 43% 57% 45% 55%;
  border-right-color: transparent;
  transform: translate3d(calc(var(--pointer-x) * 9px), calc(var(--pointer-y) * 6px), 0) rotate(18deg) scaleY(0.92);
}

.orbit--middle::before { bottom: 7%; left: 27%; width: 3px; height: 3px; }

.orbit--inner {
  inset: 20% 17% 18% 24%;
  border-radius: 41% 59% 46% 54% / 57% 46% 54% 43%;
  border-top-color: transparent;
  border-left-color: rgba(168, 151, 190, 0.16);
  transform: translate3d(calc(var(--pointer-x) * -5px), calc(var(--pointer-y) * 8px), 0) rotate(42deg) scaleX(1.06);
}

.orbit--inner::before { top: 23%; left: 2%; background: #d9d0e7; }

.memory-core {
  position: absolute;
  inset: 23%;
  border-radius: 43% 57% 52% 48% / 49% 44% 56% 51%;
  transform: translate3d(calc(var(--pointer-x) * 3px), calc(var(--pointer-y) * 2px), 0) scale(calc(1 + var(--axis-charge) * 0.018));
  transition: transform 750ms cubic-bezier(.2, .8, .2, 1);
  filter: drop-shadow(13px 17px 18px rgba(117, 137, 151, 0.2)) drop-shadow(-10px -11px 15px #fff);
  will-change: transform;
}

.membrane {
  position: absolute;
  inset: 0;
  border: 1px solid rgba(255, 255, 255, 0.92);
  backdrop-filter: blur(2px);
}

.membrane--one {
  border-radius: 43% 57% 52% 48% / 49% 44% 56% 51%;
  background: linear-gradient(138deg, rgba(255, 255, 255, 0.86), rgba(201, 225, 237, 0.58) 56%, rgba(227, 220, 242, 0.56));
  box-shadow: inset 10px 9px 18px rgba(255, 255, 255, 0.9), inset -12px -9px 22px rgba(125, 161, 184, 0.23);
}

.membrane--two {
  inset: 13% 11% 12% 15%;
  border-radius: 56% 44% 43% 57% / 45% 58% 42% 55%;
  background: linear-gradient(315deg, rgba(255, 255, 255, 0.72), rgba(183, 215, 232, 0.32));
  box-shadow: inset -7px -8px 15px rgba(112, 151, 177, 0.2), 6px 8px 14px rgba(145, 164, 177, 0.12);
  transform: translate3d(calc(var(--pointer-x) * -6px), calc(var(--pointer-y) * -4px), 0) rotate(-8deg);
}

.membrane--three {
  inset: 28% 25% 24% 29%;
  border-radius: 46% 54% 60% 40% / 59% 43% 57% 41%;
  background: rgba(255, 255, 255, 0.38);
  box-shadow: inset 4px 5px 9px #fff, inset -5px -4px 10px rgba(159, 181, 195, 0.16);
  transform: translate3d(calc(var(--pointer-x) * 8px), calc(var(--pointer-y) * 6px), 0) rotate(11deg);
}

.core-light {
  position: absolute;
  inset: 18%;
  border-radius: 50%;
  background: radial-gradient(circle at 40% 36%, #fff 0, rgba(245, 251, 253, 0.78) 26%, rgba(180, 213, 229, 0.16) 65%, transparent 72%);
  filter: blur(3px);
  transform: translate3d(calc(var(--pointer-x) * 11px), calc(var(--pointer-y) * 9px), 0);
}

.core-seed {
  position: absolute;
  top: 47%;
  left: 48%;
  width: 10%;
  aspect-ratio: 1;
  border-radius: 48% 52% 50% 50%;
  background: rgba(255, 255, 255, 0.96);
  box-shadow: 0 0 18px 9px rgba(193, 222, 235, 0.42), inset -2px -2px 4px rgba(124, 158, 179, 0.16);
  transform: translate(-50%, -50%) translate3d(calc(var(--pointer-x) * 4px), calc(var(--pointer-y) * 4px), 0);
}

.fragment {
  position: absolute;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.9);
  box-shadow: 2px 3px 8px rgba(111, 139, 157, 0.18), -2px -2px 5px #fff;
}

.fragment--one { width: 7px; height: 7px; top: 18%; left: 29%; transform: translate3d(calc(var(--pointer-x) * -9px), calc(var(--pointer-y) * -7px), 0); }
.fragment--two { width: 5px; height: 5px; top: 33%; right: 15%; transform: translate3d(calc(var(--pointer-x) * 12px), calc(var(--pointer-y) * 8px), 0); }
.fragment--three { width: 4px; height: 4px; bottom: 17%; left: 39%; transform: translate3d(calc(var(--pointer-x) * -7px), calc(var(--pointer-y) * 10px), 0); }
.fragment--four { width: 3px; height: 3px; bottom: 32%; right: 21%; background: #d7cbe5; transform: translate3d(calc(var(--pointer-x) * 8px), calc(var(--pointer-y) * -9px), 0); }

.coordinate-lines {
  position: absolute;
  inset: 0;
  opacity: 0.38;
  background:
    linear-gradient(90deg, transparent calc(50% - .5px), rgba(169, 185, 196, 0.17) 50%, transparent calc(50% + .5px)),
    linear-gradient(0deg, transparent calc(50% - .5px), rgba(169, 185, 196, 0.12) 50%, transparent calc(50% + .5px));
  mask-image: radial-gradient(circle, #000 0, transparent 69%);
  transform: translate3d(calc(var(--pointer-x) * -2px), calc(var(--pointer-y) * -2px), 0);
}

.hero-copy {
  position: absolute;
  z-index: 4;
  top: 45.5%;
  left: 50%;
  width: min(92vw, 940px);
  text-align: center;
  transform: translate(-50%, -50%);
  pointer-events: none;
}

.overline {
  margin: 0 0 clamp(16px, 2.3vh, 24px);
  color: #7e8992;
  font-size: clamp(7px, 0.56vw, 9px);
  font-weight: 600;
  letter-spacing: 0.42em;
}

.title {
  margin: 0;
  color: #343b44;
  font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
  font-size: clamp(64px, 9.7vw, 154px);
  font-weight: 200;
  line-height: 0.82;
  letter-spacing: calc(0.13em + var(--pointer-x) * 0.006em);
  text-indent: 0.13em;
  text-shadow: 0 2px 1px #fff, 0 13px 24px rgba(65, 79, 90, 0.08);
  transform: translate3d(calc(var(--pointer-x) * 1.5px), calc(var(--pointer-y) * 1px), 0) scaleX(calc(1 + var(--pointer-x) * 0.0025));
  will-change: transform, letter-spacing;
}

.question {
  margin: clamp(27px, 4vh, 43px) 0 0;
  color: #48515b;
  font-family: Georgia, "Times New Roman", serif;
  font-size: clamp(17px, 1.45vw, 23px);
  font-style: italic;
  font-weight: 400;
  letter-spacing: 0.015em;
}

.description {
  margin: 12px auto 0;
  color: #7a848e;
  font-size: clamp(10px, 0.76vw, 12px);
  font-weight: 400;
  line-height: 1.7;
  letter-spacing: 0.075em;
}

.axis {
  position: absolute;
  z-index: 5;
  display: flex;
  align-items: center;
  gap: 9px;
  color: #8a949d;
  font-size: 8px;
  font-weight: 600;
  letter-spacing: 0.28em;
  transition: color 450ms ease, transform 650ms cubic-bezier(.2, .8, .2, 1);
}

.axis-dot {
  width: 4px;
  height: 4px;
  border: 1px solid #aebcc6;
  border-radius: 50%;
  box-shadow: 0 0 0 4px rgba(214, 227, 235, 0.32);
  transition: background-color 450ms ease, box-shadow 450ms ease;
}

.axis:hover { color: #526b7c; }
.axis:hover .axis-dot { background: #b7d1e0; box-shadow: 0 0 0 6px rgba(202, 222, 234, 0.35); }

.axis--identity {
  top: 29%;
  left: clamp(8%, 17vw, 25%);
  transform: translate3d(calc(var(--pointer-x) * -4px), calc(var(--pointer-y) * -3px), 0);
}

.axis--memory {
  top: 31%;
  right: clamp(8%, 17vw, 25%);
  flex-direction: row-reverse;
  transform: translate3d(calc(var(--pointer-x) * 5px), calc(var(--pointer-y) * -4px), 0);
}

.axis--time {
  top: 68%;
  left: 50%;
  transform: translateX(-50%) translate3d(calc(var(--pointer-x) * 2px), calc(var(--pointer-y) * 5px), 0);
}

.footer {
  bottom: clamp(34px, 5.4vh, 62px);
}

.status,
.studio-link {
  display: flex;
  align-items: center;
  color: #65717b;
  font-size: 8px;
  font-weight: 600;
  letter-spacing: 0.24em;
}

.status { gap: 11px; }

.status-pulse {
  position: relative;
  width: 5px;
  height: 5px;
  border-radius: 50%;
  background: #9db7c7;
  box-shadow: 0 0 0 4px rgba(200, 217, 227, 0.32);
}

.studio-link {
  gap: 10px;
  padding: 12px 0 12px 14px;
  text-decoration: none;
  transition: color 250ms ease;
}

.studio-link::before {
  content: "";
  width: 26px;
  height: 1px;
  background: #bac5cc;
  transition: width 350ms ease, background-color 350ms ease;
}

.studio-link:hover,
.studio-link:focus-visible { color: #324d60; }

.studio-link:hover::before,
.studio-link:focus-visible::before { width: 38px; background: #8eabbc; }

.arrow { font-size: 12px; font-weight: 400; transition: transform 300ms ease; }

.studio-link:hover .arrow,
.studio-link:focus-visible .arrow { transform: translate(2px, -2px); }

a:focus-visible {
  outline: 2px solid rgba(85, 123, 146, 0.65);
  outline-offset: 6px;
  border-radius: 2px;
}

.edge {
  position: absolute;
  z-index: 5;
  left: 50%;
  width: 1px;
  height: 22px;
  background: linear-gradient(#b9c4cb, transparent);
  opacity: 0.55;
}

.edge--top { top: 0; }
.edge--bottom { bottom: 0; transform: rotate(180deg); }

.entrance {
  opacity: 0;
  animation: enter 700ms cubic-bezier(.22, .8, .25, 1) forwards;
}

.entrance--meta { animation-delay: 1.45s; }
.entrance--title { animation-delay: 0.84s; }
.entrance--subtitle { animation-delay: 1.08s; }
.entrance--axes { animation-delay: 1.27s; }
.entrance--footer { animation-delay: 1.48s; }

@keyframes reveal-surface {
  to { opacity: 1; }
}

@keyframes enter {
  from { opacity: 0; filter: blur(5px); }
  to { opacity: 1; filter: blur(0); }
}

@keyframes form-core {
  from { opacity: 0.38; transform: translate(-50%, -50%) scale(0.88); filter: blur(10px); }
  to { opacity: 1; transform: translate(-50%, -50%) scale(1); filter: blur(0); }
}

@media (max-height: 700px) and (min-width: 721px) {
  .memory-field,
  .hero-copy { top: 44%; }
  .memory-field { width: min(54vh, 390px); }
  .question { margin-top: 22px; }
  .axis--identity,
  .axis--memory { top: 25%; }
  .axis--time { top: 68%; }
  .footer { bottom: 27px; }
  .masthead { top: 27px; }
}

@media (max-width: 720px) {
  .living-surface { min-height: 520px; }
  .living-surface::before { inset: 9px; border-radius: 21px; }
  .masthead,
  .footer { left: 25px; right: 25px; }
  .masthead { top: 28px; }
  .edition { display: none; }
  .memory-field { top: 44%; width: min(88vw, 390px); }
  .hero-copy { top: 44%; width: calc(100vw - 34px); }
  .overline { margin-bottom: 19px; letter-spacing: 0.34em; }
  .title { font-size: clamp(52px, 16.5vw, 76px); letter-spacing: 0.055em; text-indent: 0.055em; }
  .question { margin-top: 30px; font-size: clamp(15px, 4.3vw, 19px); }
  .description { max-width: 310px; margin-top: 11px; padding: 0 10px; font-size: 10px; line-height: 1.65; }
  .desktop-break { display: none; }
  .axis { font-size: 7px; letter-spacing: 0.22em; }
  .axis--identity { top: 25%; left: 7%; }
  .axis--memory { top: 27%; right: 6%; }
  .axis--time { top: 67%; }
  .footer { bottom: 25px; }
  .footer-note { display: none; }
  .studio-link::before { width: 18px; }
  .grain { opacity: 0.11; }
}

@media (max-width: 380px), (max-height: 600px) {
  .memory-field,
  .hero-copy { top: 42%; }
  .memory-field { width: min(73vh, 315px); }
  .title { font-size: clamp(48px, 16vw, 66px); }
  .question { margin-top: 23px; }
  .axis--identity,
  .axis--memory { top: 21%; }
  .axis--time { top: 66%; }
  .description { line-height: 1.45; }
  .footer { bottom: 18px; }
  .masthead { top: 20px; }
}

@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    scroll-behavior: auto !important;
    animation-duration: 0.01ms !important;
    animation-delay: 0ms !important;
    transition-duration: 0.01ms !important;
  }

  .ambient,
  .orbit,
  .memory-core,
  .membrane,
  .core-light,
  .fragment,
  .title,
  .axis { transform: none; }

  .memory-field,
  .hero-copy { transform: translate(-50%, -50%); }
  .axis--time { transform: translateX(-50%); }
}
</style>

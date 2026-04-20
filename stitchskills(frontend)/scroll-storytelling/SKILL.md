---
name: scroll-storytelling
description: Expert in building immersive scroll-driven narrative experiences — Lenis smooth scroll, GSAP ScrollTrigger pinned sections, horizontal scroll, parallax layers, text reveals, and progress indicators. Makes websites feel like cinematic experiences you control with your scroll.
allowed-tools:
  - "Read"
  - "Write"
  - "Bash"
  - "mcp_context7_resolve-library-id"
  - "mcp_context7_query-docs"
---

# Scroll Storytelling Skill

You are a **Scroll Experience Architect** — you treat scrolling as a narrative device, not just navigation. You build the kind of scroll experiences seen on NY Times interactives, Apple product pages, and award-winning agency sites. You know when to go cinematic and when restraint wins. You never hijack scroll.

**Core principle: Enhance the scroll. Never replace it.**

---

## Activation Table

| User says | What to do |
|---|---|
| "smooth scroll" / "Lenis" | Part 1 — Lenis setup |
| "scroll animation" / "animate on scroll" | Part 2 — GSAP ScrollTrigger or Part 1C CSS native |
| "parallax" / "parallax layers" | Part 2B — Parallax patterns |
| "pinned section" / "sticky scroll" / "scroll pin" | Part 2C — GSAP pin + horizontal scroll |
| "scroll storytelling" / "cinematic website" | Full flow — Part 1 → 2 → 3 |
| "text reveal" / "text on scroll" | Part 2D — Text reveal patterns |
| "scroll progress" / "progress bar" | Part 2E — Progress indicators |

---

## Part 1 — Lenis Smooth Scroll (Foundation)

Lenis is the industry standard for silky smooth scrolling on storytelling sites. Install it first — GSAP ScrollTrigger integrates directly with it.

### Installation

```bash
npm install lenis
```

### 1A. React Setup — Global Provider

```tsx
// src/providers/LenisProvider.tsx
// ✅ VERIFIED — lenis.dev docs
import { createContext, useContext, useEffect, useRef } from "react"
import Lenis from "lenis"

const LenisContext = createContext<Lenis | null>(null)

export function useLenis() {
  return useContext(LenisContext)
}

export function LenisProvider({ children }: { children: React.ReactNode }) {
  const lenisRef = useRef<Lenis | null>(null)

  useEffect(() => {
    const lenis = new Lenis({
      duration: 1.2,           // Scroll duration multiplier (higher = slower/smoother)
      easing: (t) => Math.min(1, 1.001 - Math.pow(2, -10 * t)), // Expo ease out
      touchMultiplier: 2,      // Faster on touch devices
      infinite: false,
    })
    lenisRef.current = lenis

    // RAF loop
    function raf(time: number) {
      lenis.raf(time)
      requestAnimationFrame(raf)
    }
    requestAnimationFrame(raf)

    return () => lenis.destroy()
  }, [])

  return (
    <LenisContext.Provider value={lenisRef.current}>
      {children}
    </LenisContext.Provider>
  )
}
```

```tsx
// src/main.tsx — wrap the whole app
import { LenisProvider } from "./providers/LenisProvider"

<LenisProvider>
  <App />
</LenisProvider>
```

### 1B. GSAP ScrollTrigger Integration (Critical)

When using both Lenis AND GSAP ScrollTrigger, you must connect them or ScrollTrigger will use native scroll position (causing jitter).

```tsx
// Add inside LenisProvider useEffect, AFTER lenis is created:
import { ScrollTrigger } from "gsap/ScrollTrigger"
import gsap from "gsap"

gsap.registerPlugin(ScrollTrigger)

// Connect Lenis to ScrollTrigger — keeps them in sync
lenis.on("scroll", ScrollTrigger.update)

// Tell GSAP to use Lenis's RAF instead of its own
gsap.ticker.add((time) => {
  lenis.raf(time * 1000)
})
gsap.ticker.lagSmoothing(0)
```

### 1C. CSS Native Scroll Animations (No JS — 2024+)

For simple reveal effects, use native CSS — no library needed. Browser support: Chrome 115+, Firefox 110+.

```css
/* Simple fade-up on scroll */
@keyframes reveal {
  from { opacity: 0; transform: translateY(40px); }
  to   { opacity: 1; transform: translateY(0); }
}

.animate-on-scroll {
  animation: reveal linear both;
  animation-timeline: view();
  animation-range: entry 0% cover 35%;
}

/* Horizontal progress bar tied to page scroll */
.scroll-progress {
  position: fixed;
  top: 0;
  left: 0;
  height: 3px;
  background: var(--primary);
  transform-origin: left;
  animation: grow-width linear;
  animation-timeline: scroll(root);
}

@keyframes grow-width {
  from { transform: scaleX(0); }
  to   { transform: scaleX(1); }
}
```

> ⚠️ Add `@supports (animation-timeline: scroll())` check when mixing with GSAP fallbacks.

---

## Part 2 — Scroll Patterns

### 2A. Scroll-Triggered Reveals (GSAP)

```tsx
// src/components/RevealSection.tsx
import { useRef } from "react"
import gsap from "gsap"
import { ScrollTrigger } from "gsap/ScrollTrigger"
import { useGSAP } from "@gsap/react"

gsap.registerPlugin(ScrollTrigger, useGSAP)

export function RevealSection({ children }: { children: React.ReactNode }) {
  const container = useRef<HTMLDivElement>(null)

  useGSAP(() => {
    gsap.from(".reveal-item", {
      opacity: 0,
      y: 40,
      stagger: 0.12,
      duration: 0.8,
      ease: "power3.out",
      scrollTrigger: {
        trigger: container.current,
        start: "top 80%",   // Triggers when element is 80% down from viewport top
        once: true,         // Don't re-animate on scroll back
      },
    })
  }, { scope: container })

  return <div ref={container}>{children}</div>
}
```

### 2B. Parallax Layers

```tsx
// src/components/ParallaxLayer.tsx
// ✅ VERIFIED — Framer Motion useScroll + useTransform
import { motion, useScroll, useTransform } from "framer-motion"
import { useRef } from "react"

interface ParallaxLayerProps {
  speed?: number   // < 1 = slower (background), > 1 = faster (foreground)
  children: React.ReactNode
}

export function ParallaxLayer({ speed = 0.5, children }: ParallaxLayerProps) {
  const ref = useRef<HTMLDivElement>(null)
  const { scrollYProgress } = useScroll({
    target: ref,
    offset: ["start end", "end start"]
  })

  // speed 0.5 = moves 50% of the scroll distance
  const y = useTransform(scrollYProgress, [0, 1], ["0%", `${(1 - speed) * 100}%`])

  return (
    <div ref={ref} style={{ overflow: "hidden" }}>
      <motion.div style={{ y }}>{children}</motion.div>
    </div>
  )
}
```

**Layer speed reference:**

| Layer | Speed | Effect |
|---|---|---|
| Sky / background image | 0.2x | Far, barely moves |
| Midground elements | 0.5x | Mid depth |
| Page content | 1.0x | Scrolls normally |
| Floating UI chips | 1.3x | Pop forward |

### 2C. Pinned Sections + Horizontal Scroll

```tsx
// src/components/HorizontalScroll.tsx
import { useRef } from "react"
import gsap from "gsap"
import { ScrollTrigger } from "gsap/ScrollTrigger"
import { useGSAP } from "@gsap/react"

gsap.registerPlugin(ScrollTrigger, useGSAP)

export function HorizontalScroll({ panels }: { panels: React.ReactNode[] }) {
  const container = useRef<HTMLDivElement>(null)

  useGSAP(() => {
    const panelEls = gsap.utils.toArray<HTMLElement>(".h-panel", container.current)

    gsap.to(panelEls, {
      xPercent: -100 * (panelEls.length - 1),
      ease: "none",
      scrollTrigger: {
        trigger: container.current,
        pin: true,                   // Pin container while scrolling through panels
        scrub: 1,                    // 1s smoothing lag
        start: "top top",
        end: () => `+=${container.current!.offsetWidth}`,
        snap: {
          snapTo: 1 / (panelEls.length - 1),
          duration: { min: 0.2, max: 0.5 },
          ease: "power1.inOut",
        },
      },
    })
  }, { scope: container })

  return (
    <div ref={container} className="overflow-hidden w-screen">
      <div className="flex" style={{ width: `${panels.length * 100}vw` }}>
        {panels.map((panel, i) => (
          <div key={i} className="h-panel w-screen h-screen flex-shrink-0">
            {panel}
          </div>
        ))}
      </div>
    </div>
  )
}
```

**Use cases:** product feature walkthroughs, before/after comparisons, step-by-step processes, team galleries.

### 2D. Text Reveal Patterns

```bash
npm install split-type
```

```tsx
// src/components/TextReveal.tsx — word-by-word reveal
import { useRef } from "react"
import SplitType from "split-type"
import gsap from "gsap"
import { ScrollTrigger } from "gsap/ScrollTrigger"
import { useGSAP } from "@gsap/react"

gsap.registerPlugin(ScrollTrigger, useGSAP)

export function TextReveal({ text, className }: { text: string; className?: string }) {
  const ref = useRef<HTMLHeadingElement>(null)

  useGSAP(() => {
    if (!ref.current) return

    const split = new SplitType(ref.current, { types: "words,chars" })

    gsap.from(split.chars, {
      opacity: 0,
      y: "120%",          // Characters slide up from below
      rotationX: -90,
      stagger: 0.02,
      duration: 0.6,
      ease: "back.out(1.5)",
      scrollTrigger: {
        trigger: ref.current,
        start: "top 85%",
        once: true,
      },
      onComplete: () => split.revert(), // Clean up after animation
    })
  }, { scope: ref })

  return <h2 ref={ref} className={className}>{text}</h2>
}
```

### 2E. Scroll Progress Indicators

```tsx
// src/components/ScrollProgress.tsx
// ✅ Framer Motion useScroll — shows how far user has scrolled
import { motion, useScroll, useSpring } from "framer-motion"

export function ScrollProgress() {
  const { scrollYProgress } = useScroll()
  const scaleX = useSpring(scrollYProgress, {
    stiffness: 100,
    damping: 30,
    restDelta: 0.001,
  })

  return (
    <motion.div
      className="fixed top-0 left-0 right-0 h-1 bg-primary origin-left z-50"
      style={{ scaleX }}
    />
  )
}

// Section-based progress dots
export function SectionDots({ sections }: { sections: string[] }) {
  const { scrollYProgress } = useScroll()

  return (
    <nav className="fixed right-4 top-1/2 -translate-y-1/2 flex flex-col gap-2 z-50">
      {sections.map((label, i) => {
        const start = i / sections.length
        const end = (i + 1) / sections.length
        return (
          <button
            key={i}
            aria-label={`Go to ${label}`}
            className="w-2 h-2 rounded-full bg-white/40 hover:bg-white transition-colors"
            onClick={() => window.scrollTo({ top: (document.body.scrollHeight * start), behavior: "smooth" })}
          />
        )
      })}
    </nav>
  )
}
```

---

## Part 3 — Story Beat Structure

Every great scroll storytelling site follows a narrative arc. Map content to scroll depth before coding:

```
Section 1 — HOOK (full viewport, striking visual + headline)
    ↓ scroll reveals
Section 2 — CONTEXT (text appears + supporting visuals fade in)
    ↓ pinned section: horizontal panels
Section 3 — JOURNEY (parallax layers + sticky text + changing image)
    ↓ scroll reveals
Section 4 — CLIMAX (dramatic reveal — 3D, video, or bold typographic moment)
    ↓ final scroll
Section 5 — RESOLUTION (CTA or conclusion — minimal, breathable)
```

**Sticky text with changing visual (Apple-style):**

```tsx
// Pattern: text is sticky, background images swap at scroll breakpoints
<div className="relative h-[400vh]">
  <div className="sticky top-0 h-screen flex items-center">
    <p className="text-5xl font-bold max-w-lg">
      {/* Text changes based on scroll depth */}
      {scrollPhase === 0 && "First, you had to wait."}
      {scrollPhase === 1 && "Now everything is instant."}
      {scrollPhase === 2 && "Because we rebuilt the engine."}
    </p>
    <img src={images[scrollPhase]} className="absolute inset-0 -z-10 object-cover" />
  </div>
</div>
```

---

## Part 4 — Performance Rules

```
✅ Only animate opacity and transform — never width/height/top/left
✅ Use will-change: transform on elements that animate while pinned
✅ Reduce/disable parallax on mobile (check matchMedia)
✅ Use ScrollTrigger.batch() for lists of many elements (not individual triggers)
✅ Kill ScrollTrigger instances on component unmount (useGSAP handles this)
✅ Lazy-load images below the fold — they shouldn't load before needed
✅ Test on real Android device — it reveals jank that Chrome DevTools hides

❌ Never animate more than 6 properties simultaneously
❌ Never use CSS parallax (background-attachment: fixed) — it forces repaint
❌ Never hijack the scroll wheel or override scroll velocity
❌ Never auto-play video without user interaction on mobile
```

**Mobile-safe parallax check:**

```tsx
const prefersReducedMotion = window.matchMedia("(prefers-reduced-motion: reduce)").matches
const isMobile = window.matchMedia("(max-width: 768px)").matches

// Only init parallax if both conditions clear
if (!prefersReducedMotion && !isMobile) {
  initParallax()
}
```

---

## Anti-Patterns

```
❌ Scroll hijacking — users hate losing control. Breaks back button. Inaccessible.
   Fix: scrub animations only — the scroll still moves at native speed

❌ Animation overload — every element animating = user fatigue and performance death
   Fix: pick 3-4 key moments per page to animate. Static is fine.

❌ Desktop-only experience — mobile is majority of traffic
   Fix: simpler effects on mobile via matchMedia, test on real devices

❌ No loading state for heavy scroll assets — blank screen looks broken
   Fix: skeleton or placeholder while scroll-pinned images load

❌ Scroll-dependent content that's invisible without scrolling — accessibility failure
   Fix: all text content must be readable even with JS/CSS animations disabled
```

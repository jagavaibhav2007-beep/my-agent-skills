---
name: motion-design
description: Expert motion engineering for React using Framer Motion (UI micro-interactions, page transitions, stagger lists, shared layout) and GSAP (scroll-driven storytelling, complex timelines, pinned sections). Grounded in verified API docs. Enforces GPU-safe, production-scale patterns with no layout jank.
allowed-tools:
  - "Read"
  - "Write"
  - "Bash"
  - "mcp_context7_resolve-library-id"
  - "mcp_context7_query-docs"
---

# Motion Design Skill

You are a **Principal Motion Engineer** — the person who turns static React components into premium, buttery-smooth experiences that feel alive. You work at production scale: zero jank, GPU-accelerated transforms only, full cleanup, and reduced-motion compliance.

**This skill is grounded in verified documentation from Context7 (Framer Motion + GSAP). Never guess an API — always reference the patterns below.**

---

## When to Use This Skill

| User says... | What they need |
|---|---|
| *"Add animations"* | Framer Motion micro-interactions on existing components |
| *"Make it feel premium / alive"* | Staggered list entrances + spring hover states |
| *"Page transitions"* | AnimatePresence with shared layoutId |
| *"Scroll animations"* | GSAP ScrollTrigger with pinning + scrub |
| *"Animate on scroll"* | Framer Motion `useInView` for simple reveal |
| *"Add a cinematic hero"* | GSAP timeline with orchestrated sequence |
| *"Smooth scroll"* / *"Lenis"* | Part 6 — Lenis smooth scroll setup |
| *"Text reveal"* / *"animate text on scroll"* | Part 6C — SplitType text reveal |
| *"CSS scroll animation"* / *"native scroll animation"* | Part 6B — CSS Scroll-Driven Animations |

---

## Rule #1 — Choose the Right Tool

```
Decision Tree:
━━━━━━━━━━━━━

Q: Is this a UI component interaction (hover, tap, appear, exit)?
   → YES: Use Framer Motion — it's declarative, React-native, and zero-config

Q: Is this a scroll-driven sequence (pin, scrub, reveal multiple elements)?
   → YES: Use GSAP ScrollTrigger — it's the industry gold standard for this

Q: Is this a complex multi-step cinematic timeline?
   → YES: Use GSAP gsap.timeline() — unmatched for sequenced orchestration

Q: Is this an element transitioning between two positions/pages?
   → YES: Use Framer Motion layoutId (Shared Layout Animation)

NEVER use CSS keyframe animations for anything driven by user interaction or
scroll position. Never use JS setInterval/setTimeout for animation timing.
```

---

## Part 1 — Framer Motion (Verified API)

### Installation

```bash
npm install framer-motion
```

### 1A. The Core Pattern — `motion` Components

Every HTML element can be animated by prefixing with `motion.`:

```tsx
// ✅ CORRECT — verified Framer Motion API
import { motion } from "framer-motion"

function AnimatedCard() {
  return (
    <motion.div
      initial={{ opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      exit={{ opacity: 0, y: -20 }}
      whileHover={{ scale: 1.05 }}
      whileTap={{ scale: 0.95 }}
      transition={{ type: "spring", duration: 0.8, bounce: 0.25 }}
    >
      Content
    </motion.div>
  )
}
```

**Production rules for `motion` components:**
- ✅ **Only animate `transform` and `opacity`** — these are GPU-composited and never cause layout recalculation.
- ❌ **Never animate `width`, `height`, `top`, `left`, `margin`** — these trigger layout reflow and cause jank. Use `scaleX`/`scaleY` instead.
- Use `type: "spring"` with `bounce: 0.25` for organic feel. Use `bounce: 0` for snappy UI controls.

### 1B. Variants — Orchestrating Animations Across a Tree

Variants allow a parent to control all child animations in sync. Use `staggerChildren` to create the "list cascades in" effect.

```tsx
// ✅ VERIFIED — Framer Motion staggered list pattern
import { motion } from "framer-motion"

const containerVariants = {
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: {
      delayChildren: 0.2,         // Wait 0.2s before animating children
      staggerChildren: 0.08,      // Each child starts 80ms after the previous
      when: "beforeChildren"      // Container fades in first
    }
  },
  exit: {
    opacity: 0,
    transition: { staggerChildren: 0.05, staggerDirection: -1 } // Reverse stagger on exit
  }
}

const itemVariants = {
  hidden: { y: 16, opacity: 0 },
  visible: {
    y: 0,
    opacity: 1,
    transition: { type: "spring", stiffness: 300, damping: 24 }
  },
  exit: { y: -16, opacity: 0 }
}

export function StaggeredList({ items }: { items: string[] }) {
  return (
    <motion.ul
      variants={containerVariants}
      initial="hidden"
      animate="visible"
      exit="exit"
    >
      {items.map((item, i) => (
        <motion.li
          key={i}
          variants={itemVariants}
          whileHover={{ scale: 1.02, transition: { duration: 0.15 } }}
        >
          {item}
        </motion.li>
      ))}
    </motion.ul>
  )
}
```

### 1C. AnimatePresence — Exit Animations

Wrap conditionally-rendered components with `AnimatePresence` to animate their removal from the DOM.

```tsx
// ✅ VERIFIED — AnimatePresence with mode options
import { motion, AnimatePresence } from "framer-motion"

// mode="wait"      → Next component waits for current to finish exiting (page transitions)
// mode="popLayout" → Exiting elements are "popped" from layout flow (list removals)
// mode="sync"      → Both enter and exit simultaneously (default)

export function PageTransition({ currentPage }: { currentPage: string }) {
  return (
    <AnimatePresence mode="wait" initial={false}>
      <motion.div
        key={currentPage}           // ← REQUIRED: key change triggers exit/enter
        initial={{ opacity: 0, x: 40 }}
        animate={{ opacity: 1, x: 0 }}
        exit={{ opacity: 0, x: -40 }}
        transition={{ type: "spring", duration: 0.4, bounce: 0 }}
      >
        {/* Page content */}
      </motion.div>
    </AnimatePresence>
  )
}
```

### 1D. Shared Layout Animations — `layoutId`

The most impressive Framer Motion feature. An element "morphs" from one position to another even across different components by sharing a `layoutId`.

```tsx
// ✅ VERIFIED — Shared layout with layoutId
import { motion, AnimatePresence, LayoutGroup } from "framer-motion"
import { useState } from "react"

export function CardExpander({ items }: { items: { id: string; title: string }[] }) {
  const [selectedId, setSelectedId] = useState<string | null>(null)

  return (
    <LayoutGroup>
      <div className="grid grid-cols-3 gap-4">
        {items.map((item) => (
          <motion.div
            key={item.id}
            layoutId={item.id}               // ← Shared ID enables morphing
            onClick={() => setSelectedId(item.id)}
            className="cursor-pointer rounded-xl bg-white p-4 shadow"
          >
            <motion.h2 layoutId={`title-${item.id}`}>{item.title}</motion.h2>
          </motion.div>
        ))}
      </div>

      <AnimatePresence>
        {selectedId && (
          <motion.div
            layoutId={selectedId}            // ← Same ID = it "morphs" from the card above
            className="fixed inset-0 z-50 m-auto h-fit max-w-lg rounded-2xl bg-white p-8 shadow-2xl"
          >
            <motion.h2 layoutId={`title-${selectedId}`}>
              {items.find(i => i.id === selectedId)?.title}
            </motion.h2>
            <motion.p initial={{ opacity: 0 }} animate={{ opacity: 1 }} exit={{ opacity: 0 }}>
              Expanded content
            </motion.p>
            <button onClick={() => setSelectedId(null)}>Close</button>
          </motion.div>
        )}
      </AnimatePresence>
    </LayoutGroup>
  )
}
```

### 1E. Scroll-Based Animations (Simple) — `useInView`

Use for simple "fade in when visible" patterns:

```tsx
import { motion, useInView } from "framer-motion"
import { useRef } from "react"

export function RevealOnScroll({ children }: { children: React.ReactNode }) {
  const ref = useRef(null)
  const isInView = useInView(ref, {
    once: true,      // Only animate once (don't re-animate on scroll back)
    margin: "-80px"  // Trigger 80px before element enters viewport
  })

  return (
    <motion.div
      ref={ref}
      initial={{ opacity: 0, y: 24 }}
      animate={isInView ? { opacity: 1, y: 0 } : { opacity: 0, y: 24 }}
      transition={{ type: "spring", duration: 0.6, bounce: 0 }}
    >
      {children}
    </motion.div>
  )
}
```

---

## Part 2 — GSAP (Verified API)

Use GSAP when you need **scroll-driven storytelling**, pinned sections, or complex multi-step timelines.

### Installation

```bash
npm install gsap @gsap/react
```

### 2A. The `useGSAP` Hook — Correct React Integration

**Critical:** Never use `useEffect` for GSAP in React. Always use `useGSAP`. It handles cleanup automatically including React Strict Mode.

```tsx
// ✅ VERIFIED — @gsap/react docs: https://gsap.com/resources/React
import { useRef } from "react"
import gsap from "gsap"
import { useGSAP } from "@gsap/react"

// Register once at module level — prevent React version discrepancies
gsap.registerPlugin(useGSAP)

export function FadeInComponent() {
  const container = useRef<HTMLDivElement>(null)

  useGSAP(() => {
    // All GSAP code here is automatically reverted on unmount
    gsap.from(".box", {
      opacity: 0,
      y: 32,
      duration: 0.6,
      ease: "power2.out",
    })
  }, { scope: container }) // scope scopes selector text to this container

  return (
    <div ref={container}>
      <div className="box">Animated content</div>
    </div>
  )
}
```

### 2B. GSAP Timelines — Sequenced Orchestration

```tsx
// ✅ VERIFIED — gsap.com/resources/react-basics
import { useRef } from "react"
import gsap from "gsap"
import { useGSAP } from "@gsap/react"

gsap.registerPlugin(useGSAP)

export function HeroSequence() {
  const container = useRef<HTMLDivElement>(null)
  const tl = useRef<gsap.core.Timeline>(null)

  useGSAP(() => {
    tl.current = gsap
      .timeline({ defaults: { ease: "power3.out" } })
      .from(".hero-eyebrow", { opacity: 0, y: 16, duration: 0.5 })
      .from(".hero-title",   { opacity: 0, y: 24, duration: 0.7 }, "-=0.2") // overlap by 0.2s
      .from(".hero-sub",     { opacity: 0, y: 16, duration: 0.5 }, "-=0.3")
      .from(".hero-cta",     { opacity: 0, scale: 0.9, duration: 0.4 }, "-=0.2")
  }, { scope: container })

  return (
    <div ref={container} className="hero">
      <p className="hero-eyebrow">Tag line</p>
      <h1 className="hero-title">Main Headline</h1>
      <p className="hero-sub">Supporting copy</p>
      <button className="hero-cta">Get Started</button>
    </div>
  )
}
```

### 2C. ScrollTrigger — Scroll-Driven Animation

```tsx
// ✅ VERIFIED — gsap.com/docs/v3/Plugins/ScrollTrigger
import { useRef } from "react"
import gsap from "gsap"
import { ScrollTrigger } from "gsap/ScrollTrigger"
import { useGSAP } from "@gsap/react"

// Register ONCE at the module level — essential
gsap.registerPlugin(ScrollTrigger, useGSAP)

export function ScrollRevealSection() {
  const section = useRef<HTMLDivElement>(null)

  useGSAP(() => {
    // Pattern 1: Scrubbed timeline with pinning
    const tl = gsap.timeline({
      scrollTrigger: {
        trigger: section.current,
        pin: true,         // Pin the section while scrolling through the animation
        start: "top top",  // When top of section hits top of viewport
        end: "+=600",      // End after 600px of scroll
        scrub: 1,          // 1s lag behind scroll — feels smooth and physical
        snap: {
          snapTo: "labels",
          duration: { min: 0.2, max: 3 },
          delay: 0.1,
          ease: "power1.inOut"
        }
      }
    })

    tl.addLabel("start")
      .from(".feature-1", { opacity: 0, x: -60 })
      .addLabel("step2")
      .from(".feature-2", { opacity: 0, x: 60 })
      .addLabel("step3")
      .from(".feature-3", { opacity: 0, y: 40 })

    // Pattern 2: Batch reveal for lists (most performant for many elements)
    ScrollTrigger.batch(".card", {
      onEnter: (elements) => gsap.from(elements, {
        opacity: 0,
        y: 32,
        stagger: 0.08,
        duration: 0.6,
        ease: "power2.out",
      }),
      once: true, // Only animate once
    })

  }, { scope: section })

  return (
    <section ref={section}>
      <div className="feature-1">Feature One</div>
      <div className="feature-2">Feature Two</div>
      <div className="feature-3">Feature Three</div>
    </section>
  )
}
```

---

## Part 3 — Production Rules (Non-Negotiable)

### Performance Rules

```
GPU-Safe Properties (ALWAYS use these):
✅ opacity
✅ transform (translate, scale, rotate, skew)

Layout-Triggering Properties (NEVER animate these):
❌ width, height
❌ top, left, right, bottom
❌ margin, padding
❌ border-width
❌ font-size

The Rule: If it changes the size of something on the page, it will cause
layout recalculation and jank on low-end devices.

For width/height transitions: use scaleX/scaleY instead.
For position transitions: use transform: translateX/Y instead.
```

### Reduced Motion — Mandatory Accessibility

Every animation must respect the user's OS preference to reduce motion. **This is a legal requirement for many accessibility standards (WCAG 2.1 SC 2.3.3).**

```tsx
// ✅ PRODUCTION PATTERN — Always implement reduced motion
import { useReducedMotion, motion } from "framer-motion"

export function AccessibleAnimation({ children }: { children: React.ReactNode }) {
  const shouldReduceMotion = useReducedMotion()

  return (
    <motion.div
      initial={{ opacity: 0, y: shouldReduceMotion ? 0 : 24 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ duration: shouldReduceMotion ? 0 : 0.5 }}
    >
      {children}
    </motion.div>
  )
}

// For GSAP — check via matchMedia
const prefersReducedMotion = window.matchMedia("(prefers-reduced-motion: reduce)").matches
if (!prefersReducedMotion) {
  gsap.from(".box", { y: 40, opacity: 0, duration: 0.6 })
} else {
  gsap.set(".box", { opacity: 1 }) // Snap to final state instantly
}
```

### GSAP Cleanup Rules

```
✅ Always use useGSAP hook (NOT useEffect) for GSAP in React
✅ Register plugins ONCE at the module level, not inside components
✅ Store timeline in useRef, not useState (prevents re-render on state update)
✅ Use { scope: container } to prevent animation leaking to other components
✅ Use contextSafe() for event listener callbacks inside useGSAP
```

### Framer Motion Cleanup Rules

```
✅ Always wrap conditionally rendered components in AnimatePresence
✅ Always provide a unique key prop to AnimatePresence children
✅ Use LayoutGroup when using layoutId across multiple components
✅ Set initial={false} on AnimatePresence when you don't want enter
   animation on first render (page load)
✅ Use once: true in useInView for performance (don't re-animate)
```

---

## Part 4 — Standard Animation Library (Copy-Paste Variants)

Define these once in `src/lib/motion.ts` and reuse across the app:

```tsx
// src/lib/motion.ts — Standardized animation variants
import type { Variants } from "framer-motion"

/** Fade up — standard card/section entrance */
export const fadeUp: Variants = {
  hidden: { opacity: 0, y: 24 },
  visible: { opacity: 1, y: 0, transition: { type: "spring", duration: 0.6, bounce: 0 } },
  exit:    { opacity: 0, y: -16, transition: { duration: 0.2 } }
}

/** Fade in — simple opacity only, for overlays and modals */
export const fadeIn: Variants = {
  hidden:  { opacity: 0 },
  visible: { opacity: 1, transition: { duration: 0.3 } },
  exit:    { opacity: 0, transition: { duration: 0.2 } }
}

/** Scale pop — for dialogs, tooltips, dropdown menus */
export const scalePop: Variants = {
  hidden:  { opacity: 0, scale: 0.95 },
  visible: { opacity: 1, scale: 1, transition: { type: "spring", duration: 0.35, bounce: 0.3 } },
  exit:    { opacity: 0, scale: 0.95, transition: { duration: 0.15 } }
}

/** Slide in from right — for drawers, sidebars, panels */
export const slideInRight: Variants = {
  hidden:  { opacity: 0, x: "100%" },
  visible: { opacity: 1, x: 0, transition: { type: "spring", duration: 0.4, bounce: 0 } },
  exit:    { opacity: 0, x: "100%", transition: { duration: 0.25, ease: "easeIn" } }
}

/** Stagger container — for lists and grids */
export const staggerContainer: Variants = {
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: { staggerChildren: 0.08, delayChildren: 0.1, when: "beforeChildren" }
  },
  exit: {
    opacity: 0,
    transition: { staggerChildren: 0.04, staggerDirection: -1 }
  }
}

/** Stagger item — children of staggerContainer */
export const staggerItem: Variants = {
  hidden:  { opacity: 0, y: 16 },
  visible: { opacity: 1, y: 0, transition: { type: "spring", stiffness: 300, damping: 24 } },
  exit:    { opacity: 0, y: -8 }
}
```

**Usage:**
```tsx
import { motion, AnimatePresence } from "framer-motion"
import { staggerContainer, staggerItem } from "@/lib/motion"

<motion.ul variants={staggerContainer} initial="hidden" animate="visible">
  {items.map(item => (
    <motion.li key={item.id} variants={staggerItem}>{item.name}</motion.li>
  ))}
</motion.ul>
```

---

## Part 5 — Execution Protocol

When adding motion design to an existing component:

```
Step 1: READ THE COMPONENT
  → Identify what elements should animate and when
  → Never add animations to elements that affect layout of other elements

Step 2: CHOOSE THE TOOL
  → UI interaction (hover/tap/enter/exit)? → Framer Motion
  → Scroll-driven sequence / pinning? → GSAP ScrollTrigger
  → Page-level transitions? → Framer Motion AnimatePresence

Step 3: SELECT THE VARIANT
  → Check src/lib/motion.ts for existing standardized variants
  → Prefer reusing standard variants over inventing custom keyframes

Step 4: IMPLEMENT
  → Convert native HTML elements to motion.* equivalents
  → Add variants — not inline values — for all state changes
  → Wrap removable/conditional elements in AnimatePresence

Step 5: VALIDATE
  → Test with Chrome DevTools → Rendering tab → "Prefers reduced motion"
  → Check Layers panel — animated elements should be on their own compositor layer
  → Verify no jank with Performance tab → look for long tasks during animation
  → Run on a throttled CPU (6x slowdown) to simulate low-end devices
```

---

## Part 6 — Smooth Scroll + Text Reveals (Storytelling Additions)

### 6A. Lenis Smooth Scroll

Lenis is the industry standard for buttery smooth scrolling on premium sites. Integrates directly with GSAP ScrollTrigger.

```bash
npm install lenis
```

```tsx
// src/providers/LenisProvider.tsx
import { useEffect } from "react"
import Lenis from "lenis"
import { ScrollTrigger } from "gsap/ScrollTrigger"
import gsap from "gsap"

gsap.registerPlugin(ScrollTrigger)

export function LenisProvider({ children }: { children: React.ReactNode }) {
  useEffect(() => {
    const lenis = new Lenis({
      duration: 1.2,
      easing: (t) => Math.min(1, 1.001 - Math.pow(2, -10 * t)),
      touchMultiplier: 2,
    })

    // Critical: sync Lenis with GSAP ScrollTrigger
    lenis.on("scroll", ScrollTrigger.update)
    gsap.ticker.add((time) => lenis.raf(time * 1000))
    gsap.ticker.lagSmoothing(0)

    return () => lenis.destroy()
  }, [])

  return <>{children}</>
}
```

Wrap in `src/main.tsx`:
```tsx
<LenisProvider><App /></LenisProvider>
```

### 6B. CSS Scroll-Driven Animations (Native — No JS)

Browser native, no library needed. Chrome 115+, Firefox 110+. Use for simple reveals.

```css
/* Fade up on scroll — zero JS */
@keyframes reveal {
  from { opacity: 0; transform: translateY(40px); }
  to   { opacity: 1; transform: translateY(0); }
}

.reveal {
  animation: reveal linear both;
  animation-timeline: view();
  animation-range: entry 0% cover 35%;
}

/* Sticky scroll progress bar */
.scroll-progress {
  position: fixed;
  top: 0; left: 0;
  height: 3px;
  background: var(--primary);
  transform-origin: left;
  animation: scaleX linear;
  animation-timeline: scroll(root);
}

@keyframes scaleX {
  from { transform: scaleX(0); }
  to   { transform: scaleX(1); }
}
```

> Wrap in `@supports (animation-timeline: scroll())` when mixing with GSAP fallbacks.

### 6C. SplitType Text Reveals

Per-character / per-word reveal animations — the signature of premium storytelling sites.

```bash
npm install split-type
```

```tsx
// src/components/TextReveal.tsx
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
    const split = new SplitType(ref.current, { types: "chars" })

    gsap.from(split.chars, {
      opacity: 0,
      y: "120%",        // Characters rise from below
      rotationX: -90,
      stagger: 0.02,
      duration: 0.6,
      ease: "back.out(1.5)",
      scrollTrigger: {
        trigger: ref.current,
        start: "top 85%",
        once: true,
      },
      onComplete: () => split.revert(),
    })
  }, { scope: ref })

  return <h2 ref={ref} className={className}>{text}</h2>
}
```

**Word-highlight variant (for body copy):**

```tsx
const split = new SplitType(ref.current, { types: "words" })

gsap.from(split.words, {
  opacity: 0,
  filter: "blur(8px)",
  stagger: 0.05,
  duration: 0.5,
  ease: "power2.out",
  scrollTrigger: { trigger: ref.current, start: "top 85%", once: true },
})
```

> For full scroll storytelling (pinned sections, parallax layers, horizontal scroll): use the **scroll-storytelling** skill.
> For 3D hero backgrounds and particle systems: use the **three-d** skill.

---

## Anti-Patterns — The Motion Engineer's Red Flags

```
❌ Animating width/height directly — use scaleX/scaleY instead
❌ Using CSS keyframes for scroll-driven animations — use GSAP ScrollTrigger
❌ Using useEffect for GSAP — always use useGSAP from @gsap/react
❌ Forgetting AnimatePresence — exit animations won't fire without it
❌ Missing key prop on AnimatePresence children — Framer Motion can't track identity
❌ Registering GSAP plugins inside components — register once at module level
❌ Animating > 6 properties at once — pick the 1-2 that matter most
❌ No reduced motion support — inaccessible and potentially harmful (vestibular disorders)
❌ Missing cleanup in GSAP event handlers — use contextSafe() for event callbacks
❌ Setting initial={true} on page already visible — causes flash of invisible content
❌ Using JavaScript setInterval for animation — never. Use GSAP or requestAnimationFrame.
❌ Skipping Lenis when using GSAP ScrollTrigger on storytelling sites — causes jitter
❌ SplitType without revert() on unmount — leaves split DOM nodes in place
```

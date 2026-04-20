---
name: three-d
description: Expert in building 3D web experiences using React Three Fiber and Drei — 3D hero backgrounds, particle systems, scroll-synced 3D, Spline embeds, and WebGL performance optimization. For product configurators, 3D portfolios, immersive storytelling, and depth-driven landing pages.
allowed-tools:
  - "Read"
  - "Write"
  - "Bash"
  - "mcp_context7_resolve-library-id"
  - "mcp_context7_query-docs"
---

# 3D Web Experience Skill

You are a **3D Web Experience Architect** — you bring the third dimension to the web. You know when 3D creates genuine wonder and when it just burns battery. You build with React Three Fiber and Drei, you optimize models for the web, and you always provide graceful fallbacks for low-end devices.

**Core principle: 3D should serve a purpose. If a static image does the job, use a static image.**

---

## Activation Table

| User says | What to do |
|---|---|
| "3D hero" / "3D background" | Part 2A — Canvas setup + basic scene |
| "Spline" / "embed Spline" | Part 1C — Spline quick-start |
| "load a 3D model" / "GLB" / "GLTF" | Part 2B — Model pipeline |
| "particles" / "particle system" | Part 2C — Particle clouds |
| "3D + scroll" / "rotate on scroll" | Part 3 — Scroll-synced 3D |
| "3D performance" / "WebGL optimize" | Part 4 — Optimization rules |

---

## Part 1 — Stack Selection

| Tool | Best for | Learning curve | Control |
|---|---|---|---|
| **Spline** | Quick 3D elements, no code | Low | Medium |
| **React Three Fiber** | React apps, interactive scenes | Medium | High |
| **Three.js vanilla** | Non-React, max control | High | Maximum |

**Decision rule:**
- Need a 3D element fast, no complex interaction → Spline
- React app, interactive or scroll-driven → React Three Fiber + Drei
- Pure performance / game-like → Three.js vanilla

### 1A. Installation

```bash
npm install three @react-three/fiber @react-three/drei
npm install -D @types/three
```

### 1B. React Three Fiber — Basic Scene

```tsx
// src/components/Scene3D.tsx
// ✅ VERIFIED — docs.pmnd.rs/react-three-fiber
import { Canvas } from "@react-three/fiber"
import { OrbitControls, Environment } from "@react-three/drei"
import { Suspense } from "react"

function Box() {
  return (
    <mesh>
      <boxGeometry args={[1, 1, 1]} />
      <meshStandardMaterial color="#4f46e5" metalness={0.5} roughness={0.2} />
    </mesh>
  )
}

export function Scene3D() {
  return (
    <Canvas
      camera={{ position: [0, 0, 5], fov: 45 }}
      gl={{ antialias: true, alpha: true }}   // alpha: true = transparent background
      style={{ background: "transparent" }}
    >
      {/* Lighting */}
      <ambientLight intensity={0.5} />
      <directionalLight position={[10, 10, 5]} intensity={1} />

      {/* Environment map for reflections */}
      <Environment preset="city" />

      {/* Scene content */}
      <Suspense fallback={null}>
        <Box />
      </Suspense>

      {/* Controls — remove for production hero (no interaction) */}
      <OrbitControls enableZoom={false} />
    </Canvas>
  )
}
```

### 1C. Spline (Fastest Start — No 3D Expertise Required)

```bash
npm install @splinetool/react-spline
```

```tsx
// src/components/SplineHero.tsx
import Spline from "@splinetool/react-spline"
import { Suspense } from "react"

export function SplineHero() {
  return (
    <div className="relative w-full h-screen">
      <Suspense fallback={<div className="animate-pulse bg-neutral-900 h-full" />}>
        <Spline
          scene="https://prod.spline.design/YOUR_SCENE_ID/scene.splinecode"
          className="w-full h-full"
        />
      </Suspense>
    </div>
  )
}
```

> Design your scene at [spline.design](https://spline.design), then copy the embed URL from the Export → Viewer panel.

---

## Part 2 — Core Patterns

### 2A. 3D Hero Background

Full-viewport 3D scene sitting behind page content:

```tsx
// src/components/HeroScene.tsx
import { Canvas, useFrame } from "@react-three/fiber"
import { Float, Sphere, MeshDistortMaterial } from "@react-three/drei"
import { useRef } from "react"
import type { Mesh } from "three"

// Animated distorted sphere — the "liquid blob" hero background
function LiquidSphere() {
  const ref = useRef<Mesh>(null)

  useFrame((state) => {
    if (!ref.current) return
    // Gentle idle rotation
    ref.current.rotation.x = Math.sin(state.clock.elapsedTime * 0.3) * 0.2
    ref.current.rotation.y += 0.005
  })

  return (
    <Float speed={2} rotationIntensity={0.5} floatIntensity={0.5}>
      <Sphere ref={ref} args={[1.5, 64, 64]}>
        <MeshDistortMaterial
          color="#6366f1"
          roughness={0.1}
          metalness={0.8}
          distort={0.4}   // 0 = smooth sphere, 1 = heavily distorted
          speed={2}
        />
      </Sphere>
    </Float>
  )
}

export function HeroScene() {
  return (
    // z-index -1 puts 3D behind page content
    <div className="absolute inset-0 -z-10">
      <Canvas
        camera={{ position: [0, 0, 4], fov: 50 }}
        gl={{ alpha: true, antialias: true }}
        dpr={[1, 2]}  // Max 2x pixel ratio — prevents GPU overload on Retina
      >
        <ambientLight intensity={0.3} />
        <pointLight position={[5, 5, 5]} intensity={2} />
        <LiquidSphere />
      </Canvas>
    </div>
  )
}
```

### 2B. Loading 3D Models

**Model optimization pipeline (before import):**

```bash
# Install gltf-transform CLI
npm install -g @gltf-transform/cli

# Compress GLB: Draco compression + WebP textures
gltf-transform optimize input.glb output.glb \
  --compress draco \
  --texture-compress webp

# Verify output file size — target < 5MB, ideal < 2MB
ls -lh output.glb
```

**Web-ready model rules:**
- Poly count: < 100K triangles for mobile, < 500K for desktop-only
- Textures: 1024×1024 max for most models
- Format: GLB (binary GLTF) — smallest, single file

```tsx
// src/components/ProductModel.tsx
// ✅ VERIFIED — @react-three/drei useGLTF
import { useGLTF, useProgress, Html, ContactShadows } from "@react-three/drei"
import { Canvas, useFrame } from "@react-three/fiber"
import { useRef, Suspense } from "react"
import type { Group } from "three"

function LoadingBar() {
  const { progress } = useProgress()
  return (
    <Html center>
      <div className="text-white text-sm">{progress.toFixed(0)}% loaded</div>
    </Html>
  )
}

function Model({ url }: { url: string }) {
  const { scene } = useGLTF(url)
  const ref = useRef<Group>(null)

  useFrame((state) => {
    if (!ref.current) return
    // Gentle idle rotation
    ref.current.rotation.y = Math.sin(state.clock.elapsedTime * 0.5) * 0.3
  })

  return (
    <group ref={ref}>
      <primitive object={scene} />
      <ContactShadows
        position={[0, -1.5, 0]}
        opacity={0.4}
        scale={3}
        blur={2}
      />
    </group>
  )
}

export function ProductViewer({ modelUrl }: { modelUrl: string }) {
  return (
    <Canvas camera={{ position: [0, 0, 3], fov: 40 }} dpr={[1, 2]}>
      <ambientLight intensity={0.5} />
      <directionalLight position={[5, 5, 3]} intensity={1.5} />
      <Suspense fallback={<LoadingBar />}>
        <Model url={modelUrl} />
      </Suspense>
    </Canvas>
  )
}

// Preload model so it's ready before mount
useGLTF.preload("/model.glb")
```

### 2C. Particle System

```tsx
// src/components/ParticleField.tsx
import { useRef, useMemo } from "react"
import { Canvas, useFrame } from "@react-three/fiber"
import * as THREE from "three"

function Particles({ count = 3000 }: { count?: number }) {
  const ref = useRef<THREE.Points>(null)

  // Generate random positions ONCE using useMemo — not on every render
  const positions = useMemo(() => {
    const arr = new Float32Array(count * 3)
    for (let i = 0; i < count; i++) {
      arr[i * 3]     = (Math.random() - 0.5) * 10   // x
      arr[i * 3 + 1] = (Math.random() - 0.5) * 10   // y
      arr[i * 3 + 2] = (Math.random() - 0.5) * 10   // z
    }
    return arr
  }, [count])

  useFrame((state) => {
    if (!ref.current) return
    // Slow rotation of entire field
    ref.current.rotation.y = state.clock.elapsedTime * 0.05
    ref.current.rotation.x = state.clock.elapsedTime * 0.02
  })

  return (
    <points ref={ref}>
      <bufferGeometry>
        <bufferAttribute
          attach="attributes-position"
          args={[positions, 3]}
        />
      </bufferGeometry>
      <pointsMaterial
        size={0.02}
        color="#a5b4fc"
        transparent
        opacity={0.6}
        sizeAttenuation  // Points shrink with distance (3D depth)
      />
    </points>
  )
}

export function ParticleField() {
  return (
    <div className="absolute inset-0 -z-10">
      <Canvas camera={{ position: [0, 0, 5], fov: 60 }}>
        <Particles />
      </Canvas>
    </div>
  )
}
```

---

## Part 3 — Scroll-Synced 3D

### 3A. Scroll Controls (Drei built-in)

```tsx
// ✅ VERIFIED — @react-three/drei ScrollControls
import { ScrollControls, useScroll } from "@react-three/drei"
import { useFrame } from "@react-three/fiber"
import { useRef } from "react"
import type { Group } from "three"

function ScrollRotatingModel() {
  const scroll = useScroll()
  const ref = useRef<Group>(null)

  useFrame(() => {
    if (!ref.current) return
    // Full 360° rotation over 1 page of scroll
    ref.current.rotation.y = scroll.offset * Math.PI * 2
    // Camera-like z-movement
    ref.current.position.z = scroll.offset * -2
  })

  return (
    <group ref={ref}>
      {/* Your model here */}
    </group>
  )
}

export function ScrollScene() {
  return (
    <Canvas>
      <ScrollControls pages={3} damping={0.1}>
        <ScrollRotatingModel />
      </ScrollControls>
    </Canvas>
  )
}
```

### 3B. GSAP + Three.js Camera (Advanced)

```tsx
// Move the camera through the 3D scene on scroll
import { useRef } from "react"
import { Canvas, useThree } from "@react-three/fiber"
import gsap from "gsap"
import { ScrollTrigger } from "gsap/ScrollTrigger"
import { useGSAP } from "@gsap/react"

function CameraPath() {
  const { camera } = useThree()

  useGSAP(() => {
    gsap.to(camera.position, {
      scrollTrigger: {
        trigger: "body",
        start: "top top",
        end: "bottom bottom",
        scrub: 2,
      },
      z: 2,       // Camera moves forward
      y: -1,      // Camera tilts down
      ease: "none",
    })
  })

  return null
}
```

---

## Part 4 — Performance Rules

```
✅ dpr={[1, 2]} — cap pixel ratio at 2, prevents GPU overload on Retina
✅ alpha: true on gl — transparent canvas = composite over page content cheaply
✅ Compress GLBs with gltf-transform before loading
✅ useGLTF.preload() — start loading model before component mounts
✅ Suspense fallback — always show loading state (blank = broken to users)
✅ Dispose geometries/textures on unmount — prevent WebGL memory leaks
✅ Reduce particle count on mobile (use matchMedia to detect)
✅ frameloop="demand" on Canvas when scene only updates on interaction (not animation)

❌ Never create geometries/materials inside render function — triggers GC every frame
❌ Never use useEffect for Three.js side effects — use useFrame
❌ Never put 3D on every page — one impactful scene > five mediocre ones
❌ Never assume mobile can handle desktop-quality 3D — always test on real phone
❌ Never skip the Suspense wrapper — 3D assets are large and take time
```

**Mobile fallback pattern:**

```tsx
// Render 3D only on capable devices
function Scene3DWithFallback() {
  const isMobile = window.matchMedia("(max-width: 768px)").matches
  const prefersReducedMotion = window.matchMedia("(prefers-reduced-motion: reduce)").matches

  if (isMobile || prefersReducedMotion) {
    return <img src="/hero-static.webp" alt="Hero" className="w-full h-full object-cover" />
  }

  return <HeroScene />
}
```

---

## Part 5 — Anti-Patterns

```
❌ 3D for 3D's sake — rotating cube behind hero text adds nothing
   Ask: does this 3D element help the user understand the product?

❌ Desktop-only 3D without fallback — most traffic is mobile
   Fix: static image fallback, detect via matchMedia

❌ No loading state — 3D assets take 2-10s to load, blank screen = user leaves
   Fix: always wrap in Suspense with a visible fallback

❌ Uncompressed GLB files — 30MB model in a landing page
   Fix: gltf-transform optimize → compress → target < 5MB

❌ Scroll-hijacked 3D experience — forcing camera moves user didn't initiate
   Fix: scrub scroll to 3D animations, never override native scroll
```

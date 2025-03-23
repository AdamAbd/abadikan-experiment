<template>
  <div class="scroll-container" ref="scrollContainer">
    <!-- Canvas for image sequence -->
    <div class="canvas-container" ref="canvasContainer">
      <canvas ref="canvas" :width="576" :height="1024"></canvas>
    </div>

    <!-- Text Overlays -->
    <div class="text-overlays">
      <div class="text-element" ref="text1">
        <h2>First Text Element</h2>
        <p>This text will fade in and out during scrolling</p>
      </div>

      <div class="text-element" ref="text2">
        <h2>Second Text Element</h2>
        <p>This appears later in the scroll sequence</p>
      </div>

      <div class="text-element" ref="text3">
        <h2>Final Text Element</h2>
        <p>The last overlay as we approach the end of the animation</p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import gsap from 'gsap'
import ScrollTrigger from 'gsap/ScrollTrigger'
import Lenis from 'lenis' // Updated import from the renamed package

// Register ScrollTrigger plugin
gsap.registerPlugin(ScrollTrigger)

// Refs
const scrollContainer = ref<HTMLElement | null>(null)
const canvasContainer = ref<HTMLElement | null>(null)
const canvas = ref<HTMLCanvasElement | null>(null)
const text1 = ref<HTMLElement | null>(null)
const text2 = ref<HTMLElement | null>(null)
const text3 = ref<HTMLElement | null>(null)

// Variables for animation
const totalFrames = 123
const images: HTMLImageElement[] = []
let ctx: CanvasRenderingContext2D | null = null
let scrollTriggerInstance: ScrollTrigger | null = null
let lenis: Lenis | null = null

// Function to preload all images
const preloadImages = async (): Promise<void> => {
  const loadImage = (index: number): Promise<HTMLImageElement> => {
    return new Promise((resolve) => {
      const img = new Image()
      const formattedIndex = index.toString().padStart(3, '0')
      img.src = `/assets/phone/ezgif-frame-${formattedIndex}.jpg`
      img.onload = () => resolve(img)
      img.onerror = () => {
        console.error(`Failed to load image ${index}`)
        resolve(img)
      }
    })
  }

  const promises: Promise<HTMLImageElement>[] = []
  for (let i = 1; i <= totalFrames; i++) {
    promises.push(loadImage(i))
  }

  // Wait for all images to load
  images.push(...(await Promise.all(promises)))
  console.log(`Loaded ${images.length} frames`)

  // Draw first frame once loaded
  drawFrame(0)
}

// Function to draw a specific frame on the canvas
const drawFrame = (frameIndex: number): void => {
  if (!canvas.value || !ctx || frameIndex >= images.length) return

  ctx.clearRect(0, 0, canvas.value.width, canvas.value.height)
  ctx.drawImage(images[frameIndex], 0, 0, canvas.value.width, canvas.value.height)
}

// Setup Lenis smooth scrolling
const setupLenis = (): void => {
  lenis = new Lenis({
    duration: 1.2,
    easing: (t) => Math.min(1, 1.001 - Math.pow(2, -10 * t)), // https://www.desmos.com/calculator/brs54l4xou
    // direction: 'vertical', // vertical, horizontal
    // gestureDirection: 'vertical',
    // smooth: true,
    // mouseMultiplier: 1,
    // smoothTouch: false,
    touchMultiplier: 2,
    infinite: false,
  })

  // GSAP ticker integration for Lenis
  const raf = (time: number) => {
    lenis?.raf(time)
    requestAnimationFrame(raf)
  }

  // Start the animation loop
  requestAnimationFrame(raf)

  // Connect to ScrollTrigger for proper synchronization
  ScrollTrigger.normalizeScroll(true)
  ScrollTrigger.config({ ignoreMobileResize: true })

  // Update ScrollTrigger when Lenis scrolls
  lenis.on('scroll', ScrollTrigger.update)
}

// Setup the scroll animation
const setupScrollAnimation = (): void => {
  if (!canvas.value || !scrollContainer.value || !canvasContainer.value) return

  // Calculate the total scroll height
  const scrollHeight = totalFrames * 30 // 30px per frame for smooth scrolling
  scrollContainer.value.style.height = `${scrollHeight}px`

  // Create ScrollTrigger
  scrollTriggerInstance = ScrollTrigger.create({
    trigger: scrollContainer.value,
    start: 'top top',
    end: 'bottom bottom',
    scrub: 0.5, // Smoother scrubbing effect to work with Lenis
    pin: canvasContainer.value,
    onUpdate: (self) => {
      // Calculate current frame based on scroll progress
      const frameIndex = Math.min(Math.floor(self.progress * totalFrames), totalFrames - 1)
      drawFrame(frameIndex)
    },
  })

  // Text animations
  const textTimeline = gsap.timeline({
    scrollTrigger: {
      trigger: scrollContainer.value,
      start: 'top top',
      end: 'bottom bottom',
      scrub: 0.5,
    },
  })

  if (text1.value && text2.value && text3.value) {
    // Initial state
    gsap.set([text1.value, text2.value, text3.value], {
      opacity: 0,
      y: 50,
    })

    // Text 1 animation (appears at the beginning)
    textTimeline.to(
      text1.value,
      {
        opacity: 1,
        y: 0,
        duration: 0.2,
      },
      0,
    )

    // Text 1 fade out
    textTimeline.to(
      text1.value,
      {
        opacity: 0,
        y: -50,
        duration: 0.2,
      },
      0.3,
    )

    // Text 2 animation (appears in the middle)
    textTimeline.to(
      text2.value,
      {
        opacity: 1,
        y: 0,
        duration: 0.2,
      },
      0.4,
    )

    // Text 2 fade out
    textTimeline.to(
      text2.value,
      {
        opacity: 0,
        y: -50,
        duration: 0.2,
      },
      0.7,
    )

    // Text 3 animation (appears at the end)
    textTimeline.to(
      text3.value,
      {
        opacity: 1,
        y: 0,
        duration: 0.2,
      },
      0.8,
    )
  }
}

// Lifecycle hooks
onMounted(async () => {
  if (canvas.value) {
    ctx = canvas.value.getContext('2d')

    // Setup Lenis smooth scrolling first
    setupLenis()

    // Then preload images and setup animation
    await preloadImages()
    setupScrollAnimation()

    // Resize handler
    const handleResize = () => {
      if (scrollTriggerInstance) {
        scrollTriggerInstance.refresh()
      }
    }

    window.addEventListener('resize', handleResize)
  }
})

onUnmounted(() => {
  // Clean up
  if (scrollTriggerInstance) {
    scrollTriggerInstance.kill()
  }

  // Clean up Lenis
  if (lenis) {
    lenis.destroy()
  }

  window.removeEventListener('resize', () => {})
  ScrollTrigger.getAll().forEach((trigger) => trigger.kill())
})
</script>

<style scoped>
.scroll-container {
  width: 100%;
  position: relative;
}

.canvas-container {
  position: sticky;
  top: 0;
  height: 100vh;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1;
}

canvas {
  max-height: 100vh;
  max-width: 100%;
  object-fit: contain;
}

.text-overlays {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 2;
}

.text-element {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: white;
  text-align: center;
  padding: 2rem;
  background-color: rgba(0, 0, 0, 0.5);
  border-radius: 1rem;
  max-width: 80%;
}

/* Add smooth transitions for all animations */
.text-element {
  transition: transform 0.5s cubic-bezier(0.33, 1, 0.68, 1);
}
</style>

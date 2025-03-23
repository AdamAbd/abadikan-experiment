<template>
  <div class="container">
    <div class="spacer"></div>

    <section class="animation-section" ref="animationSection">
      <div class="canvas-container" ref="canvasContainer">
        <canvas ref="canvas" :width="canvasWidth" :height="canvasHeight"></canvas>

        <div class="text-overlay">
          <div class="text-element" ref="text1">Discover the journey</div>
          <div class="text-element" ref="text2">Explore new perspectives</div>
          <div class="text-element" ref="text3">Transform your understanding</div>
        </div>
      </div>
    </section>

    <div class="spacer"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import gsap from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'

// Register ScrollTrigger plugin
gsap.registerPlugin(ScrollTrigger)

// Canvas refs and dimensions
const canvas = ref<HTMLCanvasElement | null>(null)
const canvasContainer = ref<HTMLElement | null>(null)
const animationSection = ref<HTMLElement | null>(null)
const canvasWidth = ref(1920)
const canvasHeight = ref(1080)

// Text elements refs
const text1 = ref<HTMLElement | null>(null)
const text2 = ref<HTMLElement | null>(null)
const text3 = ref<HTMLElement | null>(null)

// Image sequence vars
const totalFrames = 100
const images: HTMLImageElement[] = []
let currentFrame = 0
let ctx: CanvasRenderingContext2D | null = null
let scrollTriggerInstance: gsap.core.Timeline | null = null

// Preload all images from assets
const preloadImages = async () => {
  const loadImage = (index: number): Promise<HTMLImageElement> => {
    return new Promise((resolve) => {
      const img = new Image()
      // Format the index with leading zeros (001, 002, etc.)
      const formattedIndex = String(index).padStart(3, '0')
      //   /Users/adamabdurrahman/Code/Vue/abadikan-experiment/public/assets/ezgif-split/ezgif-frame-001.jpg
      img.src = `/assets/ezgif-split/ezgif-frame-${formattedIndex}.jpg` // Adjust path as needed
      img.onload = () => resolve(img)
      return img
    })
  }

  const loadPromises: Promise<HTMLImageElement>[] = []
  for (let i = 1; i <= totalFrames; i++) {
    loadPromises.push(loadImage(i))
  }

  images.push(...(await Promise.all(loadPromises)))
  console.log(`Loaded ${images.length} images`)

  // Draw first frame once all images are loaded
  if (canvas.value && images.length > 0) {
    drawFrame(0)
  }
}

// Function to draw a specific frame on the canvas
const drawFrame = (frameIndex: number) => {
  if (!canvas.value || images.length <= frameIndex) return

  if (!ctx) {
    ctx = canvas.value.getContext('2d')
    if (!ctx) return
  }

  const img = images[frameIndex]

  // Clear the canvas
  ctx.clearRect(0, 0, canvas.value.width, canvas.value.height)

  // Draw the image centered and scaled to fill the canvas
  const scale = Math.max(canvas.value.width / img.width, canvas.value.height / img.height)

  const scaledWidth = img.width * scale
  const scaledHeight = img.height * scale

  const x = (canvas.value.width - scaledWidth) / 2
  const y = (canvas.value.height - scaledHeight) / 2

  ctx.drawImage(img, x, y, scaledWidth, scaledHeight)
}

// Set up the scroll trigger animation
const setupScrollAnimation = () => {
  if (!animationSection.value || !canvas.value) return

  // Create timeline for the animation
  scrollTriggerInstance = gsap.timeline({
    scrollTrigger: {
      trigger: animationSection.value,
      pin: true,
      start: 'top top',
      end: '+=5000', // Adjust the scrolling length as needed
      scrub: 1, // Smooth scrubbing effect
      //   markers: process.env.NODE_ENV === 'development', // Show markers in dev mode
      onUpdate: (self) => {
        // Calculate the current frame based on scroll progress
        const frameIndex = Math.min(Math.floor(self.progress * (totalFrames - 1)), totalFrames - 1)

        if (frameIndex !== currentFrame) {
          currentFrame = frameIndex
          drawFrame(currentFrame)
        }
      },
    },
  })

  // Add text animations
  if (text1.value && text2.value && text3.value) {
    // Initial state: all texts invisible
    // gsap.set([text1.value, text2.value, text3.value], {
    //   opacity: 0,
    //   y: 50,
    // })
    // // Text 1 animation
    // scrollTriggerInstance
    //   .to(text1.value, {
    //     opacity: 1,
    //     y: 0,
    //     duration: 0.2,
    //     scrollTrigger: {
    //       trigger: animationSection.value,
    //       start: 'top+=10% top',
    //       end: 'top+=30% top',
    //       scrub: 1,
    //       markers: process.env.NODE_ENV === 'development', // Show markers in dev mode
    //       containerAnimation: scrollTriggerInstance,
    //     },
    //   })
    //   .to(text1.value, {
    //     opacity: 0,
    //     y: -50,
    //     duration: 0.2,
    //     scrollTrigger: {
    //       trigger: animationSection.value,
    //       start: 'top+=30% top',
    //       end: 'top+=40% top',
    //       scrub: 1,
    //       containerAnimation: scrollTriggerInstance,
    //     },
    //   })
    // // Text 2 animation
    // scrollTriggerInstance
    //   .to(text2.value, {
    //     opacity: 1,
    //     y: 0,
    //     duration: 0.2,
    //     scrollTrigger: {
    //       trigger: animationSection.value,
    //       start: 'top+=40% top',
    //       end: 'top+=50% top',
    //       scrub: 1,
    //       containerAnimation: scrollTriggerInstance,
    //     },
    //   })
    //   .to(text2.value, {
    //     opacity: 0,
    //     y: -50,
    //     duration: 0.2,
    //     scrollTrigger: {
    //       trigger: animationSection.value,
    //       start: 'top+=60% top',
    //       end: 'top+=70% top',
    //       scrub: 1,
    //       containerAnimation: scrollTriggerInstance,
    //     },
    //   })
    // // Text 3 animation
    // scrollTriggerInstance
    //   .to(text3.value, {
    //     opacity: 1,
    //     y: 0,
    //     duration: 0.2,
    //     scrollTrigger: {
    //       trigger: animationSection.value,
    //       start: 'top+=70% top',
    //       end: 'top+=80% top',
    //       scrub: 1,
    //       containerAnimation: scrollTriggerInstance,
    //     },
    //   })
    //   .to(text3.value, {
    //     opacity: 0,
    //     y: -50,
    //     duration: 0.2,
    //     scrollTrigger: {
    //       trigger: animationSection.value,
    //       start: 'top+=90% top',
    //       end: 'top+=100% top',
    //       scrub: 1,
    //       containerAnimation: scrollTriggerInstance,
    //     },
    //   })
  }
}

// Handle window resize
const handleResize = () => {
  if (!canvasContainer.value || !canvas.value) return

  // Update canvas dimensions based on container size
  const containerWidth = canvasContainer.value.clientWidth
  const containerHeight = canvasContainer.value.clientHeight

  // Maintain aspect ratio
  const aspectRatio = 16 / 9 // Assuming 16:9 aspect ratio

  if (containerWidth / containerHeight > aspectRatio) {
    // Container is wider
    canvasHeight.value = containerHeight
    canvasWidth.value = containerHeight * aspectRatio
  } else {
    // Container is taller
    canvasWidth.value = containerWidth
    canvasHeight.value = containerWidth / aspectRatio
  }

  // Redraw current frame with new dimensions
  if (images.length > 0) {
    drawFrame(currentFrame)
  }
}

// Cleanup function
const cleanup = () => {
  if (scrollTriggerInstance) {
    scrollTriggerInstance.kill()
  }

  // Kill all other ScrollTrigger instances related to our elements
  ScrollTrigger.getAll().forEach((trigger) => {
    if (trigger.vars.trigger === animationSection.value) {
      trigger.kill()
    }
  })
}

onMounted(async () => {
  window.addEventListener('resize', handleResize)

  // Initial size
  if (canvasContainer.value) {
    const containerWidth = canvasContainer.value.clientWidth
    const containerHeight = canvasContainer.value.clientHeight
    canvasWidth.value = containerWidth
    canvasHeight.value = containerHeight
  }

  // Preload images
  await preloadImages()

  // Setup animation after images are loaded
  setupScrollAnimation()
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
  cleanup()
})
</script>

<style scoped>
.container {
  width: 100%;
}

.spacer {
  height: 100vh;
}

.animation-section {
  height: 100vh;
  width: 100%;
  position: relative;
  overflow: hidden;
}

.canvas-container {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

canvas {
  display: block;
  max-width: 100%;
  max-height: 100%;
}

.text-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
}

.text-element {
  position: absolute;
  font-size: 3rem;
  font-weight: bold;
  color: white;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
  opacity: 0;
}

.text-element:nth-child(1) {
  top: 20%;
  left: 10%;
}

.text-element:nth-child(2) {
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.text-element:nth-child(3) {
  bottom: 20%;
  right: 10%;
}
</style>

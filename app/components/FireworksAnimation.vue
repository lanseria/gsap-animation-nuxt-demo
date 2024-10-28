<script setup lang="ts">
import gsap from 'gsap'

const fireworksContainer = useTemplateRef('fireworksContainer')
let animationInterval: number | null = null

function createParticle() {
  const particle = document.createElement('div')
  particle.className = 'absolute w-1 h-1 bg-white rounded-full opacity-0'
  return particle
}

function launchFirework() {
  if (!fireworksContainer.value)
    return

  const startX = Math.random() * window.innerWidth
  const endY = window.innerHeight * (0.3 + Math.random() * 0.2)

  const firework = createParticle()
  fireworksContainer.value.appendChild(firework)

  // 设置初始位置
  gsap.set(firework, {
    x: startX,
    y: window.innerHeight,
    scale: 2,
  })

  // 上升动画
  gsap.to(firework, {
    y: endY,
    duration: 1,
    ease: 'power1.out',
    opacity: 1,
    onComplete: () => {
      // 爆炸效果
      const particles = []
      const particleCount = 20

      for (let i = 0; i < particleCount; i++) {
        const particle = createParticle()
        fireworksContainer.value?.appendChild(particle)
        particles.push(particle)

        const angle = (Math.PI * 2 * i) / particleCount
        const radius = 50 + Math.random() * 50

        gsap.set(particle, {
          x: startX,
          y: endY,
          opacity: 1,
        })

        gsap.to(particle, {
          x: startX + Math.cos(angle) * radius,
          y: endY + Math.sin(angle) * radius,
          opacity: 0,
          duration: 0.8,
          ease: 'power2.out',
          onComplete: () => {
            particle.remove()
          },
        })
      }

      firework.remove()
    },
  })
}

onMounted(() => {
  animationInterval = window.setInterval(() => {
    launchFirework()
  }, 2000)
})

onBeforeUnmount(() => {
  if (animationInterval) {
    clearInterval(animationInterval)
  }
})
</script>

<template>
  <div
    ref="fireworksContainer"
    class="pointer-events-none absolute inset-0 overflow-hidden"
  />
</template>

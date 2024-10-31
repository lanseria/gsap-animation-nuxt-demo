<script setup lang="ts">
import gsap from 'gsap'

const fireworksContainer = useTemplateRef('fireworksContainer')
let animationInterval: number | null = null
let isLaunching = false

// 彩色烟花的颜色数组
const colors = [
  '#FF69B4', // 粉红
  '#4169E1', // 皇家蓝
  '#FFD700', // 金色
  '#FF4500', // 橙红
  '#7CFC00', // 亮绿
  '#FF1493', // 深粉红
  '#00FFFF', // 青色
  '#9370DB', // 紫色
]

function createParticle() {
  const particle = document.createElement('div')
  particle.className = 'absolute w-1 h-1 rounded-full opacity-0'
  return particle
}

function getRandomColor() {
  return colors[Math.floor(Math.random() * colors.length)]
}

function createFireworkGroup() {
  if (!fireworksContainer.value || isLaunching)
    return

  isLaunching = true

  // 创建一组烟花（3-5个）
  const groupCount = 20 + Math.floor(Math.random() * 3)
  const baseX = Math.random() * (window.innerWidth * 0.8) + (window.innerWidth * 0.1)

  // 依次发射每个烟花
  for (let i = 0; i < groupCount; i++) {
    setTimeout(() => {
      launchFirework(baseX + (Math.random() - 0.5) * 100)
    }, i * 300) // 每300ms发射一个
  }

  // 发射完毕后重置状态
  setTimeout(() => {
    isLaunching = false
  }, groupCount * 300 + 1000)
}

function launchFirework(startX: number) {
  if (!fireworksContainer.value)
    return

  const endY = window.innerHeight * (0.2 + Math.random() * 0.3)
  const firework = createParticle()
  const mainColor = getRandomColor()

  firework.style.backgroundColor = mainColor!
  firework.style.boxShadow = `0 0 6px ${mainColor}`
  fireworksContainer.value.appendChild(firework)

  // 设置初始位置
  gsap.set(firework, {
    x: startX,
    y: window.innerHeight,
    scale: 2,
  })

  // 上升动画，添加拖尾效果
  gsap.to(firework, {
    y: endY,
    duration: 1.2,
    ease: 'power2.out',
    opacity: 1,
    onUpdate: () => {
      if (Math.random() > 0.8) {
        const trail = createParticle()
        trail.style.backgroundColor = mainColor!
        trail.style.opacity = '0.3'
        fireworksContainer.value?.appendChild(trail)

        gsap.set(trail, {
          x: gsap.getProperty(firework, 'x'),
          y: gsap.getProperty(firework, 'y'),
          scale: 1,
        })

        gsap.to(trail, {
          opacity: 0,
          duration: 0.4,
          onComplete: () => trail.remove(),
        })
      }
    },
    onComplete: () => {
      // 爆炸效果
      const particles = []
      const particleCount = 30 // 增加粒子数量

      for (let i = 0; i < particleCount; i++) {
        const particle = createParticle()
        const particleColor = mainColor
        particle.style.backgroundColor = particleColor!
        particle.style.boxShadow = `0 0 6px ${particleColor}`
        fireworksContainer.value?.appendChild(particle)
        particles.push(particle)

        const angle = (Math.PI * 2 * i) / particleCount
        const radius = 80 + Math.random() * 60 // 增加爆炸半径

        gsap.set(particle, {
          x: startX,
          y: endY,
          opacity: 1,
          scale: 1.5,
        })

        // 爆炸轨迹使用曲线运动
        gsap.to(particle, {
          x: startX + Math.cos(angle) * radius,
          y: endY + Math.sin(angle) * radius + (Math.random() * 40), // 添加重力效果
          opacity: 0,
          scale: 0.2,
          duration: 1.2,
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
    createFireworkGroup()
  }, 100) // 每4秒发射一组烟花
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
    class="pointer-events-none absolute inset-0 overflow-hidden bg-black bg-opacity-90"
  />
</template>

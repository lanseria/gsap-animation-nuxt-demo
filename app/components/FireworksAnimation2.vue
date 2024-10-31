<script setup lang="ts">
import gsap from 'gsap'

const fireworksContainer = useTemplateRef('fireworksContainer')
let animationInterval: number | null = null

// 主题色系列表
const colorSchemes = [
  ['#FF69B4', '#FF1493', '#FF0000'], // 粉红系
  ['#FF0000', '#FF4444', '#FF6B6B'], // 红色系
  ['#FF1493', '#FF69B4', '#FFB6C1'], // 深粉系
]

// 计算爱心轮廓上的点
function getHeartPoints(centerX: number, centerY: number, scale: number = 1, variance: number = 30): Array<{ x: number, y: number }> {
  const points = []
  for (let i = 0; i < 25; i++) {
    const t = (i / 25) * 2 * Math.PI
    const x = 16 * Math.sin(t) ** 3
    const y = 13 * Math.cos(t) - 5 * Math.cos(2 * t) - 2 * Math.cos(3 * t) - Math.cos(4 * t)
    // 添加随机偏移
    // const randomOffsetY = (Math.random() - 0.5) * variance
    points.push({
      x: centerX + x * scale * 8,
      y: centerY - y * scale * 8 + variance,
    })
  }
  return points
}

function createParticle() {
  const particle = document.createElement('div')
  particle.className = 'absolute w-1 h-1 rounded-full opacity-0'
  return particle
}

function getRandomColorScheme() {
  return colorSchemes[Math.floor(Math.random() * colorSchemes.length)]
}

function getRandomColorFromScheme(scheme: string[]) {
  return scheme[Math.floor(Math.random() * scheme.length)]
}

function launchFireworks() {
  if (!fireworksContainer.value)
    return

  const centerX = window.innerWidth / 2
  const centerY = window.innerHeight * 0.5
  const centerX_R = centerX + (Math.random() - 0.5) * centerX
  const centerY_R = centerY + (Math.random() - 0.5) * centerY
  const scale_R = 1 + (Math.random() - 0.5) * 1
  const heartPoints = getHeartPoints(centerX_R, centerY_R, scale_R, 60) // 增加垂直方向的随机性
  const colorScheme = getRandomColorScheme()
  const explosionDelay = 1.5 // 爆炸延迟时间

  // 记录所有烟花，用于同时爆炸
  const allFireworks: Array<{ element: HTMLElement, x: number, y: number }> = []

  // 依次发射烟花
  heartPoints.forEach((point, index) => {
    setTimeout(() => {
      const firework = createParticle()
      const mainColor = getRandomColorFromScheme(colorScheme!)

      firework.style.backgroundColor = mainColor!
      firework.style.boxShadow = `0 0 6px ${mainColor}`
      fireworksContainer.value?.appendChild(firework)

      // 设置初始位置
      const startX = centerX + (Math.random() - 0.5) * 300
      gsap.set(firework, {
        x: startX,
        y: window.innerHeight,
        scale: 2,
      })

      // 上升动画
      gsap.to(firework, {
        x: point.x,
        y: point.y,
        duration: 1,
        ease: 'power2.out',
        opacity: 1,
        onUpdate() {
          // 添加上升轨迹
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
      })

      // 记录烟花信息，用于后续同时爆炸
      allFireworks.push({
        element: firework,
        x: point.x,
        y: point.y,
      })

      // 在最后一个烟花上升后触发同时爆炸
      if (index === heartPoints.length - 1) {
        setTimeout(() => {
          // 同时触发所有烟花的爆炸效果
          allFireworks.forEach((fw) => {
            createExplosion(fw.element, fw.x, fw.y, colorScheme!)
          })
        }, explosionDelay * 1000)
      }
    }, index * 60) // 每个烟花发射间隔60ms
  })
}

function createExplosion(firework: HTMLElement, x: number, y: number, colorScheme: string[]) {
  const particles = []
  const particleCount = 20

  for (let i = 0; i < particleCount; i++) {
    const particle = createParticle()
    const particleColor = getRandomColorFromScheme(colorScheme)
    particle.style.backgroundColor = particleColor!
    particle.style.boxShadow = `0 0 6px ${particleColor}`
    fireworksContainer.value?.appendChild(particle)
    particles.push(particle)

    const angle = (Math.PI * 2 * i) / particleCount
    const radius = 40 + Math.random() * 30

    gsap.set(particle, {
      x,
      y,
      opacity: 1,
      scale: 1.5,
    })

    gsap.to(particle, {
      x: x + Math.cos(angle) * radius,
      y: y + Math.sin(angle) * radius + (Math.random() * 20),
      opacity: 0,
      scale: 0.2,
      duration: 1,
      ease: 'power2.out',
      onComplete: () => {
        particle.remove()
      },
    })
  }

  firework.remove()
}

onMounted(() => {
  animationInterval = window.setInterval(() => {
    launchFireworks()
  }, 4000) // 每4秒发射一组
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

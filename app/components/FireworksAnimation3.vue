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

// 3D变换工具函数
function rotatePoint(x: number, y: number, rotateX: number, rotateY: number, rotateZ: number): { x: number, y: number, scale: number } {
  // 将角度转换为弧度
  const radX = (rotateX * Math.PI) / 180
  const radY = (rotateY * Math.PI) / 180
  const radZ = (rotateZ * Math.PI) / 180

  // 基础3D旋转变换
  const x1 = x
  const y1 = y
  const z1 = 0

  // 绕Z轴旋转
  const x2 = x1 * Math.cos(radZ) - y1 * Math.sin(radZ)
  const y2 = x1 * Math.sin(radZ) + y1 * Math.cos(radZ)
  const z2 = z1

  // 绕X轴旋转
  const y3 = y2 * Math.cos(radX) - z2 * Math.sin(radX)
  const z3 = y2 * Math.sin(radX) + z2 * Math.cos(radX)

  // 绕Y轴旋转
  const x4 = x2 * Math.cos(radY) + z3 * Math.sin(radY)
  const z4 = -x2 * Math.sin(radY) + z3 * Math.cos(radY)

  // 计算透视效果（基于z坐标的缩放）
  const perspective = 1000
  const scale = perspective / (perspective + z4)

  return {
    x: x4 * scale,
    y: y3 * scale,
    scale,
  }
}

// 计算爱心轮廓上的点
function getHeartPoints(
  centerX: number,
  centerY: number,
  scale: number = 1,
  variance: number = 30,
  rotateX: number = 0,
  rotateY: number = 0,
  rotateZ: number = 0,
): Array<{ x: number, y: number, scale: number }> {
  const points = []
  for (let i = 0; i < 25; i++) {
    const t = (i / 25) * 2 * Math.PI
    const x = 16 * Math.sin(t) ** 3
    const y = 13 * Math.cos(t) - 5 * Math.cos(2 * t) - 2 * Math.cos(3 * t) - Math.cos(4 * t)

    // 应用3D旋转变换
    const baseX = x * scale * 8
    const baseY = -y * scale * 8 + variance
    const transformed = rotatePoint(baseX, baseY, rotateX, rotateY, rotateZ)

    points.push({
      x: centerX + transformed.x,
      y: centerY + transformed.y,
      scale: transformed.scale,
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
  const centerX_R = centerX + (Math.random() - 0.5) * centerX * 0.5
  const centerY_R = centerY + (Math.random() - 0.5) * centerY * 0.3

  // 限制旋转角度以保持基本朝向屏幕
  const rotateX = (Math.random() - 0.5) * 30 // ±15度
  const rotateY = (Math.random() - 0.5) * 30 // ±15度
  const rotateZ = (Math.random() - 0.5) * 40 // ±20度

  const scale_R = 0.8 + Math.random() * 0.4 // 0.8-1.2的随机缩放
  const heartPoints = getHeartPoints(centerX_R, centerY_R, scale_R, 60, rotateX, rotateY, rotateZ)
  const colorScheme = getRandomColorScheme()
  const explosionDelay = 1.5

  const allFireworks: Array<{ element: HTMLElement, x: number, y: number, scale: number }> = []

  heartPoints.forEach((point, index) => {
    setTimeout(() => {
      const firework = createParticle()
      const mainColor = getRandomColorFromScheme(colorScheme!)

      firework.style.backgroundColor = mainColor!
      firework.style.boxShadow = `0 0 6px ${mainColor}`
      fireworksContainer.value?.appendChild(firework)

      const startX = centerX + (Math.random() - 0.5) * 300
      gsap.set(firework, {
        x: startX,
        y: window.innerHeight,
        scale: 2 * point.scale, // 根据3D位置调整大小
      })

      gsap.to(firework, {
        x: point.x,
        y: point.y,
        duration: 1,
        ease: 'power2.out',
        opacity: 1,
        onUpdate() {
          if (Math.random() > 0.8) {
            const trail = createParticle()
            trail.style.backgroundColor = mainColor!
            trail.style.opacity = '0.3'
            fireworksContainer.value?.appendChild(trail)

            gsap.set(trail, {
              x: gsap.getProperty(firework, 'x'),
              y: gsap.getProperty(firework, 'y'),
              scale: point.scale, // 使用3D缩放
            })

            gsap.to(trail, {
              opacity: 0,
              duration: 0.4,
              onComplete: () => trail.remove(),
            })
          }
        },
      })

      allFireworks.push({
        element: firework,
        x: point.x,
        y: point.y,
        scale: point.scale,
      })

      if (index === heartPoints.length - 1) {
        setTimeout(() => {
          allFireworks.forEach((fw) => {
            createExplosion(fw.element, fw.x, fw.y, colorScheme!, fw.scale)
          })
        }, explosionDelay * 1000)
      }
    }, index * 60)
  })
}

function createExplosion(firework: HTMLElement, x: number, y: number, colorScheme: string[], scale: number) {
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
    const radius = (40 + Math.random() * 30) * scale // 根据3D位置调整爆炸范围

    gsap.set(particle, {
      x,
      y,
      opacity: 1,
      scale: 1.5 * scale,
    })

    gsap.to(particle, {
      x: x + Math.cos(angle) * radius,
      y: y + Math.sin(angle) * radius + (Math.random() * 20 * scale),
      opacity: 0,
      scale: 0.2 * scale,
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
  }, 4000)
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
    style="perspective: 1000px;"
  />
</template>

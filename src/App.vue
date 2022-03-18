<script setup lang="ts">
const el = $ref<HTMLCanvasElement>()
const ctx = $computed(() => el.getContext('2d')!)

const WIDTH = 450
const HEIGHT = 600

interface Point {
  x: number
  y: number
}

interface Branch {
  start: Point
  length: number
  theta: number
}

function init() {
  ctx.strokeStyle = '#808080'

  const branch = {
    start: { x: WIDTH / 2, y: HEIGHT },
    length: 20,
    theta: -Math.PI / 2,
  }
  setup(branch)
}

function lineTo(p1: Point, p2: Point) {
  ctx.beginPath()
  ctx.moveTo(p1.x, p1.y)
  ctx.lineTo(p2.x, p2.y)
  ctx.stroke()
}

const paddingTask: Function[] = []

function setup(b: Branch, depth = 0) {
  const endPoint = getEndPoint(b)
  drawBranch(b)

  if (depth < 5 || Math.random() < 0.5) {
    paddingTask.push(() => setup({
      start: endPoint,
      length: b.length + (Math.random() * 10 - 5),
      theta: b.theta - 0.4 * Math.random(),
    }, depth + 1))
  }
  if (depth < 5 || Math.random() < 0.5) {
    paddingTask.push(() => setup({
      start: endPoint,
      length: b.length + (Math.random() * 10 - 5),
      theta: b.theta + 0.4 * Math.random(),
    }, depth + 1))
  }
}

function frame() {
  const tasks = [...paddingTask]
  paddingTask.length = 0
  tasks.forEach(fn => fn())
}

let framesCount = 0

function startFrame() {
  requestAnimationFrame(() => {
    framesCount += 1
    if (framesCount % 3 === 0)
      frame()
    startFrame()
  })
}
startFrame()
function getEndPoint(b: Branch) {
  return {
    x: b.start.x + b.length * Math.cos(b.theta),
    y: b.start.y + b.length * Math.sin(b.theta),
  }
}

function drawBranch(b: Branch) {
  lineTo(b.start, getEndPoint(b))
}

onMounted(() => {
  init()
})
</script>

<template>
  <div flex justify-center>
    <canvas ref="el" width="450" height="600" border-1 />
  </div>
</template>

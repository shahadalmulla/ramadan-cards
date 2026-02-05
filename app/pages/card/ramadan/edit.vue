<template>
  <div class="page">
    <div class="stage">
      <!-- البطاقة -->
      <div class="frame">
        <canvas ref="canvasRef" :width="W" :height="H" class="canvas"></canvas>

        <!-- بوكس تفاعلي فوق الكانفس -->
        <div
          class="boxOverlay"
          :class="{ active: isActive }"
          :style="boxStyle"
          @pointerdown="startDrag"
        >
          <!-- input للكتابة (يصير “غير مرئي” لأن النص ينرسم بالكانفس) -->
          <input
            v-model="name"
            class="nameInput"
            dir="rtl"
            maxlength="40"
            @focus="onFocus"
            @blur="onBlur"
          />

          <!-- مقبض التكبير -->
          <div class="resizeHandle" @pointerdown.stop="startResize" />
        </div>
      </div>

      <!-- الإعدادات -->
      <aside class="sidebar">
        <div class="panel">
          <div class="hint">اضغط على المربع الابيض ثم اسحب لتحريكه أو تكبيره من النقطة البنيه</div>

          <label class="field">
            <span class="label">الاسم</span>
            <input v-model="name" class="input" maxlength="40" @focus="onFocus" @blur="onBlur" />
          </label>

          <label class="field">
            <span class="label">الخط</span>
            <select v-model="fontFamily" class="select">
              <option value="Times New Roman">Times New Roman</option>
              <option value="Tahoma">Tahoma</option>
              <option value="Arial">Arial</option>
            </select>
          </label>

          <label class="field">
            <span class="label">حجم الخط ({{ fontSize }})</span>
            <input type="range" min="24" max="90" v-model="fontSize" class="range" />
          </label>

          <label class="field">
            <span class="label">اللون</span>
            <input type="color" v-model="textColor" class="color" />
          </label>

          <div class="btnRow">
            <button class="btn" type="button" @click="goBack">رجوع</button>
            <button class="btn primary" type="button" @click="downloadPNG">تحميل</button>
          </div>
        </div>
      </aside>
    </div>
  </div>
</template>

<script setup>
const router = useRouter()
const route = useRoute()

const W = 1080
const H = 1920

const canvasRef = ref(null)

/** قياسات الكانفس على الشاشة (DOM) — نعبّيها بعد mount فقط */
const canvasRect = ref({ width: 0, height: 0 })

/* النص */
const placeholder = "اكتب اسمك هنا"
const name = ref(placeholder)
const fontFamily = ref("Times New Roman")
const fontSize = ref(44)
const textColor = ref("#8a6f3d")
const isActive = ref(false)

/* بوكس افتراضي أصغر */
const box = reactive({
  x: 260,
  y: 1480,
  w: 560,
  h: 110,
})

/* خلفية */
const bg = computed(() => route.query.bg || "/templates/ramadan_1.png")

/* ثوابت للتحجيم */
const MIN_W = 220
const MIN_H = 80

/* سحب/تكبير */
let mode = null // "drag" | "resize" | null
let startX = 0
let startY = 0
let startW = 0
let startH = 0

function updateCanvasRect() {
  const canvas = canvasRef.value
  if (!canvas) return
  const rect = canvas.getBoundingClientRect()
  canvasRect.value = { width: rect.width, height: rect.height }
}

/** تحويل delta من شاشة -> كانفس */
function screenToCanvasDelta(dxScreen, dyScreen) {
  const w = canvasRect.value.width || 1
  const h = canvasRect.value.height || 1
  return {
    dx: dxScreen * (W / w),
    dy: dyScreen * (H / h),
  }
}

/** ستايل البوكس فوق الكانفس — يعتمد على canvasRect (بدون DOM access وقت SSR) */
const boxStyle = computed(() => {
  const w = canvasRect.value.width
  const h = canvasRect.value.height
  if (!w || !h) return { display: "none" } // قبل mount/قبل القياس

  return {
    left: `${(box.x / W) * w}px`,
    top: `${(box.y / H) * h}px`,
    width: `${(box.w / W) * w}px`,
    height: `${(box.h / H) * h}px`,
  }
})

function onFocus() {
  isActive.value = true
  if (name.value === placeholder) name.value = ""
}
function onBlur() {
  if (!name.value) name.value = placeholder
}

/* تحميل صورة */
function loadImage(src) {
  return new Promise((resolve, reject) => {
    const img = new Image()
    img.onload = () => resolve(img)
    img.onerror = reject
    img.src = src
  })
}

/* رسم RoundRect */
function roundRect(ctx, x, y, w, h, r) {
  const rr = Math.min(r, w / 2, h / 2)
  ctx.beginPath()
  ctx.moveTo(x + rr, y)
  ctx.arcTo(x + w, y, x + w, y + h, rr)
  ctx.arcTo(x + w, y + h, x, y + h, rr)
  ctx.arcTo(x, y + h, x, y, rr)
  ctx.arcTo(x, y, x + w, y, rr)
  ctx.closePath()
}

function trimText(ctx, text, maxWidth) {
  let t = String(text || "")
  while (t.length && ctx.measureText(t).width > maxWidth) t = t.slice(0, -1)
  return t
}

async function draw() {
  const canvas = canvasRef.value
  if (!canvas) return
  const ctx = canvas.getContext("2d")

  ctx.clearRect(0, 0, W, H)

  const img = await loadImage(bg.value)
  ctx.drawImage(img, 0, 0, W, H)

  // البوكس (قريب من الشفاف)
  ctx.fillStyle = "rgba(255,255,255,0.86)"
  roundRect(ctx, box.x, box.y, box.w, box.h, box.h / 2)
  ctx.fill()

  // stroke خفيف يعطي فخامة
  ctx.strokeStyle = "rgba(255,255,255,0.28)"
  ctx.lineWidth = 2
  ctx.stroke()

  // النص
  ctx.save()
  roundRect(ctx, box.x, box.y, box.w, box.h, box.h / 2)
  ctx.clip()

  ctx.font = `700 ${fontSize.value}px ${fontFamily.value}`
  ctx.fillStyle = textColor.value
  ctx.textAlign = "center"
  ctx.textBaseline = "middle"

  const textToDraw = name.value === placeholder ? "" : name.value
  const safe = trimText(ctx, textToDraw, box.w - 60)
  ctx.fillText(safe, box.x + box.w / 2, box.y + box.h / 2)

  ctx.restore()
}

/* Drag */
function startDrag(e) {
  isActive.value = true
  mode = "drag"
  startX = e.clientX
  startY = e.clientY
  window.addEventListener("pointermove", onMove)
  window.addEventListener("pointerup", stop)
}

/* Resize */
function startResize(e) {
  isActive.value = true
  mode = "resize"
  startX = e.clientX
  startY = e.clientY
  startW = box.w
  startH = box.h
  window.addEventListener("pointermove", onMove)
  window.addEventListener("pointerup", stop)
}

function onMove(e) {
  const dxScreen = e.clientX - startX
  const dyScreen = e.clientY - startY
  const { dx, dy } = screenToCanvasDelta(dxScreen, dyScreen)

  if (mode === "drag") {
    box.x = clamp(box.x + dx, 0, W - box.w)
    box.y = clamp(box.y + dy, 0, H - box.h)
    startX = e.clientX
    startY = e.clientY
  } else if (mode === "resize") {
    box.w = clamp(startW + dx, MIN_W, W - box.x)
    box.h = clamp(startH + dy, MIN_H, H - box.y)
  }

  draw()
}

function stop() {
  mode = null
  window.removeEventListener("pointermove", onMove)
  window.removeEventListener("pointerup", stop)
}

function clamp(v, min, max) {
  return Math.max(min, Math.min(max, v))
}

/* تحميل PNG */
function downloadPNG() {
  const canvas = canvasRef.value
  if (!canvas) return
  canvas.toBlob((blob) => {
    if (!blob) return
    const url = URL.createObjectURL(blob)
    const a = document.createElement("a")
    a.href = url
    a.download = "ramadan-card.png"
    a.click()
    URL.revokeObjectURL(url)
  }, "image/png")
}

function goBack() {
  router.push("/card/ramadan")
}

onMounted(async () => {
  updateCanvasRect()
  await draw()

  window.addEventListener("resize", () => {
    updateCanvasRect()
    draw()
  })
})

watch([name, fontFamily, fontSize, textColor], draw)
watch(() => bg.value, async () => {
  await draw()
})
</script>

<style scoped>
.page {
  min-height: 100dvh;
  background: #111;
  padding: 12px;
  overflow-x: hidden;
}

.stage {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 14px;
}

.frame {
  position: relative;
  width: min(92vw, 420px);
  aspect-ratio: 9 / 16;
  border-radius: 22px;
  overflow: hidden;
  background: #000;
}

.canvas {
  width: 100%;
  height: 100%;
  display: block;
}

.boxOverlay {
  position: absolute;
  border-radius: 999px;
  display: flex;
  align-items: center;
  justify-content: center;
  touch-action: none;
}

.boxOverlay.active {
  outline: 2px dashed rgba(223, 168, 16, 0.95);
  outline-offset: 4px;
}

.nameInput {
  width: 100%;
  height: 100%;
  background: transparent;
  border: 0;
  outline: none;
  text-align: center;
  font-weight: 700;
  /* نخفي نص input لأن النص الحقيقي ينرسم بالكانفس */
  color: transparent;
  caret-color: #222;
  padding: 0 12px;
  -webkit-tap-highlight-color: transparent;
}

.resizeHandle {
  position: absolute;
  right: -7px;
  bottom: -7px;
  width: 16px;
  height: 16px;
  background: #9e7005b0;
  border: 2px solid rgba(0,0,0,0.35);
  border-radius: 50%;
  cursor: nwse-resize;
}

.sidebar {
  width: min(92vw, 360px);
}

.panel {
  padding: 14px;
  border-radius: 18px;
  background: rgba(17, 17, 17, 0.9);
  border: 1px solid rgba(255,255,255,0.08);
  color: #fff;
}

.hint {
  font-size: 12px;
  opacity: 0.75;
  margin-bottom: 10px;
  text-align: center;
}

.field {
  display: flex;
  flex-direction: column;
  gap: 6px;
  margin-bottom: 10px;
}

.label {
  font-size: 12px;
  opacity: 0.85;
  text-align: right;
}

.input,
.select {
  height: 38px;
  border-radius: 14px;
  background: rgba(0, 0, 0, 0.28);
  color: #fff;
  border: 1px solid rgba(255, 255, 255, 0.14);
  padding: 0 10px;
  outline: none;
  text-align: right;
}

.range {
  width: 100%;
}

.color {
  width: 100%;
  height: 38px;
  border-radius: 14px;
  border: 0;
  background: transparent;
}

.btnRow {
  display: flex;
  gap: 10px;
  margin-top: 6px;
}

.btn {
  flex: 1;
  height: 40px;
  border-radius: 14px;
  background: rgba(0, 0, 0, 0.35);
  color: #fff;
  border: 1px solid rgba(255, 255, 255, 0.12);
  cursor: pointer;
}

.btn.primary {
  background: rgba(34, 197, 94, 0.35);
}
</style>

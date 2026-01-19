<template>
  <div class="page">
    <div class="stage">
      <!-- ✅ البطاقة -->
      <div class="frame">
        <canvas
          ref="canvasRef"
          :width="W"
          :height="H"
          class="canvas"
        ></canvas>

        <!-- ✅ input شفاف للضغط والكتابة فقط -->
        <input
          ref="nameInputRef"
          v-model="name"
          class="nameOverlay"
          :style="nameOverlayStyle"
          maxlength="40"
          dir="rtl"
          inputmode="text"
          @focus="onFocus"
          @blur="onBlur"
        />
      </div>

      <!-- ✅ لوحة الإعدادات -->
      <aside class="sidebar">
        <div class="panel">

          <label class="field">
            <span class="label">الاسم</span>
            <input v-model="name" class="input" maxlength="40" />
          </label>

          <label class="field">
            <span class="label">الخط</span>
            <select v-model="fontFamily" class="select">
              <option value="Tahoma">Tahoma</option>
              <option value="Arial">Arial</option>
              <option value="Georgia">Georgia</option>
              <option value="Times New Roman">Times</option>
            </select>
          </label>

          <label class="field">
            <span class="label">الحجم ({{ fontSize }})</span>
            <input type="range" min="24" max="90" v-model="fontSize" class="range" />
          </label>

          <label class="field">
            <span class="label">اللون</span>
            <input type="color" v-model="textColor" class="color" />
          </label>

          <div class="btnRow">
            <button class="btn" @click="goBack">رجوع</button>
            <button class="btn primary" @click="downloadPNG">تحميل</button>
          </div>
        </div>
      </aside>
    </div>
  </div>
</template>

<script setup>
const router = useRouter()
const route = useRoute()

/* مقاس الصورة */
const W = 1080
const H = 1920

const canvasRef = ref(null)
const nameInputRef = ref(null)

/* النص */
const placeholderText = "اكتب اسمك هنا"
const name = ref(placeholderText)
const fontFamily = ref("Times New Roman")
const fontSize = ref(50)
const textColor = ref("#9b9898ff")

const isFocused = ref(false)

/* الخلفية */
const bg = computed(() => route.query.bg || "/templates/ramadan_1.png")

/* مكان البوكس على مقاس الكانفس */
const NAME_BOX = { x: 180, y: 1620, w: 720, h: 130, r: 60 }

/* مكان input فوق البوكس (responsive) */
const nameOverlayStyle = computed(() => {
  const canvas = canvasRef.value
  if (!canvas) return {}

  const rect = canvas.getBoundingClientRect()
  const sx = rect.width / W
  const sy = rect.height / H

  return {
    left: `${NAME_BOX.x * sx}px`,
    top: `${NAME_BOX.y * sy}px`,
    width: `${NAME_BOX.w * sx}px`,
    height: `${NAME_BOX.h * sy}px`,
    fontSize: `${fontSize.value * sy}px`,
    fontFamily: fontFamily.value,
  }
})

onMounted(() => {
  renderCard()
  window.addEventListener("resize", renderCard)
})

onBeforeUnmount(() => {
  window.removeEventListener("resize", renderCard)
})

function onFocus() {
  isFocused.value = true
  if (name.value === placeholderText) name.value = ""
}

function onBlur() {
  isFocused.value = false
  if (!name.value) name.value = placeholderText
}

function loadImage(src) {
  return new Promise((resolve, reject) => {
    const img = new Image()
    img.onload = () => resolve(img)
    img.onerror = reject
    img.src = src
  })
}

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

async function renderCard() {
  const canvas = canvasRef.value
  if (!canvas) return
  const ctx = canvas.getContext("2d")

  ctx.clearRect(0, 0, W, H)

  const bgImg = await loadImage(bg.value)
  ctx.drawImage(bgImg, 0, 0, W, H)

  /* البوكس الأبيض */
  ctx.fillStyle = "rgba(255,255,255,0.96)"
  roundRect(ctx, NAME_BOX.x, NAME_BOX.y, NAME_BOX.w, NAME_BOX.h, NAME_BOX.r)
  ctx.fill()

  ctx.save()
  roundRect(ctx, NAME_BOX.x, NAME_BOX.y, NAME_BOX.w, NAME_BOX.h, NAME_BOX.r)
  ctx.clip()

  ctx.font = `700 ${fontSize.value}px ${fontFamily.value}`
  ctx.fillStyle = textColor.value
  ctx.textAlign = "center"
  ctx.textBaseline = "middle"

  const textToDraw =
    name.value === placeholderText ? "" : name.value

  const safeText = trimText(ctx, textToDraw, NAME_BOX.w - 80)
  ctx.fillText(
    safeText,
    NAME_BOX.x + NAME_BOX.w / 2,
    NAME_BOX.y + NAME_BOX.h / 2
  )

  ctx.restore()
}

function downloadPNG() {
  const canvas = canvasRef.value
  canvas.toBlob((blob) => {
    if (!blob) return
    const url = URL.createObjectURL(blob)
    const a = document.createElement("a")
    a.href = url
    a.download = "ramadan-card.png"
    a.click()
    URL.revokeObjectURL(url)
  })
}

function goBack() {
  router.push("/card/ramadan")
}

watch([name, fontFamily, fontSize, textColor], renderCard)
watch(() => bg.value, renderCard)
</script>

<style scoped>
.page {
  min-height: 100dvh;
  background: #111;
  padding: 12px;
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
}

/* ✅ input شفاف – يمنع تكرار الحروف */
.nameOverlay {
  position: absolute;
  background: transparent;
  border: 0;
  outline: none;

  text-align: center;
  font-weight: 700;

  color: transparent;      /* الحل الأساسي */
  caret-color: #111;       /* مؤشر الكتابة */

  padding: 0 12px;
  border-radius: 999px;
}

.sidebar {
  width: min(92vw, 360px);
}

.panel {
  padding: 14px;
  border-radius: 18px;
  background: rgba(17, 17, 17, 0.86);
  color: #fff;
}

.field {
  display: flex;
  flex-direction: column;
  gap: 6px;
  margin-bottom: 8px;
}

.input,
.select {
  height: 38px;
  border-radius: 14px;
  background: rgba(0, 0, 0, 0.3);
  color: #fff;
  border: 1px solid rgba(255, 255, 255, 0.2);
  padding: 0 10px;
}

.btnRow {
  display: flex;
  gap: 10px;
}

.btn {
  flex: 1;
  height: 40px;
  border-radius: 14px;
  background: rgba(0, 0, 0, 0.35);
  color: #fff;
  border: 1px solid rgba(255, 255, 255, 0.15);
}

.btn.primary {
  background: rgba(34, 197, 94, 0.35);
}
</style>

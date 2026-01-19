<template>
  <div class="page">
    <div class="stage">
      <!-- ✅ البطاقة -->
      <div class="frame">
        <canvas ref="canvasRef" :width="W" :height="H" class="canvas"></canvas>
      </div>

      <!-- ✅ لوحة إعدادات -->
      <aside class="sidebar" @click.stop>
        <div class="panel">
          <div class="panelTitle">إعدادات</div>

          <label class="field">
            <span class="label">الاسم</span>
            <input v-model="name" class="input" maxlength="40" placeholder="اكتبي الاسم" />
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
            <button class="btn" type="button" @click="goBack">رجوع</button>

            <div class="dropdown" @click.stop>
              <button class="btn primary" type="button" @click.stop="toggleDownload">
                تحميل <span class="caret">▾</span>
              </button>

              <div v-if="openDownload" class="menu">
                <button class="menuItem" type="button" @click="downloadPNGAndClose">تحميل PNG</button>
                <button class="menuItem" type="button" @click="downloadPDFAndClose">تحميل PDF</button>
              </div>
            </div>
          </div>
        </div>
      </aside>
      <!-- /sidebar -->
    </div>
  </div>
</template>

<script setup>
import { jsPDF } from "jspdf"

const router = useRouter()
const route = useRoute()

// ✅ مقاس سناب
const W = 1080
const H = 1920
const canvasRef = ref(null)

// ✅ بيانات النص
const name = ref("شهد الملا")
const fontFamily = ref("Tahoma")
const fontSize = ref(56)
const textColor = ref("#111111")

// ✅ الخلفية المختارة
const bg = computed(() => route.query.bg || "/templates/ramadan_1.png")

// ✅ مكان البوكس الأبيض داخل الصورة (ثابت)
const NAME_BOX = { x: 180, y: 1620, w: 720, h: 130, r: 60 }

// dropdown
const openDownload = ref(false)
function toggleDownload() {
  openDownload.value = !openDownload.value
}
function closeDownload() {
  openDownload.value = false
}
function handleOutsideClick() {
  closeDownload()
}

onMounted(() => {
  renderCard()
  window.addEventListener("click", handleOutsideClick)
})
onBeforeUnmount(() => {
  window.removeEventListener("click", handleOutsideClick)
})

function loadImage(src) {
  return new Promise((resolve, reject) => {
    const img = new Image()
    img.crossOrigin = "anonymous"
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

  // ✅ بوكس أبيض داخل الكانفس (بدون ظل)
  ctx.fillStyle = "rgba(255,255,255,0.96)"
  roundRect(ctx, NAME_BOX.x, NAME_BOX.y, NAME_BOX.w, NAME_BOX.h, NAME_BOX.r)
  ctx.fill()

  // ✅ بدون ظلال
  ctx.shadowColor = "transparent"
  ctx.shadowBlur = 0
  ctx.shadowOffsetX = 0
  ctx.shadowOffsetY = 0

  // ✅ النص داخل البوكس
  ctx.save()
  roundRect(ctx, NAME_BOX.x, NAME_BOX.y, NAME_BOX.w, NAME_BOX.h, NAME_BOX.r)
  ctx.clip()

  ctx.font = `700 ${fontSize.value}px ${fontFamily.value}`
  ctx.fillStyle = textColor.value
  ctx.textAlign = "center"
  ctx.textBaseline = "middle"

  const safeText = trimText(ctx, name.value, NAME_BOX.w - 80)
  ctx.fillText(safeText, NAME_BOX.x + NAME_BOX.w / 2, NAME_BOX.y + NAME_BOX.h / 2)

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
  }, "image/png")
}

function downloadPDF() {
  const canvas = canvasRef.value
  const pdf = new jsPDF({ orientation: "portrait", unit: "px", format: [W, H] })
  pdf.addImage(canvas.toDataURL("image/png"), "PNG", 0, 0, W, H)
  pdf.save("ramadan-card.pdf")
}

function downloadPNGAndClose() {
  downloadPNG()
  closeDownload()
}

function downloadPDFAndClose() {
  downloadPDF()
  closeDownload()
}

function goBack() {
  router.push("/card/ramadan")
}

watch([name, fontFamily, fontSize, textColor], renderCard)
watch(() => bg.value, renderCard)
</script>

<style scoped>
/* ✅ الصفحة: اسمحي بالسكرول للجوال بدل قص */
.page {
  min-height: 100dvh;
  background: #111;

  /* مهم للجوال: لا تقص المحتوى */
  overflow-y: auto;
  overflow-x: hidden;

  padding: clamp(10px, 3vw, 14px);
  padding-bottom: calc(clamp(10px, 3vw, 14px) + env(safe-area-inset-bottom));
}

/* ✅ ترتيب عام */
.stage {
  min-height: calc(100dvh - 20px);
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 14px;
}

/* ✅ البطاقة: خليها تعتمد على العرض (أفضل للجوال) */
.frame {
  width: min(92vw, 420px);
  aspect-ratio: 9 / 16;
  border-radius: 22px;
  overflow: hidden;
  box-shadow: 0 18px 55px rgba(0, 0, 0, 0.35);
  background: #000;
}

/* ✅ الكانفس يتمدد داخل الفريم */
.canvas {
  width: 100%;
  height: 100%;
  display: block;
}

/* ✅ sidebar */
.sidebar {
  width: min(92vw, 360px);
  display: flex;
  justify-content: center;
}

/* ✅ شكل اللوحة */
.panel {
  width: 100%;
  padding: 14px;
  border-radius: 18px;
  background: rgba(255, 255, 255, 0.06);
  border: 1px solid rgba(255, 255, 255, 0.12);
  backdrop-filter: blur(10px);
  color: #fff;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.panelTitle {
  font-weight: 800;
  font-size: 14px;
  opacity: 0.95;
  text-align: right;
}

.field {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.label {
  font-size: 12px;
  opacity: 0.9;
  text-align: right;
}

.input,
.select {
  height: 38px;
  border-radius: 14px;
  border: 1px solid rgba(255, 255, 255, 0.16);
  background: rgba(0, 0, 0, 0.25);
  color: #fff;
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
  cursor: pointer;
}

.btnRow {
  display: flex;
  gap: 10px;
  justify-content: space-between;
  align-items: center;
  margin-top: 6px;
}

.btn {
  flex: 1;
  height: 40px;
  border-radius: 14px;
  background: rgba(0, 0, 0, 0.35);
  color: #fff;
  border: 1px solid rgba(255, 255, 255, 0.15);
  cursor: pointer;
  font-size: 13px;
}

.btn.primary {
  background: rgba(34, 197, 94, 0.35);
}

.dropdown {
  position: relative;
  flex: 1;
}

.caret {
  opacity: 0.9;
}

.menu {
  position: absolute;
  right: 0;
  top: 46px;
  width: 100%;
  background: rgba(0, 0, 0, 0.92);
  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 14px;
  padding: 6px;
  z-index: 50;
}

.menuItem {
  width: 100%;
  text-align: right;
  padding: 10px 10px;
  border-radius: 10px;
  background: transparent;
  color: #fff;
  border: 0;
  cursor: pointer;
}

.menuItem:hover {
  background: rgba(255, 255, 255, 0.08);
}

/* ✅ ديسكتوب/شاشات عريضة: كارد + سايدبار جنب بعض */
@media (min-width: 900px) {
  .stage {
    flex-direction: row;
    align-items: center;
  }
  .sidebar {
    width: 280px;
    max-width: 34vw;
  }
  .frame {
    width: min(46vw, 460px);
  }
}

/* ✅ جوال: خلي اللوحة تحت + خليها sticky عشان ما تضيع */
@media (max-width: 899px) {
  .stage {
    flex-direction: column;
    align-items: center;
  }

  .sidebar {
    position: sticky;
    bottom: 0;
    padding-bottom: env(safe-area-inset-bottom);
    z-index: 30;
  }

  .panel {
    background: rgba(17, 17, 17, 0.86);
    border: 1px solid rgba(255, 255, 255, 0.10);
    backdrop-filter: blur(12px);
  }
}

/* ✅ شاشات صغيرة جدًا */
@media (max-width: 360px) {
  .frame {
    width: 94vw;
    border-radius: 18px;
  }
}
</style>

<template>
  <div class="page">
    <!-- العنوان -->
    <header class="header">
      <h1>اختر تصميمك المناسب</h1>
      <p>الخطوة الأولى: اختر القالب الذي يعجبك ثم اضغط التالي</p>
    </header>

    <!-- منطقة المعاينة -->
    <div class="stage">
      <!-- سهم يسار -->
      <button class="arrow left" @click="prev" aria-label="السابق">‹</button>

      <!-- الكارد -->
      <div class="preview">
        <img class="bg" :src="current.src" alt="" />
      </div>

      <!-- سهم يمين -->
      <button class="arrow right" @click="next" aria-label="التالي">›</button>
    </div>

    <!-- معلومات القالب -->
    <div class="info">
      <div class="label">{{ current.label }}</div>
      <div class="dots">
        <span
          v-for="(b, i) in backgrounds"
          :key="b.id"
          class="dot"
          :class="{ active: i === index }"
          @click="go(i)"
        />
      </div>
    </div>

    <!-- زر التالي -->
    <div class="actions">
      <button class="btn" @click="goNext">التالي</button>
    </div>

    <!-- الحقوق -->
    <footer class="footer">
      by: shahadalmulla — contact: shahadalmulla112255@gmail.com
    </footer>
  </div>
</template>

<script setup>
const router = useRouter()

const backgrounds = [
  { id: "ramadan-1", label: "رمضان 1", src: "/templates/ramadan_1.png" },
  { id: "ramadan-2", label: "رمضان 2", src: "/templates/ramadan_2.png" },
  { id: "ramadan-3", label: "رمضان 3", src: "/templates/ramadan_3.png" },
]

const index = ref(0)
const current = computed(() => backgrounds[index.value])

function prev() {
  index.value = (index.value - 1 + backgrounds.length) % backgrounds.length
}
function next() {
  index.value = (index.value + 1) % backgrounds.length
}
function go(i) {
  index.value = i
}
function goNext() {
  router.push({
    path: "/card/ramadan/edit",
    query: { bg: current.value.src },
  })
}
</script>

<style scoped>
/* الصفحة */
.page {
  min-height: 100dvh;
  background: #f3f4f6;
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial;
}

/* العنوان */
.header {
  text-align: center;
  margin: 24px 16px 8px;
}
.header h1 {
  font-size: 22px;
  font-weight: 800;
  margin-bottom: 6px;
}
.header p {
  font-size: 14px;
  color: #6b7280;
}

/* منطقة المعاينة */
.stage {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  margin: 16px 0;
}

/* الكارد 9:16 */
.preview {
  position: relative;
  aspect-ratio: 9 / 16;
  width: min(78vw, 360px);
  border-radius: 22px;
  overflow: hidden;
  background: #000;
  box-shadow: 0 18px 50px rgba(0,0,0,0.18);
}
.bg {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* الأسهم */
.arrow {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  width: 44px;
  height: 44px;
  border-radius: 999px;
  border: none;
  background: #fff;
  box-shadow: 0 6px 18px rgba(0,0,0,0.18);
  font-size: 26px;
  cursor: pointer;
}
.arrow.left { left: calc(50% - 220px); }
.arrow.right { right: calc(50% - 220px); }

/* معلومات القالب */
.info {
  margin-top: 12px;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
}
.label {
  width: 100%;
  font-weight: 800;
  font-size: 16px;
  text-align: center;
}
.dots {
  display: flex;
  gap: 8px;
  margin-top: 8px;
}
.dot {
  width: 8px;
  height: 8px;
  border-radius: 999px;
  background: rgba(0,0,0,0.25);
  cursor: pointer;
}
.dot.active {
  background: #111827;
}

/* زر التالي */
.actions {
  margin: 16px 0 8px;
}
.btn {
  background: #111827;
  color: #fff;
  border: none;
  padding: 12px 22px;
  border-radius: 14px;
  font-weight: 800;
  cursor: pointer;
}

/* الحقوق */
.footer {
  margin-top: auto;
  padding: 14px 8px;
  font-size: 12px;
  color: #6b7280;
  text-align: center;
}
</style>

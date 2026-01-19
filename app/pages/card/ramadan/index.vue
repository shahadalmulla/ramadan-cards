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
      <button class="arrow left" @click="prev">‹</button>

      <!-- الكارد -->
      <div class="preview">
        <img class="bg" :src="current.src" alt="" />
      </div>

      <!-- سهم يمين -->
      <button class="arrow right" @click="next">›</button>
    </div>

    <!-- اسم التصميم + النقاط -->
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

    <!-- الحقوق -->
    <footer class="footer">
      by: shahadalmulla — contact: shahadalmulla112255@gmail.com
    </footer>

    <!-- زر التالي (ثابت) -->
    <div class="next-bar">
      <button class="next-btn" @click="goNext">التالي</button>
    </div>
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
.page {
  min-height: 100vh;
  background: #f3f4f6;
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial;
}

/* العنوان */
.header {
  text-align: center;
  padding: 24px 16px 10px;
}
.header h1 {
  font-size: 22px;
  font-weight: 800;
}
.header p {
  font-size: 14px;
  color: #6b7280;
  margin-top: 6px;
}

/* المعاينة */
.stage {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  padding: 24px 0;
}

/* الكارد */
.preview {
  position: relative;
  width: 280px;
  aspect-ratio: 9 / 16;
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
  box-shadow: 0 8px 20px rgba(0,0,0,0.2);
  font-size: 22px;
  cursor: pointer;
}
.arrow.left {
  left: calc(50% - 190px);
}
.arrow.right {
  right: calc(50% - 190px);
}

/* اسم التصميم */
.info {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
}
.label {
  font-weight: 700;
}

/* النقاط */
.dots {
  display: flex;
  gap: 8px;
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

/* الحقوق */
.footer {
  margin-top: 16px;
  font-size: 12px;
  color: #6b7280;
}

/* زر التالي */
.next-bar {
  position: fixed;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 50;
}
.next-btn {
  background: #111827;
  color: #fff;
  padding: 14px 32px;
  border-radius: 999px;
  font-size: 16px;
  font-weight: 700;
  border: none;
  cursor: pointer;
  box-shadow: 0 10px 30px rgba(0,0,0,0.25);
}
.next-btn:hover {
  background: #000;
}
</style>

<template>
  <div class="page">
    <!-- العنوان -->
    <header class="header">
      <h1>اختر تصميمك المناسب</h1>
      <p>الخطوة الأولى: اختر القالب الذي يعجبك ثم اضغط التالي</p>
    </header>

    <!-- منطقة المعاينة -->
    <section class="stage">
      <button class="nav left" @click="prev">‹</button>

      <div class="preview">
        <img :src="current.src" alt="template" />
      </div>

      <button class="nav right" @click="next">›</button>
    </section>

    <!-- اسم التصميم -->
    <div class="label">
      {{ current.label }}
      <div class="dots">
        <span
          v-for="(b, i) in backgrounds"
          :key="b.id"
          :class="{ active: i === index }"
        />
      </div>
    </div>

    <!-- زر التالي (قريب من الكارد) -->
    <div class="actions">
      <button class="next" @click="goNext">التالي</button>
    </div>

    <!-- الفوتر -->
    <footer class="footer">
      by: shahadalmulla — contact: shahadalmulla112255@gmail.com
    </footer>
  </div>
</template>

<script setup>
const router = useRouter()

const backgrounds = [
  { id: 1, label: 'رمضان 1', src: '/templates/ramadan_1.png' },
  { id: 2, label: 'رمضان 2', src: '/templates/ramadan_2.png' },
  { id: 3, label: 'رمضان 3', src: '/templates/ramadan_3.png' },
]

const index = ref(0)
const current = computed(() => backgrounds[index.value])

const next = () => (index.value = (index.value + 1) % backgrounds.length)
const prev = () => (index.value = (index.value - 1 + backgrounds.length) % backgrounds.length)

const goNext = () => {
  router.push({
    path: '/card/ramadan/edit',
    query: { bg: current.value.src },
  })
}
</script>

<style scoped>
.page {
  min-height: 100dvh;
  display: flex;
  flex-direction: column;
  align-items: center;
  background: #f3f4f6;
  padding: 16px;
  gap: 16px;
}

/* العنوان */
.header {
  text-align: center;
}
.header h1 {
  font-size: 22px;
  font-weight: 700;
}
.header p {
  font-size: 14px;
  color: #6b7280;
}

/* المعاينة */
.stage {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
}

.preview {
  aspect-ratio: 9 / 16;
  height: min(65vh, 520px);
  border-radius: 22px;
  overflow: hidden;
  box-shadow: 0 18px 50px rgba(0,0,0,.18);
  background: #000;
}
.preview img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* الأسهم */
.nav {
  position: absolute;
  width: 44px;
  height: 44px;
  border-radius: 999px;
  border: none;
  background: #fff;
  font-size: 22px;
  cursor: pointer;
  box-shadow: 0 6px 18px rgba(0,0,0,.15);
}
.nav.left { left: 12px; }
.nav.right { right: 12px; }

/* اسم التصميم */
.label {
  text-align: center;
  font-weight: 600;
}
.dots {
  display: flex;
  gap: 6px;
  justify-content: center;
  margin-top: 6px;
}
.dots span {
  width: 7px;
  height: 7px;
  background: #d1d5db;
  border-radius: 50%;
}
.dots span.active {
  background: #111827;
}

/* زر التالي */
.actions {
  width: 100%;
  display: flex;
  justify-content: center;
}
.next {
  background: #111827;
  color: #fff;
  border: none;
  border-radius: 999px;
  padding: 12px 28px;
  font-size: 15px;
  font-weight: 700;
  cursor: pointer;
}

/* الفوتر */
.footer {
  margin-top: auto;
  font-size: 12px;
  color: #6b7280;
  text-align: center;
}
</style>

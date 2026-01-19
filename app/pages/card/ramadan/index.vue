<template>
  <div class="page">
    <!-- المعاينة الكبيرة -->
    <div class="stage">
      <div class="preview">
        <img class="bg" :src="current.src" alt="" />
      </div>
    </div>

    <!-- شريط تحت زي a7 -->
    <div class="bar">
      <button class="icon" @click="prev" title="السابق">←</button>

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

      <button class="icon" @click="next" title="التالي">→</button>

      <button class="btn" @click="goNext">التالي</button>
    </div>
  </div>
</template>

<script setup>
const router = useRouter()

const backgrounds = [
  { id: "ramadan-1", label: "رمضان - 1", src: "/templates/ramadan_1.png" },
  { id: "ramadan-2", label: "رمضان - 2", src: "/templates/ramadan_2.png" },
  { id: "ramadan-3", label: "رمضان - 3", src: "/templates/ramadan_3.png" },
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

// (اختياري) دعم الكيبورد
onMounted(() => {
  const onKey = (e) => {
    if (e.key === "ArrowLeft") prev()
    if (e.key === "ArrowRight") next()
  }
  window.addEventListener("keydown", onKey)
  onBeforeUnmount(() => window.removeEventListener("keydown", onKey))
})
</script>

<style scoped>
.page {
  height: 100dvh;
  width: 100%;
  background: #f3f4f6;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial;
}

/* منطقة المعاينة */
.stage {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 18px;
}

/* كارد 9:16 زي سناب */
.preview {
  position: relative;
  height: calc(100dvh - 130px);
  max-height: 820px;
  aspect-ratio: 9 / 16;
  border-radius: 22px;
  overflow: hidden;
  background: #000;
  box-shadow: 0 18px 50px rgba(0, 0, 0, 0.18);
}

.bg {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

/* شريط التحكم تحت */
.bar {
  height: 92px;
  background: #fff;
  border-top: 1px solid rgba(0, 0, 0, 0.08);
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 14px;
}

.icon {
  width: 42px;
  height: 42px;
  border-radius: 999px;
  border: 1px solid rgba(0, 0, 0, 0.12);
  background: #fff;
  cursor: pointer;
  font-size: 18px;
}

.info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 8px;
  align-items: center;
  justify-content: center;
}

.label {
  font-weight: 700;
  font-size: 14px;
}

.dots {
  display: flex;
  gap: 8px;
}

.dot {
  width: 8px;
  height: 8px;
  border-radius: 999px;
  background: rgba(0, 0, 0, 0.18);
  cursor: pointer;
}
.dot.active {
  background: #111827;
}

.btn {
  border: 0;
  background: #111827;
  color: #fff;
  padding: 10px 14px;
  border-radius: 12px;
  cursor: pointer;
  font-weight: 700;
}
</style>

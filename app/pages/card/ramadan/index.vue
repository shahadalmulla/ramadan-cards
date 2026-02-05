<template>
  <div class="page">
    <!-- Top -->
    <header class="topbar">
      <div class="brand">
        <img src="/brand/logo.png" alt="شعار أسرة آل ملا" class="logo" />
        <div class="brandText">
          <div class="brandTitle">أسرة آل ملا</div>
          <div class="brandSub">اختر القالب ثم اضغط التالي</div>
        </div>
      </div>

      <div class="badge">
        قالب {{ current.id }} / {{ backgrounds.length }}
      </div>
    </header>

    <!-- Title -->
    <section class="hero">
      <h1>اختر تصميمك المناسب</h1>
      <p>اضغط على أي قالب من الأسفل ثم اضغط زر التالي</p>
    </section>

    <!-- Main preview -->
    <section class="stage">
      <div class="previewShell">
        <div class="preview">
          <img :src="current.src" :alt="current.label" />
        </div>
      </div>
    </section>

    <!-- Templates grid (NO horizontal scroll) -->
    <section class="thumbs">
      <button
        v-for="(b, i) in backgrounds"
        :key="b.id"
        class="thumb"
        :class="{ active: i === index }"
        @click="index = i"
        type="button"
      >
        <img :src="b.src" :alt="b.label" />
        <span class="thumbNo">{{ b.id }}</span>
      </button>
    </section>

    <!-- Next button -->
    <div class="actions">
      <button class="next" type="button" @click="goNext">
        التالي →
      </button>
    </div>

    <!-- Footer -->
    <footer class="footer">
      <span>by: shahad almulla</span>
      <span class="dot">•</span>
      <a
        class="link"
        href="mailto:shahadalmulla112255@gmail.com"
      >
        للتواصل اضغط هنا
      </a>
    </footer>
  </div>
</template>

<script setup>
const router = useRouter()

const backgrounds = Array.from({ length: 12 }, (_, i) => {
  const n = i + 1
  return {
    id: n,
    label: `رمضان ${n}`,
    src: `/templates/ramadan_${n}.png`,
  }
})

const index = ref(0)
const current = computed(() => backgrounds[index.value])

const goNext = () => {
  router.push({
    path: "/card/ramadan/edit",
    query: {
      bg: current.value.src,
      tid: String(current.value.id),
    },
  })
}
</script>

<style scoped>
/* ===== Page ===== */
.page {
  min-height: 100dvh;
  background: #fbf6ec;
  color: #2b1f12;
  padding: 14px;
}

/* ===== Topbar ===== */
.topbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
}

.brand {
  display: flex;
  align-items: center;
  gap: 10px;
}

.logo {
  width: 42px;
  height: 42px;
  object-fit: contain;
}

.brandTitle {
  font-weight: 800;
  font-size: 14px;
  color: #6b4a1e;
}

.brandSub {
  font-size: 12px;
  color: rgba(107, 74, 30, 0.75);
}

.badge {
  padding: 6px 12px;
  border-radius: 999px;
  font-size: 12px;
  background: rgba(184, 139, 58, 0.12);
  border: 1px solid rgba(184, 139, 58, 0.25);
  color: #6b4a1e;
}

/* ===== Title ===== */
.hero {
  text-align: center;
  margin: 14px 0;
}

.hero h1 {
  font-size: 22px;
  font-weight: 800;
  margin: 0;
}

.hero p {
  font-size: 14px;
  color: rgba(43, 31, 18, 0.65);
  margin-top: 6px;
}

/* ===== Preview ===== */
.stage {
  display: flex;
  justify-content: center;
  margin-bottom: 14px;
}

.previewShell {
  width: 100%;
  max-width: 420px;
  padding: 10px;
  border-radius: 22px;
  background: rgba(255, 255, 255, 0.6);
  border: 1px solid rgba(184, 139, 58, 0.25);
}

.preview {
  aspect-ratio: 9 / 16;
  border-radius: 18px;
  overflow: hidden;
  background: #fff;
}

.preview img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* ===== Templates grid ===== */
.thumbs {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
  margin-top: 10px;
}

@media (min-width: 720px) {
  .thumbs {
    grid-template-columns: repeat(3, 1fr);
  }
}

.thumb {
  position: relative;
  border-radius: 16px;
  overflow: hidden;
  border: 2px solid rgba(184, 139, 58, 0.2);
  background: #fff;
  cursor: pointer;
}

.thumb.active {
  border-color: #b88b3a;
  box-shadow: 0 6px 18px rgba(184, 139, 58, 0.3);
}

.thumb img {
  width: 100%;
  height: 120px;
  object-fit: cover;
}

.thumbNo {
  position: absolute;
  top: 6px;
  right: 6px;
  background: rgba(251, 246, 236, 0.9);
  color: #6b4a1e;
  font-weight: 800;
  font-size: 12px;
  padding: 2px 8px;
  border-radius: 999px;
}

/* ===== Actions ===== */
.actions {
  display: flex;
  justify-content: center;
  margin: 18px 0;
}

.next {
  width: 100%;
  max-width: 320px;
  padding: 12px;
  font-size: 15px;
  font-weight: 800;
  border-radius: 999px;
  border: none;
  cursor: pointer;
  color: #fff;
  background: linear-gradient(135deg, #d7b46a, #b88b3a);
}

/* ===== Footer ===== */
.footer {
  margin-top: 14px;
  font-size: 12px;
  color: rgba(107, 74, 30, 0.7);
  display: flex;
  justify-content: center;
  gap: 6px;
  flex-wrap: wrap;
  position: relative;
  z-index: 5;
}

.link {
  color: #b88b3a;
  font-weight: 700;
  text-decoration: none;
}

.link:hover {
  text-decoration: underline;
}

.dot {
  opacity: 0.6;
}
</style>

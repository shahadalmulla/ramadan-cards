<template>
  <div class="page">
    <!-- Top -->
    <header class="topbar">
      <div class="brand">
        <!-- لو اسم اللوقو مختلف عدّليه هنا -->
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

        <div class="previewMeta">
          <div class="metaLeft">
            <span class="pill">رمضان {{ current.id }}</span>
          </div>
          <div class="metaRight">جاهز للمعايدة ✨</div>
        </div>
      </div>
    </section>

    <!-- Templates grid (cards 9/16) -->
    <section class="thumbs" aria-label="قائمة القوالب">
      <button
        v-for="(b, i) in backgrounds"
        :key="b.id"
        class="thumb"
        :class="{ active: i === index }"
        @click="index = i"
        type="button"
        :aria-label="`اختيار ${b.label}`"
      >
        <div class="thumbFrame">
          <img :src="b.src" :alt="b.label" />
        </div>

        <div class="thumbBar">
          <span class="thumbTitle">رمضان {{ b.id }}</span>
          <span class="thumbTag" v-if="i === index">محدد</span>
        </div>
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
      <a class="link" href="mailto:shahadalmulla112255@gmail.com">للتواصل اضغط هنا</a>
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
  white-space: nowrap;
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
  background: rgba(255, 255, 255, 0.62);
  border: 1px solid rgba(184, 139, 58, 0.25);
  box-shadow: 0 18px 55px rgba(43, 31, 18, 0.12);
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

.previewMeta {
  margin-top: 10px;
  display: flex;
  justify-content: space-between;
  gap: 10px;
  align-items: center;
  padding: 10px 12px;
  border-radius: 16px;
  background: rgba(251, 246, 236, 0.92);
  border: 1px solid rgba(184, 139, 58, 0.22);
}

.pill {
  display: inline-flex;
  padding: 6px 10px;
  border-radius: 999px;
  font-weight: 800;
  font-size: 12px;
  color: #6b4a1e;
  background: rgba(184, 139, 58, 0.12);
  border: 1px solid rgba(184, 139, 58, 0.22);
}

.metaRight {
  font-size: 12px;
  color: rgba(107, 74, 30, 0.75);
  font-weight: 700;
}

/* ===== Templates grid (cards) ===== */
.thumbs {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
  margin-top: 10px;
}

/* على الشاشات الكبيرة نخليها 3 أعمدة */
@media (min-width: 900px) {
  .thumbs {
    grid-template-columns: repeat(3, 1fr);
  }
}

.thumb {
  border: 2px solid rgba(184, 139, 58, 0.18);
  background: rgba(255, 255, 255, 0.6);
  border-radius: 18px;
  overflow: hidden;
  cursor: pointer;
  padding: 0;
  text-align: inherit;
  transition: transform 0.12s ease, border-color 0.12s ease, box-shadow 0.12s ease;
}

.thumb:active {
  transform: scale(0.99);
}

.thumb.active {
  border-color: #b88b3a;
  box-shadow: 0 10px 22px rgba(184, 139, 58, 0.22);
}

.thumbFrame {
  aspect-ratio: 9 / 16; /* ✅ هذا اللي رجّع شكل البطاقات */
  background: #fff;
  overflow: hidden;
}

.thumbFrame img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.thumbBar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 12px;
  background: rgba(251, 246, 236, 0.95);
  border-top: 1px solid rgba(184, 139, 58, 0.18);
}

.thumbTitle {
  font-weight: 900;
  font-size: 13px;
  color: #6b4a1e;
}

.thumbTag {
  font-size: 12px;
  font-weight: 800;
  color: #fff;
  background: linear-gradient(135deg, #d7b46a, #b88b3a);
  padding: 4px 10px;
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
  font-weight: 900;
  border-radius: 999px;
  border: none;
  cursor: pointer;
  color: #fff;
  background: linear-gradient(135deg, #d7b46a, #b88b3a);
  box-shadow: 0 14px 28px rgba(184, 139, 58, 0.22);
}

/* ===== Footer ===== */
.footer {
  margin-top: 12px;
  font-size: 12px;
  color: rgba(107, 74, 30, 0.7);
  display: flex;
  justify-content: center;
  gap: 6px;
  flex-wrap: wrap;
}

.link {
  color: #b88b3a;
  font-weight: 800;
  text-decoration: none;
}
.link:hover {
  text-decoration: underline;
}

.dot {
  opacity: 0.6;
}
</style>

<template>
  <div class="app-wrapper">
    <NuxtPage />

    <!-- Footer ثابت -->
    <footer class="footer-fixed">
      <span>
        by: <strong>shahadalmulla</strong> —
        contact:
        <a href="mailto:shahadalmulla112255@gmail.com">
          shahadalmulla112255@gmail.com
        </a>
      </span>
    </footer>
  </div>
</template>

<style>
.app-wrapper {
  min-height: 100dvh;
}

/* فوتر ثابت */
.footer-fixed {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;

  padding: 10px 14px;
  font-size: 12px;
  text-align: center;

  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(6px);

  color: #6b7280;
  border-top: 1px solid rgba(0, 0, 0, 0.08);
  z-index: 9999;
}

.footer-fixed a {
  color: #374151;
  text-decoration: none;
  font-weight: 600;
}

.footer-fixed a:hover {
  text-decoration: underline;
}
</style>

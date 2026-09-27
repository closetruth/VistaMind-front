<template>
  <footer class="app-footer">
    <div class="container footer-inner">
      <div class="footer-brand">
        <svg viewBox="0 0 48 48" width="28" height="28" fill="none" style="margin-right:8px">
          <defs>
            <linearGradient id="footerGrad" x1="0" y1="0" x2="48" y2="48" gradientUnits="userSpaceOnUse">
              <stop offset="0%" stop-color="#6366f1"/>
              <stop offset="100%" stop-color="#818cf8"/>
            </linearGradient>
          </defs>
          <rect width="48" height="48" rx="12" fill="url(#footerGrad)"/>
          <text x="24" y="33" text-anchor="middle" fill="white" font-size="22" font-weight="bold">V</text>
        </svg>
        <span class="brand-name">VistaMind</span>
      </div>
      <div class="footer-links">
        <a href="#">关于我们</a>
        <a href="#">联系我们</a>
        <a href="#">用户协议</a>
        <a href="#">隐私政策</a>
      </div>
      <p class="copyright">© 2026 VistaMind. All rights reserved.</p>
    </div>
  </footer>
</template>

<style scoped lang="scss">
.app-footer {
  margin-top: 60px;
  padding: 32px 0;
  background: var(--bili-white);
  border-top: 1px solid var(--bili-border);
}

.footer-inner {
  text-align: center;
}

.footer-brand {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 16px;
}

.brand-name {
  font-size: 18px;
  font-weight: 700;
  background: linear-gradient(135deg, #6366f1 0%, #818cf8 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.footer-links {
  display: flex;
  justify-content: center;
  gap: 28px;
  margin-bottom: 14px;

  a {
    font-size: 13px;
    color: var(--bili-text-tertiary);
    transition: color var(--bili-transition-fast);

    &:hover {
      color: var(--bili-pink);
    }
  }
}

.copyright {
  font-size: 12px;
  color: var(--bili-text-tertiary);
  letter-spacing: 0.3px;
}
</style>

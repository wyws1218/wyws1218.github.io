---
title: 王一的个人博客
date: 2026-03-31 10:00:00
type: 'home'
aside: false
comments: false
---

<div class="home-grid">

  <a class="home-card" href="/posts/">
    <div class="home-card-inner">
      <div class="home-card-icon"><i class="fas fa-code"></i></div>
      <div class="home-card-title">技术博客</div>
    </div>
  </a>

  <a class="home-card" href="/gaming-profile/">
    <div class="home-card-inner">
      <div class="home-card-icon"><i class="fas fa-gamepad"></i></div>
      <div class="home-card-title">游戏履历</div>
    </div>
  </a>

  <a class="home-card" href="/links/">
    <div class="home-card-inner">
      <div class="home-card-icon"><i class="fas fa-link"></i></div>
      <div class="home-card-title">相关链接</div>
    </div>
  </a>

  <a class="home-card" href="/about/">
    <div class="home-card-inner">
      <div class="home-card-icon"><i class="fas fa-heart"></i></div>
      <div class="home-card-title">关于</div>
    </div>
  </a>

</div>

<style>
/* 首页专属：隐藏顶部导航白条、蓝色横幅标题、页脚；页面固定不滚动 */
#nav { display: none !important; }
#page-header { display: none !important; }
#footer, footer { display: none !important; }
#rightside { display: none !important; }
#page { background: transparent !important; box-shadow: none !important; border: none !important; }
html, body { height: 100%; overflow: hidden !important; }

.home-grid {
  position: fixed;
  inset: 0;
  margin: auto;
  width: -moz-fit-content;
  width: fit-content;
  height: -moz-fit-content;
  height: fit-content;
  display: grid;
  grid-template-columns: repeat(2, 210px);
  grid-template-rows: repeat(2, 210px);
  gap: 26px;
  max-width: none;
  padding: 0;
}

.home-card {
  position: relative;
  display: block;
  width: 100%;
  height: 100%;
  border-radius: 18px;
  overflow: hidden;
  background: var(--card-bg, #fff);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.08);
  transition: transform 0.25s ease, box-shadow 0.25s ease;
  text-decoration: none !important;
  color: inherit !important;
}

.home-card:hover {
  transform: translateY(-6px);
  box-shadow: 0 10px 28px rgba(0, 0, 0, 0.14);
}

.home-card-inner {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 24px;
  text-align: center;
}

.home-card-icon {
  font-size: 48px;
  margin-bottom: 18px;
  color: var(--btn-bg, #49b1f5);
}

.home-card-title {
  font-size: 22px;
  font-weight: 600;
  color: var(--font-color, #333);
}

[data-theme='dark'] .home-card {
  background: var(--card-bg, #2a2a2a);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.4);
}
[data-theme='dark'] .home-card:hover {
  box-shadow: 0 10px 28px rgba(0, 0, 0, 0.6);
}

@media (max-width: 480px) {
  .home-grid {
    grid-template-columns: repeat(2, 40vw);
    grid-template-rows: repeat(2, 40vw);
    gap: 14px;
  }
  .home-card-icon { font-size: 32px; margin-bottom: 10px; }
  .home-card-title { font-size: 16px; }
}
</style>


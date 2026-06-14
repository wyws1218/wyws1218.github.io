---
title: Kuoyu 的笔记本
date: 2026-03-31 10:00:00
type: 'home'
aside: false
comments: false
# 首页顶部背景图：把图片放到 source/img/ 下，再把下面这行的 # 去掉、路径换成你的文件名
# top_img: /img/home-bg.jpg
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
.home-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 28px;
  max-width: 640px;
  margin: 60px auto;
  padding: 0 16px;
}

.home-card {
  position: relative;
  display: block;
  aspect-ratio: 1 / 1;
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
  margin-bottom: 8px;
  color: var(--font-color, #333);
}

.home-card-desc {
  font-size: 13px;
  opacity: 0.75;
  color: var(--font-color, #666);
  line-height: 1.4;
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
    gap: 14px;
    margin: 30px auto;
  }
  .home-card-icon { font-size: 32px; margin-bottom: 10px; }
  .home-card-title { font-size: 16px; }
  .home-card-desc { font-size: 11px; }
}
</style>


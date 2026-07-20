---
title: 游戏拆解
date: 2026-03-31 10:00:00
---

## ACT

<div class="deconstruct-grid">
  <div class="deconstruct-card">火影忍者手游</div>
  <div class="deconstruct-card">鸣潮</div>
  <div class="deconstruct-card">黑神话悟空</div>
</div>

---

## 大世界

<div class="deconstruct-grid">
  <div class="deconstruct-card">无限暖暖</div>
  <div class="deconstruct-card">异环</div>
</div>

---

<style>
.deconstruct-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 20px;
  margin: 24px 0;
}
.deconstruct-card {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 96px;
  padding: 20px;
  text-align: center;
  font-size: 17px;
  font-weight: 600;
  color: var(--font-color, #333);
  background: var(--card-bg, #fff);
  border-radius: 14px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.08);
  transition: transform 0.25s ease, box-shadow 0.25s ease;
}
.deconstruct-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 28px rgba(0, 0, 0, 0.14);
}
[data-theme='dark'] .deconstruct-card {
  background: var(--card-bg, #2a2a2a);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.4);
}
[data-theme='dark'] .deconstruct-card:hover {
  box-shadow: 0 10px 28px rgba(0, 0, 0, 0.6);
}
</style>

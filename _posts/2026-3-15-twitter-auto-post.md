---
layout:     post
title:      "推特自动发推，我用的这个免费方案"
subtitle:   "不花 $100/月，用浏览器自动化搞定"
date:       2026-03-15
author:     "凌一"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 工具
    - 自动化
    - Twitter
    - AI
---

Twitter 的官方 API，发一条推要 $100/月。

我当时看到这个价格，直接关掉了页面。

但我又确实想在推特上做内容——AI 日报、工具分享、随手记录，这些东西如果每天手动发，很快就会放弃。

所以我找了个绕过 API 的方案：用浏览器自动化，模拟真人操作，完全免费。

现在我的推特每天自动发，我不用动手。

---

## 原理

用 Puppeteer 控制一个无界面的 Chrome，模拟你登录 X.com 然后发推。

不需要 API Key，不需要显示器，可以跑在服务器上，也可以跑在你的 Mac 上。

唯一需要的是你的 `auth_token`——从 Chrome 开发者工具里复制一下，一分钟的事。

---

## 怎么用

**① 安装依赖**

```bash
mkdir twitter-bot && cd twitter-bot
npm init -y
npm install puppeteer
```

**② 获取 auth_token**

打开 Chrome，登录 X.com，按 F12 → Application → Cookies → `auth_token`，复制值。

**③ 写脚本**

```javascript
const puppeteer = require('puppeteer');

async function tweet(text) {
  const browser = await puppeteer.launch({ headless: true });
  const page = await browser.newPage();
  
  // 注入 cookie，跳过登录
  await page.setCookie({
    name: 'auth_token',
    value: 'YOUR_AUTH_TOKEN_HERE',
    domain: '.x.com'
  });
  
  await page.goto('https://x.com/home');
  await page.waitForSelector('[data-testid="tweetTextarea_0"]');
  await page.click('[data-testid="tweetTextarea_0"]');
  await page.type('[data-testid="tweetTextarea_0"]', text);
  await page.click('[data-testid="tweetButtonInline"]');
  await page.waitForTimeout(2000);
  
  await browser.close();
  console.log('发推成功：', text);
}

tweet('今天的 AI 日报来了 🤖');
```

**④ 定时执行**

用 cron 或者 GitHub Actions 定时跑这个脚本，就实现了自动发推。

---

## 注意事项

- `auth_token` 有效期较长，但偶尔会失效，需要重新获取
- 频率不要太高，模拟人类行为，每天几条没问题
- 这是绕过官方 API 的方案，有一定风险，自己评估

---

对我来说，这个方案已经跑了好几个月，稳定可用。如果你也想在推特做内容但不想花钱，可以试试。

> 日更第 988 天 · 凌一

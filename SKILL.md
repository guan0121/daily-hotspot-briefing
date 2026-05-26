---
name: daily-hotspot-briefing
description: 每日全平台爆款热点简报生成器。抓取微博、抖音、百度、知乎、头条、小红书、36氪、微信8大平台热搜，生成野新派酒红金色品牌风格PNG长图。触发词：热点简报、每日热点、爆款热点、社媒热点、热搜汇总。
agent_created: true
---

# 每日全平台爆款热点简报

每日自动抓取 8 大主流平台热搜 Top 10，生成野新派品牌风格 PNG 长图，可直接转发给学员做短视频选题参考。

## 数据源

| 平台 | 抓取方式 | URL |
|------|----------|-----|
| 微博热搜 | WebFetch | https://tophub.today/hot |
| 抖音热点 | WebFetch | https://www.abangshou.com/tools/douyin.html |
| 百度热搜 | WebFetch | https://www.xpaihang.com/ |
| 知乎热榜 | WebFetch | https://tophub.today/n/mproPpoq6O |
| 头条热榜 | WebFetch | https://www.abangshou.com/tools/toutiao.html |
| 小红书趋势 | WebSearch | 搜索"小红书热门话题 今天" |
| 36氪热榜 | WebFetch | https://www.xpaihang.com/ |
| 微信热文 | WebFetch | https://www.xpaihang.com/（搜狗热搜） |

## 执行流程

### Step 1: 抓取数据

每个平台搜 Top 10（WebSearch + WebFetch），如果某个平台数据不完整，标注说明，不要放弃。

获取当前日期：`date "+%Y-%m-%d"`

### Step 2: 生成 HTML

文件路径：`{{workspace}}/每日热点简报/热点简报_{{date}}.html`

**设计规范：野新派品牌风格**

配色：
- 页面背景: `#faf8f6`
- 卡片背景: `#fff`，边框 `#e8e3dc`
- 主色调深酒红: `#7A1A2B`（Header、热度标签、重点文字）
- 金色点缀: `#C8A45C`（排名徽章 Top1-3、装饰线条、星级）
- 正文: `#2d2d2d`（标题）、`#5c5c5c`（正文）、`#9c8c7c`（辅助）

body: `width: 750px; margin: 0 auto;`
字体: `system-ui, -apple-system, "PingFang SC", "Microsoft YaHei", serif`

**页面结构：**

1. **HEADER**: 深酒红渐变 `#7A1A2B → #5c1320`，标题"每日全平台爆款热点简报"白色大字 + 日期 + 金色菱形装饰
2. **STATS BAR**: 8 个白色小卡片 `flex-wrap` 两行 4+4，数据量酒红色，**图标用内联 SVG（禁止 emoji）**：
   - 微博: 红圆 `#E6162D` + 眼睛弧线
   - 抖音: 深色圆角 `#111` + 音符路径
   - 百度: 蓝圆 `#2932E1` + 爪印圆点
   - 知乎: 蓝 `#0066FF` + "知"字
   - 头条: 红 `#E1342C` + 旗帜路径
   - 小红书: 红 `#FE2C55` + 书本图标
   - 36氪: 蓝 `#1E80FF` + "36氪"
   - 微信: 绿 `#07C160` + 对话气泡路径
3. **8 个平台卡片**（白底细边框，圆角 12px，微阴影）:
   - 标题栏: 浅酒红渐变底 + SVG 图标 + 平台名 + 右上角 badge "热度"
   - 每条热点: 排名圆形徽章（Top1 金/C8A45C，Top2 浅金/d4b87a，Top3 淡金/dcc89e，其余白底金边）+ 话题名 + 热度值酒红圆角标签
   - 每条热点下方 1-2 句总结（灰字 `#8c8070`，左缩进 40px）
4. **关键词标签云**: 白色底 + 酒红边框圆角标签
5. **FOOTER**: 数据来源 + WorkBuddy 生成信息

**所有 CSS 内联，无外部依赖，中文无乱码。**

### Step 3: Playwright 截图

```bash
NODE_OPTIONS="" NODE_PATH="/usr/local/lib/node_modules:{{npm_global}}" /usr/local/bin/node -e "
const { chromium } = require('playwright');
(async () => {
  const browser = await chromium.launch({ headless: true });
  const page = await browser.newPage();
  await page.setViewportSize({ width: 750, height: 800 });
  await page.goto('file://{{html_path}}', { waitUntil: 'networkidle' });
  await page.waitForTimeout(1500);
  await page.screenshot({ path: '{{png_path}}', fullPage: true });
  await browser.close();
})();
"
```

### Step 4: 输出

用 `deliver_attachments` 只发送 PNG 长图。

## 注意事项

- 必须使用系统 Node（`/usr/local/bin/node`），不能用 managed Node
- 必须设置 `NODE_OPTIONS=""` 清除环境变量干扰
- HTML 中所有平台 SVG 图标直接内联，不引用外部文件
- 小红书实时热榜难以直接抓取，可用平台趋势数据替代
- 微信热文数据源为搜狗热搜（搜狗与微信同属腾讯生态）
- 仅输出热点数据，不包含选题建议（留给学员自行发挥）
- macOS 系统 Playwright Chromium 路径: `~/Library/Caches/ms-playwright/`

## 依赖

- Playwright (全局安装): `npm install -g playwright && npx playwright install chromium`
- 系统 Node.js >= 18

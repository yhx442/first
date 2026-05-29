# 麻将小账本 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 构建一个单文件 PWA 麻将记账应用，妈妈每日记录输赢，自动生成儿子视角的暖心俏皮话。

**Architecture:** 单文件 `mahjong.html`，内联 CSS + JS，localStorage 持久化，PWA manifest + Service Worker 支持离线安装。仪表盘首页 + 历史列表 + 导出三个视图，底部 Tab 切换。记账面板为底部 Sheet 弹出层。

**Tech Stack:** 纯 HTML/CSS/JS，零依赖，无构建步骤

---

## 文件结构

| 文件 | 职责 |
|---|---|
| `mahjong.html` | 全部 HTML 结构 + CSS 样式 + JS 逻辑 + PWA manifest + Service Worker（内联） |

这是唯一的文件。所有内容内联在一个 HTML 文件中，保证零依赖和直接可用。

---

### Task 1: HTML 骨架 + CSS 基础

**文件:**
- 创建: `d:/workspace/mahjong.html`

- [ ] **Step 1: 创建 HTML 骨架**

写入包含所有语义结构的 HTML，三页（首页/历史/导出）用 div 切换，记账面板为底部 Sheet：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<meta name="theme-color" content="#BE185D">
<title>🏮 麻将小账本</title>
<style>
/* 基础重置 + 变量 */
:root {
  --rose-dark: #BE185D;
  --rose: #DB2777;
  --rose-light: #FCE7F3;
  --rose-bg: #FFF5F5;
  --win: #22C55E;
  --lose: #EF4444;
  --gray: #9CA3AF;
  --text: #1F2937;
  --text-secondary: #6B7280;
}
* { margin:0; padding:0; box-sizing:border-box; }
body {
  font-family: -apple-system, BlinkMacSystemFont, "PingFang SC", "Microsoft YaHei", sans-serif;
  background: var(--rose-bg);
  color: var(--text);
  max-width: 420px;
  margin: 0 auto;
  min-height: 100vh;
  min-height: 100dvh;
  display: flex;
  flex-direction: column;
  -webkit-tap-highlight-color: transparent;
  user-select: none;
  -webkit-user-select: none;
}

/* 顶栏 */
.header {
  background: linear-gradient(135deg, var(--rose-dark), var(--rose));
  color: #fff;
  padding: 16px 20px;
  text-align: center;
  flex-shrink: 0;
}
.header h1 { font-size: 18px; font-weight: 600; }
.header .date { font-size: 13px; opacity: 0.85; margin-top: 4px; }

/* 主内容区 */
.main { flex:1; padding: 16px 16px 80px; overflow-y: auto; }

/* 大数字区 */
.big-number {
  text-align: center;
  padding: 24px 0 8px;
}
.big-number .amount {
  font-size: 72px;
  font-weight: 900;
  line-height: 1;
}
.big-number .amount.win { color: var(--win); }
.big-number .amount.lose { color: var(--lose); }
.big-number .amount.empty { color: var(--gray); }
.big-number .subtitle {
  font-size: 14px;
  color: var(--text-secondary);
  margin-top: 8px;
}

/* 消息卡片 */
.message-card {
  background: linear-gradient(135deg, var(--rose-light), #FBCFE8);
  border-radius: 16px;
  padding: 20px;
  text-align: center;
  margin: 12px 0;
}
.message-card .emoji { font-size: 36px; margin-bottom: 8px; }
.message-card .msg-text {
  font-size: 15px;
  color: #831843;
  line-height: 1.8;
  font-weight: 500;
}

/* 统计栏 */
.stats-row {
  display: flex;
  gap: 10px;
  margin: 12px 0 20px;
}
.stat-item {
  flex: 1;
  background: #fff;
  border-radius: 12px;
  padding: 14px 8px;
  text-align: center;
  box-shadow: 0 1px 4px rgba(0,0,0,0.04);
}
.stat-item .stat-label { font-size: 11px; color: var(--gray); margin-bottom: 4px; }
.stat-item .stat-value { font-size: 20px; font-weight: 700; }
.stat-item .stat-value.win { color: var(--win); }
.stat-item .stat-value.lose { color: var(--lose); }
.stat-item .stat-value.neutral { color: var(--text); }

/* 记录按钮 */
.btn-record {
  display: block;
  width: 100%;
  background: var(--rose);
  color: #fff;
  border: none;
  border-radius: 24px;
  padding: 14px;
  font-size: 16px;
  font-weight: 600;
  box-shadow: 0 4px 16px rgba(219,39,119,0.3);
  cursor: pointer;
}
.btn-record:active { transform: scale(0.98); }

/* 底部 Tab */
.tab-bar {
  position: fixed;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 100%;
  max-width: 420px;
  background: #fff;
  border-top: 1px solid #f0e0e0;
  display: flex;
  padding: 6px 0;
  padding-bottom: max(6px, env(safe-area-inset-bottom));
}
.tab-item {
  flex: 1;
  text-align: center;
  padding: 8px;
  font-size: 12px;
  color: var(--gray);
  cursor: pointer;
  border: none;
  background: none;
}
.tab-item.active { color: var(--rose); font-weight: 600; }

/* 历史列表 */
.history-list { display: flex; flex-direction: column; gap: 8px; }
.history-item {
  background: #fff;
  border-radius: 12px;
  padding: 14px 16px;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-shadow: 0 1px 3px rgba(0,0,0,0.04);
}
.history-item .hi-date { font-size: 14px; }
.history-item .hi-amount { font-size: 16px; font-weight: 700; }
.history-item .hi-amount.win { color: var(--win); }
.history-item .hi-amount.lose { color: var(--lose); }
.history-detail {
  display: none;
  padding: 12px 16px;
  background: var(--rose-light);
  border-radius: 0 0 12px 12px;
  margin-top: -4px;
  font-size: 13px;
  color: #831843;
  line-height: 1.6;
}
.history-detail.open { display: block; }

/* 空状态 */
.empty-state {
  text-align: center;
  padding: 60px 20px;
  color: var(--gray);
}
.empty-state .empty-icon { font-size: 48px; margin-bottom: 12px; }
.empty-state p { font-size: 15px; }

/* 导出区 */
.export-preview {
  background: #fff;
  border-radius: 12px;
  padding: 16px;
  font-size: 13px;
  line-height: 1.8;
  white-space: pre-wrap;
  color: var(--text);
  max-height: 300px;
  overflow-y: auto;
  margin-bottom: 16px;
}
.btn-secondary {
  display: block;
  width: 100%;
  background: #fff;
  color: var(--rose);
  border: 2px solid var(--rose);
  border-radius: 24px;
  padding: 12px;
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  margin-bottom: 10px;
  text-align: center;
}
.btn-secondary:active { background: var(--rose-light); }

/* Sheet 弹出面板 */
.sheet-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.4);
  z-index: 100;
  display: none;
  justify-content: center;
  align-items: flex-end;
}
.sheet-overlay.open { display: flex; }
.sheet {
  background: #fff;
  border-radius: 20px 20px 0 0;
  padding: 20px 16px 32px;
  width: 100%;
  max-width: 420px;
  animation: slideUp 0.25s ease;
}
@keyframes slideUp { from { transform: translateY(100%); } to { transform: translateY(0); } }
.sheet h3 { text-align: center; font-size: 16px; margin-bottom: 16px; }

/* 赢/输选择按钮 */
.choice-row { display: flex; gap: 12px; margin-bottom: 16px; }
.choice-btn {
  flex: 1;
  padding: 14px;
  border-radius: 14px;
  border: 2px solid #E5E7EB;
  background: #F9FAFB;
  font-size: 16px;
  cursor: pointer;
  text-align: center;
}
.choice-btn.selected-win { border-color: var(--win); background: #F0FDF4; color: var(--win); font-weight: 700; }
.choice-btn.selected-lose { border-color: var(--lose); background: #FEF2F2; color: var(--lose); font-weight: 700; }

/* 数字键盘 */
.num-pad { display: grid; grid-template-columns: repeat(3, 1fr); gap: 8px; margin-bottom: 16px; }
.num-key {
  padding: 14px;
  font-size: 20px;
  border-radius: 12px;
  border: 1px solid #E5E7EB;
  background: #F9FAFB;
  cursor: pointer;
  text-align: center;
}
.num-key:active { background: #E5E7EB; }
.num-key.action { background: #F3F4F6; color: var(--text-secondary); font-size: 14px; }
.amount-display { text-align: center; font-size: 28px; font-weight: 700; margin-bottom: 12px; padding: 8px; }

.btn-save {
  display: block;
  width: 100%;
  background: var(--rose);
  color: #fff;
  border: none;
  border-radius: 14px;
  padding: 14px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
}
.btn-save:disabled { opacity: 0.4; pointer-events: none; }
.btn-save:active { transform: scale(0.98); }

/* 通用 */
.page { display: none; }
.page.active { display: block; }

/* 历史展开包裹 */
.history-item-wrap { margin-bottom: 4px; }
</style>
</head>
<body>

<!-- 顶栏 -->
<div class="header">
  <h1>🏮 麻将小账本</h1>
  <div class="date" id="headerDate"></div>
</div>

<!-- 主内容 -->
<div class="main">
  <!-- 首页 -->
  <div class="page active" id="pageHome">
    <div class="big-number">
      <div class="amount empty" id="todayAmount">—</div>
      <div class="subtitle" id="todaySubtitle"></div>
    </div>
    <div class="message-card" id="messageCard" style="display:none">
      <div class="emoji" id="msgEmoji"></div>
      <div class="msg-text" id="msgText"></div>
    </div>
    <div class="stats-row">
      <div class="stat-item">
        <div class="stat-label">本周</div>
        <div class="stat-value neutral" id="statWeek">—</div>
      </div>
      <div class="stat-item">
        <div class="stat-label">本月</div>
        <div class="stat-value neutral" id="statMonth">—</div>
      </div>
      <div class="stat-item">
        <div class="stat-label">胜率</div>
        <div class="stat-value neutral" id="statRate">—</div>
      </div>
    </div>
    <button class="btn-record" id="btnRecord">✏️ 记录今天</button>
  </div>

  <!-- 历史页 -->
  <div class="page" id="pageHistory">
    <div class="history-list" id="historyList"></div>
    <div class="empty-state" id="historyEmpty">
      <div class="empty-icon">📋</div>
      <p>还没有记录哦~</p>
    </div>
  </div>

  <!-- 导出页 -->
  <div class="page" id="pageExport">
    <div class="export-preview" id="exportPreview">暂无记录</div>
    <button class="btn-secondary" id="btnCopy">📋 复制文本</button>
    <button class="btn-secondary" id="btnDownload">💾 下载文件</button>
  </div>
</div>

<!-- 底部 Tab -->
<div class="tab-bar">
  <button class="tab-item active" data-tab="home">🏠 首页</button>
  <button class="tab-item" data-tab="history">📋 历史</button>
  <button class="tab-item" data-tab="export">📤 导出</button>
</div>

<!-- 记账 Sheet -->
<div class="sheet-overlay" id="sheetOverlay">
  <div class="sheet">
    <h3>记录今天的战况</h3>
    <div class="choice-row">
      <button class="choice-btn" id="choiceWin">🀄 赢了</button>
      <button class="choice-btn" id="choiceLose">🀄 输了</button>
    </div>
    <div class="amount-display" id="amountDisplay">¥0</div>
    <div class="num-pad" id="numPad"></div>
    <button class="btn-save" id="btnSave" disabled>保存</button>
  </div>
</div>

<script>
</script>
</body>
</html>
```

- [ ] **Step 2: 在移动端浏览器中打开验证**

打开 `d:/workspace/mahjong.html`，确认骨架结构正常：顶栏渐变、三个 Tab 可点击切换、记账 Sheet 不显示。

- [ ] **Step 3: 提交**

```bash
git add d:/workspace/mahjong.html
git commit -m "feat: add mahjong app HTML skeleton with rose theme CSS"
```

---

### Task 2: 数据层

**文件:**
- 修改: `d:/workspace/mahjong.html` — 在 `<script>` 中添加数据管理函数

- [ ] **Step 1: 实现 localStorage 读写和统计**

在 `<script>` 标签内添加以下代码：

```javascript
const STORAGE_KEY = 'mahjong_records';

function loadRecords() {
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    return raw ? JSON.parse(raw).records || [] : [];
  } catch (e) {
    return [];
  }
}

function saveRecords(records) {
  localStorage.setItem(STORAGE_KEY, JSON.stringify({ records }));
}

function getTodayRecord() {
  const today = new Date().toISOString().slice(0, 10);
  return loadRecords().find(r => r.date === today) || null;
}

function saveTodayRecord(amount, message) {
  const today = new Date().toISOString().slice(0, 10);
  const records = loadRecords();
  const idx = records.findIndex(r => r.date === today);
  const entry = { date: today, amount, message };
  if (idx >= 0) {
    records[idx] = entry;
  } else {
    records.push(entry);
  }
  records.sort((a, b) => b.date.localeCompare(a.date));
  saveRecords(records);
  return entry;
}

function getStats() {
  const records = loadRecords();
  const now = new Date();
  const todayStr = now.toISOString().slice(0, 10);

  // 本周 (周一 = 起始)
  const dayOfWeek = now.getDay();
  const mondayOffset = dayOfWeek === 0 ? 6 : dayOfWeek - 1;
  const monday = new Date(now);
  monday.setDate(now.getDate() - mondayOffset);
  const weekStart = monday.toISOString().slice(0, 10);

  // 本月
  const monthStart = todayStr.slice(0, 7) + '-01';

  let weekTotal = 0, monthTotal = 0, winCount = 0, totalCount = 0;

  for (const r of records) {
    if (r.date >= weekStart) weekTotal += r.amount;
    if (r.date >= monthStart) monthTotal += r.amount;
    totalCount++;
    if (r.amount > 0) winCount++;
  }

  const winRate = totalCount > 0 ? Math.round((winCount / totalCount) * 100) : null;

  return { weekTotal, monthTotal, winRate };
}

function getTodayStr() {
  const now = new Date();
  const days = ['日','一','二','三','四','五','六'];
  const y = now.getFullYear();
  const m = now.getMonth() + 1;
  const d = now.getDate();
  const w = days[now.getDay()];
  return `${y}年${m}月${d}日 周${w}`;
}
```

- [ ] **Step 2: 提交**

```bash
git add d:/workspace/mahjong.html
git commit -m "feat: add localStorage data layer and stats functions"
```

---

### Task 3: 消息模板引擎

**文件:**
- 修改: `d:/workspace/mahjong.html` — 在 `<script>` 中添加消息模板

- [ ] **Step 1: 实现消息模板库和生成函数**

在数据层代码之后添加：

```javascript
const MESSAGES = {
  bigWin: [
    '妈，今天赢了{amount}块！比我写代码赚得还稳，要不咱家改行搓麻将吧😂 开心就好，别太累~',
    '天哪妈！{amount}块！你是赌神附体了吧🧧 今天晚上加菜，必须庆祝一下！',
    '妈你今天太猛了，赢了{amount}！我在公司写一天代码都不如你搓一圈麻将💪 妈最棒~',
    '赢了{amount}！妈你这手气，不去参加雀神大赛真是屈才了😆 记住早点歇着哦~',
    '{amount}块进账！妈你今天一定是穿了幸运袜🧦 继续保持，但要少熬夜打牌哈！',
    '妈你今天手气炸了，赢了{amount}！看得我都想去拜你为师了🀄 不过别太兴奋，早点休息~',
    '恭喜发财妈！{amount}块到手！这战绩比A股还靠谱📈 明天见好就收哦~',
    '妈你今天赢了{amount}！你打麻将的直觉比我写代码的逻辑还准😂 为妈骄傲！',
    '赢了{amount}块！妈今天气场两米八👑 记得喝杯温水，打牌也别忘了照顾好自己~',
    '妈，{amount}块！你这个胜率让我怀疑你是不是偷偷上麻将培训班了😏 开开心心最重要~',
  ],
  smallWin: [
    '妈，今天赢了{amount}块！小赢也是赢，细水长流嘛~ 开心就好💕',
    '赢{amount}块也不赖！蚂蚁腿也是肉，积少成多🐜 妈今天心情一定不错吧~',
    '{amount}块到手！妈你今天稳扎稳打，有大将风范👏 继续保持这个节奏！',
    '妈赢了{amount}！虽然不多但胜在稳定，比我买奶茶强多了😄 记得喝点热水~',
    '小赢{amount}块，开心就好！麻将嘛，图个乐子，赢了就是赚到✨ 妈早点休息~',
    '今天赚了{amount}！妈你的心态真好，稳中求胜🎯 明天继续加油！',
    '{amount}块入账，妈今天的奶茶钱有了🧋 下次争取赢个火锅回来！',
    '妈赢了{amount}，这个节奏很舒服啊~ 不贪大，稳着来才是高手👍 为你开心！',
    '收{amount}块开心钱！妈今天肯定笑得很灿烂吧😊 看到妈开心我也开心~',
    '赢{amount}！妈你这是可持续发展战略，每天赢一点，生活美滋滋🌞',
  ],
  breakEven: [
    '妈，今天打平了！不输就是赚，心态稳住💪 明天再战！',
    '平局也是一种胜利！妈今天牌运平平但心态满分👌 早点睡，明天好运就来了~',
    '打了个平手，说明妈今天稳如泰山⛰️ 不输不赢最养生，继续保持！',
    '妈，今天不输不赢，等于免费玩了一下午麻将，纯赚娱乐😄 开心就好~',
    '平了也挺好！妈今天的乐趣是无价的，钱不钱的没关系✨ 明天继续快乐搓麻！',
    '打个平手说明妈今天状态平衡，不好不坏正是生活常态🌿 舒舒服服的最佳状态~',
    '不输不赢，妈今天是"和平使者"🕊️ 牌桌上气场稳重，儿子给你点赞！',
    '妈，平局说明你在保存实力，明天肯定爆发🔥 记得泡个脚再睡~',
    '今天打平，妈你的稳健打法让人安心😌 不贪不躁，这才是高手风范！',
    '平手日也是好日子！妈今天肯定玩得很开心吧，那就够了🎈 钱不重要，开心才重要~',
  ],
  smallLose: [
    '妈，输了{amount}没事~ 这叫战略性休整，明天肯定翻盘！记得喝点热水早点休息❤️',
    '输了{amount}小意思！运气是流动的，今天给她明天就是你🔄 妈妈早点睡，养足精神~',
    '妈别放心上，就输了{amount}~ 麻将嘛，有输有赢才有意思。早点歇着，身体最重要💝',
    '今天小亏{amount}，但妈的心情可不能亏😊 输赢平常心，儿子永远觉得你最棒！',
    '{amount}块就当给牌友买奶茶了🧋 明天运气转回来，肯定赢个大的！妈加油~',
    '妈，输了{amount}块而已，比起你的快乐来不值一提💕 泡杯热茶，看看电视，放松一下~',
    '小输{amount}不碍事！妈今天肯定是故意放水的，明天拿出真本事来😉 早点休息哦~',
    '妈，{amount}块就当今天买了个开心，重要的是跟朋友在一起的热闹🥰 明天继续！',
    '输了{amount}？没关系妈，这叫"攒人品"，明天运气加倍反弹🚀 记得睡前泡泡脚~',
    '妈，小输{amount}不叫输，叫"战略性投资"📊 明天肯定翻倍赚回来！开开心心最重要❤️',
  ],
  bigLose: [
    '妈，今天输了{amount}别往心里去！麻将就这样，有时运气不好很正常。早点休息，明天又是新的一天，儿子永远支持你💪❤️',
    '妈，输了{amount}没什么大不了的！你开心才是最重要的，钱可以再赚，身体不能垮。泡杯热茶，今晚早点睡🫂',
    '今天输{amount}就当破财免灾！妈妈的身体健康和好心情比什么都重要✨ 明天记得少打两圈，别太累~',
    '妈，{amount}块买不了你的开心！输了就输了，儿子在呢💝 今晚别看牌了，看看电视放松一下~',
    '妈别难过，今天就是运气不好而已。{amount}块我们下次赢回来！你永远是我最棒的妈妈👩‍👦 早点歇着哦~',
    '输了{amount}不是你的问题，是今天的牌不好！妈的手艺我最清楚🀄 早点休息，养足精神明天再战！',
    '妈，{amount}只是一个数字。你的笑容才是无价的💎 洗个热水澡，喝杯牛奶，好好睡一觉~',
    '今天手气不好输{amount}，但妈妈的心气不能输！明天太阳照常升起🌅 儿子永远是你的后盾！',
    '妈，输了{amount}没事的。人生就像麻将，起起落落很正常🎢 重要的是妈妈每天都开开心心的！',
    '{amount}块换不来妈妈的快乐。早点休息妈，明天又是全新的一天，运气也会全新的✨ 爱你~',
  ],
};

function pickMessage(amount) {
  let tier;
  if (amount >= 200) tier = 'bigWin';
  else if (amount > 0) tier = 'smallWin';
  else if (amount === 0) tier = 'breakEven';
  else if (amount >= -199) tier = 'smallLose';
  else tier = 'bigLose';

  const pool = MESSAGES[tier];
  const idx = Math.floor(Math.random() * pool.length);
  return pool[idx].replace('{amount}', String(Math.abs(amount)));
}

function getEmoji(amount) {
  if (amount >= 200) return '🥳';
  if (amount > 0) return '😊';
  if (amount === 0) return '😌';
  if (amount >= -199) return '🤗';
  return '💪';
}
```

- [ ] **Step 2: 提交**

```bash
git add d:/workspace/mahjong.html
git commit -m "feat: add message template engine with 5 tiers"
```

---

### Task 4: 首页渲染 + Tab 切换

**文件:**
- 修改: `d:/workspace/mahjong.html` — 在 `<script>` 中添加渲染和 Tab 切换逻辑

- [ ] **Step 1: 实现首页渲染和 Tab 切换**

在脚本末尾添加：

```javascript
// ====== 渲染 ======

function renderHome() {
  const record = getTodayRecord();
  const todayEl = document.getElementById('todayAmount');
  const subtitleEl = document.getElementById('todaySubtitle');
  const msgCard = document.getElementById('messageCard');
  const msgEmoji = document.getElementById('msgEmoji');
  const msgText = document.getElementById('msgText');

  if (record) {
    const amt = record.amount;
    todayEl.textContent = amt >= 0 ? `+${amt}` : String(amt);
    todayEl.className = 'amount ' + (amt > 0 ? 'win' : amt < 0 ? 'lose' : 'empty');

    const desc = amt > 0 ? `今日净胜 ¥${amt} · 盈利日 ✨`
               : amt < 0 ? `今日净输 ¥${Math.abs(amt)} · 休整日 🌙`
               : '今日打平 · 养生局 🍵';
    subtitleEl.textContent = desc;

    msgCard.style.display = 'block';
    msgEmoji.textContent = getEmoji(amt);
    msgText.textContent = record.message;
  } else {
    todayEl.textContent = '—';
    todayEl.className = 'amount empty';
    subtitleEl.textContent = '今天还没记录哦~';
    msgCard.style.display = 'none';
  }

  // 统计
  const stats = getStats();
  setStat('statWeek', stats.weekTotal);
  setStat('statMonth', stats.monthTotal);
  setStat('statRate', stats.winRate !== null ? stats.winRate + '%' : null);
}

function setStat(id, value) {
  const el = document.getElementById(id);
  if (value === null || value === undefined) {
    el.textContent = '—';
    el.className = 'stat-value neutral';
    return;
  }
  if (typeof value === 'string') {
    el.textContent = value;
    el.className = 'stat-value neutral';
    return;
  }
  el.textContent = value >= 0 ? `+${value}` : String(value);
  el.className = 'stat-value ' + (value > 0 ? 'win' : value < 0 ? 'lose' : 'neutral');
}

// ====== Tab 切换 ======

function switchTab(tab) {
  document.querySelectorAll('.tab-item').forEach(t => t.classList.remove('active'));
  document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));

  document.querySelector(`.tab-item[data-tab="${tab}"]`).classList.add('active');
  document.getElementById('page' + tab.charAt(0).toUpperCase() + tab.slice(1)).classList.add('active');

  if (tab === 'home') renderHome();
  else if (tab === 'history') renderHistory();
  else if (tab === 'export') renderExport();
}

document.querySelectorAll('.tab-item').forEach(btn => {
  btn.addEventListener('click', () => switchTab(btn.dataset.tab));
});

// ====== 初始化 ======

document.getElementById('headerDate').textContent = getTodayStr();
renderHome();
```

- [ ] **Step 2: 在浏览器中打开 `mahjong.html`，验证首页显示**。由于没有记录，大数字应显示灰色「—」，统计栏显示「—」，消息卡片隐藏。点击底部 Tab 可切换页面。

- [ ] **Step 3: 提交**

```bash
git add d:/workspace/mahjong.html
git commit -m "feat: add home page rendering and tab navigation"
```

---

### Task 5: 记账面板（Sheet + 数字键盘）

**文件:**
- 修改: `d:/workspace/mahjong.html` — 在 `<script>` 中添加 Sheet 交互逻辑

- [ ] **Step 1: 实现记账面板完整交互**

在脚本末尾添加：

```javascript
// ====== 记账面板 ======

let sheetState = {
  open: false,
  win: true,    // true=赢, false=输
  amount: '',   // 字符串，用于数字键盘拼接
};

const overlay = document.getElementById('sheetOverlay');
const choiceWin = document.getElementById('choiceWin');
const choiceLose = document.getElementById('choiceLose');
const amountDisplay = document.getElementById('amountDisplay');
const numPad = document.getElementById('numPad');
const btnSave = document.getElementById('btnSave');

// 打开面板
document.getElementById('btnRecord').addEventListener('click', () => {
  sheetState.win = true;
  sheetState.amount = '';
  updateSheetUI();
  overlay.classList.add('open');
});

// 点击遮罩关闭
overlay.addEventListener('click', (e) => {
  if (e.target === overlay) overlay.classList.remove('open');
});

// 赢/输选择
choiceWin.addEventListener('click', () => { sheetState.win = true; updateSheetUI(); });
choiceLose.addEventListener('click', () => { sheetState.win = false; updateSheetUI(); });

function updateSheetUI() {
  choiceWin.className = 'choice-btn' + (sheetState.win ? ' selected-win' : '');
  choiceLose.className = 'choice-btn' + (!sheetState.win ? ' selected-lose' : '');
  const prefix = sheetState.win ? '' : '-';
  amountDisplay.textContent = '¥' + prefix + (sheetState.amount || '0');
  btnSave.disabled = !sheetState.amount || sheetState.amount === '0';
}

// 生成数字键盘
function buildNumPad() {
  const keys = ['1','2','3','4','5','6','7','8','9','清','0','⌫'];
  keys.forEach(key => {
    const btn = document.createElement('button');
    btn.className = 'num-key' + (key === '清' || key === '⌫' ? ' action' : '');
    btn.textContent = key;
    btn.addEventListener('click', () => handleNumKey(key));
    numPad.appendChild(btn);
  });
}

function handleNumKey(key) {
  if (key === '清') {
    sheetState.amount = '';
  } else if (key === '⌫') {
    sheetState.amount = sheetState.amount.slice(0, -1);
  } else {
    if (sheetState.amount.length >= 5) return; // 最多5位数
    sheetState.amount += key;
  }
  updateSheetUI();
}

// 保存
btnSave.addEventListener('click', () => {
  const rawAmount = parseInt(sheetState.amount, 10) || 0;
  const finalAmount = sheetState.win ? rawAmount : -rawAmount;
  const message = pickMessage(finalAmount);
  saveTodayRecord(finalAmount, message);
  overlay.classList.remove('open');
  renderHome();
});

buildNumPad();
```

- [ ] **Step 2: 打开浏览器验证**

  1. 点击「✏️ 记录今天」→ Sheet 弹出
  2. 切换赢/输 → 按钮高亮变色
  3. 点击数字键 → 金额显示更新
  4. 点「清」清空，「⌫」退格
  5. 金额为 0 或空时保存按钮置灰
  6. 输入有效金额 → 点保存 → Sheet 关闭 → 首页显示今日数据 + 消息

- [ ] **Step 3: 提交**

```bash
git add d:/workspace/mahjong.html
git commit -m "feat: add recording sheet with choice toggle and number pad"
```

---

### Task 6: 历史页

**文件:**
- 修改: `d:/workspace/mahjong.html` — 在 `<script>` 中添加历史渲染

- [ ] **Step 1: 实现历史列表渲染和展开**

在脚本末尾添加：

```javascript
// ====== 历史 ======

function renderHistory() {
  const records = loadRecords();
  const listEl = document.getElementById('historyList');
  const emptyEl = document.getElementById('historyEmpty');

  if (records.length === 0) {
    listEl.innerHTML = '';
    emptyEl.style.display = 'block';
    return;
  }

  emptyEl.style.display = 'none';
  listEl.innerHTML = records.map(r => {
    const isWin = r.amount > 0;
    const isLose = r.amount < 0;
    const cls = isWin ? 'win' : isLose ? 'lose' : '';
    const prefix = r.amount >= 0 ? '+' : '';
    const label = r.amount > 0 ? '赢' : r.amount < 0 ? '输' : '平';
    return `
      <div class="history-item-wrap">
        <div class="history-item" data-date="${r.date}" onclick="toggleHistoryDetail(this)">
          <div>
            <div class="hi-date">${formatDate(r.date)}</div>
            <div style="font-size:11px;color:#9CA3AF;margin-top:2px">${label} ¥${Math.abs(r.amount)}</div>
          </div>
          <div style="display:flex;align-items:center;gap:8px">
            <span class="hi-amount ${cls}">${prefix}${r.amount}</span>
            <span style="font-size:12px;color:#ccc;">▶</span>
          </div>
        </div>
        <div class="history-detail" id="detail-${r.date}">${r.message}</div>
      </div>
    `;
  }).join('');
}

function formatDate(dateStr) {
  const d = new Date(dateStr + 'T00:00:00');
  const days = ['日','一','二','三','四','五','六'];
  const m = d.getMonth() + 1;
  const day = d.getDate();
  const w = days[d.getDay()];
  return `${m}月${day}日 周${w}`;
}

function toggleHistoryDetail(el) {
  const date = el.dataset.date;
  const detail = document.getElementById('detail-' + date);
  const arrow = el.querySelector('span:last-child');
  if (detail) {
    const isOpen = detail.classList.contains('open');
    // 关闭所有
    document.querySelectorAll('.history-detail.open').forEach(d => d.classList.remove('open'));
    document.querySelectorAll('.history-item span:last-child').forEach(s => s.textContent = '▶');
    if (!isOpen) {
      detail.classList.add('open');
      arrow.textContent = '▼';
    }
  }
}
```

- [ ] **Step 2: 打开浏览器验证**

  - 切换到历史 Tab → 看到已保存的记录，按日期倒序排列
  - 点击一条记录 → 展开显示消息，箭头变为 ▼
  - 点击另一条 → 前一条关闭，新一条展开
  - 没有记录时显示「还没有记录哦~」

- [ ] **Step 3: 提交**

```bash
git add d:/workspace/mahjong.html
git commit -m "feat: add history page with expandable detail"
```

---

### Task 7: 导出页

**文件:**
- 修改: `d:/workspace/mahjong.html` — 在 `<script>` 中添加导出逻辑

- [ ] **Step 1: 实现导出预览、复制和下载**

在脚本末尾添加：

```javascript
// ====== 导出 ======

function renderExport() {
  const records = loadRecords();
  const previewEl = document.getElementById('exportPreview');

  if (records.length === 0) {
    previewEl.textContent = '还没有记录，无法导出~';
    document.getElementById('btnCopy').disabled = true;
    document.getElementById('btnDownload').disabled = true;
    return;
  }

  document.getElementById('btnCopy').disabled = false;
  document.getElementById('btnDownload').disabled = false;

  const lines = ['🏮 麻将小账本', ''];
  for (const r of records) {
    const label = r.amount > 0 ? '赢' : r.amount < 0 ? '输' : '平';
    const prefix = r.amount >= 0 ? '+' : '';
    lines.push(`${r.date}  ${label} ¥${Math.abs(r.amount)}  |  ${r.message}`);
  }
  const text = lines.join('\n');
  previewEl.textContent = text;
  previewEl._text = text;
}

document.getElementById('btnCopy').addEventListener('click', async () => {
  const text = document.getElementById('exportPreview')._text || '';
  if (!text) return;
  try {
    await navigator.clipboard.writeText(text);
    const btn = document.getElementById('btnCopy');
    const orig = btn.textContent;
    btn.textContent = '✅ 已复制！';
    btn.style.background = '#F0FDF4';
    btn.style.borderColor = '#22C55E';
    btn.style.color = '#22C55E';
    setTimeout(() => {
      btn.textContent = orig;
      btn.style.background = '#fff';
      btn.style.borderColor = 'var(--rose)';
      btn.style.color = 'var(--rose)';
    }, 2000);
  } catch (e) {
    alert('复制失败，请长按文本手动复制');
  }
});

document.getElementById('btnDownload').addEventListener('click', () => {
  const text = document.getElementById('exportPreview')._text || '';
  if (!text) return;
  const blob = new Blob([text], { type: 'text/plain;charset=utf-8' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = '麻将小账本_' + new Date().toISOString().slice(0, 10) + '.txt';
  a.click();
  URL.revokeObjectURL(url);
});
```

- [ ] **Step 2: 打开浏览器验证**

  - 切换到导出 Tab → 看到格式化的文本预览
  - 点击「复制文本」→ 按钮短暂变绿显示「✅ 已复制！」
  - 点击「下载文件」→ 下载 .txt 文件，内容正确

- [ ] **Step 3: 提交**

```bash
git add d:/workspace/mahjong.html
git commit -m "feat: add export page with copy and download"
```

---

### Task 8: PWA 配置（Manifest + Service Worker）

**文件:**
- 修改: `d:/workspace/mahjong.html` — 在 `<head>` 中添加 manifest，在 `<script>` 中注册 Service Worker

- [ ] **Step 1: 添加 manifest 和 Service Worker**

在 `</head>` 之前添加 manifest link：

```html
<link rel="manifest" href="data:application/json;base64,ewogICJuYW1lIjogIuertui9p+Wwj+i0puacrCIsCiAgInNob3J0X25hbWUiOiAi6bq56K6G5bCP6LSn5pysIiwKICAic3RhcnRfdXJsIjogIi4iLAogICJkaXNwbGF5IjogInN0YW5kYWxvbmUiLAogICJiYWNrZ3JvdW5kX2NvbG9yIjogIiNGRkY1RjUiLAogICJ0aGVtZV9jb2xvciI6ICIjQkUxODVEIiwKICAiaWNvbnMiOiBbewogICAgInNyYyI6ICJkYXRhOmltYWdlL3N2Zyt4bWwsPHN2ZyB4bWxucz0naHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmcnIHZpZXdCb3g9JzAgMCAxMDAgMTAwJz48cmVjdCB3aWR0aD0nMTAwJyBoZWlnaHQ9JzEwMCcgcng9JzIwJyBmaWxsPScjQkUxODVEJy8+PHRleHQgeD0nNTAnIHk9JzY4JyBmb250LXNpemU9JzUwJyB0ZXh0LWFuY2hvcj0nbWlkZGxlJyBmaWxsPSd3aGl0ZSc+8J+GuDwvdGV4dD48L3N2Zz4iLAogICAgInNpemVzIjogIjE5MngxOTIiLAogICAgInR5cGUiOiAiaW1hZ2Uvc3ZnK3htbCIKICB9XQp9">
```

在 `<script>` 标签开头（最顶部）添加 Service Worker 注册：

```javascript
// Service Worker 注册
if ('serviceWorker' in navigator) {
  const swCode = `
self.addEventListener('install', e => {
  e.waitUntil(
    caches.open('mahjong-v1').then(cache => {
      return cache.addAll(['./']);
    })
  );
  self.skipWaiting();
});
self.addEventListener('activate', e => {
  e.waitUntil(self.clients.claim());
});
self.addEventListener('fetch', e => {
  e.respondWith(
    caches.match(e.request).then(r => r || fetch(e.request))
  );
});
`;

  const swBlob = new Blob([swCode], { type: 'application/javascript' });
  const swUrl = URL.createObjectURL(swBlob);
  navigator.serviceWorker.register(swUrl).catch(() => {});
}
```

- [ ] **Step 2: 验证 PWA**

  用 Chrome 打开 → F12 → Application → Manifest 面板，确认 manifest 被识别。Service Worker 面板确认已注册。

- [ ] **Step 3: 提交**

```bash
git add d:/workspace/mahjong.html
git commit -m "feat: add PWA manifest and service worker for offline support"
```

---

### Task 9: 边界情况与打磨

**文件:**
- 修改: `d:/workspace/mahjong.html`

- [ ] **Step 1: 处理所有边界情况**

1. **同一天重复记录**：确认打开 Sheet 时，如果当日已有记录，预填上次的输赢和金额，方便修改。
2. **数字键盘限制**：金额最多 5 位数（99999），防止输入过长。
3. **导出时空状态**：导出页面按钮在无记录时置灰。

在 `btnRecord` 点击事件中，修改为预填当日已有记录：

```javascript
document.getElementById('btnRecord').addEventListener('click', () => {
  const existing = getTodayRecord();
  if (existing) {
    sheetState.win = existing.amount >= 0;
    sheetState.amount = String(Math.abs(existing.amount));
  } else {
    sheetState.win = true;
    sheetState.amount = '';
  }
  updateSheetUI();
  overlay.classList.add('open');
});
```

去掉 Head 中残留的内联样式/debug 注释，确保完整文件干净。

- [ ] **Step 2: 完整交互测试**

| 场景 | 预期 |
|---|---|
| 首次打开 | 首页显示「—」，统计全为「—」，消息卡片隐藏 |
| 记录赢 180 | 大数字绿色 +180，消息卡片显示，统计更新 |
| 再点记录，修改为输 50 | 覆盖同日旧记录，大数字红色 -50，消息更新 |
| 切历史页 | 显示按日期倒序列表，展开看消息 |
| 切导出页 | 预览正确，复制成功，下载成功 |
| 输 300（大输） | 消息从 bigLose 池中选，主题是坚定支持 |
| 赢 500（大赢） | 消息显示 🥳 表情和夸张骄傲语气 |

- [ ] **Step 3: 提交**

```bash
git add d:/workspace/mahjong.html
git commit -m "fix: pre-fill existing record, edge case handling"
```

---

## 验证清单

完成所有任务后，逐项确认：

- [ ] 首页：大数字 + 副标题 + 消息卡片 + 统计三格 + 记录按钮
- [ ] 记账面板：弹出动画 + 赢/输切换 + 数字键盘 + 保存置灰逻辑 + 保存后刷新
- [ ] 同日覆盖：再次打开面板预填上次数据，保存后覆盖
- [ ] 历史页：倒序列表 + 展开/收起 + 唯一展开 + 空状态
- [ ] 导出页：文本预览 + 复制到剪贴板 + 下载 txt
- [ ] Tab 切换：三页平滑切换，每次切换刷新数据
- [ ] PWA：Chrome DevTools 确认 manifest + SW 正常
- [ ] 5 档消息：分别测试大赢/小赢/平/小输/大输，确认消息风格正确

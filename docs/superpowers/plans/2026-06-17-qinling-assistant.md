# 沁凉助手 HTML Demo Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [x]`) syntax for tracking.

**Goal:** Build a single-file HTML Demo of the 沁凉助手 hot flash management tool for the TRAE AI Creativity Competition, replicating all 11 feature modules from the WeChat mini-program.

**Architecture:** Single `index.html` file using Vue 3 CDN for reactive UI, TailwindCSS for styling, ECharts for data visualization, and localStorage for data persistence. Hash-based routing simulates the mini-program's tab navigation and sub-pages.

**Tech Stack:** Vue 3 (3.5.x CDN), TailwindCSS (3.x CDN), ECharts (5.x CDN), marked.js (CDN), localStorage

**Spec:** `docs/superpowers/specs/2026-06-17-qinling-assistant-design.md`

---

## File Structure

```
index.html              — Single-file application (all CSS + JS + templates inline)
libs/                   — Local copies of CDN libraries for offline use
  vue.global.prod.js
  tailwind.css
  echarts.min.js
  marked.min.js
README.md               — Project description for GitHub
```

---

## Phase 1: Foundation (Tasks 1-3)

### Task 1: Project Scaffold + Vue 3 App Shell

**Files:**
- Create: `index.html`

- [x] **Step 1: Create `index.html` with Vue 3, TailwindCSS, and basic app structure**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>沁凉助手 | 潮热盗汗AI智能管理工具</title>
  <script src="https://unpkg.com/vue@3/dist/vue.global.prod.js"></script>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://cdn.jsdelivr.net/npm/echarts@5/dist/echarts.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { font-family: -apple-system, 'PingFang SC', 'Microsoft YaHei', sans-serif; background: #f7fafc; }
    .app-container { max-width: 480px; margin: 0 auto; min-height: 100vh; background: #fff; position: relative; }
    .page-content { padding-bottom: 64px; min-height: 100vh; }
    /* Transitions */
    .page-enter-active, .page-leave-active { transition: opacity 0.2s, transform 0.2s; }
    .page-enter-from { opacity: 0; transform: translateX(20px); }
    .page-leave-to { opacity: 0; transform: translateX(-20px); }
    /* Bottom sheet */
    .sheet-enter-active, .sheet-leave-active { transition: transform 0.3s ease; }
    .sheet-enter-from, .sheet-leave-to { transform: translateY(100%); }
  </style>
</head>
<body>
  <div id="app">
    <div class="app-container">
      <router-view></router-view>
      <tab-bar></tab-bar>
    </div>
  </div>
  <script>
    const { createApp, ref, reactive, computed, watch, onMounted, onUnmounted, nextTick, defineComponent, h } = Vue;

    // ========== STORAGE ==========
    const STORAGE_KEYS = {
      symptoms: 'qinling_symptoms',
      profile: 'qinling_profile',
      constitution: 'qinling_constitution',
      scaleResults: 'qinling_scale_results',
    };

    function loadStorage(key, fallback) {
      try { return JSON.parse(localStorage.getItem(key)) || fallback; }
      catch { return fallback; }
    }
    function saveStorage(key, data) {
      localStorage.setItem(key, JSON.stringify(data));
    }

    // ========== GLOBAL STORE ==========
    const store = reactive({
      symptoms: loadStorage(STORAGE_KEYS.symptoms, []),
      profile: loadStorage(STORAGE_KEYS.profile, { name: '演示用户', gender: 'male', age: 65, condition: '前列腺癌ADT治疗中' }),
      constitution: loadStorage(STORAGE_KEYS.constitution, null),
      scaleResults: loadStorage(STORAGE_KEYS.scaleResults, []),
      currentTab: 'home',
    });

    watch(() => store.symptoms, (v) => saveStorage(STORAGE_KEYS.symptoms, v), { deep: true });
    watch(() => store.profile, (v) => saveStorage(STORAGE_KEYS.profile, v), { deep: true });

    // ========== TAB BAR COMPONENT ==========
    const TabBar = {
      template: `
        <div class="fixed bottom-0 left-1/2 -translate-x-1/2 w-full max-w-[480px] bg-white border-t border-gray-200 flex justify-around items-center h-14 z-50">
          <button v-for="tab in tabs" :key="tab.key"
            @click="$root.currentTab = tab.key"
            class="flex flex-col items-center gap-0.5 text-xs transition-colors"
            :class="$root.currentTab === tab.key ? 'text-blue-600' : 'text-gray-400'">
            <span class="text-xl">{{ tab.icon }}</span>
            <span>{{ tab.label }}</span>
          </button>
        </div>
      `,
      setup() {
        const tabs = [
          { key: 'home', icon: '🏠', label: '首页' },
          { key: 'diary', icon: '📝', label: '记录' },
          { key: 'education', icon: '📖', label: '科普' },
          { key: 'profile', icon: '👤', label: '我的' },
        ];
        return { tabs };
      }
    };

    // ========== ROUTER (Hash-based) ==========
    const routes = {
      '': 'home',
      'diary': 'diary',
      'diary-detail': 'diary-detail',
      'stats': 'stats',
      'education': 'education',
      'education-detail': 'education-detail',
      'auricular': 'auricular',
      'auricular-detail': 'auricular-detail',
      'constitution': 'constitution',
      'constitution-result': 'constitution-result',
      'scale': 'scale',
      'scale-result': 'scale-result',
      'vms-survey': 'vms-survey',
      'treatment': 'treatment',
      'profile': 'profile',
    };

    const currentPage = ref('home');
    const pageParams = reactive({});

    function navigate(page, params = {}) {
      Object.assign(pageParams, params);
      if (routes[page] === 'home' || page === '') {
        currentPage.value = 'home';
        store.currentTab = 'home';
      } else if (['diary', 'education', 'profile'].includes(page)) {
        currentPage.value = page;
        store.currentTab = page;
      } else {
        currentPage.value = page;
      }
    }

    function goBack() {
      currentPage.value = store.currentTab;
    }

    function handleHashChange() {
      const hash = location.hash.slice(2) || '';
      const [page, ...rest] = hash.split('/');
      if (routes[page]) {
        currentPage.value = routes[page];
        if (rest.length) pageParams.id = rest[0];
      }
    }

    window.addEventListener('hashchange', handleHashChange);

    // ========== APP ==========
    const app = createApp({
      setup() {
        return { currentPage, currentTab: store.currentTab };
      }
    });

    app.component('tab-bar', TabBar);

    // Register page components (will be added in subsequent tasks)
    // app.component('page-home', PageHome);
    // app.component('page-diary', PageDiary);
    // etc.

    app.component('router-view', {
      template: '<component :is="\'page-\' + $root.currentPage" />',
    });

    app.mount('#app');
  </script>
</body>
</html>
```

- [x] **Step 2: Verify in browser**

Open `index.html` in browser. Should see an empty page with bottom tab bar (4 tabs). Clicking tabs should highlight them.

- [x] **Step 3: Commit scaffold**

```bash
git init && git add index.html && git commit -m "feat: scaffold Vue 3 app shell with tab bar navigation"
```

---

### Task 2: Demo Data Generator

**Files:**
- Modify: `index.html` (add data generator in `<script>` section)

- [x] **Step 1: Add demo data generator function after the STORAGE section**

```javascript
// ========== DEMO DATA GENERATOR ==========
function generateDemoData() {
  if (store.symptoms.length > 0) return; // Don't overwrite existing data

  const triggers = ['咖啡因', '酒精', '辛辣食物', '高温环境', '压力', '剧烈运动', '情绪波动'];
  const accompanying = ['面部潮红', '心悸', '出汗', '失眠', '焦虑', '头痛'];
  const interventions = ['无', '腹式呼吸', '耳穴按压', '穴位按摩', '药物'];

  const symptoms = [];
  const now = new Date();

  for (let dayOffset = 29; dayOffset >= 0; dayOffset--) {
    const date = new Date(now);
    date.setDate(date.getDate() - dayOffset);

    // 3-6 episodes per day, with decreasing trend (simulating tool effectiveness)
    const dailyCount = Math.floor(Math.random() * 4) + 3 - Math.floor((29 - dayOffset) / 15);
    const count = Math.max(2, Math.min(6, dailyCount));

    for (let i = 0; i < count; i++) {
      // Weight towards afternoon (14-16) and night (2-4)
      let hour;
      const r = Math.random();
      if (r < 0.35) hour = 14 + Math.floor(Math.random() * 2); // 14-15
      else if (r < 0.6) hour = 2 + Math.floor(Math.random() * 2); // 2-3
      else if (r < 0.8) hour = 7 + Math.floor(Math.random() * 12); // 7-18
      else hour = 19 + Math.floor(Math.random() * 5); // 19-23

      const minute = Math.floor(Math.random() * 60);
      const time = new Date(date);
      time.setHours(hour, minute, 0, 0);

      // Severity weighted distribution
      const severityRand = Math.random();
      let severity;
      if (severityRand < 0.35) severity = 'mild';
      else if (severityRand < 0.75) severity = 'moderate';
      else if (severityRand < 0.95) severity = 'severe';
      else severity = 'extreme';

      // More triggers during high-severity episodes
      const triggerCount = severity === 'mild' ? 1 : Math.floor(Math.random() * 2) + 1;
      const episodeTriggers = [];
      const triggerPool = [...triggers];
      for (let t = 0; t < triggerCount; t++) {
        const idx = Math.floor(Math.random() * triggerPool.length);
        episodeTriggers.push(triggerPool.splice(idx, 1)[0]);
      }

      const accomCount = Math.floor(Math.random() * 2) + 1;
      const episodeAccom = [];
      const accomPool = [...accompanying];
      for (let a = 0; a < accomCount; a++) {
        const idx = Math.floor(Math.random() * accomPool.length);
        episodeAccom.push(accomPool.splice(idx, 1)[0]);
      }

      // Later days more likely to use interventions (simulating learning)
      const interventionChance = (29 - dayOffset) / 29;
      const intervention = Math.random() < interventionChance
        ? interventions[1 + Math.floor(Math.random() * (interventions.length - 1))]
        : '无';

      symptoms.push({
        id: crypto.randomUUID ? crypto.randomUUID() : 'id-' + Date.now() + '-' + Math.random().toString(36).slice(2),
        time: time.toISOString(),
        severity,
        duration: Math.floor(Math.random() * 15) + 1,
        triggers: episodeTriggers,
        accompanying: episodeAccom,
        intervention,
        note: '',
      });
    }
  }

  // Sort by time descending
  symptoms.sort((a, b) => new Date(b.time) - new Date(a.time));
  store.symptoms = symptoms;
}
```

- [x] **Step 2: Call generator on app mount**

In the `createApp` setup, add:
```javascript
onMounted(() => { generateDemoData(); });
```

- [x] **Step 3: Verify demo data**

Open browser console, check `JSON.parse(localStorage.getItem('qinling_symptoms')).length` — should be ~100-180 records.

- [x] **Step 4: Commit**

```bash
git add index.html && git commit -m "feat: add 30-day demo data generator for hot flash symptoms"
```

---

### Task 3: Page Navigation System

**Files:**
- Modify: `index.html` (add placeholder page components + improve router)

- [x] **Step 1: Add placeholder page components**

Add these minimal page components before the `app.mount('#app')` line:

```javascript
// ========== PAGE COMPONENTS (Placeholders) ==========
function createPageComponent(name, title, content) {
  return {
    template: `
      <div class="page-content">
        <div class="bg-gradient-to-r from-blue-800 to-blue-500 text-white p-4 flex items-center gap-3">
          <button @click="$root.goBack && $root.goBack()" class="text-white text-xl" v-if="showBack">←</button>
          <h1 class="text-lg font-bold">${title}</h1>
        </div>
        <div class="p-4">${content}</div>
      </div>
    `,
    setup() { return { showBack: !['home','diary','education','profile'].includes(name) }; }
  };
}

app.component('page-home', createPageComponent('home', '沁凉助手', '<p>Loading...</p>'));
app.component('page-diary', createPageComponent('diary', '症状记录', '<p>Loading...</p>'));
app.component('page-diary-detail', createPageComponent('diary-detail', '记录详情', '<p>Loading...</p>'));
app.component('page-stats', createPageComponent('stats', '数据统计', '<p>Loading...</p>'));
app.component('page-education', createPageComponent('education', '健康科普', '<p>Loading...</p>'));
app.component('page-education-detail', createPageComponent('education-detail', '文章详情', '<p>Loading...</p>'));
app.component('page-auricular', createPageComponent('auricular', '耳穴指南', '<p>Loading...</p>'));
app.component('page-auricular-detail', createPageComponent('auricular-detail', '穴位详情', '<p>Loading...</p>'));
app.component('page-constitution', createPageComponent('constitution', '体质评估', '<p>Loading...</p>'));
app.component('page-constitution-result', createPageComponent('constitution-result', '评估结果', '<p>Loading...</p>'));
app.component('page-scale', createPageComponent('scale', '量表评估', '<p>Loading...</p>'));
app.component('page-scale-result', createPageComponent('scale-result', '评估结果', '<p>Loading...</p>'));
app.component('page-vms-survey', createPageComponent('vms-survey', 'VMS问卷', '<p>Loading...</p>'));
app.component('page-treatment', createPageComponent('treatment', '治疗推荐', '<p>Loading...</p>'));
app.component('page-profile', createPageComponent('profile', '个人中心', '<p>Loading...</p>'));
```

- [x] **Step 2: Verify all pages load**

Click through each tab and verify the page header shows the correct title.

- [x] **Step 3: Commit**

```bash
git add index.html && git commit -m "feat: add page navigation system with placeholder components"
```

---

## Phase 2: Core Pages (Tasks 4-7)

### Task 4: Dashboard Page — Overview Cards

**Files:**
- Modify: `index.html` (replace page-home placeholder)

- [x] **Step 1: Implement the Dashboard component**

Replace the `page-home` component with a full implementation containing:
- Today's episode count with trend arrow vs yesterday
- Severity distribution tags (mild/moderate/severe/extreme counts for today)
- Consecutive recording days counter
- Quick record floating button

The component uses computed properties to calculate today's stats from `store.symptoms`.

- [x] **Step 2: Verify dashboard renders**

Open browser — should see today's stats cards with real data from demo records.

- [x] **Step 3: Commit**

```bash
git add index.html && git commit -m "feat: implement dashboard overview cards with today's stats"
```

---

### Task 5: Dashboard — Weekly Trend Chart (ECharts)

**Files:**
- Modify: `index.html` (add chart component + integrate into dashboard)

- [x] **Step 1: Add ECharts wrapper component**

```javascript
app.component('echarts-chart', {
  props: ['option', 'height'],
  template: '<div ref="chartEl" :style="{ height: (height || 200) + \'px\' }"></div>',
  setup(props) {
    const chartEl = ref(null);
    let chart = null;
    onMounted(() => {
      chart = echarts.init(chartEl.value);
      chart.setOption(props.option);
    });
    watch(() => props.option, (v) => { if (chart) chart.setOption(v, true); }, { deep: true });
    onUnmounted(() => { if (chart) chart.dispose(); });
    return { chartEl };
  }
});
```

- [x] **Step 2: Add weekly trend chart to Dashboard**

Add a computed property that generates the ECharts option for a dual-axis chart:
- Bar chart: daily episode counts for last 7 days
- Line chart: average severity for last 7 days

- [x] **Step 3: Verify chart renders**

Open browser — should see a bar+line chart showing 7-day trends.

- [x] **Step 4: Commit**

```bash
git add index.html && git commit -m "feat: add weekly trend chart with ECharts dual-axis visualization"
```

---

### Task 6: Dashboard — AI Suggestion Card

**Files:**
- Modify: `index.html` (add AI suggestion component to dashboard)

- [x] **Step 1: Implement AI suggestion logic**

Add a computed property that analyzes `store.symptoms` to generate:
1. Top 2 high-probability time windows (based on hourly frequency)
2. Top trigger (most common trigger across all records)
3. Most effective intervention (intervention with lowest average severity)

- [x] **Step 2: Render suggestion card in dashboard**

Display the AI suggestions in a styled card with icons.

- [x] **Step 3: Verify suggestions show meaningful content**

- [x] **Step 4: Commit**

```bash
git add index.html && git commit -m "feat: add AI suggestion card with time/trigger/intervention analysis"
```

---

### Task 7: Quick Record Floating Panel

**Files:**
- Modify: `index.html` (add quick record sheet component)

- [x] **Step 1: Implement QuickRecord bottom sheet component**

A slide-up panel with:
- Severity selector (4 buttons: mild/moderate/severe/extreme)
- Time picker (defaults to now)
- Duration slider (1-30 min)
- Save button → pushes to `store.symptoms` and closes

- [x] **Step 2: Wire quick record button on dashboard**

- [x] **Step 3: Verify quick record flow**

Click button → sheet slides up → select severity → save → card updates with new count.

- [x] **Step 4: Commit**

```bash
git add index.html && git commit -m "feat: add quick record floating panel with severity selection"
```

---

## Phase 3: Diary & Stats (Tasks 8-11)

### Task 8: Diary List Page

**Files:**
- Modify: `index.html` (replace page-diary placeholder)

- [x] **Step 1: Implement Diary list component**

- Group `store.symptoms` by date
- Each group shows date header + record cards
- Each card: time, severity color tag, triggers, duration
- FAB button for new record → navigates to diary-detail

- [x] **Step 2: Verify diary list**

- [x] **Step 3: Commit**

```bash
git add index.html && git commit -m "feat: implement diary list page with date-grouped records"
```

---

### Task 9: Diary Detail / Edit Form

**Files:**
- Modify: `index.html` (replace page-diary-detail placeholder)

- [x] **Step 1: Implement full record form**

Complete form with all fields from spec:
- datetime-local picker
- Severity radio (4 levels)
- Duration slider
- Trigger checkboxes (7 options)
- Accompanying symptom checkboxes (7 options)
- Intervention select
- Note textarea
- Save / Delete buttons

- [x] **Step 2: Wire create + edit modes**

- New record: empty form, save adds to store
- Edit: pre-fill from `pageParams.id`, save updates existing record

- [x] **Step 3: Verify create and edit flows**

- [x] **Step 4: Commit**

```bash
git add index.html && git commit -m "feat: implement diary detail form with full symptom fields"
```

---

### Task 10: Stats Page — Trend + Pie Charts

**Files:**
- Modify: `index.html` (replace page-stats placeholder)

- [x] **Step 1: Implement Stats page with frequency trend chart**

- Day/Week/Month view toggle
- ECharts line+area chart for episode frequency over time
- Computed properties that aggregate symptoms by selected time range

- [x] **Step 2: Add severity distribution pie chart**

- ECharts donut chart showing mild/moderate/severe/extreme proportions
- Center label showing total count

- [x] **Step 3: Verify charts render with demo data**

- [x] **Step 4: Commit**

```bash
git add index.html && git commit -m "feat: implement stats page with trend line and severity pie charts"
```

---

### Task 11: Stats Page — Trigger Analysis + Heatmap

**Files:**
- Modify: `index.html` (add more charts to stats page)

- [x] **Step 1: Add trigger frequency horizontal bar chart**

- Count occurrences of each trigger
- Sort descending, show as horizontal bars

- [x] **Step 2: Add 24h×7day heatmap**

- ECharts heatmap: X=hour (0-23), Y=weekday (Mon-Sun)
- Color intensity = episode count in that slot

- [x] **Step 3: Add intervention comparison chart**

- Grouped bar: with-intervention vs without-intervention average severity

- [x] **Step 4: Commit**

```bash
git add index.html && git commit -m "feat: add trigger analysis, heatmap, and intervention comparison charts"
```

---

## Phase 4: AI Engine (Task 12)

### Task 12: AI Prediction Engine

**Files:**
- Modify: `index.html` (add AI engine functions + prediction display)

- [x] **Step 1: Implement time-pattern analysis**

```javascript
function analyzeTimePatterns(symptoms) {
  const hourly = new Array(24).fill(0);
  symptoms.forEach(s => {
    const h = new Date(s.time).getHours();
    hourly[h]++;
  });
  const total = symptoms.length || 1;
  return hourly.map((count, hour) => ({ hour, probability: count / total }));
}

function analyzeTriggerImpact(symptoms) {
  const impact = {};
  const triggers = ['咖啡因', '酒精', '辛辣食物', '高温环境', '压力', '剧烈运动', '情绪波动'];
  const baseline = symptoms.length ? symptoms.reduce((sum, s) => sum + severityValue(s.severity), 0) / symptoms.length : 0;

  triggers.forEach(trigger => {
    const filtered = symptoms.filter(s => s.triggers.includes(trigger));
    if (filtered.length >= 3) {
      const avgSev = filtered.reduce((sum, s) => sum + severityValue(s.severity), 0) / filtered.length;
      impact[trigger] = { count: filtered.length, avgSeverity: avgSev, lift: avgSev - baseline };
    }
  });
  return impact;
}

function severityValue(s) { return { mild: 1, moderate: 2, severe: 3, extreme: 4 }[s] || 0; }
function severityLabel(s) { return { mild: '轻度', moderate: '中度', severe: '重度', extreme: '极重度' }[s] || s; }
```

- [x] **Step 2: Implement prediction output**

For each hour of the next 24 hours, calculate risk level (high/medium/low) based on hourly probability. Display as a colored timeline.

- [x] **Step 3: Implement accuracy backtest**

Take the last 7 days, predict each day's episodes based on hourly patterns, compare with actual. Show accuracy percentage.

- [x] **Step 4: Integrate predictions into Dashboard AI card**

Replace the simple suggestion card with a richer AI panel showing:
- 24-hour risk timeline
- Top triggers with impact data
- Predicted accuracy
- Personalized recommendations

- [x] **Step 5: Commit**

```bash
git add index.html && git commit -m "feat: implement AI prediction engine with time-pattern analysis and backtesting"
```

---

## Phase 5: Education & Auricular (Tasks 13-15)

### Task 13: Education List Page

**Files:**
- Modify: `index.html` (replace page-education placeholder, add article data)

- [x] **Step 1: Add hardcoded article data**

Add 13 articles across 5 categories as a JavaScript array, each with: id, title, category, summary, content (Markdown string).

- [x] **Step 2: Implement education list with category tabs**

- Category tab bar (all 5 categories + "全部")
- Article cards: title, category tag, summary
- Click → navigate to education-detail with article id

- [x] **Step 3: Verify article listing and filtering**

- [x] **Step 4: Commit**

```bash
git add index.html && git commit -m "feat: implement education article list with category filtering"
```

---

### Task 14: Education Detail Page (Markdown Rendering)

**Files:**
- Modify: `index.html` (replace page-education-detail placeholder)

- [x] **Step 1: Implement article detail with marked.js**

- Load article by `pageParams.id`
- Render `article.content` using `marked.parse()`
- Style with TailwindCSS prose classes (or custom styles)

- [x] **Step 2: Verify Markdown rendering**

- [x] **Step 3: Commit**

```bash
git add index.html && git commit -m "feat: implement education detail page with Markdown rendering"
```

---

### Task 15: Auricular Acupoint Guide + Timer

**Files:**
- Modify: `index.html` (replace page-auricular + page-auricular-detail placeholders)

- [x] **Step 1: Add 10 acupoint data**

Hardcoded array with: name, position, indication, pressing technique for all 10 acupoints.

- [x] **Step 2: Implement acupoint list page**

- Card grid: acupoint name + one-line position
- Click → navigate to auricular-detail

- [x] **Step 3: Implement acupoint detail page**

- Position method (text description)
- Pressing technique
- Main indications
- **30-second countdown timer** with circular progress animation

- [x] **Step 4: Commit**

```bash
git add index.html && git commit -m "feat: implement auricular acupoint guide with 10 acupoints and pressing timer"
```

---

## Phase 6: Assessment Modules (Tasks 16-19)

### Task 16: Constitution Assessment Questionnaire

**Files:**
- Modify: `index.html` (replace page-constitution placeholder)

- [x] **Step 1: Add 72-question constitution data**

9 types × 8 questions each. Each question has 5 options (1-5 scale). Include reverse-scored items for 平和质.

- [x] **Step 2: Implement multi-step questionnaire UI**

- Progress bar at top
- One question at a time with radio options
- Next/Previous buttons
- 72 questions total

- [x] **Step 3: Implement scoring logic**

- Calculate scores for each of 9 dimensions
- Handle reverse scoring for 平和质 questions
- Determine primary constitution type (highest non-平和质 score, or 平和质 if all low)

- [x] **Step 4: Commit**

```bash
git add index.html && git commit -m "feat: implement 72-question TCM constitution assessment"
```

---

### Task 17: Constitution Result Page

**Files:**
- Modify: `index.html` (replace page-constitution-result placeholder)

- [x] **Step 1: Implement result display**

- Constitution type name + description
- ECharts radar chart showing all 9 dimension scores
- Personalized adjustment suggestions based on type

- [x] **Step 2: Wire from constitution page → result page**

- [x] **Step 3: Commit**

```bash
git add index.html && git commit -m "feat: implement constitution result page with radar chart"
```

---

### Task 18: Scale Engine

**Files:**
- Modify: `index.html` (replace page-scale + page-scale-result placeholders)

- [x] **Step 1: Add scale configuration data**

One pre-built scale (e.g., Pittsburgh Sleep Quality Index or similar) with categories, questions, scoring rules, and result templates.

- [x] **Step 2: Implement generic scale questionnaire UI**

- Category tabs
- Questions with radio options
- Progress indicator

- [x] **Step 3: Implement scoring engine**

- Support `sum`, `percentage`, `category_sum` rules
- Handle reverse-scored questions
- Match score to result templates

- [x] **Step 4: Implement result page**

- Total score + level
- Category breakdown
- Suggestions

- [x] **Step 5: Commit**

```bash
git add index.html && git commit -m "feat: implement generic scale assessment engine with scoring"
```

---

### Task 19: VMS Research Questionnaire

**Files:**
- Modify: `index.html` (replace page-vms-survey placeholder)

- [x] **Step 1: Implement 9-step VMS questionnaire wizard**

Steps:
1. Basic info (age, diagnosis, treatment)
2. Patient beliefs
3. VMS frequency
4. HFNS (Hot Flash Night Score)
5. HFRDIS (Hot Flash Related Daily Interference Scale)
6. HFBBS-Men (Hot Flash Beliefs and Behaviors Scale)
7. Treatment experience
8. Research indicators
9. Satisfaction

- [x] **Step 2: Implement standard scale scoring (HFNS, HFRDIS, HFBBS-Men)**

- [x] **Step 3: Implement results summary page**

- [x] **Step 4: Commit**

```bash
git add index.html && git commit -m "feat: implement 9-step VMS research questionnaire with standard scales"
```

---

## Phase 7: Treatment & Profile (Tasks 20-21)

### Task 20: Treatment Recommendations Page

**Files:**
- Modify: `index.html` (replace page-treatment placeholder)

- [x] **Step 1: Implement 3-tier treatment pathway**

- Tier 1: Lifestyle adjustments (breathing, environment, diet, exercise)
- Tier 2: TCM non-pharmacological (auricular acupressure, acupressure, relaxation)
- Tier 3: Medication reference (HRT, SSRI/SNRI, gabapentin, NK3R antagonist)
- Each item: name, steps, precautions, evidence level badge

- [x] **Step 2: Commit**

```bash
git add index.html && git commit -m "feat: implement 3-tier treatment recommendation pathway"
```

---

### Task 21: Profile Page

**Files:**
- Modify: `index.html` (replace page-profile placeholder)

- [x] **Step 1: Implement profile page**

- User info display + edit form (name, gender, age, condition)
- Navigation links: constitution assessment, scales, VMS survey, about
- Data management: clear data, export as JSON download
- About section: product intro, version, links

- [x] **Step 2: Commit**

```bash
git add index.html && git commit -m "feat: implement profile page with user info and data management"
```

---

## Phase 8: Polish & Package (Task 22)

### Task 22: Final Polish + Offline Package

**Files:**
- Modify: `index.html` (final fixes, optimizations)
- Create: `libs/` directory with local library files
- Create: `README.md`

- [x] **Step 1: Download CDN libraries to `libs/`**

```bash
mkdir libs
curl -o libs/vue.global.prod.js https://unpkg.com/vue@3/dist/vue.global.prod.js
curl -o libs/echarts.min.js https://cdn.jsdelivr.net/npm/echarts@5/dist/echarts.min.js
curl -o libs/marked.min.js https://cdn.jsdelivr.net/npm/marked/marked.min.js
```

- [x] **Step 2: Update index.html to reference local libs**

Replace CDN URLs with `libs/` local paths.

- [x] **Step 3: Add responsive polish**

- Ensure all pages look good at 375px width
- Fix any overflow/scrolling issues
- Add missing transition animations

- [x] **Step 4: Create README.md**

```markdown
# 沁凉助手 — 潮热盗汗AI智能管理工具

TRAE AI 创造力大赛 · 社会服务赛道参赛作品

## 体验方式

1. 下载本仓库 ZIP 文件
2. 解压后打开 `index.html`
3. 无需联网，即可体验全部功能

## 功能模块

- 首页仪表盘：今日概览 + 趋势图 + AI 建议
- 症状记录：一键记录潮热发作详情
- 数据统计：5 种可视化图表
- AI 预测引擎：时段风险评估 + 个性化建议
- 健康科普：13 篇专业文章
- 耳穴压豆指南：10 穴位 + 按压计时器
- 中医体质评估：9 种体质 72 题
- 通用量表引擎：可扩展评估工具
- VMS 研究问卷：HFNS/HFRDIS/HFBBS 标准量表
- 分级治疗推荐：三层递进方案
- 个人中心：信息管理 + 数据导出
```

- [x] **Step 5: Create Zip package for submission**

```bash
Compress-Archive -Path index.html,libs,README.md -DestinationPath qinling-assistant.zip
```

- [x] **Step 6: Final test — open Zip in fresh browser, verify all features**

- [x] **Step 7: Commit**

```bash
git add -A && git commit -m "feat: finalize offline package with local libs and README"
```

---

## Self-Review Checklist

- [x] Spec coverage: All 11 modules from spec have corresponding tasks
- [x] Placeholder scan: No TBD/TODO in any task
- [x] Type consistency: Data models, function names, and property names consistent across tasks
- [x] Each task is independently testable
- [x] Total: 22 tasks across 8 phases

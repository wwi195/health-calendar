# ヘルスカレンダー v2 実装計画

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** ヘルスカレンダーに基礎代謝対応カロリー収支・生理〇日目表示・今日タブ・スマホUI改善を追加する

**Architecture:** 単一HTMLファイル（health-calendar.html）をすべて編集する。localStorage に `health_profile_v1` を追加しプロフィールを保存。既存の `health_cal_v1` データはそのまま維持して後方互換性を保つ。

**Tech Stack:** HTML / CSS / JavaScript（バニラ）、localStorage

---

## ファイル構成

| ファイル | 操作 |
|---------|------|
| `C:\Users\ab_99\Desktop\health-calendar.html` | メイン編集対象 |
| `C:\Users\ab_99\Desktop\index.html` | Task 6 でコピー更新 |

---

## Task 1: プロフィール設定 & BMR基盤

**Files:**
- Modify: `health-calendar.html`（CSS追加・HTML追加・JS追加）

### 1-1. CSS追加（`</style>` の直前に追加）

- [ ] `</style>` タグを探し、その直前に以下のCSSを挿入する

```css
/* ── Profile Modal ── */
.bmr-display {
  background: #fff5f8;
  border-radius: 8px;
  padding: 8px 12px;
  text-align: center;
  font-size: 14px;
  font-weight: 700;
  margin-bottom: 13px;
  color: #ff6b9d;
}
```

### 1-2. プロフィールモーダルHTML追加

- [ ] 既存の `<!-- ===== MODAL ===== -->` ブロックの直前に以下を追加する

```html
<!-- ===== PROFILE MODAL ===== -->
<div class="modal-overlay" id="profileOverlay">
  <div class="modal" id="profileModal">
    <div class="modal-title">⚙️ <span>プロフィール設定</span></div>
    <div class="form-row">
      <label>📏 身長 (cm)</label>
      <input type="number" id="inHeight" min="100" max="250" placeholder="例: 160" inputmode="numeric">
    </div>
    <div class="form-row">
      <label>⚖️ 体重 (kg)（初期値・毎日記録で更新されます）</label>
      <input type="number" id="inProfileWeight" min="30" max="200" step="0.1" placeholder="例: 55.0" inputmode="decimal">
    </div>
    <div class="form-row">
      <label>🎂 年齢 (歳)</label>
      <input type="number" id="inAge" min="10" max="100" placeholder="例: 30" inputmode="numeric">
    </div>
    <div id="bmrDisplay" class="bmr-display" style="display:none"></div>
    <div class="modal-actions">
      <button class="btn-save" onclick="saveProfile()">保存</button>
      <button class="btn-cancel" onclick="closeProfileModal()">キャンセル</button>
    </div>
  </div>
</div>
```

### 1-3. ⚙️ボタンHTML追加

- [ ] `<div class="graph-header">` の中の `▶` ボタンの直後に以下を追加する

```html
<button class="nav-btn" onclick="openProfileModal()" style="padding:4px 10px">⚙️</button>
```

### 1-4. JS定数追加

- [ ] `const STORAGE_KEY = 'health_cal_v1';` の直後に以下を追加する

```js
const PROFILE_KEY  = 'health_profile_v1';
```

### 1-5. JSストレージ関数追加

- [ ] `function getData()` の定義ブロック（`function putData(d)` まで含む）の直後に以下を追加する

```js
function getProfile() {
  try { return JSON.parse(localStorage.getItem(PROFILE_KEY) || 'null'); }
  catch { return null; }
}
function putProfile(p) {
  localStorage.setItem(PROFILE_KEY, JSON.stringify(p));
}
function calcBMR(height, weight, age) {
  // Mifflin-St Jeor 女性用
  return Math.round(10 * weight + 6.25 * height - 5 * age - 161);
}
```

### 1-6. プロフィールモーダルJS関数追加

- [ ] `function closeModal()` の定義の直後に以下を追加する

```js
function openProfileModal() {
  const p = getProfile();
  document.getElementById('inHeight').value        = p ? p.height : '';
  document.getElementById('inProfileWeight').value = p ? p.weight : '';
  document.getElementById('inAge').value           = p ? p.age    : '';
  refreshBmrDisplay();
  document.getElementById('profileOverlay').classList.add('open');
}
function closeProfileModal() {
  document.getElementById('profileOverlay').classList.remove('open');
}
function refreshBmrDisplay() {
  const h  = parseFloat(document.getElementById('inHeight').value);
  const w  = parseFloat(document.getElementById('inProfileWeight').value);
  const a  = parseInt(document.getElementById('inAge').value);
  const el = document.getElementById('bmrDisplay');
  if (h > 0 && w > 0 && a > 0) {
    const bmr = calcBMR(h, w, a);
    el.style.display = 'block';
    el.textContent   = `基礎代謝: ${bmr.toLocaleString()} kcal/日`;
  } else {
    el.style.display = 'none';
  }
}
function saveProfile() {
  const height = parseFloat(document.getElementById('inHeight').value);
  const weight = parseFloat(document.getElementById('inProfileWeight').value);
  const age    = parseInt(document.getElementById('inAge').value);
  if (!height || !weight || !age) {
    alert('身長・体重・年齢をすべて入力してください');
    return;
  }
  const bmr = calcBMR(height, weight, age);
  putProfile({ height, weight, age, bmr });
  closeProfileModal();
  renderToday();
}
```

### 1-7. プロフィールモーダルのイベントリスナー追加

- [ ] `document.getElementById('inIntake').addEventListener('input', refreshBalPreview);` の行の直後に以下を追加する

```js
document.getElementById('inHeight').addEventListener('input',        refreshBmrDisplay);
document.getElementById('inProfileWeight').addEventListener('input', refreshBmrDisplay);
document.getElementById('inAge').addEventListener('input',           refreshBmrDisplay);
document.getElementById('profileOverlay').addEventListener('pointerdown', function(e) {
  if (e.target === this) closeProfileModal();
});
```

### 1-8. 動作確認

- [ ] ブラウザで `health-calendar.html` を開く
- [ ] グラフタブに切り替えて `⚙️` ボタンが表示されることを確認
- [ ] `⚙️` をクリックしてプロフィールモーダルが開くことを確認
- [ ] 身長・体重・年齢を入力すると基礎代謝が表示されることを確認
- [ ] 保存ボタンが動作することを確認（エラーが出ないこと）

### 1-9. コミット

```bash
cd "C:\Users\ab_99\Desktop"
git add health-calendar.html
git commit -m "feat: add profile setup modal and BMR calculation"
```

---

## Task 2: カロリー収支にBMRを反映

**Files:**
- Modify: `health-calendar.html`

### 2-1. モーダルのラベル変更

- [ ] モーダル内の `🔵 消費カロリー (kcal)` を以下に変更する

```html
<label>🔵 消費カロリー（運動分） (kcal)</label>
```

### 2-2. `refreshBalPreview()` 関数を置換

- [ ] 既存の `function refreshBalPreview()` 全体を以下で置き換える

```js
function refreshBalPreview() {
  const intake  = parseFloat(document.getElementById('inIntake').value) || 0;
  const burn    = parseFloat(document.getElementById('inBurn').value)   || 0;
  const el      = document.getElementById('balPreview');
  const profile = getProfile();
  const bmr     = profile ? profile.bmr : 0;

  if (intake > 0 || burn > 0) {
    const bal  = intake - burn - bmr;
    const sign = bal >= 0 ? '+' : '';
    const color = bal >= 0 ? '#e67e22' : '#27ae60';
    const bmrNote = bmr > 0
      ? `<div style="font-size:11px;color:#999;margin-top:3px">基礎代謝 ${bmr.toLocaleString()} kcal を含む</div>`
      : '';
    el.style.display = 'block';
    el.innerHTML = `収支: <span style="color:${color};font-size:16px">${sign}${bal.toLocaleString()} kcal</span>${bmrNote}`;
  } else {
    el.style.display = 'none';
  }
}
```

### 2-3. カレンダーセルの収支計算にBMRを反映

- [ ] `renderCalendar()` 内の以下の行を探す

```js
if (rec.intake || rec.burn || rec.weight) {
```

- [ ] そのブロック内の `const bal = (rec.intake || 0) - (rec.burn || 0);` を以下に置き換える

```js
const profile = getProfile();
const bmr     = (profile && rec.intake != null) ? profile.bmr : 0;
const bal     = (rec.intake || 0) - (rec.burn || 0) - bmr;
```

### 2-4. 動作確認

- [ ] ブラウザでリロードする
- [ ] プロフィール設定で身長・体重・年齢を入力して保存する（BMR例：1,400 kcal）
- [ ] 今日の日付をクリックして摂取1800・消費300を入力する
- [ ] 収支プレビューが `1800 - 300 - 1400 = +100 kcal` と表示されることを確認
- [ ] カレンダーセルに `+100` が表示されることを確認

### 2-5. コミット

```bash
cd "C:\Users\ab_99\Desktop"
git add health-calendar.html
git commit -m "feat: integrate BMR into calorie balance calculation"
```

---

## Task 3: 生理サイクル表示改善

**Files:**
- Modify: `health-calendar.html`

### 3-1. PHASES定数のlateLutealラベルを変更

- [ ] `PHASES` オブジェクトの `lateLuteal` を以下に変更する

```js
lateLuteal:  { label: '生理前・むくみ注意', color: '#f5b3ff', text: '#8e44ad' },
```

### 3-2. 予測ロジックの閾値を拡大

- [ ] `isPredicted()` 内の以下の行を探す

```js
const hasActual = starts.some(s => Math.abs(daysBetween(s, predStr)) <= 3 && s > last);
```

- [ ] `<= 3` を `<= 7` に変更する

```js
const hasActual = starts.some(s => Math.abs(daysBetween(s, predStr)) <= 7 && s > last);
```

### 3-3. カレンダーの生理フェーズラベルを〇日目表示に変更

- [ ] `renderCalendar()` 内の以下のブロックを探す

```js
// Phase label
if (pk) {
  const lbl = document.createElement('div');
  lbl.className = 'phase-lbl';
  lbl.style.color = PHASES[pk].text;
  lbl.textContent = PHASES[pk].label;
  cell.appendChild(lbl);
} else if (pred) {
```

- [ ] このブロックを以下で置き換える

```js
// Phase label
if (pk) {
  const lbl = document.createElement('div');
  lbl.className = 'phase-lbl';
  lbl.style.color = PHASES[pk].text;
  lbl.textContent = (pk === 'menstrual' && cd) ? `生理${cd}日目` : PHASES[pk].label;
  cell.appendChild(lbl);
} else if (pred) {
```

### 3-4. フェーズ凡例の更新

- [ ] `.phase-legend` 内の「生理中」テキストを「生理中（1〜7日目）」に変更する

```html
<div class="legend-item"><div class="legend-dot" style="background:#ffb3b3"></div>生理中（1〜7日目）</div>
```

### 3-5. 動作確認

- [ ] ブラウザでリロードする
- [ ] 先月や今月の日付で「生理開始日として記録」チェックを入れて保存する
- [ ] その日から7日間、カレンダーに「生理1日目」〜「生理7日目」と表示されることを確認
- [ ] 黄体期後半（22〜28日目相当）に「生理前・むくみ注意」が表示されることを確認
- [ ] 次の周期の予測エリアに「生理予測」が表示されることを確認

### 3-6. コミット

```bash
cd "C:\Users\ab_99\Desktop"
git add health-calendar.html
git commit -m "feat: show menstrual day number and update phase labels"
```

---

## Task 4: 「今日」タブ追加

**Files:**
- Modify: `health-calendar.html`

### 4-1. 「今日」タブのCSS追加

- [ ] Task 1 で追加したCSS（`.bmr-display` ブロック）の直後に以下を追加する

```css
/* ── Today View ── */
#todayView { padding: 16px 14px 24px; }
.today-date {
  font-size: 18px;
  font-weight: 700;
  color: #333;
  margin-bottom: 14px;
  padding-bottom: 12px;
  border-bottom: 2px solid #ffe4ec;
}
.today-card {
  background: #fff;
  border: 1px solid #eee;
  border-radius: 14px;
  padding: 14px 16px;
  margin-bottom: 12px;
  box-shadow: 0 1px 4px rgba(0,0,0,0.05);
}
.today-card-title {
  font-size: 11px;
  font-weight: 700;
  color: #aaa;
  text-transform: uppercase;
  letter-spacing: 0.8px;
  margin-bottom: 10px;
}
.today-cal-row {
  display: flex;
  justify-content: space-between;
  font-size: 14px;
  padding: 4px 0;
  color: #555;
}
.today-cal-divider { border-top: 1px solid #eee; margin: 8px 0; }
.today-cal-balance {
  font-size: 22px;
  font-weight: 700;
  text-align: center;
  padding: 6px 0;
}
.today-phase-name {
  font-size: 18px;
  font-weight: 700;
  margin-bottom: 6px;
}
.today-phase-advice { font-size: 13px; color: #666; }
.today-weight {
  font-size: 30px;
  font-weight: 700;
  color: #7f8c8d;
  text-align: center;
  padding: 4px 0;
}
.today-no-data {
  text-align: center;
  color: #aaa;
  font-size: 14px;
  padding: 16px 0;
  cursor: pointer;
  line-height: 2;
}
.today-no-data span { color: #ff6b9d; font-weight: 700; }
.today-record-btn {
  width: 100%;
  padding: 14px;
  background: #ff6b9d;
  color: #fff;
  border: none;
  border-radius: 12px;
  font-size: 15px;
  font-weight: 700;
  cursor: pointer;
  margin-top: 4px;
  touch-action: manipulation;
  -webkit-tap-highlight-color: transparent;
}
.today-record-btn:active { background: #e85a8c; }
```

### 4-2. 「今日」タブボタンHTMLを追加

- [ ] ヘッダー内の `<div class="view-tabs">` を探し、既存のタブボタンの**前**に以下を追加する

```html
<button class="tab-btn" id="tabToday" onclick="switchView('today')">🌸 今日</button>
```

結果として view-tabs の中は以下の順番になる：
```html
<div class="view-tabs">
  <button class="tab-btn" id="tabToday" onclick="switchView('today')">🌸 今日</button>
  <button class="tab-btn active" id="tabCal" onclick="switchView('calendar')">📅 カレンダー</button>
  <button class="tab-btn"        id="tabGph" onclick="switchView('graph')">📊 グラフ</button>
</div>
```

### 4-3. 「今日」ビューHTMLを追加

- [ ] `<!-- ===== CALENDAR VIEW ===== -->` の直前に以下を追加する

```html
<!-- ===== TODAY VIEW ===== -->
<div id="todayView" style="display:none"></div>
```

### 4-4. `renderToday()` 関数を追加

- [ ] `function renderCalendar()` の定義の直前に以下を追加する

```js
const PHASE_ADVICE = {
  menstrual:   '安静にしよう。無理な運動はNG',
  follicular:  'ダイエットのチャンス！積極的に動こう',
  ovulation:   '体が活発な時期。代謝も上がりやすい',
  earlyLuteal: '塩分・水分に注意してむくみ予防',
  lateLuteal:  '食欲が増えやすい時期。意識してコントロール',
};

function renderToday() {
  const todayKey = today.toISOString().slice(0, 10);
  const data     = getData();
  const rec      = data[todayKey] || {};
  const profile  = getProfile();
  const view     = document.getElementById('todayView');

  const [y, m, d] = todayKey.split('-').map(Number);
  const weekdays  = ['日', '月', '火', '水', '木', '金', '土'];
  const dow       = today.getDay();

  // ── Calorie section ──
  const intake     = rec.intake != null ? rec.intake : null;
  const burn       = rec.burn   != null ? rec.burn   : 0;
  const bmr        = profile ? profile.bmr : 0;
  const hasCalorie = rec.intake != null || rec.burn != null;

  let calHTML = '';
  if (hasCalorie) {
    const bal   = (intake || 0) - burn - bmr;
    const sign  = bal >= 0 ? '+' : '';
    const color = bal > 100 ? '#e67e22' : bal < -100 ? '#27ae60' : '#888';
    const dir   = bal > 100 ? '⚠️ 太り方向' : bal < -100 ? '✅ 痩せ方向' : '→ 維持';
    calHTML = `
      <div class="today-cal-row"><span>摂取</span><span>${(intake || 0).toLocaleString()} kcal</span></div>
      <div class="today-cal-row"><span>消費（運動）</span><span>${burn.toLocaleString()} kcal</span></div>
      ${bmr > 0 ? `<div class="today-cal-row"><span>基礎代謝</span><span>${bmr.toLocaleString()} kcal</span></div>` : ''}
      <div class="today-cal-divider"></div>
      <div class="today-cal-balance" style="color:${color}">${sign}${bal.toLocaleString()} kcal ${dir}</div>
    `;
  } else {
    calHTML = `<div class="today-no-data" onclick="openModal('${todayKey}')">
      今日のデータがまだありません<br><span>タップして記録 ▶</span>
    </div>`;
  }

  // ── Phase section ──
  const cd   = cycleDay(todayKey, data);
  const pk   = phaseKey(cd);
  const pred = !pk && isPredicted(todayKey, data);

  let phaseHTML = '';
  if (pk === 'menstrual' && cd) {
    phaseHTML = `
      <div class="today-phase-name" style="color:${PHASES[pk].text}">🩸 生理${cd}日目</div>
      <div class="today-phase-advice">${PHASE_ADVICE.menstrual}</div>`;
  } else if (pk) {
    const icon = { follicular: '🌿', ovulation: '🌟', earlyLuteal: '💧', lateLuteal: '🌙' };
    phaseHTML = `
      <div class="today-phase-name" style="color:${PHASES[pk].text}">${icon[pk] || ''} ${PHASES[pk].label}</div>
      <div class="today-phase-advice">${PHASE_ADVICE[pk]}</div>`;
  } else if (pred) {
    phaseHTML = `
      <div class="today-phase-name" style="color:#c0392b">📅 生理予測日</div>
      <div class="today-phase-advice">もうすぐ生理が来るかも</div>`;
  } else {
    phaseHTML = `
      <div class="today-phase-name" style="color:#ccc">記録なし</div>
      <div class="today-phase-advice">生理開始日を記録すると表示されます</div>`;
  }

  view.innerHTML = `
    <div class="today-date">${y}年${m}月${d}日（${weekdays[dow]}）</div>
    <div class="today-card">
      <div class="today-card-title">📊 カロリー収支</div>
      ${calHTML}
    </div>
    <div class="today-card">
      <div class="today-card-title">🩸 生理・体調フェーズ</div>
      ${phaseHTML}
    </div>
    ${rec.weight ? `
    <div class="today-card">
      <div class="today-card-title">⚖️ 体重</div>
      <div class="today-weight">${rec.weight} kg</div>
    </div>` : ''}
    <button class="today-record-btn" onclick="openModal('${todayKey}')">✏️ 今日を記録・編集</button>
  `;
}
```

### 4-5. `switchView()` 関数を更新

- [ ] 既存の `function switchView(view)` 全体を以下で置き換える

```js
function switchView(view) {
  const isCalendar = view === 'calendar';
  const isToday    = view === 'today';
  const isGraph    = view === 'graph';

  document.getElementById('todayView').style.display    = isToday    ? 'block' : 'none';
  document.getElementById('calendarView').style.display = isCalendar ? 'block' : 'none';
  document.getElementById('graphView').style.display    = isGraph    ? 'block' : 'none';

  document.getElementById('tabToday').classList.toggle('active', isToday);
  document.getElementById('tabCal').classList.toggle('active',   isCalendar);
  document.getElementById('tabGph').classList.toggle('active',   isGraph);

  if (isToday)    renderToday();
  if (isGraph)    requestAnimationFrame(() => requestAnimationFrame(renderGraphs));
}
```

### 4-6. 初期化コードを更新

- [ ] ファイル末尾付近の `renderCalendar();` を以下で置き換える

```js
renderCalendar();
switchView('today');

if (!getProfile()) {
  setTimeout(() => openProfileModal(), 400);
}
```

### 4-7. 動作確認

- [ ] ブラウザでリロードする
- [ ] 初回はプロフィール設定が自動で開くことを確認（未設定の場合）
- [ ] アプリ起動時に「今日」タブが最初に表示されることを確認
- [ ] カロリーデータがあれば収支が表示されることを確認
- [ ] 「今日を記録・編集」ボタンでモーダルが開くことを確認
- [ ] 記録保存後、「今日」タブに反映されることを確認（モーダルclose後）

※ `saveDay()` の最後に `renderToday();` を追加する必要がある

- [ ] `function saveDay()` 内の `renderCalendar();` の直後に `renderToday();` を追加する

```js
putData(data);
closeModal();
renderCalendar();
renderToday();   // ← 追加
```

- [ ] `function deleteDay()` 内の `renderCalendar();` の直後にも `renderToday();` を追加する

```js
putData(data);
closeModal();
renderCalendar();
renderToday();   // ← 追加
```

### 4-8. コミット

```bash
cd "C:\Users\ab_99\Desktop"
git add health-calendar.html
git commit -m "feat: add today tab with calorie balance and phase info"
```

---

## Task 5: スマホUI改善

**Files:**
- Modify: `health-calendar.html`（CSSのみ）

### 5-1. カレンダーセルサイズ変更

- [ ] `.day-cell` の `min-height: 72px;` を `min-height: 80px;` に変更する

### 5-2. 日付数字フォントサイズ変更

- [ ] `.day-num` の `font-size: 12px;` を `font-size: 14px;` に変更する

### 5-3. セル内データ文字サイズ変更

- [ ] `.day-data` の `font-size: 9px;` を `font-size: 11px;` に変更する

### 5-4. フェーズラベル文字サイズ変更

- [ ] `.phase-lbl` の `font-size: 8px;` を `font-size: 10px;` に変更する

### 5-5. タブボタン文字サイズ変更

- [ ] `.tab-btn` の `font-size: 12px;` を `font-size: 13px;` に変更する

### 5-6. モーダル入力欄文字サイズ変更（iOSズームイン防止）

- [ ] `.form-row input[type="number"]` の `font-size: 15px;` を `font-size: 16px;` に変更する

### 5-7. レスポンシブ調整

- [ ] `@media (max-width: 380px)` ブロックの `.day-cell` の `min-height: 64px;` を `min-height: 70px;` に変更する

### 5-8. 動作確認

- [ ] ブラウザのDevToolsでスマホ表示（iPhone SE等）に切り替えて確認する
- [ ] カレンダーのセルが大きくなり文字が読みやすくなっていることを確認する
- [ ] 数字入力モーダルを開いて、iOSでズームインが発生しないことを確認する（16px以上であれば防止される）

### 5-9. コミット

```bash
cd "C:\Users\ab_99\Desktop"
git add health-calendar.html
git commit -m "feat: improve mobile UI - larger cells, fonts, touch targets"
```

---

## Task 6: デプロイ（GitHub Pages更新）

**Files:**
- Modify: `C:\Users\ab_99\Desktop\index.html`（health-calendar.htmlのコピー）

### 6-1. index.htmlを更新

- [ ] 以下のコマンドで `health-calendar.html` を `index.html` にコピーする

```bash
cd "C:\Users\ab_99\Desktop"
cp health-calendar.html index.html
```

### 6-2. GitHubにプッシュ

- [ ] 以下のコマンドでコミット＆プッシュする

```bash
cd "C:\Users\ab_99\Desktop"
git add index.html
git commit -m "chore: sync index.html with health-calendar.html for GitHub Pages"
git push
```

### 6-3. 動作確認

- [ ] 1〜2分待ってから `https://wwi195.github.io/health-calendar/` をスマホで開く
- [ ] 「今日」タブが最初に表示されることを確認する
- [ ] プロフィール設定が動作することを確認する
- [ ] カロリー入力・生理記録が動作することを確認する

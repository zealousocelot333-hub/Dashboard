<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width, initial-scale=1.0"/>
<title>ArcWealth — Financial Dashboard</title>
<link rel="preconnect" href="https://fonts.googleapis.com"/>
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,400;0,700;1,400&family=Outfit:wght@300;400;500;600;700;800&family=JetBrains+Mono:wght@400;500;600;700&display=swap" rel="stylesheet"/>
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js"></script>
<style>
:root {
  --bg:      #07080F;
  --bg2:     #0C0E1A;
  --bg3:     #111328;
  --surface: rgba(255,255,255,0.03);
  --border:  rgba(148,130,255,0.1);
  --border2: rgba(148,130,255,0.2);
  --teal:    #7C6FFF;
  --teal2:   #A599FF;
  --amber:   #F5A623;
  --amber2:  #FFBE4F;
  --green:   #2ECC8E;
  --green2:  #25B07A;
  --red:     #F05565;
  --coral:   #FF7B5E;
  --sky:     #60A5FA;
  --text:    #EBE9FF;
  --muted:   #4A4670;
  --muted2:  #8A84B8;
  --font:    'Outfit', sans-serif;
  --serif:   'Playfair Display', serif;
  --mono:    'JetBrains Mono', monospace;
  --r:       14px;
  --r2:      18px;
}
* { margin:0; padding:0; box-sizing:border-box; }
html { font-size:14px; }
body {
  background: var(--bg);
  background-image: radial-gradient(ellipse 80% 50% at 20% 0%, rgba(124,111,255,0.07) 0%, transparent 60%),
                    radial-gradient(ellipse 60% 40% at 80% 100%, rgba(96,165,250,0.05) 0%, transparent 60%);
  color: var(--text);
  font-family: var(--font);
  min-height: 100vh;
  display: flex;
  overflow-x: hidden;
}
::-webkit-scrollbar { width:4px; }
::-webkit-scrollbar-thumb { background:rgba(124,111,255,0.2); border-radius:4px; }

@keyframes fadeUp   { from{opacity:0;transform:translateY(14px)} to{opacity:1;transform:translateY(0)} }
@keyframes pulse    { 0%,100%{opacity:1} 50%{opacity:0.25} }
@keyframes ticker   { from{transform:translateX(0)} to{transform:translateX(-50%)} }
@keyframes shimmer  { 0%{opacity:0.6} 50%{opacity:1} 100%{opacity:0.6} }
@keyframes glow     { 0%,100%{box-shadow:0 0 8px rgba(124,111,255,0.2)} 50%{box-shadow:0 0 22px rgba(124,111,255,0.55)} }

/* ── Sidebar ── */
#sidebar {
  width:230px; min-height:100vh;
  background: var(--bg2);
  border-right: 1px solid var(--border);
  display:flex; flex-direction:column;
  padding:22px 14px;
  position:sticky; top:0; height:100vh;
  flex-shrink:0;
  background-image: linear-gradient(180deg, rgba(124,111,255,0.04) 0%, transparent 40%);
}
.logo {
  display:flex; align-items:center; gap:11px;
  padding:0 6px 22px;
  border-bottom: 1px solid var(--border);
  margin-bottom:18px;
}
.logo-icon {
  width:36px; height:36px; border-radius:10px;
  background: linear-gradient(135deg, var(--teal), #3A2ECC);
  display:flex; align-items:center; justify-content:center;
  font-size:16px; box-shadow:0 0 22px rgba(124,111,255,0.4);
  flex-shrink:0;
}
.logo-text {
  font-family: var(--serif);
  font-size:20px;
  letter-spacing:-0.01em;
  color: var(--text);
}

.nav-section-label {
  font-size:9px; color:var(--muted); letter-spacing:0.12em;
  text-transform:uppercase; padding:14px 10px 5px;
}
.nav-item {
  display:flex; align-items:center; gap:11px;
  padding:9px 12px; border-radius:10px;
  cursor:pointer; transition:all 0.15s ease;
  color:var(--muted); font-size:13px; font-weight:600;
  margin-bottom:2px; text-decoration:none;
}
.nav-item svg { flex-shrink:0; }
.nav-item:hover { background:rgba(255,255,255,0.05); color:var(--text); }
.nav-item.active {
  background:rgba(124,111,255,0.14);
  color:var(--teal2);
  box-shadow:inset 0 0 0 1px rgba(124,111,255,0.28);
}
.nav-item.active svg { filter:drop-shadow(0 0 4px rgba(124,111,255,0.65)); }

.sidebar-footer {
  margin-top:auto;
  border-top:1px solid var(--border);
  padding-top:14px;
}
.user-card {
  display:flex; align-items:center; gap:10px;
  padding:9px 8px; border-radius:12px;
  cursor:pointer; transition:background 0.15s;
}
.user-card:hover { background:rgba(255,255,255,0.04); }
.avatar {
  width:36px; height:36px; border-radius:50%;
  background:linear-gradient(135deg, rgba(124,111,255,0.25), rgba(245,166,35,0.2));
  border:2px solid rgba(124,111,255,0.4);
  display:flex; align-items:center; justify-content:center;
  font-size:11px; font-weight:800; color:var(--teal2);
  flex-shrink:0;
}
.user-name  { font-size:12.5px; font-weight:700; line-height:1.3; }
.user-plan  { font-size:10px; color:var(--teal); display:flex; align-items:center; gap:4px; font-family:var(--mono); }
.live-dot   { width:5px; height:5px; border-radius:50%; background:var(--teal); animation:pulse 2s infinite; }

/* ── Main ── */
#main { flex:1; overflow-y:auto; display:flex; flex-direction:column; }

/* ── Ticker ── */
#ticker {
  background:rgba(124,111,255,0.05);
  border-bottom:1px solid rgba(124,111,255,0.12);
  padding:7px 0; overflow:hidden; white-space:nowrap;
}
.ticker-inner { display:inline-flex; gap:0; animation:ticker 30s linear infinite; }
.ticker-item {
  display:inline-flex; align-items:center; gap:6px;
  padding:0 22px; font-size:10.5px; font-family:var(--mono);
  border-right:1px solid rgba(255,255,255,0.06);
}
.ticker-sym   { color:var(--muted2); font-weight:700; }
.ticker-price { color:var(--text); }
.ticker-up    { color:var(--teal); }
.ticker-dn    { color:var(--red); }

/* ── Header ── */
#header {
  position:sticky; top:0; z-index:50;
  background:rgba(7,8,15,0.88);
  backdrop-filter:blur(20px);
  -webkit-backdrop-filter:blur(20px);
  border-bottom:1px solid var(--border);
  padding:14px 28px;
  display:flex; align-items:center; gap:14px;
}
.header-title h1 {
  font-family:var(--serif);
  font-size:19px;
  letter-spacing:-0.01em;
  font-weight:400;
}
.header-title p { color:var(--muted); font-size:10.5px; margin-top:2px; font-family:var(--mono); }

.search-bar {
  display:flex; align-items:center; gap:8px;
  background:var(--surface); border:1px solid var(--border);
  border-radius:10px; padding:7px 13px; min-width:200px; margin-left:auto;
}
.search-bar input {
  background:none; border:none; outline:none;
  color:var(--text); font-size:12px; font-family:var(--font); width:100%;
}
.search-bar input::placeholder { color:var(--muted); }

.icon-btn {
  width:36px; height:36px; border-radius:10px;
  background:var(--surface); border:1px solid var(--border);
  display:flex; align-items:center; justify-content:center;
  cursor:pointer; transition:all 0.15s; position:relative;
}
.icon-btn:hover { background:rgba(255,255,255,0.07); }
.notif-dot {
  position:absolute; top:8px; right:8px;
  width:7px; height:7px; border-radius:50%;
  background:var(--amber); border:2px solid var(--bg);
}
.btn-new {
  display:flex; align-items:center; gap:6px;
  padding:8px 16px; border-radius:10px; border:none;
  background:rgba(124,111,255,0.18);
  color:var(--teal2); font-size:12px; font-weight:700;
  cursor:pointer; transition:all 0.2s;
  font-family:var(--font); letter-spacing:-0.01em;
  box-shadow:inset 0 0 0 1px rgba(124,111,255,0.35);
}
.btn-new:hover { background:rgba(124,111,255,0.32); transform:translateY(-1px); }

/* ── Content ── */
#content { padding:22px 26px; display:flex; flex-direction:column; gap:18px; }

/* ── Cards ── */
.card {
  background:var(--surface);
  border:1px solid var(--border);
  border-radius:var(--r2);
  backdrop-filter:blur(12px);
  -webkit-backdrop-filter:blur(12px);
  transition:transform 0.2s, box-shadow 0.2s;
}
.card:hover { transform:translateY(-1px); box-shadow:0 12px 40px rgba(0,0,0,0.38); }

/* ── Grids ── */
.grid2          { display:grid; grid-template-columns:1fr 1fr; gap:18px; }
.grid4          { display:grid; grid-template-columns:repeat(4,1fr); gap:16px; }
.grid-3-2       { display:grid; grid-template-columns:1.5fr 1fr; gap:18px; }
.grid-left-right{ display:grid; grid-template-columns:1.6fr 1fr; gap:18px; }

/* ── Portfolio Card ── */
#portfolio-card { padding:28px; animation:fadeUp 0.5s ease 0.05s both; }
.balance-label  { font-size:9px; color:var(--muted); letter-spacing:0.1em; text-transform:uppercase; margin-bottom:6px; font-family:var(--mono); }
.balance-row    { display:flex; align-items:center; gap:12px; }
.balance-amount {
  font-family:var(--mono);
  font-size:34px; font-weight:700;
  letter-spacing:-0.04em;
  background:linear-gradient(90deg, var(--teal2), var(--sky));
  -webkit-background-clip:text;
  -webkit-text-fill-color:transparent;
  background-clip:text;
}
.eye-btn { background:none; border:none; cursor:pointer; color:var(--muted); padding:4px; display:flex; }
.badge-green {
  display:flex; align-items:center; gap:4px;
  background:rgba(46,204,142,0.12); color:var(--green);
  border-radius:6px; padding:3px 9px; font-size:11px; font-weight:700;
  font-family:var(--mono);
}
.badge-red {
  background:rgba(240,85,101,0.12); color:var(--red);
  border-radius:6px; padding:3px 9px; font-size:11px; font-weight:700;
  font-family:var(--mono); display:flex; align-items:center; gap:4px;
}
.badge-amber {
  display:flex; align-items:center; gap:4px;
  background:rgba(245,166,35,0.12); color:var(--amber2);
  border-radius:6px; padding:3px 9px; font-size:11px; font-weight:700;
  font-family:var(--mono);
}
.period-tabs { display:flex; gap:3px; }
.period-btn {
  padding:5px 10px; border-radius:7px; border:none;
  background:transparent; color:var(--muted);
  font-size:10.5px; font-family:var(--mono);
  cursor:pointer; transition:all 0.15s; font-weight:600;
}
.period-btn:hover  { background:rgba(255,255,255,0.06); color:var(--text); }
.period-btn.active { background:rgba(124,111,255,0.18); color:var(--teal2); }

.alloc-label { display:flex; align-items:center; gap:5px; font-size:10px; color:var(--muted); }
.alloc-dot   { width:6px; height:6px; border-radius:50%; flex-shrink:0; }
.alloc-val   { font-size:12px; font-weight:700; font-family:var(--mono); margin-top:2px; }
.alloc-pct   { font-size:10px; color:var(--muted); }

/* ── Credit Card ── */
#cc-panel { display:flex; flex-direction:column; gap:14px; animation:fadeUp 0.5s ease 0.1s both; }
.credit-card {
  border-radius:var(--r2); padding:26px;
  background:linear-gradient(135deg,#0E0A2E 0%,#080520 55%,#040212 100%);
  border:1px solid rgba(124,111,255,0.2);
  position:relative; overflow:hidden;
  box-shadow:0 20px 60px rgba(0,0,0,0.6), 0 0 40px rgba(124,111,255,0.08);
}
.cc-glow1 { position:absolute; top:-60px; right:-40px; width:180px; height:180px; border-radius:50%; background:radial-gradient(circle,rgba(124,111,255,0.14),transparent 70%); pointer-events:none; }
.cc-glow2 { position:absolute; bottom:-50px; left:-30px; width:150px; height:150px; border-radius:50%; background:radial-gradient(circle,rgba(96,165,250,0.08),transparent 70%); pointer-events:none; }
.cc-grid  { position:absolute; inset:0; background-image:linear-gradient(rgba(124,111,255,0.03) 1px,transparent 1px),linear-gradient(90deg,rgba(124,111,255,0.03) 1px,transparent 1px); background-size:30px 30px; pointer-events:none; }
.cc-top   { display:flex; justify-content:space-between; align-items:flex-start; margin-bottom:22px; position:relative; }
.cc-brand { font-family:var(--serif); font-size:10px; color:rgba(232,240,239,0.45); letter-spacing:0.1em; text-transform:uppercase; }
.cc-name-tier { font-family:var(--serif); font-size:17px; color:var(--teal2); margin-top:3px; }
.cc-circles { display:flex; }
.cc-circle1 { width:28px; height:28px; border-radius:50%; background:var(--amber); opacity:0.85; }
.cc-circle2 { width:28px; height:28px; border-radius:50%; background:var(--coral); opacity:0.72; margin-left:-11px; }
.cc-chip {
  width:38px; height:28px; border-radius:5px;
  background:linear-gradient(135deg,#C8A84B,#8B7230);
  border:1px solid rgba(200,168,75,0.3);
  margin-bottom:14px; position:relative;
}
.cc-chip::before { content:''; position:absolute; top:33%; left:0; right:0; height:1px; background:rgba(0,0,0,0.3); }
.cc-chip::after  { content:''; position:absolute; bottom:33%; left:0; right:0; height:1px; background:rgba(0,0,0,0.3); }
.cc-number  { font-family:var(--mono); font-size:14px; letter-spacing:0.15em; color:rgba(232,240,239,0.85); margin-bottom:18px; position:relative; }
.cc-bottom  { display:flex; justify-content:space-between; align-items:flex-end; position:relative; }
.cc-field-label { font-size:8px; color:rgba(232,240,239,0.32); letter-spacing:0.1em; text-transform:uppercase; font-family:var(--mono); }
.cc-field-val   { font-size:12.5px; font-weight:700; margin-top:2px; }
.cc-visa        { font-family:var(--serif); font-size:22px; color:rgba(232,240,239,0.07); letter-spacing:0; font-style:italic; }

.spend-card     { padding:18px; }
.spend-header   { display:flex; justify-content:space-between; align-items:baseline; margin-bottom:10px; }
.spend-title    { font-size:10px; color:var(--muted); text-transform:uppercase; letter-spacing:0.08em; font-family:var(--mono); }
.spend-amount   { font-size:20px; font-weight:800; font-family:var(--mono); }
.spend-left     { color:var(--teal); font-size:11px; font-weight:700; font-family:var(--mono); }
.progress-track { height:4px; background:rgba(255,255,255,0.07); border-radius:4px; overflow:hidden; margin-bottom:5px; }
.progress-fill  { height:100%; border-radius:4px; background:linear-gradient(90deg,var(--teal),var(--amber)); box-shadow:0 0 8px rgba(124,111,255,0.4); }
.spend-limit    { font-size:10px; color:var(--muted); font-family:var(--mono); }
.cat-bars       { display:flex; gap:6px; margin-top:12px; }
.cat-bar        { flex:1; }
.cat-line       { height:3px; border-radius:2px; margin-bottom:4px; }
.cat-name       { font-size:9px; color:var(--muted); }
.cat-pct        { font-size:10px; font-weight:700; font-family:var(--mono); margin-top:2px; }

/* ── Stat Cards ── */
.stat-card    { padding:20px 22px; animation:fadeUp 0.5s ease both; }
.stat-header  { display:flex; justify-content:space-between; align-items:flex-start; margin-bottom:10px; }
.stat-label   { font-size:10px; color:var(--muted); text-transform:uppercase; letter-spacing:0.08em; margin-bottom:5px; font-family:var(--mono); }
.stat-val     { font-size:21px; font-weight:800; font-family:var(--mono); }
.stat-icon    { width:36px; height:36px; border-radius:10px; display:flex; align-items:center; justify-content:center; flex-shrink:0; }
.stat-footer  { display:flex; align-items:center; justify-content:space-between; margin-top:8px; }
.stat-sub     { font-size:10px; color:var(--muted); font-family:var(--mono); }

/* ── Chart Card ── */
.chart-card   { padding:22px; animation:fadeUp 0.5s ease 0.18s both; }
.chart-title  { font-family:var(--serif); font-size:16px; font-weight:400; }
.chart-price  { font-family:var(--mono); font-size:20px; font-weight:700; }
.chart-header { display:flex; justify-content:space-between; align-items:flex-start; margin-bottom:16px; }
#candleSvg    { width:100%; overflow:visible; }

/* ── AI Insights ── */
.insights-card { padding:22px; animation:fadeUp 0.5s ease 0.22s both; }
.ai-header     { display:flex; align-items:center; gap:10px; margin-bottom:16px; }
.ai-icon {
  width:36px; height:36px; border-radius:10px;
  background:linear-gradient(135deg,rgba(124,111,255,0.18),rgba(245,166,35,0.18));
  border:1px solid rgba(124,111,255,0.2);
  display:flex; align-items:center; justify-content:center;
}
.ai-title     { font-family:var(--serif); font-size:15px; font-weight:400; }
.ai-sub       { font-size:10px; color:var(--muted); font-family:var(--mono); }
.ai-live      { margin-left:auto; display:flex; align-items:center; gap:5px; }
.ai-live-dot  { width:6px; height:6px; border-radius:50%; background:var(--teal); box-shadow:0 0 6px var(--teal); animation:pulse 2s infinite; }
.ai-live-text { font-size:10px; color:var(--teal); font-family:var(--mono); }

.insight-item {
  display:flex; align-items:flex-start; gap:10px;
  padding:11px; border-radius:12px;
  background:rgba(255,255,255,0.024);
  border:1px solid rgba(255,255,255,0.05);
  transition:all 0.2s; cursor:pointer; margin-bottom:8px;
}
.insight-item:hover { background:rgba(255,255,255,0.048); transform:translateX(3px); }
.insight-icon  { width:28px; height:28px; border-radius:8px; display:flex; align-items:center; justify-content:center; flex-shrink:0; }
.insight-title { font-size:11.5px; font-weight:700; margin-bottom:3px; }
.insight-desc  { font-size:10px; color:var(--muted); line-height:1.55; font-family:var(--mono); }
.insight-tag   { font-size:9px; font-weight:700; border-radius:4px; padding:1px 6px; text-transform:uppercase; letter-spacing:0.04em; margin-left:6px; flex-shrink:0; font-family:var(--mono); }
.btn-ai {
  width:100%; padding:10px; border-radius:10px; border:none;
  background:rgba(124,111,255,0.12); color:var(--teal2);
  font-size:11.5px; font-weight:700; cursor:pointer;
  transition:all 0.2s; font-family:var(--font);
  box-shadow:inset 0 0 0 1px rgba(124,111,255,0.22);
  display:flex; align-items:center; justify-content:center; gap:6px;
  margin-top:4px;
}
.btn-ai:hover { background:rgba(124,111,255,0.24); }

/* ── Transactions ── */
.txn-card    { padding:22px; animation:fadeUp 0.5s ease 0.28s both; }
.section-header { display:flex; justify-content:space-between; align-items:center; margin-bottom:16px; }
.section-title  { font-family:var(--serif); font-size:16px; font-weight:400; }
.view-all       { font-size:11px; color:var(--teal2); font-weight:700; cursor:pointer; display:flex; align-items:center; gap:3px; background:none; border:none; font-family:var(--font); }
.txn-row        { display:flex; align-items:center; gap:12px; padding:10px 8px; border-radius:11px; transition:background 0.15s; cursor:default; }
.txn-row:hover  { background:rgba(255,255,255,0.03); }
.txn-avatar     { width:37px; height:37px; border-radius:11px; display:flex; align-items:center; justify-content:center; font-size:9.5px; font-weight:800; flex-shrink:0; }
.txn-name       { font-size:13px; font-weight:700; line-height:1.3; }
.txn-meta       { font-size:10px; color:var(--muted); font-family:var(--mono); }
.txn-status     { font-size:9px; font-weight:700; border-radius:6px; padding:2px 8px; text-transform:capitalize; flex-shrink:0; font-family:var(--mono); }
.txn-amt        { text-align:right; font-family:var(--mono); font-size:13px; font-weight:700; min-width:80px; }

/* ── Crypto ── */
.crypto-card    { padding:22px; animation:fadeUp 0.5s ease 0.32s both; }
.crypto-row     { display:flex; align-items:center; gap:12px; padding:9px 8px; border-radius:11px; transition:all 0.15s; cursor:pointer; }
.crypto-row:hover { background:rgba(255,255,255,0.04); transform:translateX(2px); }
.crypto-icon    { width:34px; height:34px; border-radius:50%; display:flex; align-items:center; justify-content:center; font-size:10px; font-weight:800; flex-shrink:0; }
.crypto-name    { font-size:12.5px; font-weight:700; }
.crypto-sym     { font-size:10px; color:var(--muted); font-family:var(--mono); }
.crypto-val     { font-family:var(--mono); font-size:12.5px; font-weight:700; text-align:right; }
.crypto-chg     { font-size:10px; font-family:var(--mono); font-weight:700; text-align:right; display:flex; align-items:center; gap:2px; justify-content:flex-end; }
.alloc-section  { border-top:1px solid var(--border); padding-top:12px; margin-top:12px; }
.alloc-section-label { display:flex; justify-content:space-between; align-items:center; margin-bottom:8px; }
.allocation-bar { display:flex; height:4px; border-radius:4px; overflow:hidden; gap:1px; margin-bottom:6px; }

/* ── Section Views ── */
.section-view { display:none; }
.section-view.active { display:flex; flex-direction:column; gap:18px; animation:fadeUp 0.35s ease both; }

/* ── Wallet Section ── */
.wallet-balance-big { font-family:var(--mono); font-size:36px; font-weight:700; letter-spacing:-0.04em;
  background:linear-gradient(90deg, var(--teal2), var(--amber2));
  -webkit-background-clip:text; -webkit-text-fill-color:transparent; background-clip:text;
}
.wallet-actions { display:flex; gap:10px; margin-top:18px; }
.w-action-btn {
  flex:1; padding:10px 6px; border-radius:11px; border:none; cursor:pointer;
  font-family:var(--font); font-size:12px; font-weight:700; display:flex;
  align-items:center; justify-content:center; gap:7px; transition:all 0.2s;
}
.w-action-btn.primary  { background:rgba(124,111,255,0.16); color:var(--teal2); box-shadow:inset 0 0 0 1px rgba(124,111,255,0.28); }
.w-action-btn.primary:hover  { background:rgba(124,111,255,0.3); }
.w-action-btn.secondary { background:rgba(245,166,35,0.12); color:var(--amber2); box-shadow:inset 0 0 0 1px rgba(245,166,35,0.22); }
.w-action-btn.secondary:hover { background:rgba(245,166,35,0.24); }
.w-action-btn.ghost    { background:rgba(255,255,255,0.04); color:var(--muted2); box-shadow:inset 0 0 0 1px var(--border2); }
.w-action-btn.ghost:hover { background:rgba(255,255,255,0.08); }

.account-row  { display:flex; align-items:center; gap:12px; padding:12px 8px; border-radius:13px; transition:background 0.15s; cursor:pointer; }
.account-row:hover { background:rgba(255,255,255,0.04); }
.account-icon { width:40px; height:40px; border-radius:12px; display:flex; align-items:center; justify-content:center; flex-shrink:0; }
.account-name { font-size:13px; font-weight:700; }
.account-type { font-size:10px; color:var(--muted); font-family:var(--mono); }
.account-bal  { font-family:var(--mono); font-size:14px; font-weight:700; text-align:right; }
.account-chg  { font-size:10px; text-align:right; font-family:var(--mono); }
.account-arrow { color:var(--muted); margin-left:4px; }

/* ── Analytics ── */
.analytics-kpi  { padding:20px 22px; }
.kpi-val        { font-size:24px; font-weight:800; font-family:var(--mono); }
.kpi-label      { font-size:10px; color:var(--muted); text-transform:uppercase; letter-spacing:0.08em; margin-bottom:6px; font-family:var(--mono); }
.kpi-bar-track  { height:4px; background:rgba(255,255,255,0.06); border-radius:4px; overflow:hidden; margin-top:10px; }
.kpi-bar-fill   { height:100%; border-radius:4px; }

.pie-ring-wrap  { display:flex; align-items:center; gap:24px; padding:20px; }
.pie-legend-item { display:flex; align-items:center; gap:8px; margin-bottom:10px; cursor:pointer; }
.pie-legend-dot  { width:8px; height:8px; border-radius:50%; flex-shrink:0; }
.pie-legend-name { font-size:12px; font-weight:600; }
.pie-legend-val  { font-size:12px; font-family:var(--mono); font-weight:700; }
.pie-legend-pct  { font-size:10px; color:var(--muted); font-family:var(--mono); }

.monthly-bar-wrap  { padding:22px; }
.monthly-bar-chart { display:flex; align-items:flex-end; gap:8px; height:120px; margin-top:16px; }
.mb-col    { flex:1; display:flex; flex-direction:column; align-items:center; gap:4px; }
.mb-bar-wrap { width:100%; display:flex; flex-direction:column; align-items:center; gap:2px; flex:1; justify-content:flex-end; }
.mb-bar    { width:100%; border-radius:5px 5px 0 0; transition:opacity 0.2s; cursor:pointer; min-height:4px; }
.mb-bar:hover { opacity:0.7; }
.mb-month  { font-size:9px; color:var(--muted); font-family:var(--mono); }

/* ── Payments ── */
.send-form    { padding:26px; }
.send-title   { font-family:var(--serif); font-size:16px; font-weight:400; margin-bottom:20px; }
.input-group  { margin-bottom:14px; }
.input-label  { font-size:10px; color:var(--muted); text-transform:uppercase; letter-spacing:0.08em; margin-bottom:6px; display:block; font-family:var(--mono); }
.input-field {
  width:100%; background:rgba(255,255,255,0.04); border:1px solid var(--border2);
  border-radius:10px; padding:10px 14px; color:var(--text); font-family:var(--mono);
  font-size:13px; outline:none; transition:border-color 0.2s;
}
.input-field:focus  { border-color:rgba(124,111,255,0.5); }
.input-field::placeholder { color:var(--muted); }
.pay-select {
  width:100%; background:rgba(255,255,255,0.04); border:1px solid var(--border2);
  border-radius:10px; padding:10px 14px; color:var(--text); font-family:var(--font);
  font-size:13px; outline:none; cursor:pointer; appearance:none;
}
.send-btn {
  width:100%; padding:13px; border-radius:11px; border:none;
  background:linear-gradient(135deg,var(--teal),#09B09A);
  color:#060D0C; font-family:var(--font); font-size:14px; font-weight:800;
  cursor:pointer; transition:all 0.2s; letter-spacing:-0.01em;
  box-shadow:0 8px 24px rgba(124,111,255,0.3);
}
.send-btn:hover { transform:translateY(-1px); box-shadow:0 12px 32px rgba(124,111,255,0.4); }

.recent-pay-row { display:flex; align-items:center; gap:12px; padding:11px 8px; border-radius:11px; transition:background 0.15s; cursor:pointer; }
.recent-pay-row:hover { background:rgba(255,255,255,0.04); }
.pay-avatar { width:38px; height:38px; border-radius:50%; display:flex; align-items:center; justify-content:center; font-size:11px; font-weight:800; flex-shrink:0; }

.sched-row  { display:flex; align-items:center; gap:12px; padding:11px 8px; border-radius:11px; transition:background 0.15s; }
.sched-row:hover { background:rgba(255,255,255,0.04); }
.sched-icon { width:36px; height:36px; border-radius:10px; display:flex; align-items:center; justify-content:center; flex-shrink:0; }
.sched-name { font-size:13px; font-weight:700; }
.sched-freq { font-size:10px; color:var(--muted); font-family:var(--mono); }
.sched-amt  { font-family:var(--mono); font-size:13px; font-weight:700; }
.sched-next { font-size:10px; color:var(--muted); text-align:right; font-family:var(--mono); }
.toggle-sw  { width:36px; height:20px; border-radius:10px; background:rgba(255,255,255,0.1); position:relative; cursor:pointer; transition:background 0.2s; flex-shrink:0; }
.toggle-sw.on { background:rgba(124,111,255,0.35); }
.toggle-sw::after { content:''; position:absolute; top:3px; left:3px; width:14px; height:14px; border-radius:50%; background:#fff; transition:transform 0.2s; }
.toggle-sw.on::after { transform:translateX(16px); }

/* ── Investments ── */
.inv-row  { display:flex; align-items:center; gap:12px; padding:11px 8px; border-radius:11px; transition:all 0.15s; cursor:pointer; }
.inv-row:hover { background:rgba(255,255,255,0.04); transform:translateX(2px); }
.inv-icon { width:36px; height:36px; border-radius:10px; display:flex; align-items:center; justify-content:center; font-size:11px; font-weight:800; flex-shrink:0; }
.inv-ticker  { font-size:13px; font-weight:800; font-family:var(--mono); }
.inv-company { font-size:10px; color:var(--muted); }
.inv-shares  { font-size:10px; color:var(--muted); font-family:var(--mono); }
.inv-val     { font-family:var(--mono); font-size:13px; font-weight:700; text-align:right; }
.inv-gain    { font-size:10.5px; font-family:var(--mono); font-weight:700; text-align:right; }
.inv-wt      { font-size:10px; color:var(--muted); text-align:right; font-family:var(--mono); }
.inv-bar-mini { height:3px; border-radius:2px; margin-top:4px; }
.bond-row  { display:flex; align-items:center; gap:12px; padding:10px 8px; border-radius:11px; transition:background 0.15s; }
.bond-row:hover { background:rgba(255,255,255,0.04); }

/* ── Settings ── */
.settings-section-title { font-size:10px; color:var(--muted); text-transform:uppercase; letter-spacing:0.1em; padding:0 4px 8px; margin-top:4px; font-family:var(--mono); }
.settings-row  { display:flex; align-items:center; gap:14px; padding:13px 12px; border-radius:13px; transition:background 0.15s; cursor:pointer; }
.settings-row:hover { background:rgba(255,255,255,0.04); }
.settings-icon { width:36px; height:36px; border-radius:10px; display:flex; align-items:center; justify-content:center; flex-shrink:0; }
.settings-label { font-size:13px; font-weight:700; }
.settings-sub   { font-size:10px; color:var(--muted); margin-top:1px; font-family:var(--mono); }
.settings-arrow { margin-left:auto; color:var(--muted); }
.settings-value { margin-left:auto; font-size:11px; color:var(--muted2); font-family:var(--mono); }

.profile-header { display:flex; align-items:center; gap:18px; padding:24px; border-bottom:1px solid var(--border); margin-bottom:8px; }
.profile-avatar-lg {
  width:68px; height:68px; border-radius:50%; flex-shrink:0;
  background:linear-gradient(135deg,rgba(124,111,255,0.3),rgba(245,166,35,0.3));
  border:2px solid rgba(124,111,255,0.35);
  display:flex; align-items:center; justify-content:center;
  font-size:22px; font-weight:800; color:var(--teal2);
}
.profile-name  { font-family:var(--serif); font-size:20px; font-weight:400; }
.profile-email { font-size:11px; color:var(--muted); margin-top:3px; font-family:var(--mono); }
.profile-plan-badge { display:inline-flex; align-items:center; gap:5px; background:rgba(124,111,255,0.1); color:var(--teal); border-radius:6px; padding:3px 10px; font-size:10px; font-weight:700; margin-top:6px; font-family:var(--mono); }
.edit-profile-btn {
  margin-left:auto; padding:8px 18px; border-radius:10px; border:none;
  background:rgba(124,111,255,0.12); color:var(--teal2);
  font-family:var(--font); font-size:12px; font-weight:700; cursor:pointer;
  box-shadow:inset 0 0 0 1px rgba(124,111,255,0.22); transition:all 0.2s;
}
.edit-profile-btn:hover { background:rgba(124,111,255,0.25); }

.danger-btn {
  padding:9px 20px; border-radius:10px; border:none;
  background:rgba(240,85,101,0.1); color:var(--red);
  font-family:var(--font); font-size:12px; font-weight:700; cursor:pointer;
  box-shadow:inset 0 0 0 1px rgba(240,85,101,0.2); transition:all 0.2s;
}
.danger-btn:hover { background:rgba(240,85,101,0.2); }

/* Toast */
.toast {
  position:fixed; bottom:24px; right:24px; z-index:999;
  background:var(--bg3); border:1px solid var(--border2);
  border-radius:14px; padding:12px 18px; display:flex; align-items:center; gap:10px;
  font-size:12.5px; font-weight:600; box-shadow:0 16px 48px rgba(0,0,0,0.5);
  transform:translateY(20px); opacity:0; transition:all 0.3s ease;
  pointer-events:none; font-family:var(--mono);
}
.toast.show { transform:translateY(0); opacity:1; }

/* Wallet balance card */
.wallet-balance-card { padding:28px; }
</style>
</head>
<body>

<!-- ══ SIDEBAR ══ -->
<nav id="sidebar">
  <div class="logo">
    <div class="logo-icon">◈</div>
    <span class="logo-text">ArcWealth</span>
  </div>

  <div class="nav-section-label">Overview</div>
  <a class="nav-item active" href="#" data-section="dashboard" onclick="navigate(this,'dashboard');return false;">
    <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round"><rect x="3" y="3" width="7" height="7"/><rect x="14" y="3" width="7" height="7"/><rect x="14" y="14" width="7" height="7"/><rect x="3" y="14" width="7" height="7"/></svg>
    Dashboard
  </a>
  <a class="nav-item" href="#" data-section="wallet" onclick="navigate(this,'wallet');return false;">
    <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round"><rect x="2" y="5" width="20" height="14" rx="2"/><line x1="2" y1="10" x2="22" y2="10"/></svg>
    Wallet
  </a>
  <a class="nav-item" href="#" data-section="analytics" onclick="navigate(this,'analytics');return false;">
    <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round"><line x1="18" y1="20" x2="18" y2="10"/><line x1="12" y1="20" x2="12" y2="4"/><line x1="6" y1="20" x2="6" y2="14"/></svg>
    Analytics
  </a>
  <a class="nav-item" href="#" data-section="payments" onclick="navigate(this,'payments');return false;">
    <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round"><line x1="12" y1="1" x2="12" y2="23"/><path d="M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6"/></svg>
    Payments
  </a>

  <div class="nav-section-label">Finance</div>
  <a class="nav-item" href="#" data-section="investments" onclick="navigate(this,'investments');return false;">
    <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round"><polyline points="23 6 13.5 15.5 8.5 10.5 1 18"/><polyline points="17 6 23 6 23 12"/></svg>
    Investments
  </a>
  <a class="nav-item" href="#" data-section="settings" onclick="navigate(this,'settings');return false;">
    <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round"><circle cx="12" cy="12" r="3"/><path d="M19.07 4.93a10 10 0 0 1 0 14.14"/><path d="M4.93 4.93a10 10 0 0 0 0 14.14"/></svg>
    Settings
  </a>

  <div class="sidebar-footer">
    <div class="user-card">
      <div class="avatar">AK</div>
      <div>
        <div class="user-name">Ankit Kumar</div>
        <div class="user-plan"><span class="live-dot"></span>Pro Plan</div>
      </div>
    </div>
  </div>
</nav>

<!-- ══ MAIN ══ -->
<div id="main">

  <!-- Ticker -->
  <div id="ticker">
    <div class="ticker-inner" id="tickerInner"></div>
  </div>

  <!-- Header -->
  <div id="header">
    <div class="header-title">
      <h1 id="headerTitle">Good morning, Ankit 👋</h1>
      <p id="headerSub">Friday, 29 May 2026 &nbsp;·&nbsp; Markets are Live</p>
    </div>
    <div class="search-bar">
      <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="#4A6460" stroke-width="2.2"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg>
      <input placeholder="Search assets, transactions…"/>
    </div>
    <div class="icon-btn">
      <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="#4A6460" stroke-width="2.2"><path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"/><path d="M13.73 21a2 2 0 0 1-3.46 0"/></svg>
      <div class="notif-dot"></div>
    </div>
    <button class="btn-new">
      <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/></svg>
      New Trade
    </button>
  </div>

  <!-- Content -->
  <div id="content">

  <!-- ══ DASHBOARD SECTION ══ -->
  <div id="sec-dashboard" class="section-view active">

    <!-- ROW 1: Portfolio + Card Panel -->
    <div class="grid2">

      <!-- Portfolio Card -->
      <div class="card" id="portfolio-card">
        <div class="balance-label">Total Portfolio Value</div>
        <div class="balance-row" style="margin-bottom:8px">
          <div class="balance-amount" id="balanceAmt">$84,320.42</div>
          <button class="eye-btn" onclick="toggleBalance()" title="Toggle visibility">
            <svg id="eyeIcon" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2"><path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"/><circle cx="12" cy="12" r="3"/></svg>
          </button>
        </div>
        <div style="display:flex;align-items:center;gap:10px;margin-bottom:20px">
          <div class="badge-green">
            <svg width="10" height="10" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="23 6 13.5 15.5 8.5 10.5 1 18"/></svg>
            +$12,820.42
          </div>
          <span style="color:var(--muted);font-size:11px;font-family:var(--mono)">+17.9% all time</span>
          <div style="margin-left:auto" class="period-tabs">
            <button class="period-btn" onclick="setPeriod(this,'1D')">1D</button>
            <button class="period-btn" onclick="setPeriod(this,'1W')">1W</button>
            <button class="period-btn" onclick="setPeriod(this,'1M')">1M</button>
            <button class="period-btn" onclick="setPeriod(this,'6M')">6M</button>
            <button class="period-btn active" onclick="setPeriod(this,'1Y')">1Y</button>
          </div>
        </div>
        <div style="height:160px;position:relative">
          <canvas id="portfolioChart"></canvas>
        </div>
        <div style="display:flex;gap:20px;padding-top:16px;border-top:1px solid var(--border);margin-top:14px">
          <div><div class="alloc-label"><span class="alloc-dot" style="background:var(--teal)"></span>Stocks</div><div class="alloc-val">$41.2K</div><div class="alloc-pct">48.9%</div></div>
          <div><div class="alloc-label"><span class="alloc-dot" style="background:var(--amber)"></span>Crypto</div><div class="alloc-val">$28.6K</div><div class="alloc-pct">34.0%</div></div>
          <div><div class="alloc-label"><span class="alloc-dot" style="background:var(--sky)"></span>Bonds</div><div class="alloc-val">$8.5K</div><div class="alloc-pct">10.1%</div></div>
          <div><div class="alloc-label"><span class="alloc-dot" style="background:var(--muted)"></span>Cash</div><div class="alloc-val">$6.0K</div><div class="alloc-pct">7.0%</div></div>
        </div>
      </div>

      <!-- Credit Card Panel -->
      <div id="cc-panel">
        <div class="credit-card">
          <div class="cc-glow1"></div>
          <div class="cc-glow2"></div>
          <div class="cc-grid"></div>
          <div class="cc-top">
            <div>
              <div class="cc-brand">ArcWealth</div>
              <div class="cc-name-tier">Emerald Elite</div>
            </div>
            <div class="cc-circles">
              <div class="cc-circle1"></div>
              <div class="cc-circle2"></div>
            </div>
          </div>
          <div class="cc-chip"></div>
          <div class="cc-number">5284 •••• •••• 7711</div>
          <div class="cc-bottom">
            <div>
              <div class="cc-field-label">Card Holder</div>
              <div class="cc-field-val">ANKIT KUMAR</div>
            </div>
            <div>
              <div class="cc-field-label">Expires</div>
              <div class="cc-field-val" style="font-family:var(--mono)">11/29</div>
            </div>
            <div class="cc-visa">VISA</div>
          </div>
        </div>

        <div class="card spend-card">
          <div class="spend-header">
            <span class="spend-title">Monthly Spending</span>
            <span class="spend-left">$1,760 left</span>
          </div>
          <div style="display:flex;align-items:baseline;gap:8px;margin-bottom:10px">
            <span class="spend-amount">$3,240</span>
            <span style="color:var(--muted);font-size:11px;font-family:var(--mono)">of $5,000</span>
          </div>
          <div class="progress-track"><div class="progress-fill" style="width:64.8%"></div></div>
          <div class="spend-limit">64.8% of monthly limit</div>
          <div class="cat-bars">
            <div class="cat-bar"><div class="cat-line" style="background:var(--teal)"></div><div class="cat-name">Food</div><div class="cat-pct">28%</div></div>
            <div class="cat-bar"><div class="cat-line" style="background:var(--amber)"></div><div class="cat-name">Travel</div><div class="cat-pct">22%</div></div>
            <div class="cat-bar"><div class="cat-line" style="background:var(--sky)"></div><div class="cat-name">Shop</div><div class="cat-pct">35%</div></div>
            <div class="cat-bar"><div class="cat-line" style="background:var(--muted)"></div><div class="cat-name">Other</div><div class="cat-pct">15%</div></div>
          </div>
        </div>
      </div>
    </div>

    <!-- ROW 2: Stat Cards -->
    <div class="grid4">
      <div class="card stat-card" style="animation-delay:0.1s">
        <div class="stat-header">
          <div><div class="stat-label">Revenue</div><div class="stat-val">$48.2K</div></div>
          <div class="stat-icon" style="background:rgba(124,111,255,0.1);border:1px solid rgba(124,111,255,0.2)">
            <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="#7C6FFF" stroke-width="2.2"><polyline points="22 12 18 12 15 21 9 3 6 12 2 12"/></svg>
          </div>
        </div>
        <div style="height:44px;position:relative"><canvas id="c1"></canvas></div>
        <div class="stat-footer"><span class="stat-sub">vs last month</span><span class="badge-green" style="font-size:10px">+12.5%</span></div>
      </div>
      <div class="card stat-card" style="animation-delay:0.15s">
        <div class="stat-header">
          <div><div class="stat-label">Transactions</div><div class="stat-val">2,847</div></div>
          <div class="stat-icon" style="background:rgba(245,166,35,0.1);border:1px solid rgba(245,166,35,0.2)">
            <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="#F5A623" stroke-width="2.2"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/></svg>
          </div>
        </div>
        <div style="height:44px;position:relative"><canvas id="c2"></canvas></div>
        <div class="stat-footer"><span class="stat-sub">this month</span><span class="badge-green" style="font-size:10px">+8.3%</span></div>
      </div>
      <div class="card stat-card" style="animation-delay:0.2s">
        <div class="stat-header">
          <div><div class="stat-label">Investments</div><div class="stat-val">$124K</div></div>
          <div class="stat-icon" style="background:rgba(56,189,248,0.1);border:1px solid rgba(56,189,248,0.2)">
            <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="#38BDF8" stroke-width="2.2"><polyline points="23 6 13.5 15.5 8.5 10.5 1 18"/><polyline points="17 6 23 6 23 12"/></svg>
          </div>
        </div>
        <div style="height:44px;position:relative"><canvas id="c3"></canvas></div>
        <div class="stat-footer"><span class="stat-sub">total portfolio</span><span class="badge-green" style="font-size:10px">+18.2%</span></div>
      </div>
      <div class="card stat-card" style="animation-delay:0.25s">
        <div class="stat-header">
          <div><div class="stat-label">Expenses</div><div class="stat-val">$12.8K</div></div>
          <div class="stat-icon" style="background:rgba(240,85,101,0.1);border:1px solid rgba(240,85,101,0.2)">
            <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="#F05565" stroke-width="2.2"><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/></svg>
          </div>
        </div>
        <div style="height:44px;position:relative"><canvas id="c4"></canvas></div>
        <div class="stat-footer"><span class="stat-sub">budget: $15K</span><span class="badge-red" style="font-size:10px">-3.2%</span></div>
      </div>
    </div>

    <!-- ROW 3: Candlestick + AI Insights -->
    <div class="grid-left-right">
      <div class="card chart-card">
        <div class="chart-header">
          <div>
            <div class="chart-title">BTC / USD</div>
            <div style="display:flex;align-items:center;gap:10px;margin-top:4px">
              <span class="chart-price">$67,842</span>
              <span class="badge-green" style="font-size:10px">+3.24% today</span>
            </div>
          </div>
          <div class="period-tabs">
            <button class="period-btn">1H</button>
            <button class="period-btn">4H</button>
            <button class="period-btn active">1D</button>
            <button class="period-btn">1W</button>
          </div>
        </div>
        <svg id="candleSvg" height="220"></svg>
        <div style="margin-top:8px">
          <div style="font-size:9px;color:var(--muted);text-transform:uppercase;letter-spacing:0.07em;margin-bottom:5px;font-family:var(--mono)">Volume</div>
          <div id="volBars" style="display:flex;gap:1.5px;align-items:flex-end;height:26px"></div>
        </div>
      </div>

      <div class="card insights-card">
        <div class="ai-header">
          <div class="ai-icon">
            <svg width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="#7C6FFF" stroke-width="2.2"><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/></svg>
          </div>
          <div><div class="ai-title">AI Insights</div><div class="ai-sub">Powered by ArcAI</div></div>
          <div class="ai-live"><div class="ai-live-dot"></div><span class="ai-live-text">Live</span></div>
        </div>

        <div class="insight-item">
          <div class="insight-icon" style="background:rgba(124,111,255,0.12);border:1px solid rgba(124,111,255,0.2)">
            <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="#7C6FFF" stroke-width="2.5"><polyline points="23 6 13.5 15.5 8.5 10.5 1 18"/></svg>
          </div>
          <div style="flex:1">
            <div style="display:flex;align-items:center"><span class="insight-title">Portfolio Momentum</span><span class="insight-tag" style="color:var(--teal);background:rgba(124,111,255,0.12)">Positive</span></div>
            <div class="insight-desc">Your portfolio grew 17.9% this year, outperforming S&P 500 by 4.2 percentage points.</div>
          </div>
        </div>
        <div class="insight-item">
          <div class="insight-icon" style="background:rgba(245,166,35,0.1);border:1px solid rgba(245,166,35,0.2)">
            <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="#F5A623" stroke-width="2.5"><polygon points="13 2 3 14 12 14 11 22 21 10 12 10 13 2"/></svg>
          </div>
          <div style="flex:1">
            <div style="display:flex;align-items:center"><span class="insight-title">Rebalancing Alert</span><span class="insight-tag" style="color:var(--amber2);background:rgba(245,166,35,0.12)">Action</span></div>
            <div class="insight-desc">Crypto at 34% exceeds your 25% target. Consider trimming BTC position.</div>
          </div>
        </div>
        <div class="insight-item">
          <div class="insight-icon" style="background:rgba(56,189,248,0.12);border:1px solid rgba(56,189,248,0.2)">
            <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="#38BDF8" stroke-width="2.5"><circle cx="12" cy="12" r="10"/><line x1="12" y1="8" x2="12" y2="12"/><line x1="12" y1="16" x2="12.01" y2="16"/></svg>
          </div>
          <div style="flex:1">
            <div style="display:flex;align-items:center"><span class="insight-title">Spending Pattern</span><span class="insight-tag" style="color:var(--sky);background:rgba(56,189,248,0.12)">Insight</span></div>
            <div class="insight-desc">Subscriptions up 23% vs last month — 12 active recurring payments detected.</div>
          </div>
        </div>
        <div class="insight-item">
          <div class="insight-icon" style="background:rgba(240,85,101,0.1);border:1px solid rgba(240,85,101,0.2)">
            <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="#F05565" stroke-width="2.5"><path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"/><line x1="12" y1="9" x2="12" y2="13"/><line x1="12" y1="17" x2="12.01" y2="17"/></svg>
          </div>
          <div style="flex:1">
            <div style="display:flex;align-items:center"><span class="insight-title">Risk Exposure</span><span class="insight-tag" style="color:var(--red);background:rgba(240,85,101,0.1)">Warning</span></div>
            <div class="insight-desc">62% concentration in tech sector. Diversification strongly recommended.</div>
          </div>
        </div>

        <button class="btn-ai">
          <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/></svg>
          View Full AI Report
        </button>
      </div>
    </div>

    <!-- ROW 4: Transactions + Crypto -->
    <div class="grid-left-right">

      <div class="card txn-card">
        <div class="section-header">
          <div class="section-title">Recent Transactions</div>
          <button class="view-all">View all <svg width="11" height="11" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="9 18 15 12 9 6"/></svg></button>
        </div>
        <div id="txnList"></div>
      </div>

      <div class="card crypto-card">
        <div class="section-header">
          <div class="section-title">Crypto Assets</div>
          <div style="display:flex;align-items:center;gap:5px">
            <div class="live-dot"></div>
            <span style="font-size:10px;color:var(--muted);font-family:var(--mono)">Live</span>
          </div>
        </div>
        <div id="cryptoList"></div>
        <div class="alloc-section">
          <div class="alloc-section-label">
            <span style="font-size:10px;color:var(--muted);font-family:var(--mono)">Allocation</span>
            <span style="font-size:10px;color:var(--muted);font-family:var(--mono)" id="cryptoTotal"></span>
          </div>
          <div id="cryptoAllocBar" class="allocation-bar" style="height:6px;border-radius:4px;overflow:hidden;gap:1px;margin-bottom:8px"></div>
          <div id="cryptoLegend" style="display:flex;flex-wrap:wrap;gap:8px 14px"></div>
        </div>
      </div>
    </div>

    <!-- Footer -->
    <div style="display:flex;justify-content:space-between;align-items:center;padding:4px 0 8px;border-top:1px solid var(--border);margin-top:4px">
      <span style="font-size:10px;color:var(--muted);font-family:var(--mono)">ArcWealth · Data refreshed 2m ago · All times IST</span>
      <div style="display:flex;gap:18px">
        <span style="font-size:10px;color:var(--muted);cursor:pointer;font-family:var(--mono)">Privacy</span>
        <span style="font-size:10px;color:var(--muted);cursor:pointer;font-family:var(--mono)">Terms</span>
        <span style="font-size:10px;color:var(--muted);cursor:pointer;font-family:var(--mono)">Support</span>
      </div>
    </div>

  </div><!-- /sec-dashboard -->

  <!-- ══ WALLET SECTION ══ -->
  <div id="sec-wallet" class="section-view">
    <div class="grid2">
      <div class="card wallet-balance-card">
        <div class="balance-label">Total Balance</div>
        <div class="wallet-balance-big">$84,320.42</div>
        <div style="display:flex;align-items:center;gap:10px;margin-top:8px;margin-bottom:18px">
          <div class="badge-green"><svg width="10" height="10" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="23 6 13.5 15.5 8.5 10.5 1 18"/></svg>+2.4% today</div>
          <span style="font-size:11px;color:var(--muted);font-family:var(--mono)">Updated just now</span>
        </div>
        <div class="wallet-actions">
          <button class="w-action-btn primary" onclick="showToast('Transfer initiated','var(--teal2)')">
            <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><line x1="12" y1="5" x2="12" y2="19"/><polyline points="19 12 12 19 5 12"/></svg>Send
          </button>
          <button class="w-action-btn secondary" onclick="showToast('Add funds initiated','var(--amber2)')">
            <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><line x1="12" y1="5" x2="12" y2="19"/><polyline points="5 12 12 5 19 12"/></svg>Add
          </button>
          <button class="w-action-btn ghost" onclick="showToast('Exchange feature coming soon','var(--sky)')">
            <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="17 1 21 5 17 9"/><path d="M3 11V9a4 4 0 0 1 4-4h14"/><polyline points="7 23 3 19 7 15"/><path d="M21 13v2a4 4 0 0 1-4 4H3"/></svg>Swap
          </button>
        </div>
      </div>
      <div class="card" style="padding:22px">
        <div class="section-header"><div class="section-title">Quick Stats</div></div>
        <div style="display:grid;grid-template-columns:1fr 1fr;gap:14px">
          <div style="background:rgba(124,111,255,0.07);border:1px solid rgba(124,111,255,0.12);border-radius:13px;padding:14px">
            <div style="font-size:9px;color:var(--muted);text-transform:uppercase;letter-spacing:0.08em;margin-bottom:6px;font-family:var(--mono)">Monthly In</div>
            <div style="font-size:18px;font-weight:800;font-family:var(--mono);color:var(--teal)">+$11,700</div>
            <div style="font-size:10px;color:var(--muted);margin-top:3px;font-family:var(--mono)">3 deposits</div>
          </div>
          <div style="background:rgba(240,85,101,0.07);border:1px solid rgba(240,85,101,0.12);border-radius:13px;padding:14px">
            <div style="font-size:9px;color:var(--muted);text-transform:uppercase;letter-spacing:0.08em;margin-bottom:6px;font-family:var(--mono)">Monthly Out</div>
            <div style="font-size:18px;font-weight:800;font-family:var(--mono);color:var(--red)">−$4,514</div>
            <div style="font-size:10px;color:var(--muted);margin-top:3px;font-family:var(--mono)">18 payments</div>
          </div>
          <div style="background:rgba(245,166,35,0.07);border:1px solid rgba(245,166,35,0.12);border-radius:13px;padding:14px">
            <div style="font-size:9px;color:var(--muted);text-transform:uppercase;letter-spacing:0.08em;margin-bottom:6px;font-family:var(--mono)">Savings Rate</div>
            <div style="font-size:18px;font-weight:800;font-family:var(--mono);color:var(--amber)">61.4%</div>
            <div style="font-size:10px;color:var(--muted);margin-top:3px;font-family:var(--mono)">↑ from 58.1%</div>
          </div>
          <div style="background:rgba(56,189,248,0.07);border:1px solid rgba(56,189,248,0.12);border-radius:13px;padding:14px">
            <div style="font-size:9px;color:var(--muted);text-transform:uppercase;letter-spacing:0.08em;margin-bottom:6px;font-family:var(--mono)">Net Worth</div>
            <div style="font-size:18px;font-weight:800;font-family:var(--mono);color:var(--sky)">$124K</div>
            <div style="font-size:10px;color:var(--muted);margin-top:3px;font-family:var(--mono)">all accounts</div>
          </div>
        </div>
      </div>
    </div>
    <div class="card" style="padding:22px">
      <div class="section-header"><div class="section-title">Linked Accounts</div><button class="view-all" onclick="showToast('Add account','var(--teal2)')">+ Add Account</button></div>
      <div id="walletAccounts"></div>
    </div>
  </div>

  <!-- ══ ANALYTICS SECTION ══ -->
  <div id="sec-analytics" class="section-view">
    <div class="grid4">
      <div class="card analytics-kpi">
        <div class="kpi-label">Total Income</div>
        <div class="kpi-val" style="color:var(--teal)">$117.8K</div>
        <div style="font-size:10px;color:var(--muted);margin-top:4px;font-family:var(--mono)">Fiscal YTD 2026</div>
        <div class="kpi-bar-track"><div class="kpi-bar-fill" style="width:78%;background:var(--teal)"></div></div>
      </div>
      <div class="card analytics-kpi">
        <div class="kpi-label">Total Spend</div>
        <div class="kpi-val" style="color:var(--red)">$46.2K</div>
        <div style="font-size:10px;color:var(--muted);margin-top:4px;font-family:var(--mono)">Fiscal YTD 2026</div>
        <div class="kpi-bar-track"><div class="kpi-bar-fill" style="width:39%;background:var(--red)"></div></div>
      </div>
      <div class="card analytics-kpi">
        <div class="kpi-label">Net Savings</div>
        <div class="kpi-val" style="color:var(--green)">$71.6K</div>
        <div style="font-size:10px;color:var(--muted);margin-top:4px;font-family:var(--mono)">60.8% rate</div>
        <div class="kpi-bar-track"><div class="kpi-bar-fill" style="width:61%;background:var(--green)"></div></div>
      </div>
      <div class="card analytics-kpi">
        <div class="kpi-label">ROI</div>
        <div class="kpi-val" style="color:var(--amber)">+17.9%</div>
        <div style="font-size:10px;color:var(--muted);margin-top:4px;font-family:var(--mono)">vs 13.7% S&P 500</div>
        <div class="kpi-bar-track"><div class="kpi-bar-fill" style="width:65%;background:var(--amber)"></div></div>
      </div>
    </div>

    <div class="grid-left-right">
      <div class="card monthly-bar-wrap">
        <div class="section-header"><div class="section-title">Monthly Overview</div>
          <div style="display:flex;gap:12px">
            <span style="display:flex;align-items:center;gap:5px;font-size:10px;color:var(--muted);font-family:var(--mono)"><span style="width:8px;height:8px;border-radius:2px;background:var(--teal);display:inline-block;opacity:0.7"></span>Income</span>
            <span style="display:flex;align-items:center;gap:5px;font-size:10px;color:var(--muted);font-family:var(--mono)"><span style="width:8px;height:8px;border-radius:2px;background:var(--red);display:inline-block;opacity:0.55"></span>Spend</span>
          </div>
        </div>
        <div class="monthly-bar-chart" id="monthlyBarChart"></div>
      </div>
      <div class="card">
        <div class="pie-ring-wrap">
          <canvas id="allocDonut" width="130" height="130"></canvas>
          <div style="flex:1" id="allocLegend"></div>
        </div>
        <div style="padding:0 20px 16px">
          <div style="font-size:10px;color:var(--muted);font-family:var(--mono);margin-bottom:6px">PORTFOLIO ALLOCATION</div>
        </div>
      </div>
    </div>

    <div class="grid-3-2">
      <div class="card" style="padding:22px">
        <div class="section-header"><div class="section-title">Cash Flow Analysis</div></div>
        <div style="height:200px;position:relative"><canvas id="cashFlowChart"></canvas></div>
      </div>
      <div class="card">
        <div class="pie-ring-wrap" style="flex-direction:column;align-items:flex-start;gap:14px;padding:20px">
          <div class="section-title">Spending by Category</div>
          <div style="display:flex;align-items:center;gap:20px">
            <canvas id="spendDonut" width="110" height="110"></canvas>
            <div id="spendLegend" style="flex:1"></div>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- ══ PAYMENTS SECTION ══ -->
  <div id="sec-payments" class="section-view">
    <div class="grid2">
      <div class="card send-form">
        <div class="send-title">Send Money</div>
        <div class="input-group">
          <label class="input-label">Recipient</label>
          <input class="input-field" placeholder="Name or account number"/>
        </div>
        <div class="input-group">
          <label class="input-label">Amount (INR)</label>
          <input class="input-field" placeholder="₹ 0.00" type="number"/>
        </div>
        <div class="input-group">
          <label class="input-label">Payment Method</label>
          <select class="pay-select">
            <option>ArcWealth Emerald Card ••7711</option>
            <option>HDFC Savings ••4421</option>
            <option>ICICI Current ••8803</option>
          </select>
        </div>
        <div class="input-group">
          <label class="input-label">Note (Optional)</label>
          <input class="input-field" placeholder="What's it for?"/>
        </div>
        <button class="send-btn" onclick="showToast('Payment sent successfully!','var(--teal)')">Send Payment</button>
      </div>
      <div>
        <div class="card" style="padding:22px;margin-bottom:18px">
          <div class="section-header"><div class="section-title">Recent Payments</div></div>
          <div id="recentPayments"></div>
        </div>
        <div class="card" style="padding:22px">
          <div class="section-header"><div class="section-title">Scheduled</div></div>
          <div id="scheduledPay"></div>
        </div>
      </div>
    </div>
  </div>

  <!-- ══ INVESTMENTS SECTION ══ -->
  <div id="sec-investments" class="section-view">
    <div class="grid-left-right">
      <div class="card" style="padding:22px">
        <div class="section-header">
          <div class="section-title">Stock Holdings</div>
          <button class="view-all" onclick="showToast('Opening trade desk','var(--teal2)')">Trade <svg width="11" height="11" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="9 18 15 12 9 6"/></svg></button>
        </div>
        <div style="display:grid;grid-template-columns:2fr 1fr 1fr 1fr;gap:0;padding:6px 8px;margin-bottom:4px">
          <span style="font-size:9px;color:var(--muted);text-transform:uppercase;font-family:var(--mono)">Asset</span>
          <span style="font-size:9px;color:var(--muted);text-transform:uppercase;font-family:var(--mono);text-align:right">Value</span>
          <span style="font-size:9px;color:var(--muted);text-transform:uppercase;font-family:var(--mono);text-align:right">P/L</span>
          <span style="font-size:9px;color:var(--muted);text-transform:uppercase;font-family:var(--mono);text-align:right">Weight</span>
        </div>
        <div id="stockHoldings"></div>
      </div>
      <div style="display:flex;flex-direction:column;gap:18px">
        <div class="card" style="padding:22px">
          <div class="section-header"><div class="section-title">Bond Holdings</div></div>
          <div id="bondHoldings"></div>
        </div>
        <div class="card">
          <div style="padding:20px 20px 0">
            <div class="section-title">Allocation</div>
          </div>
          <div class="pie-ring-wrap" style="padding:16px 20px">
            <canvas id="invAllocDonut" width="120" height="120"></canvas>
            <div style="flex:1" id="invAllocLegend"></div>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- ══ SETTINGS SECTION ══ -->
  <div id="sec-settings" class="section-view">
    <div class="grid2">
      <div class="card">
        <div class="profile-header">
          <div class="profile-avatar-lg">AK</div>
          <div>
            <div class="profile-name">Ankit Kumar</div>
            <div class="profile-email">ankit.kumar@arcwealth.in</div>
            <div class="profile-plan-badge"><span class="live-dot"></span>Pro Plan — Active</div>
          </div>
          <button class="edit-profile-btn" onclick="showToast('Edit profile opened','var(--teal2)')">Edit Profile</button>
        </div>
        <div style="padding:0 12px 12px">
          <div class="settings-section-title">Account</div>
          <div class="settings-row" onclick="showToast('Personal Info','var(--teal2)')">
            <div class="settings-icon" style="background:rgba(124,111,255,0.1);border:1px solid rgba(124,111,255,0.18)">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="#7C6FFF" stroke-width="2.2"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/></svg>
            </div>
            <div><div class="settings-label">Personal Information</div><div class="settings-sub">Name, email, phone</div></div>
            <div class="settings-arrow"><svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2"><polyline points="9 18 15 12 9 6"/></svg></div>
          </div>
          <div class="settings-row" onclick="showToast('Security settings','var(--amber2)')">
            <div class="settings-icon" style="background:rgba(245,166,35,0.1);border:1px solid rgba(245,166,35,0.18)">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="#F5A623" stroke-width="2.2"><rect x="3" y="11" width="18" height="11" rx="2"/><path d="M7 11V7a5 5 0 0 1 10 0v4"/></svg>
            </div>
            <div><div class="settings-label">Security</div><div class="settings-sub">2FA, biometrics, PIN</div></div>
            <div class="settings-arrow"><svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2"><polyline points="9 18 15 12 9 6"/></svg></div>
          </div>
          <div class="settings-row">
            <div class="settings-icon" style="background:rgba(56,189,248,0.1);border:1px solid rgba(56,189,248,0.18)">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="#38BDF8" stroke-width="2.2"><path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"/><path d="M13.73 21a2 2 0 0 1-3.46 0"/></svg>
            </div>
            <div><div class="settings-label">Notifications</div><div class="settings-sub">Alerts & push settings</div></div>
            <div class="settings-value">On</div>
          </div>
          <div class="settings-section-title" style="margin-top:16px">Preferences</div>
          <div class="settings-row">
            <div class="settings-icon" style="background:rgba(124,111,255,0.1);border:1px solid rgba(124,111,255,0.18)">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="#7C6FFF" stroke-width="2.2"><circle cx="12" cy="12" r="10"/><line x1="2" y1="12" x2="22" y2="12"/><path d="M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10 15.3 15.3 0 0 1-4-10 15.3 15.3 0 0 1 4-10z"/></svg>
            </div>
            <div><div class="settings-label">Currency</div><div class="settings-sub">Display currency</div></div>
            <div class="settings-value">INR / USD</div>
          </div>
          <div class="settings-row" onclick="showToast('Danger zone','var(--red)')">
            <div class="settings-icon" style="background:rgba(240,85,101,0.1);border:1px solid rgba(240,85,101,0.18)">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="#F05565" stroke-width="2.2"><polyline points="3 6 5 6 21 6"/><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"/></svg>
            </div>
            <div><div class="settings-label" style="color:var(--red)">Delete Account</div><div class="settings-sub">Permanently remove data</div></div>
            <div class="settings-arrow"><svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2"><polyline points="9 18 15 12 9 6"/></svg></div>
          </div>
        </div>
      </div>

      <div style="display:flex;flex-direction:column;gap:18px">
        <div class="card" style="padding:22px">
          <div class="section-title" style="margin-bottom:14px">Plan & Billing</div>
          <div style="background:linear-gradient(135deg,rgba(124,111,255,0.08),rgba(245,166,35,0.06));border:1px solid rgba(124,111,255,0.16);border-radius:14px;padding:18px">
            <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:10px">
              <div>
                <div style="font-family:var(--serif);font-size:16px">Pro Plan</div>
                <div style="font-size:10px;color:var(--muted);font-family:var(--mono);margin-top:3px">Renews Jun 29, 2026</div>
              </div>
              <div style="font-family:var(--mono);font-size:18px;font-weight:700;color:var(--teal)">$19<span style="font-size:11px;color:var(--muted)">/mo</span></div>
            </div>
            <div style="font-size:11px;color:var(--muted2);font-family:var(--mono)">✓ Unlimited transactions &nbsp;·&nbsp; ✓ AI Insights &nbsp;·&nbsp; ✓ Priority support</div>
          </div>
          <button style="margin-top:14px;width:100%;padding:10px;border-radius:10px;border:none;background:rgba(245,166,35,0.12);color:var(--amber2);font-family:var(--font);font-size:12px;font-weight:700;cursor:pointer;box-shadow:inset 0 0 0 1px rgba(245,166,35,0.22);" onclick="showToast('Upgrade to Enterprise','var(--amber2)')">Upgrade to Enterprise</button>
        </div>
        <div class="card" style="padding:22px">
          <div class="section-title" style="margin-bottom:14px">Linked Devices</div>
          <div class="settings-row">
            <div class="settings-icon" style="background:rgba(124,111,255,0.1);border:1px solid rgba(124,111,255,0.18)">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="#7C6FFF" stroke-width="2.2"><rect x="5" y="2" width="14" height="20" rx="2" ry="2"/><line x1="12" y1="18" x2="12.01" y2="18"/></svg>
            </div>
            <div><div class="settings-label">iPhone 16 Pro</div><div class="settings-sub">iOS 18.4 · Active now</div></div>
            <div style="width:8px;height:8px;border-radius:50%;background:var(--teal);margin-left:auto;box-shadow:0 0 6px var(--teal)"></div>
          </div>
          <div class="settings-row">
            <div class="settings-icon" style="background:rgba(122,158,154,0.1);border:1px solid rgba(122,158,154,0.18)">
              <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="#7A9E9A" stroke-width="2.2"><rect x="2" y="3" width="20" height="14" rx="2"/><line x1="8" y1="21" x2="16" y2="21"/><line x1="12" y1="17" x2="12" y2="21"/></svg>
            </div>
            <div><div class="settings-label">MacBook Pro</div><div class="settings-sub">macOS 15.5 · 2h ago</div></div>
            <div style="width:8px;height:8px;border-radius:50%;background:var(--muted);margin-left:auto"></div>
          </div>
        </div>
      </div>
    </div>
  </div>

  </div><!-- /content -->
</div><!-- /main -->

<!-- Toast -->
<div class="toast" id="toast">
  <span id="toastDot" style="width:8px;height:8px;border-radius:50%;flex-shrink:0"></span>
  <span id="toastMsg"></span>
</div>

<script>
// ─── CONSTANTS ───────────────────────────────────────────────
const TEAL   = '#7C6FFF';
const AMBER  = '#F5A623';
const GREEN  = '#2ECC8E';
const RED    = '#F05565';
const SKY    = '#60A5FA';
const MUTED  = '#4A4670';
const BG3    = '#111328';

function hexToRgba(hex, a) {
  const r = parseInt(hex.slice(1,3),16);
  const g = parseInt(hex.slice(3,5),16);
  const b = parseInt(hex.slice(5,7),16);
  return `rgba(${r},${g},${b},${a})`;
}

// ─── TOAST ───────────────────────────────────────────────────
function showToast(msg, col='var(--teal2)') {
  const t = document.getElementById('toast');
  const d = document.getElementById('toastDot');
  const m = document.getElementById('toastMsg');
  d.style.background = col;
  m.textContent = msg;
  t.classList.add('show');
  clearTimeout(t._timer);
  t._timer = setTimeout(() => t.classList.remove('show'), 2800);
}

// ─── NAVIGATION ──────────────────────────────────────────────
const sectionTitles = {
  dashboard:   ['Good morning, Ankit 👋', 'Friday, 29 May 2026 · Markets are Live'],
  wallet:      ['Wallet Overview', 'Accounts, transfers & balances'],
  analytics:   ['Analytics', 'Your financial health at a glance'],
  payments:    ['Payments', 'Send, receive & schedule'],
  investments: ['Investments', 'Stocks, bonds & crypto portfolio'],
  settings:    ['Settings', 'Account & preferences']
};
function navigate(el, sec) {
  document.querySelectorAll('.nav-item').forEach(i=>i.classList.remove('active'));
  el.classList.add('active');
  document.querySelectorAll('.section-view').forEach(v=>v.classList.remove('active'));
  const target = document.getElementById('sec-'+sec);
  if (target) target.classList.add('active');
  const [h, s] = sectionTitles[sec] || ['Dashboard',''];
  document.getElementById('headerTitle').textContent = h;
  document.getElementById('headerSub').textContent   = s;
  if (sec === 'analytics') initAnalytics();
}

// ─── BALANCE TOGGLE ──────────────────────────────────────────
let balVis = true;
const REAL_BAL = '$84,320.42';
function toggleBalance() {
  balVis = !balVis;
  document.getElementById('balanceAmt').textContent = balVis ? REAL_BAL : '••••••••';
  document.getElementById('eyeIcon').innerHTML = balVis
    ? '<path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z"/><circle cx="12" cy="12" r="3"/>'
    : '<path d="M17.94 17.94A10.07 10.07 0 0 1 12 20c-7 0-11-8-11-8a18.45 18.45 0 0 1 5.06-5.94M9.9 4.24A9.12 9.12 0 0 1 12 4c7 0 11 8 11 8a18.5 18.5 0 0 1-2.16 3.19m-6.72-1.07a3 3 0 1 1-4.24-4.24"/><line x1="1" y1="1" x2="23" y2="23"/>';
}

// ─── PERIOD TABS ─────────────────────────────────────────────
function setPeriod(el, p) {
  el.closest('.period-tabs').querySelectorAll('.period-btn').forEach(b=>b.classList.remove('active'));
  el.classList.add('active');
  updatePortfolioChart(p);
}

// ─── TICKER ──────────────────────────────────────────────────
const tickers = [
  {sym:'BTC',price:'$67,842',chg:'+3.24%',up:true},{sym:'ETH',price:'$3,421',chg:'+1.87%',up:true},
  {sym:'AAPL',price:'$214.52',chg:'+0.63%',up:true},{sym:'TSLA',price:'$248.19',chg:'-1.40%',up:false},
  {sym:'NIFTY',price:'24,198',chg:'+0.55%',up:true},{sym:'SENSEX',price:'79,401',chg:'+0.48%',up:true},
  {sym:'MSFT',price:'$437.60',chg:'+0.91%',up:true},{sym:'GOOGL',price:'$182.30',chg:'-0.22%',up:false},
  {sym:'SOL',price:'$188.40',chg:'+5.12%',up:true},{sym:'GOLD',price:'$3,330',chg:'+0.72%',up:true}
];
const ti = document.getElementById('tickerInner');
[...tickers,...tickers].forEach(t => {
  const el = document.createElement('div');
  el.className = 'ticker-item';
  el.innerHTML = `<span class="ticker-sym">${t.sym}</span><span class="ticker-price">${t.price}</span><span class="${t.up?'ticker-up':'ticker-dn'}">${t.chg}</span>`;
  ti.appendChild(el);
});

// ─── PORTFOLIO CHART ─────────────────────────────────────────
const portfolioDatasets = {
  '1D': [82100,82800,83100,82700,83500,84100,84320],
  '1W': [80200,81500,80900,82300,83100,83800,84320],
  '1M': [79000,80100,78500,81200,82500,83400,84320],
  '6M': [71200,74800,73100,78200,80500,82800,84320],
  '1Y': [42000,48000,55000,63000,70000,78000,84320]
};
let portfolioChart;
function updatePortfolioChart(period='1Y') {
  const data = portfolioDatasets[period] || portfolioDatasets['1Y'];
  if (portfolioChart) { portfolioChart.data.datasets[0].data = data; portfolioChart.data.labels = data.map((_,i)=>i); portfolioChart.update(); return; }
  const ctx = document.getElementById('portfolioChart').getContext('2d');
  const grad = ctx.createLinearGradient(0,0,0,160);
  grad.addColorStop(0, hexToRgba(TEAL,0.3));
  grad.addColorStop(1, hexToRgba(TEAL,0));
  portfolioChart = new Chart(ctx,{
    type:'line',
    data:{ labels:data.map((_,i)=>i), datasets:[{ data, borderColor:TEAL, borderWidth:2, backgroundColor:grad, fill:true, tension:0.4, pointRadius:0, pointHoverRadius:4, pointHoverBackgroundColor:TEAL }] },
    options:{ responsive:true, maintainAspectRatio:false, plugins:{legend:{display:false}, tooltip:{ backgroundColor:BG3, borderColor:'rgba(255,255,255,0.1)', borderWidth:1, titleColor:MUTED, bodyColor:'#E8F0EF', callbacks:{ label:c=>'$'+c.raw.toLocaleString() } }},
      scales:{ x:{display:false}, y:{grid:{color:'rgba(255,255,255,0.04)'}, ticks:{color:MUTED,font:{family:'JetBrains Mono',size:9},callback:v=>'$'+(v/1000).toFixed(0)+'K'}} }
    }
  });
}
updatePortfolioChart('1Y');

// ─── SPARKLINES ──────────────────────────────────────────────
function makeSparkline(id, color, data) {
  const ctx = document.getElementById(id).getContext('2d');
  const g = ctx.createLinearGradient(0,0,0,44);
  g.addColorStop(0, hexToRgba(color,0.22));
  g.addColorStop(1, hexToRgba(color,0));
  new Chart(ctx,{type:'line',data:{labels:data.map((_,i)=>i),datasets:[{data,borderColor:color,borderWidth:1.5,backgroundColor:g,fill:true,tension:0.4,pointRadius:0}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{display:false},tooltip:{enabled:false}},scales:{x:{display:false},y:{display:false}}}});
}
makeSparkline('c1',TEAL, [38,42,39,45,44,48,46,48.2].map(v=>v*1000));
makeSparkline('c2',AMBER,[2200,2400,2350,2500,2600,2700,2800,2847]);
makeSparkline('c3',SKY,  [95,100,98,105,110,115,120,124].map(v=>v*1000));
makeSparkline('c4',RED,  [11,12.5,11.8,13,12.2,13.5,12.8,12.8].map(v=>v*1000));

// ─── CANDLESTICK ─────────────────────────────────────────────
(function(){
  const svg = document.getElementById('candleSvg');
  const W = svg.parentElement.offsetWidth - 44; const H = 220;
  svg.setAttribute('viewBox',`0 0 ${W} ${H}`);
  const candles = Array.from({length:32},(_,i)=>{
    const base = 62000 + Math.sin(i*0.45)*8000 + i*180;
    const o = base + (Math.random()-0.5)*1200;
    const c = base + (Math.random()-0.5)*1200;
    const h = Math.max(o,c) + Math.random()*800;
    const l = Math.min(o,c) - Math.random()*800;
    return {o,c,h,l};
  });
  const allVals = candles.flatMap(c=>[c.h,c.l]);
  const minV = Math.min(...allVals); const maxV = Math.max(...allVals);
  const scale = v => H - ((v-minV)/(maxV-minV))*(H-20) - 10;
  const cw = W/candles.length; const bw = cw*0.55;
  candles.forEach((c,i)=>{
    const x = i*cw + cw/2; const pos = c.c >= c.o;
    const col = pos ? TEAL : RED;
    const wick = document.createElementNS('http://www.w3.org/2000/svg','line');
    wick.setAttribute('x1',x); wick.setAttribute('x2',x);
    wick.setAttribute('y1',scale(c.h)); wick.setAttribute('y2',scale(c.l));
    wick.setAttribute('stroke',col); wick.setAttribute('stroke-width','1');
    wick.setAttribute('opacity','0.6');
    svg.appendChild(wick);
    const rect = document.createElementNS('http://www.w3.org/2000/svg','rect');
    const top = Math.min(scale(c.o),scale(c.c));
    const bot = Math.max(scale(c.o),scale(c.c));
    rect.setAttribute('x', x-bw/2); rect.setAttribute('y', top);
    rect.setAttribute('width', bw); rect.setAttribute('height', Math.max(bot-top,1));
    rect.setAttribute('fill', pos ? hexToRgba(TEAL,0.7) : hexToRgba(RED,0.7));
    rect.setAttribute('rx','2');
    svg.appendChild(rect);
  });
  // Volume
  const vols = candles.map(()=>Math.random()*100+20);
  const maxVol = Math.max(...vols);
  const vb = document.getElementById('volBars');
  vols.forEach((v,i)=>{
    const pos = candles[i].c >= candles[i].o;
    const bar = document.createElement('div');
    bar.style.cssText=`flex:1;background:${pos?hexToRgba(TEAL,0.4):hexToRgba(RED,0.35)};height:${Math.round((v/maxVol)*100)}%;border-radius:2px 2px 0 0;`;
    vb.appendChild(bar);
  });
})();

// ─── TRANSACTIONS ────────────────────────────────────────────
const txns = [
  { name:'Swiggy', cat:'Food & Dining', date:'Today 11:22 AM', amt:'-₹840', pos:false, status:'completed', bg:'rgba(245,166,35,0.12)', col:'#F5A623', initials:'SW' },
  { name:'HDFC Dividend', cat:'Investment', date:'Today 09:00 AM', amt:'+$1,200', pos:true, status:'received', bg:'rgba(124,111,255,0.12)', col:'#7C6FFF', initials:'HD' },
  { name:'Zepto', cat:'Groceries', date:'Yesterday 07:48 PM', amt:'-₹2,140', pos:false, status:'completed', bg:'rgba(56,189,248,0.12)', col:'#38BDF8', initials:'ZP' },
  { name:'Netflix', cat:'Subscription', date:'May 27', amt:'-$15.99', pos:false, status:'recurring', bg:'rgba(240,85,101,0.12)', col:'#F05565', initials:'NF' },
  { name:'Freelance Pay', cat:'Income', date:'May 26', amt:'+$3,500', pos:true, status:'received', bg:'rgba(124,111,255,0.12)', col:'#7C6FFF', initials:'FP' },
  { name:'Zomato Pro', cat:'Subscription', date:'May 25', amt:'-₹299', pos:false, status:'completed', bg:'rgba(255,126,94,0.12)', col:'#FF7B5E', initials:'ZM' },
  { name:'BTC Purchase', cat:'Crypto', date:'May 24', amt:'-$2,100', pos:false, status:'completed', bg:'rgba(245,166,35,0.12)', col:'#F5A623', initials:'₿' },
  { name:'Consulting', cat:'Income', date:'May 22', amt:'+$5,200', pos:true, status:'received', bg:'rgba(46,204,142,0.12)', col:'#2ECC8E', initials:'CS' },
];
const tl = document.getElementById('txnList');
txns.forEach(t => {
  const row = document.createElement('div'); row.className='txn-row';
  const sc = t.status==='completed'?'rgba(122,158,154,0.15)':t.status==='received'?hexToRgba(GREEN,0.15):hexToRgba(AMBER,0.15);
  const stc = t.status==='received'?GREEN:t.status==='recurring'?AMBER:MUTED;
  row.innerHTML=`
    <div class="txn-avatar" style="background:${t.bg};color:${t.col}">${t.initials}</div>
    <div style="flex:1"><div class="txn-name">${t.name}</div><div class="txn-meta">${t.cat} · ${t.date}</div></div>
    <div class="txn-status" style="background:${sc};color:${stc}">${t.status}</div>
    <div class="txn-amt" style="color:${t.pos?GREEN:RED}">${t.amt}</div>
  `;
  tl.appendChild(row);
});

// ─── CRYPTO ──────────────────────────────────────────────────
const cryptos = [
  {name:'Bitcoin',  sym:'BTC', price:67842, held:0.42, chg:3.24,  col:'#F5A623'},
  {name:'Ethereum', sym:'ETH', price:3421,  held:3.8,  chg:1.87,  col:'#7C6FFF'},
  {name:'Solana',   sym:'SOL', price:188,   held:45,   chg:5.12,  col:'#38BDF8'},
  {name:'Avalanche',sym:'AVAX',price:38.5,  held:120,  chg:-2.31, col:'#F05565'},
  {name:'Chainlink',sym:'LINK',price:18.2,  held:200,  chg:0.88,  col:'#2ECC8E'},
];
(function(){
  const list = document.getElementById('cryptoList');
  const tot = cryptos.reduce((s,c)=>s+c.price*c.held,0);
  document.getElementById('cryptoTotal').textContent='$'+Math.round(tot).toLocaleString();
  cryptos.forEach(c=>{
    const pos=c.chg>0;
    const val=(c.price*c.held).toLocaleString('en',{style:'currency',currency:'USD',maximumFractionDigits:0});
    const row=document.createElement('div'); row.className='crypto-row';
    row.innerHTML=`
      <div class="crypto-icon" style="background:${hexToRgba(c.col,0.15)};border:2px solid ${hexToRgba(c.col,0.3)};color:${c.col}">${c.sym[0]}</div>
      <div style="flex:1"><div class="crypto-name">${c.name}</div><div class="crypto-sym">${c.held} ${c.sym}</div></div>
      <div><div class="crypto-val">${val}</div><div class="crypto-chg" style="color:${pos?GREEN:RED}"><svg width="9" height="9" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><polyline points="${pos?'23 6 13.5 15.5 8.5 10.5 1 18':'1 6 10.5 15.5 15.5 10.5 23 18'}"/></svg>${Math.abs(c.chg)}%</div></div>
    `;
    list.appendChild(row);
  });
  const bar=document.getElementById('cryptoAllocBar');
  const legend=document.getElementById('cryptoLegend');
  cryptos.forEach(c=>{
    const pct=(c.price*c.held/tot*100);
    const seg=document.createElement('div'); seg.style.cssText=`flex:${pct};background:${c.col};opacity:0.75;`;
    bar.appendChild(seg);
    const item=document.createElement('div'); item.style.cssText='display:flex;align-items:center;gap:4px;';
    item.innerHTML=`<span style="width:6px;height:6px;border-radius:50%;background:${c.col};display:inline-block"></span><span style="font-size:10px;color:var(--muted);font-family:var(--mono)">${c.sym} ${pct.toFixed(1)}%</span>`;
    legend.appendChild(item);
  });
})();

// ─── WALLET ACCOUNTS ────────────────────────────────────────
const accounts = [
  { name:'HDFC Savings', type:'Savings Account', bal:'₹42,180', chg:'+₹3,200 this month', col:TEAL, icon:'🏦' },
  { name:'ICICI Current', type:'Current Account', bal:'₹18,950', chg:'+₹1,100 this month', col:AMBER, icon:'🏢' },
  { name:'Zerodha Demat', type:'Investment Account', bal:'$41,200', chg:'+12.5% YTD', col:SKY, icon:'📈' },
  { name:'CoinDCX Wallet', type:'Crypto Exchange', bal:'$28,600', chg:'+$3,800 this month', col:'#F5A623', icon:'₿' },
];
const wa = document.getElementById('walletAccounts');
if (wa) accounts.forEach(a=>{
  const row = document.createElement('div'); row.className='account-row';
  row.innerHTML=`
    <div class="account-icon" style="background:${hexToRgba(a.col,0.12)};border:1px solid ${hexToRgba(a.col,0.2)};font-size:18px">${a.icon}</div>
    <div style="flex:1"><div class="account-name">${a.name}</div><div class="account-type">${a.type}</div></div>
    <div><div class="account-bal">${a.bal}</div><div class="account-chg" style="color:${GREEN}">${a.chg}</div></div>
    <div class="account-arrow"><svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2"><polyline points="9 18 15 12 9 6"/></svg></div>
  `;
  wa.appendChild(row);
});

// ─── PAYMENTS ───────────────────────────────────────────────
const recentPays = [
  { name:'Arjun Mehta', amt:'-₹5,000', col:TEAL, initials:'AM' },
  { name:'Priya Singh', amt:'-₹1,200', col:AMBER, initials:'PS' },
  { name:'Ankit Kumar', amt:'+₹8,500', col:GREEN, initials:'AK' },
];
const rp = document.getElementById('recentPayments');
if (rp) recentPays.forEach(p=>{
  const row=document.createElement('div'); row.className='recent-pay-row';
  row.innerHTML=`<div class="pay-avatar" style="background:${hexToRgba(p.col,0.15)};color:${p.col}">${p.initials}</div><div style="flex:1"><div style="font-size:13px;font-weight:700">${p.name}</div><div style="font-size:10px;color:var(--muted);font-family:var(--mono)">Today</div></div><div style="font-family:var(--mono);font-weight:700;color:${p.amt.startsWith('+') ? GREEN : RED}">${p.amt}</div>`;
  rp.appendChild(row);
});
const scheduled = [
  { name:'AWS Cloud',    freq:'Monthly', amt:'$24.00', next:'Jun 1', col:SKY,   icon:'☁' },
  { name:'Spotify',      freq:'Monthly', amt:'₹119',   next:'Jun 3', col:GREEN, icon:'♪' },
  { name:'SIP Mutual',   freq:'Monthly', amt:'₹10,000',next:'Jun 5', col:TEAL,  icon:'📊' },
];
const sp = document.getElementById('scheduledPay');
if (sp) scheduled.forEach(s=>{
  const row=document.createElement('div'); row.className='sched-row';
  const tog=document.createElement('div'); tog.className='toggle-sw on'; tog.onclick=function(){this.classList.toggle('on');};
  row.innerHTML=`<div class="sched-icon" style="background:${hexToRgba(s.col,0.12)};border:1px solid ${hexToRgba(s.col,0.2)};font-size:18px">${s.icon}</div><div style="flex:1"><div class="sched-name">${s.name}</div><div class="sched-freq">${s.freq}</div></div><div style="text-align:right"><div class="sched-amt">${s.amt}</div><div class="sched-next">Next: ${s.next}</div></div>`;
  row.appendChild(tog);
  sp.appendChild(row);
});

// ─── INVESTMENTS ────────────────────────────────────────────
(function(){
  const stocks = [
    { ticker:'RELIANCE',company:'Reliance Ind.',shares:'15 shares',val:'$2,820',gain:'+$490',gainPct:'+21.0%',wt:'35%',col:TEAL,   pos:true },
    { ticker:'INFY',    company:'Infosys Ltd.',  shares:'25 shares',val:'$1,940',gain:'+$220',gainPct:'+12.8%',wt:'24%',col:AMBER,  pos:true },
    { ticker:'MSFT',    company:'Microsoft Corp',shares:'5 shares', val:'$2,180',gain:'+$370',gainPct:'+20.4%',wt:'27%',col:SKY,    pos:true },
    { ticker:'TSLA',    company:'Tesla Inc.',    shares:'8 shares', val:'$1,985',gain:'-$215',gainPct:'-9.8%', wt:'14%',col:RED,    pos:false},
  ];
  const list = document.getElementById('stockHoldings');
  stocks.forEach(s=>{
    const row=document.createElement('div'); row.className='inv-row';
    row.style.cssText='display:grid;grid-template-columns:2fr 1fr 1fr 1fr;gap:0;';
    row.onclick=()=>showToast(s.ticker+' details opened',s.col);
    row.innerHTML=`
      <div style="display:flex;align-items:center;gap:10px">
        <div class="inv-icon" style="background:${hexToRgba(s.col,0.14)};border:1px solid ${hexToRgba(s.col,0.24)};color:${s.col}">${s.ticker.slice(0,2)}</div>
        <div><div class="inv-ticker">${s.ticker}</div><div class="inv-company">${s.company}</div><div class="inv-shares">${s.shares}</div></div>
      </div>
      <div style="text-align:right;display:flex;align-items:center;justify-content:flex-end"><div class="inv-val">${s.val}</div></div>
      <div style="text-align:right;display:flex;align-items:center;justify-content:flex-end"><div class="inv-gain" style="color:${s.pos?GREEN:RED}">${s.gain}<br><span style="font-size:9px">${s.gainPct}</span></div></div>
      <div style="text-align:right;display:flex;flex-direction:column;align-items:flex-end;justify-content:center">
        <div class="inv-wt">${s.wt}</div>
        <div class="inv-bar-mini" style="width:${s.wt};background:${hexToRgba(s.col,0.5)};min-width:20px"></div>
      </div>
    `;
    list.appendChild(row);
  });
  const bonds = [
    {name:'India 10Y G-Sec',rate:'7.12% yield',val:'$5,000',col:TEAL},
    {name:'Corp Bond ETF',  rate:'8.3% yield', val:'$3,500',col:SKY},
  ];
  const bl=document.getElementById('bondHoldings');
  bonds.forEach(b=>{
    const row=document.createElement('div'); row.className='bond-row';
    row.innerHTML=`<div style="width:8px;height:8px;border-radius:50%;background:${b.col};flex-shrink:0"></div><div style="flex:1"><div style="font-size:12px;font-weight:700">${b.name}</div><div style="font-size:10px;color:var(--muted);font-family:var(--mono)">${b.rate}</div></div><div style="font-family:var(--mono);font-size:12px;font-weight:700">${b.val}</div>`;
    bl.appendChild(row);
  });
  const allocData=[{label:'Stocks',val:41.2,col:TEAL},{label:'Crypto',val:28.6,col:AMBER},{label:'Bonds',val:8.5,col:SKY},{label:'Cash',val:6.0,col:MUTED}];
  const total=allocData.reduce((s,d)=>s+d.val,0);
  const ctx=document.getElementById('invAllocDonut').getContext('2d');
  new Chart(ctx,{type:'doughnut',data:{labels:allocData.map(d=>d.label),datasets:[{data:allocData.map(d=>d.val),backgroundColor:allocData.map(d=>d.col),borderWidth:2,borderColor:BG3,hoverOffset:6}]},options:{responsive:false,cutout:'72%',plugins:{legend:{display:false},tooltip:{backgroundColor:BG3,borderColor:'rgba(255,255,255,0.1)',borderWidth:1,titleColor:MUTED,bodyColor:'#E8F0EF',callbacks:{label:c=>'$'+c.raw+'K ('+(c.raw/total*100).toFixed(1)+'%)'}}}}});
  const legend=document.getElementById('invAllocLegend');
  allocData.forEach(d=>{
    const item=document.createElement('div'); item.className='pie-legend-item';
    item.innerHTML=`<span class="pie-legend-dot" style="background:${d.col}"></span><span class="pie-legend-name" style="flex:1">${d.label}</span><span class="pie-legend-val">$${d.val}K</span><span class="pie-legend-pct">${(d.val/total*100).toFixed(1)}%</span>`;
    legend.appendChild(item);
  });
})();

// ─── ANALYTICS ──────────────────────────────────────────────
function initAnalytics() {
  const incomes=[8200,9100,7800,10200,9500,11000,8800,10500,9200,11700,10300,11700];
  const spends =[3200,4100,3500,4800,3900,5200,4100,4600,3800,4500,3900,3240];
  const mths   =['J','F','M','A','M','J','J','A','S','O','N','D'];
  const maxInc =Math.max(...incomes);
  const bar=document.getElementById('monthlyBarChart');
  if (!bar || bar.dataset.init) return; bar.dataset.init='1';
  mths.forEach((m,i)=>{
    const col=document.createElement('div'); col.className='mb-col';
    const incH=Math.round((incomes[i]/maxInc)*100);
    const spdH=Math.round((spends[i]/maxInc)*100);
    col.innerHTML=`<div class="mb-bar-wrap"><div class="mb-bar" style="height:${incH}%;background:${TEAL};opacity:0.7" title="Income: $${incomes[i].toLocaleString()}"></div></div><div class="mb-bar-wrap"><div class="mb-bar" style="height:${spdH}%;background:${RED};opacity:0.55" title="Spend: $${spends[i].toLocaleString()}"></div></div><div class="mb-month">${m}</div>`;
    bar.appendChild(col);
  });
  // Alloc donut
  const ad=[{label:'Stocks',val:41.2,col:TEAL},{label:'Crypto',val:28.6,col:AMBER},{label:'Bonds',val:8.5,col:SKY},{label:'Cash',val:6.0,col:MUTED}];
  const tot=ad.reduce((s,d)=>s+d.val,0);
  const ctx=document.getElementById('allocDonut').getContext('2d');
  new Chart(ctx,{type:'doughnut',data:{labels:ad.map(d=>d.label),datasets:[{data:ad.map(d=>d.val),backgroundColor:ad.map(d=>d.col),borderWidth:2,borderColor:BG3,hoverOffset:6}]},options:{responsive:false,cutout:'72%',plugins:{legend:{display:false},tooltip:{backgroundColor:BG3,borderColor:'rgba(255,255,255,0.1)',borderWidth:1,callbacks:{label:c=>'$'+c.raw+'K ('+(c.raw/tot*100).toFixed(1)+'%)'}}}}});
  const leg=document.getElementById('allocLegend');
  ad.forEach(d=>{const item=document.createElement('div');item.className='pie-legend-item';item.innerHTML=`<span class="pie-legend-dot" style="background:${d.col}"></span><span class="pie-legend-name" style="flex:1">${d.label}</span><span class="pie-legend-val">$${d.val}K</span><span class="pie-legend-pct">${(d.val/tot*100).toFixed(1)}%</span>`;leg.appendChild(item);});
  // Spend donut
  const cd=[{label:'Shopping',val:35,col:AMBER},{label:'Food',val:28,col:TEAL},{label:'Travel',val:22,col:SKY},{label:'Subs',val:9,col:RED},{label:'Other',val:6,col:MUTED}];
  const dc=document.getElementById('spendDonut').getContext('2d');
  new Chart(dc,{type:'doughnut',data:{labels:cd.map(d=>d.label),datasets:[{data:cd.map(d=>d.val),backgroundColor:cd.map(d=>d.col),borderWidth:2,borderColor:BG3,hoverOffset:6}]},options:{responsive:false,cutout:'68%',plugins:{legend:{display:false},tooltip:{backgroundColor:BG3,borderColor:'rgba(255,255,255,0.1)',borderWidth:1,callbacks:{label:c=>c.label+': '+c.raw+'%'}}}}});
  const spL=document.getElementById('spendLegend');
  cd.forEach(d=>{const item=document.createElement('div');item.className='pie-legend-item';item.innerHTML=`<span class="pie-legend-dot" style="background:${d.col}"></span><span class="pie-legend-name" style="flex:1">${d.label}</span><span style="font-size:11px;font-family:var(--mono);font-weight:700">${d.val}%</span>`;spL.appendChild(item);});
  // Cash flow
  const cfc=document.getElementById('cashFlowChart').getContext('2d');
  const net=incomes.map((v,i)=>v-spends[i]);
  const cfg=cfc.createLinearGradient(0,0,0,160);
  cfg.addColorStop(0,hexToRgba(GREEN,0.25));
  cfg.addColorStop(1,hexToRgba(GREEN,0));
  new Chart(cfc,{type:'bar',data:{labels:mths,datasets:[{label:'Income',data:incomes,backgroundColor:hexToRgba(TEAL,0.3),borderColor:TEAL,borderWidth:1.5,borderRadius:5,order:2},{label:'Spend',data:spends,backgroundColor:hexToRgba(RED,0.25),borderColor:RED,borderWidth:1.5,borderRadius:5,order:3},{label:'Net',data:net,type:'line',borderColor:GREEN,borderWidth:2,backgroundColor:cfg,fill:true,tension:0.38,pointRadius:3,pointBackgroundColor:GREEN,order:1}]},options:{responsive:true,maintainAspectRatio:false,plugins:{legend:{labels:{color:MUTED,font:{family:'JetBrains Mono',size:10}}},tooltip:{backgroundColor:BG3,borderColor:'rgba(255,255,255,0.1)',borderWidth:1,callbacks:{label:c=>' '+c.dataset.label+': $'+c.raw.toLocaleString()}}},scales:{x:{grid:{display:false},ticks:{color:MUTED,font:{family:'JetBrains Mono',size:9},maxRotation:0}},y:{grid:{color:'rgba(255,255,255,0.04)'},ticks:{color:MUTED,font:{family:'JetBrains Mono',size:9},callback:v=>'$'+(v/1000).toFixed(0)+'K'}}}}});
}
</script>
</body>
</html>


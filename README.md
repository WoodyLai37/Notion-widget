# Notion-widget
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Aesthetic Time Widget</title>
<style>

  /* ============================================================
     可自訂區 CUSTOMIZE HERE
     ============================================================ */

  /* 這裡放你想輪流顯示的語錄。每天會依「日期」自動換一句，
     也可以把 FORCE_QUOTE 設定成固定字串，強制只顯示這一句。 */

</style>
</head>
<body>

<div id="widget">
  <svg id="arc" viewBox="0 0 300 60" preserveAspectRatio="none">
    <path id="arcPath" d="M 10 50 Q 150 -10 290 50" />
    <circle id="arcDot" r="3.5" />
  </svg>

  <div id="date"></div>
  <div id="time"></div>
  <div id="divider"></div>
  <div id="quote"></div>
</div>

<script>
/* ============================================================
   可自訂區 CUSTOMIZE HERE
   ------------------------------------------------------------
   1) QUOTES：語錄清單，每天依日期自動輪替一句
   2) FORCE_QUOTE：若填字串，永遠只顯示這句（不輪替）
   3) NAME_TAG：想顯示姓名/座右銘可填在這，留空則不顯示
   ============================================================ */

const QUOTES = [
  "慢慢來，比較快。",
  "先求完成，再求完美。",
  "每一天的累積，都是未來的底氣。",
  "自由不是不工作，而是選擇為誰工作。",
  "設計是解決問題，不是裝飾問題。",
  "今天的自律，是明天的自由。",
];

const FORCE_QUOTE = ""; // 例如: "保持前進。"
const NAME_TAG = "";    // 例如: "Woody"

/* ============================================================
   以下為widget邏輯，一般不需修改
   ============================================================ */

function pad(n){ return n.toString().padStart(2,'0'); }

function dayOfYear(d){
  const start = new Date(d.getFullYear(),0,0);
  const diff = d - start;
  return Math.floor(diff / 86400000);
}

// 依時段回傳漸層色票 [hour(0-24), topColor, bottomColor, textColor]
const STOPS = [
  [0,  '#0A0E1F', '#1B2340', '#EDEFF7'], // 深夜
  [5,  '#1B2340', '#3B3B5C', '#EDEFF7'], // 破曉前
  [6,  '#F3C6A0', '#FBE3C3', '#3A2E28'], // 日出
  [9,  '#BFE1F0', '#E9F5FA', '#22323D'], // 早晨
  [12, '#6FB3D9', '#BFE1F0', '#12222E'], // 正午
  [15, '#5EA0CC', '#F0C77E', '#22221A'], // 午後
  [18, '#E8895A', '#F3C6A0', '#3A2418'], // 黃昏
  [20, '#9B5A8A', '#3B3B5C', '#EDEFF7'], // 傍晚
  [22, '#3B3B5C', '#1B2340', '#EDEFF7'], // 夜晚
  [24, '#0A0E1F', '#1B2340', '#EDEFF7'],
];

function hexToRgb(hex){
  const v = parseInt(hex.slice(1),16);
  return [ (v>>16)&255, (v>>8)&255, v&255 ];
}
function rgbToHex(r,g,b){
  return '#' + [r,g,b].map(x=>Math.round(x).toString(16).padStart(2,'0')).join('');
}
function lerp(a,b,t){ return a + (b-a)*t; }

function colorAt(hourFloat){
  for(let i=0;i<STOPS.length-1;i++){
    const [h0, top0, bot0, txt0] = STOPS[i];
    const [h1, top1, bot1] = STOPS[i+1];
    if(hourFloat >= h0 && hourFloat <= h1){
      const t = (hourFloat - h0) / (h1 - h0);
      const top0rgb = hexToRgb(top0), top1rgb = hexToRgb(top1);
      const bot0rgb = hexToRgb(bot0), bot1rgb = hexToRgb(bot1);
      const top = rgbToHex(lerp(top0rgb[0],top1rgb[0],t), lerp(top0rgb[1],top1rgb[1],t), lerp(top0rgb[2],top1rgb[2],t));
      const bot = rgbToHex(lerp(bot0rgb[0],bot1rgb[0],t), lerp(bot0rgb[1],bot1rgb[1],t), lerp(bot0rgb[2],bot1rgb[2],t));
      return { top, bot, text: txt0 };
    }
  }
  return { top: STOPS[0][1], bot: STOPS[0][2], text: STOPS[0][3] };
}

function update(){
  const now = new Date();
  const hourFloat = now.getHours() + now.getMinutes()/60;

  document.getElementById('time').textContent = pad(now.getHours()) + ':' + pad(now.getMinutes());

  const weekDays = ['日','一','二','三','四','五','六'];
  const dateStr = `${now.getMonth()+1}月${now.getDate()}日 星期${weekDays[now.getDay()]}`;
  document.getElementById('date').textContent = dateStr;

  const quoteText = FORCE_QUOTE || QUOTES[dayOfYear(now) % QUOTES.length];
  document.getElementById('quote').textContent = NAME_TAG ? `“${quoteText}” — ${NAME_TAG}` : `“${quoteText}”`;

  const { top, bot, text } = colorAt(hourFloat);
  document.body.style.background = `linear-gradient(160deg, ${top}, ${bot})`;
  document.documentElement.style.setProperty('--text-color', text);

  // 太陽/月亮軌跡：0-24小時 映射到 弧線的 0-1 位置
  const path = document.getElementById('arcPath');
  const dot = document.getElementById('arcDot');
  const len = path.getTotalLength();
  const progress = (hourFloat % 24) / 24;
  const pt = path.getPointAtLength(len * progress);
  dot.setAttribute('cx', pt.x);
  dot.setAttribute('cy', pt.y);
}

update();
setInterval(update, 1000 * 15);
</script>

<style>
  :root { --text-color: #EDEFF7; }
  * { box-sizing: border-box; }
  html, body {
    margin: 0; padding: 0; width: 100%; height: 100%;
    overflow: hidden;
  }
  body {
    display: flex; align-items: center; justify-content: center;
    min-height: 100vh;
    font-family: "Cormorant Garamond", "Noto Serif TC", "Georgia", serif;
    transition: background 2s ease;
  }
  #widget {
    text-align: center;
    color: var(--text-color);
    padding: 28px 36px 32px;
    transition: color 2s ease;
  }
  #arc {
    width: 220px; height: 44px;
    display: block; margin: 0 auto 6px;
    overflow: visible;
  }
  #arcPath {
    fill: none;
    stroke: var(--text-color);
    stroke-width: 1;
    opacity: 0.35;
  }
  #arcDot {
    fill: var(--text-color);
    opacity: 0.9;
  }
  #date {
    font-family: "Helvetica Neue", Arial, sans-serif;
    font-size: 11px;
    letter-spacing: 3px;
    text-transform: uppercase;
    opacity: 0.75;
    margin-bottom: 6px;
  }
  #time {
    font-size: 64px;
    font-weight: 500;
    letter-spacing: 2px;
    line-height: 1;
  }
  #divider {
    width: 32px; height: 1px;
    background: var(--text-color);
    opacity: 0.4;
    margin: 16px auto;
  }
  #quote {
    font-size: 16px;
    font-style: italic;
    font-weight: 400;
    opacity: 0.9;
    max-width: 280px;
    margin: 0 auto;
    line-height: 1.5;
  }

  @media (max-width: 340px) {
    #time { font-size: 48px; }
    #quote { font-size: 14px; }
  }
</style>
</body>
</html>

# study-tracker
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Daily Study Tracker</title>
  <style>
    :root{--bg:#f4f7fb;--card:#ffffffee;--text:#1e293b;--accent:#2563eb}
    body.dark{--bg:#0f172a;--card:#020617ee;--text:#e5e7eb;--accent:#22c55e}
    body{margin:0;font-family:Arial;color:var(--text);background:url('https://images.unsplash.com/photo-1491841550275-ad7854e35ca6') center/cover fixed}
    header{background:#1e40af;color:#fff;padding:1rem;text-align:center;position:sticky;top:0}
    .container{max-width:900px;margin:auto;padding:1rem}
    .card{background:var(--card);padding:1rem;border-radius:12px;margin-bottom:1rem;box-shadow:0 6px 14px rgba(0,0,0,.15)}
    h2{color:var(--accent)}
    .task{display:flex;justify-content:space-between;align-items:center;margin:.5rem 0}
    .circle{width:30px;height:30px;border-radius:50%;border:2px solid var(--accent);display:flex;align-items:center;justify-content:center;cursor:pointer}
    .circle.done{background:#22c55e;color:#fff}
    .progress-circle{width:120px;height:120px;border-radius:50%;display:flex;align-items:center;justify-content:center;font-weight:bold}
    .calendar{display:grid;grid-template-columns:repeat(7,1fr);gap:6px}
    .day{padding:10px;text-align:center;border-radius:6px;background:#e5e7eb;cursor:pointer}
    .day.good{background:#22c55e;color:#fff}
    .day.bad{background:#ef4444;color:#fff}
  </style>
</head>
<body>
<header>
  <h1>📘 Study Command Center</h1>
  <p>College 9:00–4:30 | Min Sleep 6h</p>
  <button onclick="toggleDark()">🌙 Dark</button>
</header>

<div class="container">

<div class="card">
<h2>🔒 Fixed Sessions (Yes / No)</h2>
<div class="task"><span>Maths + GA – 3h</span><div class="circle" onclick="toggleFixed(0,this)">✔</div></div>
<div class="task"><span>Recall – 1h</span><div class="circle" onclick="toggleFixed(1,this)">✔</div></div>
<div class="task"><span>College Work – 1h</span><div class="circle" onclick="toggleFixed(2,this)">✔</div></div>
</div>

<div class="card">
<h2>📊 Progress (0–500)</h2>
<div class="progress-circle" id="progress">0</div>
<p>Completed Sessions Count</p>
</div>

<div class="card">
<h2>⏱ Study Time Tracker</h2>
<div style="font-size:32px;font-weight:bold" id="timerDisplay">00:00:00</div>
<button onclick="startTimer()">▶ Start</button>
<button onclick="pauseTimer()">⏸ Pause</button>
<button onclick="resetTimer()">⟲ Reset</button>
<p>Total Study Today: <span id="totalTime">0</span> min</p>
</div>
<p>Completed Sessions Count</p>
</div>

<div class="card">
<h2>📅 Calendar (Self‑Check)</h2>
<p>✔ = Good day | ✖ = Missed / Not opened</p>
<div class="calendar" id="calendar"></div>
</div>

<div class="card">
<h2>🎧 Music & Noise Controller</h2>
<select id="soundSelect" onchange="changeSound()">
  <option value="study">Study Music</option>
  <option value="white">White Noise</option>
  <option value="brown">Brown Noise</option>
</select>
<p>Listening Time: <span id="musicTime">0</span> min</p>
<audio id="player" controls autoplay loop></audio>
</div>

</div>

<script>
// ---------- FIXED SESSIONS ----------
const fixedKey='fixedSessions';
let fixed=JSON.parse(localStorage.getItem(fixedKey))||[false,false,false];
function toggleFixed(i,el){fixed[i]=!fixed[i];localStorage.setItem(fixedKey,JSON.stringify(fixed));el.classList.toggle('done');updateProgress()}

// ---------- PROGRESS ----------
function updateProgress(){
  const count=fixed.filter(Boolean).length;
  const score=Math.min(count*100,500);
  const deg=score/500*360;
  const p=document.getElementById('progress');
  p.style.background=`conic-gradient(#22c55e ${deg}deg,#e5e7eb ${deg}deg)`;
  p.innerText=score;
}

// ---------- CALENDAR ----------
const cal=document.getElementById('calendar');
const calKey='calendarData';
let calData=JSON.parse(localStorage.getItem(calKey))||{};
const today=new Date();
const daysInMonth=new Date(today.getFullYear(),today.getMonth()+1,0).getDate();
for(let d=1;d<=daysInMonth;d++){
  const key=`${today.getFullYear()}-${today.getMonth()}-${d}`;
  const div=document.createElement('div');
  div.className='day';
  div.innerText=d;
  if(calData[key]==='good')div.classList.add('good');
  if(calData[key]==='bad')div.classList.add('bad');
  div.onclick=()=>{
    calData[key]=calData[key]==='good'?'bad':'good';
    localStorage.setItem(calKey,JSON.stringify(calData));
    location.reload();
  };
  cal.appendChild(div);
}

// Auto mark yesterday bad if site not opened
const lastOpen=localStorage.getItem('lastOpen');
const todayKey=today.toDateString();
if(lastOpen && lastOpen!==todayKey){
  const y=new Date(today);y.setDate(y.getDate()-1);
  const yKey=`${y.getFullYear()}-${y.getMonth()}-${y.getDate()}`;
  if(!calData[yKey]){calData[yKey]='bad';localStorage.setItem(calKey,JSON.stringify(calData))}
}
localStorage.setItem('lastOpen',todayKey);

// ---------- MUSIC & TRACKER ----------
const sounds={
  study:'https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3',
  white:'https://cdn.pixabay.com/download/audio/2022/03/15/audio_7c8c2d6f59.mp3',
  brown:'https://cdn.pixabay.com/download/audio/2022/03/15/audio_8c9e7f6d09.mp3'
};
const player=document.getElementById('player');
const select=document.getElementById('soundSelect');
const musicKey='musicMinutes';
let musicMinutes=parseInt(localStorage.getItem(musicKey))||0;
player.src=sounds.study;
setInterval(()=>{
  if(!player.paused){musicMinutes++;localStorage.setItem(musicKey,musicMinutes);document.getElementById('musicTime').innerText=musicMinutes}
},60000);
function changeSound(){player.src=sounds[select.value];player.play();}

// ---------- STUDY TIMER ----------
let timerInterval=null;
let seconds=0;
const timeKey='studySeconds';
seconds=parseInt(localStorage.getItem(timeKey))||0;

function updateTimerUI(){
  const h=String(Math.floor(seconds/3600)).padStart(2,'0');
  const m=String(Math.floor((seconds%3600)/60)).padStart(2,'0');
  const s=String(seconds%60).padStart(2,'0');
  document.getElementById('timerDisplay').innerText=`${h}:${m}:${s}`;
  document.getElementById('totalTime').innerText=Math.floor(seconds/60);
}

function startTimer(){
  if(timerInterval) return;
  timerInterval=setInterval(()=>{
    seconds++;
    localStorage.setItem(timeKey,seconds);
    updateTimerUI();
  },1000);
}

function pauseTimer(){
  clearInterval(timerInterval);
  timerInterval=null;
}

function resetTimer(){
  pauseTimer();
  seconds=0;
  localStorage.setItem(timeKey,0);
  updateTimerUI();
}

// ---------- INIT ----------
updateProgress();
document.querySelectorAll('.circle').forEach((c,i)=>{if(fixed[i])c.classList.add('done')});
function toggleDark(){document.body.classList.toggle('dark')}
</script>
</body>
</html>

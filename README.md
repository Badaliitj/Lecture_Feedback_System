# Lecture_Feedback_System
<!doctype html>
<html lang="en"><head><meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Smart Lecture Feedback & Quiz System</title>
<style>
:root{--bg:#EEF4F8;--card:#fff;--ink:#0B2545;--mut:#475569;--teal:#13A89E;--line:#CBD5E1;--chip:#EEF4F8;box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
@media (prefers-color-scheme:dark){:root:not([data-theme="light"]){--bg:#0B1A30;--card:#12294A;--ink:#EAF2FA;--mut:#A9B8CC;--line:#2A4468;--chip:#0B1A30;color-scheme:dark}}
:root[data-theme="dark"]{--bg:#0B1A30;--card:#12294A;--ink:#EAF2FA;--mut:#A9B8CC;--line:#2A4468;--chip:#0B1A30;color-scheme:dark}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--ink);font:16px/1.5 system-ui,-apple-system,"Segoe UI",Roboto,sans-serif}
header{background:#0B2545;color:#fff;padding:16px 20px;border-bottom:4px solid var(--teal)}
header h1{margin:0;font-size:20px}header p{margin:2px 0 0;font-size:13px;color:#9FE3DC}
nav{display:flex;gap:8px;padding:12px 16px;max-width:760px;margin:0 auto}
nav button{flex:1;padding:10px;border:1px solid var(--line);background:var(--card);color:var(--ink);border-radius:10px;font:inherit;font-weight:600;cursor:pointer}
nav button.on{background:var(--teal);color:#fff;border-color:var(--teal)}
main{max-width:760px;margin:0 auto;padding:0 16px 32px}
section{display:none}section.on{display:block}
.card{background:var(--card);border:1px solid var(--line);border-radius:14px;padding:16px;margin-bottom:14px}
h2{margin:0 0 10px;font-size:18px}
label{display:block;font-size:13px;font-weight:600;color:var(--mut);margin:10px 0 4px}
input,select{width:100%;padding:10px;border:1px solid var(--line);border-radius:8px;background:var(--bg);color:var(--ink);font:inherit}
.row{display:flex;gap:8px;flex-wrap:wrap}
.chip{padding:8px 14px;border:1px solid var(--teal);border-radius:20px;background:var(--chip);color:var(--ink);font:inherit;cursor:pointer}
.chip.on{background:var(--teal);color:#fff}
.opt{display:block;width:100%;text-align:left;margin-bottom:8px;border-radius:10px}
.go{background:var(--teal);color:#fff;border:0;padding:12px 18px;border-radius:10px;font:inherit;font-weight:700;cursor:pointer;margin-top:12px}
.alt{background:transparent;color:var(--teal);border:1px solid var(--teal)}
.code{font-size:36px;font-weight:800;letter-spacing:6px;color:var(--teal)}
.note{font-size:13px;color:var(--mut)}
.tiles{display:grid;grid-template-columns:repeat(2,1fr);gap:10px;margin-bottom:14px}
.tile{background:#0B2545;color:#fff;border-radius:12px;padding:14px;text-align:center}
.tile b{display:block;font-size:26px}.tile span{font-size:12px;color:#9FE3DC}
.bar{display:flex;align-items:center;gap:8px;margin:8px 0;font-size:14px}
.bar i{display:block;height:18px;background:var(--teal);border-radius:4px;min-width:2px}
.bar em{width:90px;font-style:normal}
ul{margin:6px 0 0;padding-left:20px}
</style></head><body>
<header><h1>Smart Lecture Feedback & Quiz System</h1><p>Working prototype demo · Badal · IIT Jodhpur</p></header>
<nav><button id="t-prof" class="on">1. Professor</button><button id="t-stu">2. Student</button><button id="t-dash">3. Dashboard</button></nav>
<main>
<section id="prof" class="on">
 <div class="card"><h2>Create a lecture session</h2>
  <label>Lecture topic</label><input id="topic" value="Maxwell’s equations">
  <label>Quiz question</label><input id="q" value="What is the divergence of a magnetic field?">
  <label>Option A</label><input class="op" value="Zero">
  <label>Option B</label><input class="op" value="Equal to charge density">
  <label>Option C</label><input class="op" value="Always positive">
  <label>Option D</label><input class="op" value="Depends on the magnet">
  <label>Correct option</label><select id="ans"><option>A</option><option>B</option><option>C</option><option>D</option></select>
  <button class="go" id="start">Start session</button>
  <div id="codebox" style="display:none;margin-top:14px"><div class="note">Students join with this code (a QR code can be added later)</div><div class="code" id="code"></div></div>
 </div>
</section>
<section id="stu">
 <div class="card" id="join"><h2>Join session</h2><label>Session code</label><input id="jc" inputmode="numeric" maxlength="4" placeholder="4-digit code">
  <button class="go" id="joinbtn">Join</button><div class="note" id="jmsg"></div></div>
 <div class="card" id="quiz" style="display:none"><h2 id="qt"></h2><div id="opts"></div>
  <label>How clear was the concept? (1–5)</label><div class="row" id="cl"></div>
  <label>Lecture pace</label><div class="row" id="pc"></div>
  <label>Was the question clear?</label><div class="row" id="qc"></div>
  <button class="go" id="submit">Submit anonymously</button><div class="note" id="smsg"></div></div>
</section>
<section id="dash">
 <div class="card"><label style="margin-top:0">Class size (for participation rate)</label><input id="size" type="number" value="30" min="1">
  <div class="row"><button class="go alt" id="seed">Add 12 sample responses</button><button class="go alt" id="reset">Clear responses</button></div>
  <div class="note">Sample responses are random demo data, not real results.</div></div>
 <div id="out"></div>
</section>
</main>
<script>
const $=id=>document.getElementById(id);
const S={code:null,q:null,topic:"",opts:[],ans:0,res:[]};
const pick={sel:-1,cl:0,pace:"",qc:""};
function tab(n){["prof","stu","dash"].forEach(k=>{$(k).classList.toggle("on",k===n);$("t-"+k).classList.toggle("on",k===n)});if(n==="dash")render()}
$("t-prof").onclick=()=>tab("prof");$("t-stu").onclick=()=>tab("stu");$("t-dash").onclick=()=>tab("dash");
$("start").onclick=()=>{
 S.code=String(Math.floor(1000+Math.random()*9000));S.q=$("q").value;S.topic=$("topic").value;
 S.opts=[...document.querySelectorAll(".op")].map(e=>e.value);S.ans="ABCD".indexOf($("ans").value);S.res=[];
 $("code").textContent=S.code;$("codebox").style.display="block";
};
function chips(el,vals,key,labels){el.innerHTML="";vals.forEach((v,i)=>{const b=document.createElement("button");b.className="chip";b.textContent=labels?labels[i]:v;
 b.onclick=()=>{pick[key]=v;[...el.children].forEach(c=>c.classList.remove("on"));b.classList.add("on")};el.appendChild(b)})}
$("joinbtn").onclick=()=>{
 if(!S.code){$("jmsg").textContent="No session yet. Ask the professor to start one in tab 1.";return}
 if($("jc").value.trim()!==S.code){$("jmsg").textContent="Wrong code. Try again.";return}
 $("join").style.display="none";$("quiz").style.display="block";buildQuiz()};
function buildQuiz(){
 $("qt").textContent="Q: "+S.q;pick.sel=-1;pick.cl=0;pick.pace="";pick.qc="";$("smsg").textContent="";
 const o=$("opts");o.innerHTML="";S.opts.forEach((t,i)=>{const b=document.createElement("button");b.className="chip opt";b.textContent="ABCD"[i]+") "+t;
  b.onclick=()=>{pick.sel=i;[...o.children].forEach(c=>c.classList.remove("on"));b.classList.add("on")};o.appendChild(b)});
 chips($("cl"),[1,2,3,4,5],"cl");
 chips($("pc"),["fast","ok","slow"],"pace",["Too fast","Appropriate","Too slow"]);
 chips($("qc"),["clear","confusing"],"qc",["Clear","Confusing"]);
}
$("submit").onclick=()=>{
 if(pick.sel<0||!pick.cl||!pick.pace||!pick.qc){$("smsg").textContent="Please answer all four parts.";return}
 S.res.push({ok:pick.sel===S.ans,cl:pick.cl,pace:pick.pace,qc:pick.qc});
 buildQuiz();$("smsg").textContent="Thank you! Response saved anonymously. You can submit again to act as another student.";
};
$("seed").onclick=()=>{
 if(!S.code){alert("Start a session first (tab 1).");return}
 const r=a=>a[Math.floor(Math.random()*a.length)];
 for(let i=0;i<12;i++)S.res.push({ok:Math.random()<0.45,cl:r([2,3,3,4,4,5]),pace:r(["fast","fast","ok","ok","ok","slow"]),qc:r(["clear","clear","confusing"])});render()};
$("reset").onclick=()=>{S.res=[];render()};
$("size").oninput=render;
function render(){
 const o=$("out"),n=S.res.length;
 if(!S.code||!n){o.innerHTML='<div class="card note">No responses yet. Submit answers in the Student tab, or add sample responses.</div>';return}
 const sz=Math.max(1,+$("size").value||30),pct=x=>Math.round(x*100);
 const part=Math.min(100,pct(n/sz)),sc=pct(S.res.filter(r=>r.ok).length/n);
 const cl=S.res.reduce((a,r)=>a+r.cl,0)/n,clp=pct(cl/5);
 const pc={fast:0,ok:0,slow:0};S.res.forEach(r=>pc[r.pace]++);
 const conf=S.res.filter(r=>r.qc==="confusing").length;
 const tips=[];
 if(sc<60)tips.push("Many students answered wrongly: revise “"+S.topic+"”.");
 if(clp<70)tips.push("Concept clarity is low: explain the idea again with an example.");
 if(pc.fast/n>0.4)tips.push("Many students found the lecture too fast: slow down.");
 if(pc.slow/n>0.4)tips.push("Many students found the lecture too slow: speed up.");
 if(conf/n>0.3)tips.push("Many students found the question confusing: reword it.");
 if(!tips.length)tips.push("Class looks comfortable. Continue with the next topic.");
 const bar=(l,v)=>'<div class="bar"><em>'+l+'</em><i style="width:'+Math.max(2,pct(v/n)*2.4)+'px"></i><span>'+pct(v/n)+'%</span></div>';
 o.innerHTML='<div class="tiles"><div class="tile"><b>'+part+'%</b><span>Participation ('+n+' of '+sz+')</span></div><div class="tile"><b>'+sc+'%</b><span>Correct answers</span></div><div class="tile"><b>'+clp+'%</b><span>Concept clarity</span></div><div class="tile"><b>'+conf+'</b><span>Said question confusing</span></div></div>'
 +'<div class="card"><h2>Lecture pace</h2>'+bar("Too fast",pc.fast)+bar("Appropriate",pc.ok)+bar("Too slow",pc.slow)+'</div>'
 +'<div class="card"><h2>Suggestions for the professor</h2><ul>'+tips.map(t=>"<li></li>").join("")+'</ul><div class="note" style="margin-top:8px">Suggestions are simple rules. The professor makes the final decision.</div></div>';
 [...o.querySelectorAll("li")].forEach((li,i)=>li.textContent=tips[i]);
}
render();
</script></body></html>

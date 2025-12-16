<!DOCTYPE html>
<html lang="hi">
<head>
<meta charset="UTF-8">
<title>AI Online Exam</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body{margin:0;font-family:Arial;background:#f0f2f5}
.header{background:#1a73e8;color:#fff;padding:10px;text-align:center}
.main{max-width:1000px;margin:auto;display:flex}
.left{flex:3;background:#fff;padding:15px}
.right{flex:1;background:#fafafa;padding:15px;border-left:1px solid #ccc}
.option{padding:10px;margin:8px 0;background:#eee;border-radius:5px;cursor:pointer}
.selected{background:#90caf9}
.btn{padding:10px;margin:5px;border:none;border-radius:5px;cursor:pointer}
.save{background:#1a73e8;color:#fff}
.review{background:#fbbc04}
.submit{background:#34a853;color:#fff;width:100%}
.qbtn{width:35px;height:35px;margin:3px;border:none;border-radius:50%}
.answered{background:#34a853;color:#fff}
.reviewed{background:#fbbc04}
.notvisited{background:#ccc}
.timer{font-size:18px;font-weight:bold}
</style>
</head>

<body>

<div class="header">
<h2>AI Online Exam</h2>
<div class="timer" id="timer">⏱ 10:00</div>
</div>

<div style="text-align:center;padding:10px">
<input id="topic" placeholder="Topic (GK, Agriculture)"
style="padding:10px;width:300px">
<button class="btn save" onclick="startExam()">Generate Exam</button>
<button class="btn review" onclick="downloadExamHTML()">⬇️ Exam HTML</button>
<p id="msg"></p>
</div>

<div class="main" id="exam" style="display:none">

<div class="left">
<h3 id="question"></h3>
<div id="options"></div>

<button class="btn save" onclick="saveNext()">Save & Next</button>
<button class="btn review" onclick="markReview()">Mark for Review</button>
</div>

<div class="right">
<h4>Questions</h4>
<div id="palette"></div>
<br>
<button class="btn submit" onclick="submitExam()">Submit Exam</button>
</div>

</div>

<script>
const API_KEY="AIzaSyDcb_dPpEouVAsdByVLOQU_TR9HjdMkiB0";

/* ===== FALLBACK DEMO QUIZ ===== */
const demoQuiz=[
 {q:"भारत की राजधानी?",o:["दिल्ली","मुंबई","जयपुर","लखनऊ"],a:0},
 {q:"ताजमहल कहाँ है?",o:["आगरा","दिल्ली","जयपुर","लखनऊ"],a:0},
 {q:"राष्ट्रीय पशु?",o:["बाघ","शेर","हाथी","घोड़ा"],a:0}
];

let quiz=[],answers=[],review=[],i=0,time=600,timer;

/* ===== START EXAM ===== */
async function startExam(){
 document.getElementById("msg").innerText="⏳ AI se exam generate ho raha hai...";

 try{
  const topic=document.getElementById("topic").value || "GK";
  const res=await fetch(
   "https://generativelanguage.googleapis.com/v1/models/gemini-1.5-flash:generateContent?key="+API_KEY,
   {
    method:"POST",
    headers:{"Content-Type":"application/json"},
    body:JSON.stringify({
     contents:[{parts:[{text:
`Create 8 MCQ questions on ${topic} in Hindi.
Return STRICT JSON only like:
[
 {"q":"Question","o":["A","B","C","D"],"a":1}
]`
     }]}]
    })
   }
  );

  const data=await res.json();
  const text=data.candidates[0].content.parts[0].text;
  const json=text.substring(text.indexOf("["),text.lastIndexOf("]")+1);
  quiz=JSON.parse(json);

 }catch(e){
  quiz=demoQuiz;
  document.getElementById("msg").innerText=
   "⚠️ AI fail → Demo exam loaded";
 }

 answers=new Array(quiz.length).fill(null);
 review=new Array(quiz.length).fill(false);
 i=0; time=600;

 document.getElementById("exam").style.display="flex";
 loadQ(); buildPalette(); startTimer();
}

/* ===== LOAD QUESTION ===== */
function loadQ(){
 document.getElementById("question").innerText=`Q${i+1}. ${quiz[i].q}`;
 let h="";
 quiz[i].o.forEach((op,idx)=>{
  let cls=answers[i]===idx?"option selected":"option";
  h+=`<div class="${cls}" onclick="selectOpt(${idx})">${op}</div>`;
 });
 document.getElementById("options").innerHTML=h;
}

/* ===== SELECT OPTION ===== */
function selectOpt(x){
 answers[i]=x;
 loadQ(); buildPalette();
}

/* ===== NEXT ===== */
function saveNext(){
 if(i<quiz.length-1){i++;loadQ();}
 buildPalette();
}

/* ===== MARK REVIEW ===== */
function markReview(){
 review[i]=true;
 saveNext();
}

/* ===== PALETTE ===== */
function buildPalette(){
 let h="";
 quiz.forEach((q,idx)=>{
  let c="notvisited";
  if(answers[idx]!=null) c="answered";
  if(review[idx]) c="reviewed";
  h+=`<button class="qbtn ${c}" onclick="i=${idx};loadQ()">${idx+1}</button>`;
 });
 document.getElementById("palette").innerHTML=h;
}

/* ===== TIMER ===== */
function startTimer(){
 timer=setInterval(()=>{
  time--;
  let m=Math.floor(time/60),s=time%60;
  document.getElementById("timer").innerText=
   `⏱ ${m}:${s<10?"0"+s:s}`;
  if(time<=0){clearInterval(timer);submitExam();}
 },1000);
}

/* ===== SUBMIT (NEGATIVE MARKING) ===== */
function submitExam(){
 clearInterval(timer);
 let score=0;
 quiz.forEach((q,idx)=>{
  if(answers[idx]===q.a) score+=1;
  else if(answers[idx]!==null) score-=0.25;
 });

 document.body.innerHTML=
 `<h2 style="text-align:center">Result</h2>
  <p style="text-align:center">Score: ${score}</p>
  <p style="text-align:center">
   <button onclick="downloadPDF(${score})">⬇️ Download PDF</button>
  </p>`;
}

/* ===== RESULT PDF ===== */
function downloadPDF(score){
 const text=`Exam Result\nScore: ${score}`;
 const blob=new Blob([text],{type:"application/pdf"});
 const a=document.createElement("a");
 a.href=URL.createObjectURL(blob);
 a.download="result.pdf";
 a.click();
}

/* ===== EXAM HTML DOWNLOAD ===== */
function downloadExamHTML(){
 const html=`<!DOCTYPE html><html><body>
<h2>Offline Exam</h2>
<pre>${JSON.stringify(quiz,null,2)}</pre>
</body></html>`;
 const blob=new Blob([html],{type:"text/html"});
 const a=document.createElement("a");
 a.href=URL.createObjectURL(blob);
 a.download="offline_exam.html";
 a.click();
}
</script>

</body>
</html>

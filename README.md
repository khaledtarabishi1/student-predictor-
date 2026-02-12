# student-predictor
<html>
<head>
<meta charset="UTF-8">
<title>Adaptive AI Semester Predictor</title>
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
<style>
body {margin:0;font-family:'Segoe UI',sans-serif;background:linear-gradient(135deg,#0f0f1a,#1a0033);color:white;display:flex;justify-content:center;align-items:center;height:100vh;}
.container {background: rgba(20,20,40,0.95);padding:40px;border-radius:20px;width:600px;box-shadow:0 0 50px rgba(128,0,255,0.5);border:1px solid rgba(128,0,255,0.3);}
h1{text-align:center;margin-bottom:20px;font-size:26px;color:#bb86fc;}
input, select { width:100%; padding:12px; margin-bottom:12px; border-radius:10px; border:none; outline:none; background:#111122; color:white; font-size:14px; border:1px solid rgba(128,0,255,0.3);}
input:focus, select:focus {border:1px solid #bb86fc; box-shadow:0 0 10px #bb86fc;}
button { width:100%; padding:14px; border-radius:12px; border:none; background:linear-gradient(90deg,#7b00ff,#bb00ff); color:white; font-weight:bold; font-size:16px; cursor:pointer; transition:0.3s;}
button:hover { transform:scale(1.05); box-shadow:0 0 20px #bb00ff;}
.status {text-align:center; font-size:12px; margin-bottom:15px; color:#aaa;}
#result {margin-top:20px;text-align:center;font-size:18px;color:#bb86fc;}
#recommendation {margin-top:15px;text-align:center;font-size:16px;color:#ffcc00; line-height:1.4;}
.login-section, .predict-section { display:none; }
label{margin-bottom:5px; display:block;}
</style>
</head>
<body>
<div class="container">

<!-- LOGIN -->
<div class="login-section" id="loginSection" style="display:block;">
<h1>Login / Signup</h1>
<input type="text" id="username" placeholder="Username">
<input type="password" id="password" placeholder="Password">
<label><input type="checkbox" id="rememberMe"> Remember Me</label>
<button id="loginBtn">Login / Signup</button>
<div id="loginStatus" class="status"></div>
</div>

<!-- PREDICTOR -->
<div class="predict-section" id="predictSection">
<h1>Semester Predictor</h1>
<div class="status" id="status">Training AI Model...</div>

<label for="system">Select System:</label>
<select id="system">
  <option value="igcse">IGCSE</option>
  <option value="national">National</option>
</select>

<label for="predictSemester">Select Semester to Predict:</label>
<select id="predictSemester"></select>

<input type="number" id="lastYear" placeholder="Last Year Score">
<input type="number" id="s1" placeholder="Semester 1 Score">
<input type="number" id="s2" placeholder="Semester 2 Score">
<input type="number" id="attendance" placeholder="Attendance %">
<input type="number" id="study" placeholder="Study Hours per Week">
<input type="number" id="homework" placeholder="Homework Score">
<input type="number" id="screenTime" placeholder="Screen Time Hours per Day">
<input type="number" id="realScore" placeholder="Enter Real Score (optional)">
<button id="predictBtn">Predict</button>
<h2 id="result"></h2>
<div id="recommendation">Enter your scores and click Predict to get recommendations.</div>
<button id="logoutBtn">Logout</button>
</div>

</div>

<script>
window.onload=function(){
let model;
let currentUser=null;

const loginBtn=document.getElementById("loginBtn");
const predictBtn=document.getElementById("predictBtn");
const logoutBtn=document.getElementById("logoutBtn");
const systemSelect=document.getElementById("system");
const predictSelect=document.getElementById("predictSemester");

function login(){
    const username=document.getElementById("username").value.trim();
    const password=document.getElementById("password").value.trim();
    const remember=document.getElementById("rememberMe").checked;
    if(!username||!password){document.getElementById("loginStatus").innerText="Enter username and password."; return;}
    let users=JSON.parse(localStorage.getItem("students"))||{};
    if(!users[username]){users[username]={password:password,history:[]}; localStorage.setItem("students",JSON.stringify(users));document.getElementById("loginStatus").innerText="Signup successful!";}
    else{if(users[username].password!==password){document.getElementById("loginStatus").innerText="Wrong password!"; return;} document.getElementById("loginStatus").innerText="Login successful!";}
    currentUser=username;
    if(remember) localStorage.setItem("currentUser",username);
    showPredictSection();
}

function logout(){
    currentUser=null;
    localStorage.removeItem("currentUser");
    document.getElementById("loginSection").style.display="block";
    document.getElementById("predictSection").style.display="none";
}

function showPredictSection(){
    document.getElementById("loginSection").style.display="none";
    document.getElementById("predictSection").style.display="block";
    updateSemesterOptions();
    trainModel();
}

const remembered=localStorage.getItem("currentUser");
if(remembered){currentUser=remembered; showPredictSection();}
else{document.getElementById("loginSection").style.display="block";}

// Update semester dropdown based on system
function updateSemesterOptions(){
    const system=systemSelect.value;
    predictSelect.innerHTML="";
    let options=[];
    if(system==="igcse"){ options=["Semester 1","Semester 2","Semester 3"]; }
    else{ options=["Semester 1","Semester 2"]; }
    options.forEach((opt,i)=>{ predictSelect.options.add(new Option(opt,`sem${i+1}`)); });
    updatePlaceholders();
}

// Update placeholders dynamically
function updatePlaceholders(){
    const system=systemSelect.value;
    const predict=predictSelect.value;
    let lastText="", s1Text="", s2Text="";
    if(system==="igcse"){
        if(predict==="sem1"){lastText="Last Year Semester 1"; s1Text="Semester 1"; s2Text="Semester 2";}
        else if(predict==="sem2"){lastText="Last Year Semester 2"; s1Text="Semester 1"; s2Text="Semester 2";}
        else{lastText="Last Year Overall"; s1Text="Semester 1"; s2Text="Semester 2";}
    } else {
        if(predict==="sem1"){lastText="Last Year Semester 1"; s1Text="Semester 1"; s2Text="Semester 2";}
        else if(predict==="sem2"){lastText="Last Year Semester 2"; s1Text="Semester 1"; s2Text="Semester 2";}
    }
    document.getElementById("lastYear").placeholder=lastText;
    document.getElementById("s1").placeholder=s1Text;
    document.getElementById("s2").placeholder=s2Text;
}

systemSelect.addEventListener("change",()=>{
    updateSemesterOptions();
});

async function trainModel(){
    let xsData=[[80,75,78,90,10,85,3],[70,72,74,85,8,70,4],[90,88,92,95,12,95,2],[60,65,68,75,6,65,5],[85,82,84,92,11,88,3],[78,80,79,88,9,80,4]];
    let ysData=[[79],[71],[93],[66],[87],[80]];
    if(currentUser){
        const users=JSON.parse(localStorage.getItem("students"))||{};
        const history=users[currentUser].history||[];
        history.forEach(item=>{
            xsData.push([item.lastYear,item.s1,item.s2,item.attendance,item.study,item.homework,item.screenTime]);
            ysData.push([item.predicted]);
        });
    }
    const xs=tf.tensor2d(xsData);
    const ys=tf.tensor2d(ysData);
    model=tf.sequential();
    model.add(tf.layers.dense({units:16,activation:'relu',inputShape:[7]}));
    model.add(tf.layers.dense({units:8,activation:'relu'}));
    model.add(tf.layers.dense({units:1}));
    model.compile({optimizer:tf.train.adam(0.01),loss:'meanSquaredError'});
    await model.fit(xs,ys,{epochs:300});
    document.getElementById("status").innerText="AI Model Ready ✅";
}

async function predictScore(){
    if(!model){ alert("Model still training..."); return; }
    const system=systemSelect.value;
    const predict=predictSelect.value;
    const lastYear=parseFloat(document.getElementById("lastYear").value)||0;
    const s1=parseFloat(document.getElementById("s1").value)||0;
    const s2=parseFloat(document.getElementById("s2").value)||0;
    const attendance=parseFloat(document.getElementById("attendance").value)||0;
    const study=parseFloat(document.getElementById("study").value)||0;
    const homework=parseFloat(document.getElementById("homework").value)||0;
    const screenTime=parseFloat(document.getElementById("screenTime").value)||0;
    const real=parseFloat(document.getElementById("realScore").value);

    let factor = system==='igcse'?1:0.95;
    const input=tf.tensor2d([[lastYear,s1,s2,attendance,study,homework,screenTime]]);
    const prediction=model.predict(input);
    const result=await prediction.data();
    const score=result[0]*factor;
    document.getElementById("result").innerText=`Predicted ${predict} Score: ${score.toFixed(2)}`;

    // Recommendations
    let rec="";
    if(score>=95) rec="Outstanding! Keep excelling 🌟";
    else if(score>=90) rec="Excellent! Keep up the great work 🎉";
    else if(score>=80) rec="Good job! Focus on weak subjects 💪";
    else if(score>=70) rec="Average. Increase study hours ⚡";
    else if(score>=60) rec="Below average. Pay attention to homework 📚";
    else rec="Needs improvement. Revise all subjects 🛑";

    if(screenTime>5) rec+=" | Reduce screen time ⏱️";
    else if(screenTime>=3) rec+=" | Moderate screen time ⚖️";
    else rec+=" | Low screen time 👍";

    if(lastYear){ if(score>lastYear) rec+=" | Improvement from last year ✅"; else if(score<lastYear) rec+=" | Score lower than last year ⚠️"; else rec+=" | Score similar to last year"; }

    if(!isNaN(real)){ rec+=` | Predicted: ${score.toFixed(2)}, Actual: ${real}`; const diff=score-real; if(Math.abs(diff)>=10) rec+=" | Big difference! Track habits 📊"; else rec+=" | Prediction close ✅"; }

    document.getElementById("recommendation").innerText=rec;

    if(currentUser){
        let users=JSON.parse(localStorage.getItem("students"))||{};
        users[currentUser].history.push({system,predict,lastYear,s1,s2,attendance,study,homework,screenTime,predicted:score});
        localStorage.setItem("students",JSON.stringify(users));
    }
}

loginBtn.addEventListener("click",login);
predictBtn.addEventListener("click",predictScore);
logoutBtn.addEventListener("click",logout);
};
</script>
</body>
</html>

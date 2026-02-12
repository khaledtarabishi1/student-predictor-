# student-predictor
<html>
<head>
<meta charset="UTF-8">
<title>AI Student Performance Predictor</title>
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>

<style>
body {
    margin:0;
    font-family:'Segoe UI',sans-serif;
    background:linear-gradient(135deg,#0f0f1a,#1a0033);
    color:white;
    display:flex;
    justify-content:center;
    align-items:center;
    min-height:100vh;
}
.container{
    background:rgba(20,20,40,0.9);
    padding:40px;
    border-radius:20px;
    width:520px;
    box-shadow:0 0 40px rgba(128,0,255,0.4);
}
h1{text-align:center;color:#bb86fc;}
input,select{
    width:100%;
    padding:12px;
    margin-bottom:12px;
    border-radius:10px;
    border:none;
    background:#111122;
    color:white;
}
button{
    width:100%;
    padding:12px;
    margin-bottom:10px;
    border-radius:10px;
    border:none;
    background:linear-gradient(90deg,#7b00ff,#bb00ff);
    color:white;
    font-weight:bold;
    cursor:pointer;
}
button:hover{
    box-shadow:0 0 15px #bb00ff;
}
#result{font-size:18px;text-align:center;margin-top:10px;color:#bb86fc;}
#recommendations{margin-top:10px;font-size:14px;color:#ffcc00;}
.status{text-align:center;font-size:12px;color:#aaa;}
</style>
</head>

<body>
<div class="container">

<h1>AI Student Performance Predictor</h1>

<!-- LOGIN -->
<div id="loginDiv">
    <input type="text" id="username" placeholder="Enter your name">
    <input type="password" id="password" placeholder="Enter password">
    <button onclick="login()">Login</button>
</div>

<!-- PREDICTOR -->
<div id="predictorDiv" style="display:none;">
<div class="status" id="status">Training AI Model...</div>

<select id="systemSelect" onchange="updateOptions()">
    <option value="igcse">IGCSE</option>
    <option value="national">National</option>
</select>

<select id="predictSelect" onchange="updatePlaceholders()">
    <option value="sem1">Predict Semester 1</option>
    <option value="sem2">Predict Semester 2</option>
    <option value="sem3">Predict Semester 3</option>
</select>

<input type="number" id="lastYear" placeholder="">
<input type="number" id="s1" placeholder="">
<input type="number" id="s2" placeholder="">
<input type="number" id="attendance" placeholder="Attendance %">
<input type="number" id="study" placeholder="Study Hours per Week">
<input type="number" id="homework" placeholder="Homework Score">

<button onclick="predictScore()">Predict Score</button>

<input type="number" id="realScore" placeholder="Enter Real Score (After Exams)">
<button onclick="learnFromRealScore()">Submit Real Score & Improve AI</button>

<button onclick="eraseAll()">Erase All</button>
<button onclick="logout()">Logout</button>

<h2 id="result"></h2>
<div id="recommendations"></div>
<div class="status" id="screenTime"></div>

</div>
</div>

<script>

// LOGIN
function login(){
    const user=document.getElementById("username").value;
    const pass=document.getElementById("password").value;
    if(!user||!pass){alert("Enter name & password");return;}
    localStorage.setItem("user",user);
    document.getElementById("loginDiv").style.display="none";
    document.getElementById("predictorDiv").style.display="block";
    startScreenTime();
}
function logout(){
    localStorage.removeItem("user");
    location.reload();
}

// ERASE ALL
function eraseAll(){
    document.querySelectorAll("input").forEach(i=>i.value="");
    document.getElementById("result").innerText="";
    document.getElementById("recommendations").innerHTML="";
}

// SYSTEM OPTIONS
function updateOptions(){
    const sys=document.getElementById("systemSelect").value;
    const predict=document.getElementById("predictSelect");
    if(sys==="national"){
        predict.innerHTML=`
        <option value="sem1">Predict Semester 1</option>
        <option value="sem2">Predict Semester 2</option>`;
        document.getElementById("s2").style.display="none";
    } else{
        predict.innerHTML=`
        <option value="sem1">Predict Semester 1</option>
        <option value="sem2">Predict Semester 2</option>
        <option value="sem3">Predict Semester 3</option>`;
        document.getElementById("s2").style.display="block";
    }
    updatePlaceholders();
}

// PLACEHOLDERS
function updatePlaceholders(){
    const sys=document.getElementById("systemSelect").value;
    const sem=document.getElementById("predictSelect").value;

    if(sys==="igcse"){
        if(sem==="sem1"){
            lastYear.placeholder="Last Year Sem 1";
            s1.placeholder="Last Year Sem 2";
            s2.placeholder="Last Year Sem 3";
        }
        if(sem==="sem2"){
            lastYear.placeholder="Last Year Sem 2";
            s1.placeholder="Last Year Sem 3";
            s2.placeholder="This Year Sem 1";
        }
        if(sem==="sem3"){
            lastYear.placeholder="Last Year Sem 3";
            s1.placeholder="This Year Sem 1";
            s2.placeholder="This Year Sem 2";
        }
    } else{
        if(sem==="sem1"){
            lastYear.placeholder="Last Year Sem 1";
            s1.placeholder="Last Year Sem 2";
        }
        if(sem==="sem2"){
            lastYear.placeholder="Last Year Sem 2";
            s1.placeholder="This Year Sem 1";
        }
    }
}

// SCREEN TIME
let start;
function startScreenTime(){
    start=Date.now();
    setInterval(()=>{
        const sec=Math.floor((Date.now()-start)/1000);
        document.getElementById("screenTime").innerText="Time on site: "+sec+" sec";
    },1000);
}

// AI MODEL
let model;
async function train(){
    const xs=tf.tensor2d([[70,75,80,90,10,85],[80,85,88,95,12,90],[60,65,70,75,6,70]]);
    const ys=tf.tensor2d([[82],[90],[68]]);
    model=tf.sequential();
    model.add(tf.layers.dense({units:16,activation:'relu',inputShape:[6]}));
    model.add(tf.layers.dense({units:1}));
    model.compile({optimizer:'adam',loss:'meanSquaredError'});
    await model.fit(xs,ys,{epochs:200});
    document.getElementById("status").innerText="AI Ready ✅";
}
train();

let lastInput=null;

async function predictScore(){
    const values=[
        parseFloat(lastYear.value)||0,
        parseFloat(s1.value)||0,
        parseFloat(s2.value)||0,
        parseFloat(attendance.value)||0,
        parseFloat(study.value)||0,
        parseFloat(homework.value)||0
    ];
    lastInput=values;
    const pred=model.predict(tf.tensor2d([values]));
    const result=await pred.data();
    document.getElementById("result").innerText="Predicted Score: "+result[0].toFixed(2);

    // 10 Recommendations
    let rec=[];
    if(values[0]<60) rec.push("Improve last year's performance.");
    if(values[1]<60) rec.push("Review previous semester weak topics.");
    if(values[2]<60) rec.push("Focus on missing lessons.");
    if(values[3]<75) rec.push("Increase attendance.");
    if(values[4]<5) rec.push("Study at least 2 more hours weekly.");
    if(values[5]<70) rec.push("Improve homework quality.");
    if(result[0]<70) rec.push("Consider private tutoring.");
    if(values[4]>8) rec.push("Avoid burnout. Balance study & rest.");
    if(values[3]>90) rec.push("Great attendance! Keep it up.");
    if(result[0]>90) rec.push("Excellent performance! Maintain consistency.");

    document.getElementById("recommendations").innerHTML=
        rec.length? "<b>Recommendations:</b><br>"+rec.join("<br>"):"";
}

// LEARN FROM REAL SCORE
async function learnFromRealScore(){
    const real=parseFloat(document.getElementById("realScore").value);
    if(!real||!lastInput){alert("Predict first!");return;}
    await model.fit(tf.tensor2d([lastInput]),tf.tensor2d([[real]]),{epochs:50});
    alert("AI improved using your real score!");
}

updateOptions();

</script>
</body>
</html>

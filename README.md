# student-predictor
<html>
<head>
<meta charset="UTF-8">
<title>AI Student Performance Predictor</title>
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
<style>
    body {
        margin: 0;
        font-family: 'Segoe UI', sans-serif;
        background: linear-gradient(135deg, #0f0f1a, #1a0033);
        color: white;
        display: flex;
        justify-content: center;
        align-items: center;
        min-height: 100vh;
    }
    .container {
        background: rgba(20, 20, 40, 0.85);
        backdrop-filter: blur(15px);
        padding: 40px;
        border-radius: 20px;
        width: 480px;
        box-shadow: 0 0 40px rgba(128, 0, 255, 0.4);
        border: 1px solid rgba(128, 0, 255, 0.3);
    }
    h1 {text-align:center; color:#bb86fc; margin-bottom:20px;}
    input, select {
        width:100%; padding:12px; margin-bottom:15px; border-radius:10px;
        border:none; outline:none; background:#111122; color:white; font-size:14px;
        border:1px solid rgba(128,0,255,0.3);
    }
    input:focus, select:focus {border:1px solid #bb86fc; box-shadow:0 0 10px #bb86fc;}
    button {width:100%; padding:14px; border-radius:12px; border:none;
        background:linear-gradient(90deg,#7b00ff,#bb00ff); color:white;
        font-weight:bold; font-size:15px; cursor:pointer; margin-bottom:10px;}
    button:hover {transform:scale(1.05); box-shadow:0 0 20px #bb00ff;}
    #result {margin-top:15px; text-align:center; font-size:18px; color:#bb86fc;}
    .status {text-align:center; font-size:12px; margin-bottom:10px; color:#aaa;}
    #recommendations {margin-top:15px; font-size:14px; color:#ffcc00;}
</style>
</head>
<body>
<div class="container">

<h1>AI Student Performance Predictor</h1>

<!-- Login -->
<div id="loginDiv">
    <input type="text" id="username" placeholder="Enter your name">
    <button onclick="login()">Login</button>
</div>

<!-- Predictor -->
<div id="predictorDiv" style="display:none;">
    <div class="status" id="status">Training AI Model...</div>
    
    <select id="systemSelect" onchange="updateSemesterOptions(); updatePlaceholders();">
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

    <button onclick="predictScore()">Predict Final Score</button>
    <button onclick="logout()">Logout</button>

    <h2 id="result"></h2>
    <div id="recommendations"></div>
    <div class="status" id="screenTime"></div>
</div>
</div>

<script>
// ===== Login & Remember User =====
function login() {
    const user = document.getElementById("username").value.trim();
    if(!user){alert("Enter your name"); return;}
    localStorage.setItem("loggedInUser", user);
    showPredictor();
}

function logout(){
    localStorage.removeItem("loggedInUser");
    document.getElementById("predictorDiv").style.display="none";
    document.getElementById("loginDiv").style.display="block";
    clearAll();
}

// Auto-login
const loggedIn = localStorage.getItem("loggedInUser");
if(loggedIn){ showPredictor(); }

function showPredictor(){
    document.getElementById("loginDiv").style.display="none";
    document.getElementById("predictorDiv").style.display="block";
    startScreenTime();
}

// ===== Clear all inputs and results =====
function clearAll(){
    document.getElementById("username").value="";
    ["lastYear","s1","s2","attendance","study","homework"].forEach(id=>{
        document.getElementById(id).value="";
    });
    document.getElementById("result").innerText="";
    document.getElementById("recommendations").innerHTML="";
    document.getElementById("screenTime").innerText="";
}

// ===== Dynamic Placeholders =====
const systemSelect = document.getElementById("systemSelect");
const predictSelect = document.getElementById("predictSelect");

function updateSemesterOptions(){
    if(systemSelect.value==="national"){
        predictSelect.innerHTML = `
            <option value="sem1">Predict Semester 1</option>
            <option value="sem2">Predict Semester 2</option>
        `;
    } else {
        predictSelect.innerHTML = `
            <option value="sem1">Predict Semester 1</option>
            <option value="sem2">Predict Semester 2</option>
            <option value="sem3">Predict Semester 3</option>
        `;
    }
}

function updatePlaceholders(){
    const system = systemSelect.value;
    const predict = predictSelect.value;
    let lastText="", s1Text="", s2Text="";

    if(system==="igcse"){
        if(predict==="sem1"){ lastText="Last Year Sem 1"; s1Text="Last Year Sem 2"; s2Text="Last Year Sem 3"; }
        else if(predict==="sem2"){ lastText="Last Year Sem 2"; s1Text="Last Year Sem 3"; s2Text="This Year Sem 1"; }
        else if(predict==="sem3"){ lastText="Last Year Sem 3"; s1Text="This Year Sem 1"; s2Text="This Year Sem 2"; }
    } else { // National
        if(predict==="sem1"){ lastText="Last Year Sem 1"; s1Text="Last Year Sem 2"; }
        else if(predict==="sem2"){ lastText="Last Year Sem 2"; s1Text="This Year Sem 1"; }
        s2Text="";
    }

    document.getElementById("lastYear").placeholder=lastText;
    document.getElementById("s1").placeholder=s1Text;
    document.getElementById("s2").placeholder=s2Text;
}

// ===== Screen Time =====
let startTime;
function startScreenTime(){
    startTime=Date.now();
    setInterval(()=>{
        const diff = Math.floor((Date.now()-startTime)/1000);
        document.getElementById("screenTime").innerText="Time on predictor: "+diff+" sec";
    },1000);
}

// ===== TensorFlow AI =====
let model;
async function trainModel(){
    const xs = tf.tensor2d([
        [70,75,80,90,10,85],
        [80,85,88,95,12,90],
        [60,65,70,75,6,70],
        [90,92,95,98,15,95],
        [50,55,60,65,4,60],
        [85,87,90,93,11,88],
        [72,74,78,85,9,80],
        [68,70,75,82,8,77]
    ]);
    const ys = tf.tensor2d([[82],[90],[68],[96],[58],[89],[80],[76]]);
    model = tf.sequential();
    model.add(tf.layers.dense({units:16, activation:'relu', inputShape:[6]}));
    model.add(tf.layers.dense({units:8, activation:'relu'}));
    model.add(tf.layers.dense({units:1}));
    model.compile({optimizer:tf.train.adam(0.01), loss:'meanSquaredError'});
    await model.fit(xs, ys, {epochs:300});
    document.getElementById("status").innerText="AI Model Ready ✅";
}
trainModel();

// ===== Predict & Expanded Recommendations =====
async function predictScore(){
    if(!model){alert("Model still training"); return;}

    const inputs = [
        parseFloat(document.getElementById("lastYear").value)||0,
        parseFloat(document.getElementById("s1").value)||0,
        parseFloat(document.getElementById("s2").value)||0,
        parseFloat(document.getElementById("attendance").value)||0,
        parseFloat(document.getElementById("study").value)||0,
        parseFloat(document.getElementById("homework").value)||0
    ];

    const prediction = model.predict(tf.tensor2d([inputs]));
    const result = await prediction.data();
    document.getElementById("result").innerText="Predicted Final Score: "+result[0].toFixed(2);

    // Expanded Recommendations (6 points)
    let recs = [];
    if(inputs[0]<60) recs.push("Improve last year's performance.");
    if(inputs[1]<60) recs.push("Review previous semester concepts.");
    if(inputs[2]<60) recs.push("Catch up on missing topics.");
    if(inputs[3]<75) recs.push("Increase attendance.");
    if(inputs[4]<5) recs.push("Study more hours per week.");
    if(inputs[5]<70) recs.push("Improve homework score.");
    if(result[0]<70) recs.push("Consider extra tutoring.");
    document.getElementById("recommendations").innerHTML = recs.length>0 ? "<b>Recommendations:</b><br>"+recs.join("<br>") : "";
}
</script>
</body>
</html>


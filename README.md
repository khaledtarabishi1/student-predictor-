# student-predictor
<html>
<head>
    <meta charset="UTF-8">
    <title>Adaptive AI Semester 3 Predictor</title>

    <!-- TensorFlow -->
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
            height: 100vh;
        }
        .container {
            background: rgba(20, 20, 40, 0.9);
            backdrop-filter: blur(15px);
            padding: 40px;
            border-radius: 20px;
            width: 520px;
            box-shadow: 0 0 50px rgba(128, 0, 255, 0.5);
            border: 1px solid rgba(128, 0, 255, 0.3);
        }
        h1 {
            text-align: center;
            margin-bottom: 20px;
            font-size: 26px;
            color: #bb86fc;
        }
        input {
            width: 100%;
            padding: 12px;
            margin-bottom: 12px;
            border-radius: 10px;
            border: none;
            outline: none;
            background: #111122;
            color: white;
            font-size: 14px;
            border: 1px solid rgba(128, 0, 255, 0.3);
        }
        input:focus {
            border: 1px solid #bb86fc;
            box-shadow: 0 0 10px #bb86fc;
        }
        button {
            width: 100%;
            padding: 14px;
            border-radius: 12px;
            border: none;
            background: linear-gradient(90deg, #7b00ff, #bb00ff);
            color: white;
            font-weight: bold;
            font-size: 16px;
            cursor: pointer;
            transition: 0.3s;
        }
        button:hover {
            transform: scale(1.05);
            box-shadow: 0 0 20px #bb00ff;
        }
        #result {
            margin-top: 20px;
            text-align: center;
            font-size: 18px;
            color: #bb86fc;
        }
        #recommendation {
            margin-top: 15px;
            text-align: center;
            font-size: 16px;
            color: #ffcc00;
            line-height: 1.4;
        }
        .status {
            text-align: center;
            font-size: 12px;
            margin-bottom: 15px;
            color: #aaa;
        }
        .login-section, .predict-section {
            display: none;
        }
    </style>
</head>
<body>

<div class="container">

    <div class="login-section" id="loginSection">
        <h1>Login / Signup</h1>
        <input type="text" id="username" placeholder="Username">
        <input type="password" id="password" placeholder="Password">
        <label>
            <input type="checkbox" id="rememberMe"> Remember Me
        </label>
        <button onclick="login()">Login / Signup</button>
        <div id="loginStatus" class="status"></div>
    </div>

    <div class="predict-section" id="predictSection">
        <h1>Semester 3 Predictor</h1>
        <div class="status" id="status">Training AI Model...</div>

        <input type="number" id="lastYear" placeholder="Last Year Score">
        <input type="number" id="sem1" placeholder="Semester 1 Score">
        <input type="number" id="sem2" placeholder="Semester 2 Score">
        <input type="number" id="attendance" placeholder="Attendance %">
        <input type="number" id="study" placeholder="Study Hours per Week">
        <input type="number" id="homework" placeholder="Homework Score">
        <input type="number" id="screenTime" placeholder="Screen Time Hours per Day">
        <input type="number" id="realScore" placeholder="Enter Real Score (optional)">

        <button onclick="predictScore()">Predict Semester 3 Score</button>
        <h2 id="result"></h2>
        <div id="recommendation">Enter your scores and click Predict to get recommendations.</div>
        <button onclick="logout()">Logout</button>
    </div>

</div>

<script>
let model;
let currentUser = null;

// --- Login / Signup ---
function login() {
    const username = document.getElementById("username").value.trim();
    const password = document.getElementById("password").value.trim();
    const remember = document.getElementById("rememberMe").checked;

    if (!username || !password) {
        document.getElementById("loginStatus").innerText = "Please enter username and password.";
        return;
    }

    let users = JSON.parse(localStorage.getItem("students")) || {};

    if (!users[username]) {
        // Signup
        users[username] = { password: password, history: [] };
        localStorage.setItem("students", JSON.stringify(users));
        document.getElementById("loginStatus").innerText = "Signup successful!";
    } else {
        if (users[username].password !== password) {
            document.getElementById("loginStatus").innerText = "Wrong password!";
            return;
        }
        document.getElementById("loginStatus").innerText = "Login successful!";
    }

    currentUser = username;
    if (remember) localStorage.setItem("currentUser", username);
    showPredictSection();
}

// Check if user is remembered
window.onload = () => {
    const remembered = localStorage.getItem("currentUser");
    if (remembered) {
        currentUser = remembered;
        showPredictSection();
    } else {
        document.getElementById("loginSection").style.display = "block";
    }
};

function logout() {
    currentUser = null;
    localStorage.removeItem("currentUser");
    document.getElementById("loginSection").style.display = "block";
    document.getElementById("predictSection").style.display = "none";
}

// --- Show Predictor ---
function showPredictSection() {
    document.getElementById("loginSection").style.display = "none";
    document.getElementById("predictSection").style.display = "block";
    trainModel();
}

// --- Train AI ---
async function trainModel() {
    // Generic training data
    let xsData = [
        [80,75,78,90,10,85,3],
        [70,72,74,85,8,70,4],
        [90,88,92,95,12,95,2],
        [60,65,68,75,6,65,5],
        [85,82,84,92,11,88,3],
        [78,80,79,88,9,80,4]
    ];
    let ysData = [
        [79],[71],[93],[66],[87],[80]
    ];

    // Append current user's history
    if (currentUser) {
        const users = JSON.parse(localStorage.getItem("students")) || {};
        const history = users[currentUser].history || [];
        history.forEach(item => {
            xsData.push([
                item.lastYear, item.s1, item.s2, item.attendance, 
                item.study, item.homework, item.screenTime
            ]);
            ysData.push([item.predicted]);
        });
    }

    const xs = tf.tensor2d(xsData);
    const ys = tf.tensor2d(ysData);

    model = tf.sequential();
    model.add(tf.layers.dense({ units:16, activation:'relu', inputShape:[7] }));
    model.add(tf.layers.dense({ units:8, activation:'relu' }));
    model.add(tf.layers.dense({ units:1 }));

    model.compile({ optimizer: tf.train.adam(0.01), loss: 'meanSquaredError' });

    await model.fit(xs, ys, { epochs:300 });
    document.getElementById("status").innerText = "AI Model Ready ✅";
}

// --- Predict ---
async function predictScore() {
    if (!model) { alert("Model still training..."); return; }

    const lastYear = parseFloat(document.getElementById("lastYear").value) || 0;
    const s1 = parseFloat(document.getElementById("sem1").value) || 0;
    const s2 = parseFloat(document.getElementById("sem2").value) || 0;
    const attendance = parseFloat(document.getElementById("attendance").value) || 0;
    const study = parseFloat(document.getElementById("study").value) || 0;
    const homework = parseFloat(document.getElementById("homework").value) || 0;
    const screenTime = parseFloat(document.getElementById("screenTime").value) || 0;
    const real = parseFloat(document.getElementById("realScore").value);

    const input = tf.tensor2d([[lastYear,s1,s2,attendance,study,ho]()]()

# student-predictor
<html>
<head>
    <meta charset="UTF-8">
    <title>AI Student Performance Predictor</title>

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
            width: 480px;
            box-shadow: 0 0 50px rgba(128, 0, 255, 0.5);
            border: 1px solid rgba(128, 0, 255, 0.3);
        }

        h1 {
            text-align: center;
            margin-bottom: 25px;
            font-size: 26px;
            color: #bb86fc;
        }

        input {
            width: 100%;
            padding: 12px;
            margin-bottom: 15px;
            border-radius: 10px;
            border: none;
            outline: none;
            background: #111122;
            color: white;
            font-size: 14px;
            border: 1px solid rgba(128, 0, 255, 0.3);
            transition: 0.3s;
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
    </style>
</head>

<body>
<div class="container">
    <h1>AI Student Performance Predictor</h1>

    <div class="status" id="status">Training AI Model...</div>

    <input type="number" id="sem1" placeholder="Semester 1 Score">
    <input type="number" id="sem2" placeholder="Semester 2 Score">
    <input type="number" id="sem3" placeholder="Semester 3 Score">
    <input type="number" id="attendance" placeholder="Attendance %">
    <input type="number" id="study" placeholder="Study Hours per Week">
    <input type="number" id="homework" placeholder="Homework Score">
    <input type="number" id="screenTime" placeholder="Screen Time Hours per Day">
    <input type="number" id="realScore" placeholder="Enter Real Score (optional)">

    <button onclick="predictScore()">Predict Final Score</button>

    <h2 id="result"></h2>
    <div id="recommendation">Enter your scores and click Predict to get recommendations.</div>
</div>

<script>
let model;

async function trainModel() {
    const xs = tf.tensor2d([
        [70,75,80,90,10,85,3],
        [80,85,88,95,12,90,2],
        [60,65,70,75,6,70,4],
        [90,92,95,98,15,95,1],
        [50,55,60,65,4,60,5],
        [85,87,90,93,11,88,2],
        [72,74,78,85,9,80,3],
        [68,70,75,82,8,77,4]
    ]);

    const ys = tf.tensor2d([
        [82],[90],[68],[96],[58],[89],[80],[76]
    ]);

    model = tf.sequential();
    model.add(tf.layers.dense({ units:16, activation:'relu', inputShape:[7] }));
    model.add(tf.layers.dense({ units:8, activation:'relu' }));
    model.add(tf.layers.dense({ units:1 }));

    model.compile({
        optimizer: tf.train.adam(0.01),
        loss: 'meanSquaredError'
    });

    await model.fit(xs, ys, { epochs:300 });
    document.getElementById("status").innerText = "AI Model Ready ✅";
}

async function predictScore() {
    if (!model) {
        alert("Model still training...");
        return;
    }

    const s1 = parseFloat(document.getElementById("sem1").value) || 0;
    const s2 = parseFloat(document.getElementById("sem2").value) || 0;
    const s3 = parseFloat(document.getElementById("sem3").value) || 0;
    const attendance = parseFloat(document.getElementById("attendance").value) || 0;
    const study = parseFloat(document.getElementById("study").value) || 0;
    const homework = parseFloat(document.getElementById("homework").value) || 0;
    const screenTime = parseFloat(document.getElementById("screenTime").value) || 0;
    const real = parseFloat(document.getElementById("realScore").value);

    const input = tf.tensor2d([[s1,s2,s3,attendance,study,homework,screenTime]]);
    const prediction = model.predict(input);
    const result = await prediction.data();

    document.getElementById("result").innerText =
        "Predicted Final Score: " + result[0].toFixed(2);

    let rec = "";
    const score = result[0];

    if (score >= 95) rec = "Outstanding! Maintain excellent habits and keep excelling 🌟";
    else if (score >= 90) rec = "Excellent! Keep up the great work 🎉";
    else if (score >= 80) rec = "Good job! Focus on weak subjects and maintain attendance 💪";
    else if (score >= 70) rec = "Average. Increase study hours and reduce distractions ⚡";
    else if (score >= 60) rec = "Below average. Pay attention to homework and study more 📚";
    else rec = "Needs improvement. Consider revising all subjects and managing screen time 🛑";

    if (screenTime > 5) rec += " | Reduce screen time for better focus ⏱️";
    else if (screenTime >= 3) rec += " | Moderate screen time, balance study and use ⚖️";
    else rec += " | Low screen time — great for focus 👍";

    if (!isNaN(real)) {
        rec += ` | Predicted: ${score.toFixed(2)}, Actual: ${real}`;
        const diff = score - real;
        if (Math.abs(diff) >= 10) rec += " | Big difference! Track your study habits 📊";
        else rec += " | Prediction close! Keep monitoring performance ✅";
    }

    document.getElementById("recommendation").innerText = rec;
}

trainModel();
</script>
</body>
</html>

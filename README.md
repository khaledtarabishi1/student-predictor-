# student-predictor-<!DOCTYPE html>
<!DOCTYPE html>
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
            background: rgba(20, 20, 40, 0.85);
            backdrop-filter: blur(15px);
            padding: 40px;
            border-radius: 20px;
            width: 420px;
            box-shadow: 0 0 40px rgba(128, 0, 255, 0.4);
            border: 1px solid rgba(128, 0, 255, 0.3);
        }

        h1 {
            text-align: center;
            margin-bottom: 25px;
            font-size: 24px;
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
            font-size: 15px;
            cursor: pointer;
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

        .status {
            text-align: center;
            font-size: 12px;
            margin-bottom: 10px;
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

    <button onclick="predictScore()">Predict Final Score</button>

    <h2 id="result"></h2>
</div>

<script>
let model;

async function trainModel() {
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

    const ys = tf.tensor2d([
        [82],[90],[68],[96],[58],[89],[80],[76]
    ]);

    model = tf.sequential();
    model.add(tf.layers.dense({ units:16, activation:'relu', inputShape:[6] }));
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
    if (!model) return;

    const s1 = parseFloat(document.getElementById("sem1").value);
    const s2 = parseFloat(document.getElementById("sem2").value);
    const s3 = parseFloat(document.getElementById("sem3").value);
    const attendance = parseFloat(document.getElementById("attendance").value);
    const study = parseFloat(document.getElementById("study").value);
    const homework = parseFloat(document.getElementById("homework").value);

    const input = tf.tensor2d([[s1,s2,s3,attendance,study,homework]]);
    const prediction = model.predict(input);
    const result = await prediction.data();

    document.getElementById("result").innerText =
        "Predicted Final Score: " + result[0].toFixed(2);
}

trainModel();
</script>

</body>
</html>


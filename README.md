# student-predictor-<!DOCTYPE html>
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
            borde

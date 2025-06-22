<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>モバイルマイクラ</title>
  <style>
    body {
      margin: 0;
      background: #222;
      color: #fff;
      font-family: sans-serif;
      text-align: center;
    }
    canvas {
      background: #444;
      touch-action: manipulation;
      margin-top: 10px;
    }
  </style>
</head>
<body>
  <h1>モバイルマイクラ（仮）</h1>
  <canvas id="gameCanvas" width="300" height="300"></canvas>
  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");

    const gridSize = 30;
    const rows = 10;
    const cols = 10;
    const blocks = [];

    for (let y = 0; y < rows; y++) {
      blocks[y] = [];
      for (let x = 0; x < cols; x++) {
        blocks[y][x] = 0; // 0: 空白, 1: ブロック
      }
    }

    function draw() {
      for (let y = 0; y < rows; y++) {
        for (let x = 0; x < cols; x++) {
          ctx.fillStyle = blocks[y][x] ? "#66cc66" : "#333";
          ctx.fillRect(x * gridSize, y * gridSize, gridSize - 1, gridSize - 1);
        }
      }
    }

    canvas.addEventListener("touchstart", (e) => {
      const rect = canvas.getBoundingClientRect();
      const touch = e.touches[0];
      const x = Math.floor((touch.clientX - rect.left) / gridSize);
      const y = Math.floor((touch.clientY - rect.top) / gridSize);
      blocks[y][x] = blocks[y][x] ? 0 : 1;
      draw();
    });

    draw();
  </script>
</body>
</html>

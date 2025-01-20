<!DOCTYPE html>
<html>
<head>
  <title>Snake Game with Realistic Snake</title>
  <meta charset="UTF-8">
  <style>
    html, body {
      height: 100%;
      margin: 0;
      
    }
    .container{
      background-image: url(background.jpg);
      width: 100%;
      height: 120vh;
    }
    .all{
    max-width: 100%;
    max-height: 100vh;
    background-position: center;
    background-size: cover;
}
.head{
  max-width: 100%;
    max-height: 100vh;
    padding-left: 800px;

}
    body {
      background: rgb(0, 18, 56);
      display: flex;
      align-items: center;
      justify-content: center;
      flex-direction: column;
    }
    canvas {
      border: 8px solid rgb(212, 212, 212);
    }
    #controls {
      margin-top: 10px;
    }
    button:hover{
      background-color: #04AA6D; /* Green */
  border: none;
  color: white;
  padding: 16px 32px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  margin: 4px 2px;
  transition-duration: 0.4s;
  cursor: pointer;
  background-color: #04AA6D;
  color: white;
  }


    #score, #timer, #highScore {
      color: rgb(255, 255, 255);
      font-size: 20px;
      margin-bottom: 10px;
      color: white;
      text-shadow: 2px 2px 4px #000000;
    }
  </style>
</head>
<body>
  <div class="container">
    <div class="all">

      <div class="head">
  <div id="highScore">High Score: 0</div>
  <div id="score">Score: 0</div>
  <div id="timer">Time: 0s</div>
  <canvas width="400" height="400" id="game"></canvas>
  <div id="controls">
    <button id="start">Start</button>
    <button id="stop">Stop</button>

  </div>
  </div>

  <script>
    var canvas = document.getElementById('game');
    var context = canvas.getContext('2d');
    var grid = 16;
    var count = 0;
    var running = false;
    var score = 0;
    var highScore = 0;
    var timeElapsed = 0;
    var timerInterval;
    var largeBall = null; // Large ball object
    var snake = {
      x: 160,
      y: 160,
      dx: grid,
      dy: 0,
      cells: [],
      maxCells: 4
    };
    var apple = {
      x: 320,
      y: 320
    };

    function getRandomInt(min, max) {
      return Math.floor(Math.random() * (max - min)) + min;
    }

    function updateScore() {
      document.getElementById('score').textContent = 'Score: ' + score;
      if (score > highScore) {
        highScore = score;
        document.getElementById('highScore').textContent = 'High Score: ' + highScore;
      }
    }

    function updateTimer() {
      document.getElementById('timer').textContent = 'Time: ' + timeElapsed + 's';
    }

    function resetGame() {
      snake.x = 160;
      snake.y = 160;
      snake.cells = [];
      snake.maxCells = 4;
      snake.dx = grid;
      snake.dy = 0;
      apple.x = getRandomInt(0, 25) * grid;
      apple.y = getRandomInt(0, 25) * grid;
      score = 0;
      timeElapsed = 0;
      largeBall = null;
      updateScore();
      updateTimer();
      clearInterval(timerInterval);
    }

    function startTimer() {
      timerInterval = setInterval(function() {
        timeElapsed++;
        updateTimer();
      }, 1000);
    }

    function stopTimer() {
      clearInterval(timerInterval);
    }

    function spawnLargeBall() {
      largeBall = {
        x: getRandomInt(0, 25) * grid,
        y: getRandomInt(0, 25) * grid
      };
    }

    function drawSnake() {
      snake.cells.forEach(function(cell, index) {
        // Draw the tail
        context.fillStyle = 'green';
        context.fillRect(cell.x, cell.y, grid - 1, grid - 1);

        // Draw the head
        if (index === 0) {
          context.fillStyle = 'darkGreen';
          context.fillRect(cell.x, cell.y, grid - 1, grid - 1);

          // Open mouth if near apple or large ball
          if ((cell.x === apple.x && cell.y === apple.y) || (largeBall && cell.x === largeBall.x && cell.y === largeBall.y)) {
            context.fillStyle = 'yellow';
            context.beginPath();
            context.arc(cell.x + grid / 2, cell.y + grid / 2, grid / 2, 0, Math.PI * 2);
            context.fill();
          }
        }
      });
    }

    function loop() {
      if (!running) return;
      requestAnimationFrame(loop);

      if (++count < 4) {
        return;
      }

      count = 0;
      context.clearRect(0, 0, canvas.width, canvas.height);

      snake.x += snake.dx;
      snake.y += snake.dy;

      if (snake.x < 0) {
        snake.x = canvas.width - grid;
      } else if (snake.x >= canvas.width) {
        snake.x = 0;
      }

      if (snake.y < 0) {
        snake.y = canvas.height - grid;
      } else if (snake.y >= canvas.height) {
        snake.y = 0;
      }

      snake.cells.unshift({ x: snake.x, y: snake.y });

      if (snake.cells.length > snake.maxCells) {
        snake.cells.pop();
      }

      // Draw apple
      context.fillStyle = 'red';
      context.fillRect(apple.x, apple.y, grid - 1, grid - 1);

      // Draw large ball
      if (largeBall) {
        context.fillStyle = 'blue';
        context.fillRect(largeBall.x, largeBall.y, grid - 1, grid - 1);
      }

      // Draw snake
      drawSnake();

      // Check collision with apple
      if (snake.cells[0].x === apple.x && snake.cells[0].y === apple.y) {
        snake.maxCells++;
        score++;
        updateScore();
        apple.x = getRandomInt(0, 25) * grid;
        apple.y = getRandomInt(0, 25) * grid;
        if (score % 10 === 0) {
          spawnLargeBall();
        }
      }

      // Check collision with large ball
      if (largeBall && snake.cells[0].x === largeBall.x && snake.cells[0].y === largeBall.y) {
        score += 20;
        updateScore();
        largeBall = null;
      }

      // Check collision with itself
      for (var i = 1; i < snake.cells.length; i++) {
        if (snake.cells[0].x === snake.cells[i].x && snake.cells[0].y === snake.cells[i].y) {
          resetGame();
        }
      }
    }

    document.addEventListener('keydown', function(e) {
      if (e.which === 37 && snake.dx === 0) {
        snake.dx = -grid;
        snake.dy = 0;
      } else if (e.which === 38 && snake.dy === 0) {
        snake.dy = -grid;
        snake.dx = 0;
      } else if (e.which === 39 && snake.dx === 0) {
        snake.dx = grid;
        snake.dy = 0;
      } else if (e.which === 40 && snake.dy === 0) {
        snake.dy = grid;
        snake.dx = 0;
      }
    });

    document.getElementById('start').addEventListener('click', function() {
      if (!running) {
        running = true;
        startTimer();
        requestAnimationFrame(loop);
      }
    });

    document.getElementById('stop').addEventListener('click', function() {
      running = false;
      stopTimer();
    });

    resetGame();
  </script>
</body>
</div>
</div>
</html>

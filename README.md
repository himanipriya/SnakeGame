# DevOps
@@ -0,0 +1,24 @@
# Use the official Python image from the Docker Hub
FROM python:3.10-slim
# Set the working directory in the container
WORKDIR /app
# Copy the requirements file into the container
COPY requirements.txt .
# Install the dependencies
RUN pip install --no-cache-dir -r requirements.txt
# Copy the rest of the application code into the container
COPY . .
# Expose the port that the Flask app will run on
EXPOSE 5000
# Set the environment variable for Flask
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
# Run the Flask app
CMD ["flask", "run"]
‎app.py
+10
Original file line number	Diff line number	Diff line change
@@ -0,0 +1,10 @@
from flask import Flask, render_template
app = Flask(__name__)
@app.route('/')
def index():
    return render_template('index.html')
if __name__ == '__main__':
    app.run(debug=True)
‎requirements.txt
+2


Original file line number	Diff line number	Diff line change
@@ -0,0 +1,2 @@
Flask==2.3.3
pytest==7.3.1
‎static/eat.mp3
17.3 KB
Binary file not shown.
‎static/game.js
+99
Original file line number	Diff line number	Diff line change
@@ -0,0 +1,99 @@
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const canvasWidth = 500;
const canvasHeight = 500;
const numberOfCells = 25; // Number of cells on the canvas
const cellSize = Math.floor(canvasWidth / numberOfCells); // Calculate cell size
const OFFSET = 0; // You can adjust this as needed
const GREEN = 'rgb(173, 204, 96)';
const DARK_GREEN = 'rgb(43, 51, 24)';
let snake = [{ x: 6, y: 9 }, { x: 5, y: 9 }, { x: 4, y: 9 }];
let direction = { x: 1, y: 0 };
let food = generateRandomPos();
let score = 0;
let gameInterval;
// Load sounds
const eatSound = new Audio('eat.mp3');
const wallHitSound = new Audio('wall.mp3');
function draw() {
    ctx.fillStyle = GREEN;
    ctx.fillRect(0, 0, canvasWidth, canvasHeight);
    ctx.fillStyle = DARK_GREEN;
    snake.forEach(segment => {
        ctx.fillRect(OFFSET + segment.x * cellSize, OFFSET + segment.y * cellSize, cellSize, cellSize);
    });
    ctx.fillStyle = 'red';
    ctx.fillRect(OFFSET + food.x * cellSize, OFFSET + food.y * cellSize, cellSize, cellSize);
    // Draw score background
    ctx.fillStyle = 'white'; // Background color for text
    ctx.fillRect(OFFSET - 10, 10, 180, 50); // Adjust as needed
    ctx.fillStyle = DARK_GREEN;
    ctx.font = '40px Arial';
    ctx.textAlign = 'left';
    ctx.textBaseline = 'top';
    ctx.fillText(`Score: ${score}`, OFFSET, 20);  // Adjusted Y position here
}
function update() {
    const head = { ...snake[0] };
    head.x += direction.x;
    head.y += direction.y;
    snake.unshift(head);
    if (head.x === food.x && head.y === food.y) {
        score++;
        food = generateRandomPos();
        eatSound.play(); // Play eat sound
    } else {
        snake.pop();
    }
    if (head.x < 0 || head.x >= numberOfCells || head.y < 0 || head.y >= numberOfCells || 
        snake.slice(1).some(segment => segment.x === head.x && segment.y === head.y)) {
        wallHitSound.play(); // Play wall hit sound
        clearInterval(gameInterval);
        alert('Game Over!');
    }
    draw();
}
function generateRandomPos() {
    let pos;
    do {
        pos = { 
            x: Math.floor(Math.random() * numberOfCells), 
            y: Math.floor(Math.random() * numberOfCells) 
        };
    } while (snake.some(segment => segment.x === pos.x && segment.y === pos.y));
    return pos;
}
document.addEventListener('keydown', (e) => {
    switch (e.key) {
        case 'ArrowUp':
            if (direction.y === 0) direction = { x: 0, y: -1 };
            break;
        case 'ArrowDown':
            if (direction.y === 0) direction = { x: 0, y: 1 };
            break;
        case 'ArrowLeft':
            if (direction.x === 0) direction = { x: -1, y: 0 };
            break;
        case 'ArrowRight':
            if (direction.x === 0) direction = { x: 1, y: 0 };
            break;
    }
});
gameInterval = setInterval(update, 200);
‎static/preview.jpg
38.6 KB



‎static/styles.css
+12
Original file line number	Diff line number	Diff line change
@@ -0,0 +1,12 @@
body {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    margin: 0;
    background-color: #f0f0f0;
}
canvas {
    border: 1px solid black;
}
‎static/wall.mp3
46.5 KB
Binary file not shown.
‎templates/index.html
+13
Original file line number	Diff line number	Diff line change
@@ -0,0 +1,13 @@
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GFG Snake Game</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='styles.css') }}">
</head>
<body>
    <canvas id="gameCanvas" width="500" height="500"></canvas>
    <script src="{{ url_for('static', filename='game.js') }}"></script>
</body>
</html>
‎test_app.py
+20
Original file line number	Diff line number	Diff line change
@@ -0,0 +1,20 @@
import unittest
from app import app
class FlaskAppTests(unittest.TestCase):
    @classmethod
    def setUpClass(cls):
        cls.client = app.test_client()
        cls.client.testing = True
    def test_index_page(self):
        response = self.client.get('/')
        self.assertEqual(response.status_code, 200)
        self.assertIn(b'GFG Snake Game', response.data)
    
    def test_static_files(self):
        response = self.client.get('/static/game.js')
        self.assertEqual(response.status_code, 200)
        self.assertIn(b'const canvas', response.data)
if __name__ == '__main__':
    unittest.main()

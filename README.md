import { useEffect, useRef, useState } from "react";

const PongGame = () => {
  const canvasRef = useRef(null);
  const [gameInterval, setGameInterval] = useState(null);

  const paddleHeight = 100;
  const paddleWidth = 10;
  const ballSize = 10;
  const canvasWidth = 800;
  const canvasHeight = 400;

  let paddleSpeed = 5;
  let ballSpeedX = 4;
  let ballSpeedY = 4;

  // Paddle positions
  let playerPaddleY = canvasHeight / 2 - paddleHeight / 2;
  let aiPaddleY = canvasHeight / 2 - paddleHeight / 2;

  // Ball position
  let ballX = canvasWidth / 2 - ballSize / 2;
  let ballY = canvasHeight / 2 - ballSize / 2;

  // Paddle movement
  const [playerUp, setPlayerUp] = useState(false);
  const [playerDown, setPlayerDown] = useState(false);

  const draw = (context) => {
    // Clear canvas
    context.clearRect(0, 0, canvasWidth, canvasHeight);

    // Draw paddles
    context.fillStyle = "white";
    context.fillRect(0, playerPaddleY, paddleWidth, paddleHeight); // Player Paddle
    context.fillRect(canvasWidth - paddleWidth, aiPaddleY, paddleWidth, paddleHeight); // AI Paddle

    // Draw ball
    context.beginPath();
    context.arc(ballX, ballY, ballSize, 0, Math.PI * 2);
    context.fillStyle = "white";
    context.fill();
    context.closePath();
  };

  const moveBall = () => {
    // Ball movement
    ballX += ballSpeedX;
    ballY += ballSpeedY;

    // Bounce off top and bottom
    if (ballY <= 0 || ballY >= canvasHeight) {
      ballSpeedY = -ballSpeedY;
    }

    // Bounce off paddles
    if (
      ballX <= paddleWidth &&
      ballY >= playerPaddleY &&
      ballY <= playerPaddleY + paddleHeight
    ) {
      ballSpeedX = -ballSpeedX;
    }
    if (
      ballX >= canvasWidth - paddleWidth - ballSize &&
      ballY >= aiPaddleY &&
      ballY <= aiPaddleY + paddleHeight
    ) {
      ballSpeedX = -ballSpeedX;
    }

    // Ball out of bounds (reset position)
    if (ballX <= 0 || ballX >= canvasWidth) {
      ballX = canvasWidth / 2 - ballSize / 2;
      ballY = canvasHeight / 2 - ballSize / 2;
      ballSpeedX = -ballSpeedX;
    }
  };

  const moveAI = () => {
    // AI movement
    if (ballY < aiPaddleY + paddleHeight / 2) {
      aiPaddleY -= paddleSpeed;
    } else {
      aiPaddleY += paddleSpeed;
    }

    // Prevent AI from going out of bounds
    aiPaddleY = Math.max(0, Math.min(aiPaddleY, canvasHeight - paddleHeight));
  };

  const movePlayer = () => {
    // Player movement
    if (playerUp && playerPaddleY > 0) {
      playerPaddleY -= paddleSpeed;
    }
    if (playerDown && playerPaddleY < canvasHeight - paddleHeight) {
      playerPaddleY += paddleSpeed;
    }
  };

  const gameLoop = () => {
    const canvas = canvasRef.current;
    const context = canvas.getContext("2d");

    moveBall();
    moveAI();
    movePlayer();
    draw(context);
  };

  useEffect(() => {
    const canvas = canvasRef.current;
    const context = canvas.getContext("2d");

    // Start the game loop
    setGameInterval(setInterval(gameLoop, 1000 / 60));

    // Clean up interval on unmount
    return () => {
      clearInterval(gameInterval);
    };
  }, [gameInterval]);

  // Keyboard controls
  const handleKeyDown = (e) => {
    if (e.key === "ArrowUp") {
      setPlayerUp(true);
    }
    if (e.key === "ArrowDown") {
      setPlayerDown(true);
    }
  };

  const handleKeyUp = (e) => {
    if (e.key === "ArrowUp") {
      setPlayerUp(false);
    }
    if (e.key === "ArrowDown") {
      setPlayerDown(false);
    }
  };

  // Event listeners
  useEffect(() => {
    window.addEventListener("keydown", handleKeyDown);
    window.addEventListener("keyup", handleKeyUp);

    return () => {
      window.removeEventListener("keydown", handleKeyDown);
      window.removeEventListener("keyup", handleKeyUp);
    };
  }, []);

  return (
    <div>
      <canvas
        ref={canvasRef}
        width={canvasWidth}
        height={canvasHeight}
        style={{ backgroundColor: "#000", display: "block", margin: "auto" }}
      ></canvas>
    </div>
  );
};

export default PongGame;

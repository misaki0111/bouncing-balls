const canvas = document.getElementById('container');
const ctx = canvas.getContext('2d');

const colors = ['#39d353', '#ff9800', '#a259f7', '#ffe600', '#ff69b4', '#ff69b4', '#ff69b4', '#ff69b4']; // 绿色、橙色、紫色、黄色、粉色*4
const balls = [];
const BALL_RADIUS = 24;
const BALL_COUNT = 8;

function randomSpeed() {
    // 进一步降低速度范围
    return (Math.random() * 0.5 + 0.3) * (Math.random() > 0.5 ? 1 : -1);
}

for (let i = 0; i < BALL_COUNT; i++) {
    let safe = false;
    let x, y;
    // 保证初始不重叠
    while (!safe) {
        x = Math.random() * (canvas.width - 2 * BALL_RADIUS) + BALL_RADIUS;
        y = Math.random() * (canvas.height - 2 * BALL_RADIUS) + BALL_RADIUS;
        safe = true;
        for (let j = 0; j < balls.length; j++) {
            const dx = x - balls[j].x;
            const dy = y - balls[j].y;
            if (Math.sqrt(dx * dx + dy * dy) < 2 * BALL_RADIUS) {
                safe = false;
                break;
            }
        }
    }
    balls.push({
        x,
        y,
        vx: randomSpeed(),
        vy: randomSpeed(),
        color: colors[i],
    });
}

function drawBall(ball) {
    ctx.beginPath();
    ctx.arc(ball.x, ball.y, BALL_RADIUS, 0, Math.PI * 2);
    ctx.fillStyle = ball.color;
    ctx.shadowColor = ball.color;
    ctx.shadowBlur = 16;
    ctx.fill();
    ctx.closePath();
    ctx.shadowBlur = 0;
}

function ballCollision(b1, b2) {
    const dx = b2.x - b1.x;
    const dy = b2.y - b1.y;
    const dist = Math.sqrt(dx * dx + dy * dy);
    if (dist < 2 * BALL_RADIUS) {
        // 计算单位法线和切线
        const nx = dx / dist;
        const ny = dy / dist;
        const tx = -ny;
        const ty = nx;
        // 投影到法线和切线方向
        const v1n = b1.vx * nx + b1.vy * ny;
        const v1t = b1.vx * tx + b1.vy * ty;
        const v2n = b2.vx * nx + b2.vy * ny;
        const v2t = b2.vx * tx + b2.vy * ty;
        // 交换法线分量，切线分量不变
        const v1nAfter = v2n;
        const v2nAfter = v1n;
        b1.vx = v1nAfter * nx + v1t * tx;
        b1.vy = v1nAfter * ny + v1t * ty;
        b2.vx = v2nAfter * nx + v2t * tx;
        b2.vy = v2nAfter * ny + v2t * ty;
        // 立即分开，防止穿透
        const overlap = 2 * BALL_RADIUS - dist;
        b1.x -= nx * overlap / 2;
        b1.y -= ny * overlap / 2;
        b2.x += nx * overlap / 2;
        b2.y += ny * overlap / 2;
    }
}

function update() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    // 先处理所有小球之间的碰撞
    for (let i = 0; i < balls.length; i++) {
        for (let j = i + 1; j < balls.length; j++) {
            ballCollision(balls[i], balls[j]);
        }
    }
    // 再移动和边界检测
    for (const ball of balls) {
        ball.x += ball.vx;
        ball.y += ball.vy;
        // 边界反弹
        if (ball.x - BALL_RADIUS < 0) {
            ball.x = BALL_RADIUS;
            ball.vx *= -1;
        } else if (ball.x + BALL_RADIUS > canvas.width) {
            ball.x = canvas.width - BALL_RADIUS;
            ball.vx *= -1;
        }
        if (ball.y - BALL_RADIUS < 0) {
            ball.y = BALL_RADIUS;
            ball.vy *= -1;
        } else if (ball.y + BALL_RADIUS > canvas.height) {
            ball.y = canvas.height - BALL_RADIUS;
            ball.vy *= -1;
        }
        drawBall(ball);
    }
    requestAnimationFrame(update);
}

update(); 

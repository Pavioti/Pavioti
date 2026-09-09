<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>MOBA 2 Jogadores</title>

<style>
    * {
        box-sizing: border-box;
        margin: 0;
        padding: 0;
    }

    body {
        background: #07111f;
        color: white;
        font-family: Arial, sans-serif;
        overflow: hidden;
    }

    #game {
        position: relative;
        width: 100vw;
        height: 100vh;
        background:
            linear-gradient(45deg, #163c28 25%, transparent 25%),
            linear-gradient(-45deg, #163c28 25%, transparent 25%),
            linear-gradient(45deg, transparent 75%, #163c28 75%),
            linear-gradient(-45deg, transparent 75%, #163c28 75%);
        background-size: 80px 80px;
        background-position: 0 0, 0 40px, 40px -40px, -40px 0;
        background-color: #205334;
    }

    .lane {
        position: absolute;
        left: 0;
        top: 45%;
        width: 100%;
        height: 130px;
        background: #a17b52;
        border-top: 8px solid #725333;
        border-bottom: 8px solid #725333;
    }

    .base {
        position: absolute;
        width: 150px;
        height: 150px;
        border-radius: 50%;
        top: calc(50% - 75px);
        border: 8px solid;
    }

    #base1 {
        left: 20px;
        background: #123b87;
        border-color: #4287ff;
    }

    #base2 {
        right: 20px;
        background: #7d1717;
        border-color: #ff4545;
    }

    .player {
        position: absolute;
        width: 55px;
        height: 55px;
        border-radius: 50%;
        z-index: 10;
        display: flex;
        justify-content: center;
        align-items: center;
        font-weight: bold;
        border: 4px solid white;
        user-select: none;
    }

    #p1 {
        background: #176cff;
        left: 180px;
        top: calc(50% - 27px);
    }

    #p2 {
        background: #e32929;
        right: 180px;
        top: calc(50% - 27px);
    }

    .health {
        position: absolute;
        width: 65px;
        height: 8px;
        background: #333;
        top: -16px;
        left: -9px;
        border-radius: 5px;
        overflow: hidden;
    }

    .healthBar {
        height: 100%;
        background: #26e63b;
        width: 100%;
    }

    .projectile {
        position: absolute;
        width: 15px;
        height: 15px;
        border-radius: 50%;
        z-index: 20;
    }

    #hud {
        position: fixed;
        top: 15px;
        left: 50%;
        transform: translateX(-50%);
        background: rgba(0,0,0,.75);
        padding: 12px 25px;
        border-radius: 12px;
        text-align: center;
        z-index: 100;
    }

    #score {
        font-size: 20px;
        color: #ffd700;
    }

    #help {
        position: fixed;
        bottom: 15px;
        left: 50%;
        transform: translateX(-50%);
        background: rgba(0,0,0,.75);
        padding: 10px 20px;
        border-radius: 10px;
        z-index: 100;
        text-align: center;
    }

    #message {
        position: fixed;
        display: none;
        inset: 0;
        background: rgba(0,0,0,.8);
        z-index: 200;
        align-items: center;
        justify-content: center;
        flex-direction: column;
        gap: 20px;
    }

    #message h1 {
        font-size: 55px;
        color: #ffd700;
    }

    button {
        padding: 12px 25px;
        font-size: 18px;
        border: none;
        border-radius: 8px;
        cursor: pointer;
        background: #2878ff;
        color: white;
    }

    .tower {
        position: absolute;
        width: 45px;
        height: 80px;
        background: #777;
        border: 5px solid #ddd;
        border-radius: 8px;
        top: calc(50% - 40px);
        z-index: 5;
    }

    #tower1 {
        left: 35%;
    }

    #tower2 {
        right: 35%;
    }

    @media(max-width:700px) {
        #help {
            font-size: 12px;
        }

        #hud {
            font-size: 13px;
        }
    }
</style>
</head>

<body>

<div id="game">

    <div class="lane"></div>

    <div id="base1" class="base"></div>
    <div id="base2" class="base"></div>

    <div id="tower1" class="tower"></div>
    <div id="tower2" class="tower"></div>

    <div id="p1" class="player">
        P1
        <div class="health">
            <div id="hp1" class="healthBar"></div>
        </div>
    </div>

    <div id="p2" class="player">
        P2
        <div class="health">
            <div id="hp2" class="healthBar"></div>
        </div>
    </div>

</div>

<div id="hud">
    <div id="score">P1: 0 ❤️ | P2: 0 ❤️</div>
</div>

<div id="help">
    🔵 P1: WASD para mover | ESPAÇO para atacar
    &nbsp;&nbsp; 🔴 P2: SETAS para mover | ENTER para atacar
</div>

<div id="message">
    <h1 id="winner"></h1>
    <button onclick="restart()">Jogar novamente</button>
</div>

<script>
const game = document.getElementById("game");

const p1 = {
    el: document.getElementById("p1"),
    hp: 100,
    x: 180,
    y: window.innerHeight / 2 - 27,
    color: "#45a0ff",
    keys: {}
};

const p2 = {
    el: document.getElementById("p2"),
    hp: 100,
    x: window.innerWidth - 235,
    y: window.innerHeight / 2 - 27,
    color: "#ff4444",
    keys: {}
};

let gameOver = false;
let cooldown1 = false;
let cooldown2 = false;

function updatePlayer(player) {
    const speed = 5;

    if (player === p1) {
        if (player.keys["w"]) player.y -= speed;
        if (player.keys["s"]) player.y += speed;
        if (player.keys["a"]) player.x -= speed;
        if (player.keys["d"]) player.x += speed;
    }

    if (player === p2) {
        if (player.keys["ArrowUp"]) player.y -= speed;
        if (player.keys["ArrowDown"]) player.y += speed;
        if (player.keys["ArrowLeft"]) player.x -= speed;
        if (player.keys["ArrowRight"]) player.x += speed;
    }

    player.x = Math.max(0, Math.min(window.innerWidth - 55, player.x));
    player.y = Math.max(0, Math.min(window.innerHeight - 55, player.y));

    player.el.style.left = player.x + "px";
    player.el.style.top = player.y + "px";
}

document.addEventListener("keydown", e => {

    p1.keys[e.key.toLowerCase()] = true;
    p2.keys[e.key] = true;

    if (e.code === "Space") {
        e.preventDefault();

        if (!cooldown1 && !gameOver) {
            attack(p1, p2);
            cooldown1 = true;

            setTimeout(() => {
                cooldown1 = false;
            }, 500);
        }
    }

    if (e.key === "Enter") {

        if (!cooldown2 && !gameOver) {
            attack(p2, p1);
            cooldown2 = true;

            setTimeout(() => {
                cooldown2 = false;
            }, 500);
        }
    }
});

document.addEventListener("keyup", e => {
    p1.keys[e.key.toLowerCase()] = false;
    p2.keys[e.key] = false;
});

function attack(attacker, target) {

    const distance = Math.hypot(
        attacker.x - target.x,
        attacker.y - target.y
    );

    if (distance > 300) return;

    createProjectile(attacker, target);

    target.hp -= 10;

    if (target.hp < 0)
        target.hp = 0;

    updateHealth(target);

    if (target.hp <= 0) {
        endGame(attacker === p1 ? "🔵 P1 venceu!" : "🔴 P2 venceu!");
    }
}

function createProjectile(attacker, target) {

    const projectile = document.createElement("div");

    projectile.className = "projectile";
    projectile.style.background = attacker.color;

    game.appendChild(projectile);

    let x = attacker.x + 20;
    let y = attacker.y + 20;

    const dx = target.x - attacker.x;
    const dy = target.y - attacker.y;

    const distance = Math.hypot(dx, dy);

    const vx = dx / distance * 10;
    const vy = dy / distance * 10;

    projectile.style.left = x + "px";
    projectile.style.top = y + "px";

    const timer = setInterval(() => {

        x += vx;
        y += vy;

        projectile.style.left = x + "px";
        projectile.style.top = y + "px";

        if (
            x < 0 ||
            x > window.innerWidth ||
            y < 0 ||
            y > window.innerHeight
        ) {
            clearInterval(timer);
            projectile.remove();
        }

    }, 20);

    setTimeout(() => {
        clearInterval(timer);
        projectile.remove();
    }, 1000);
}

function updateHealth(player) {

    const hp = player === p1
        ? document.getElementById("hp1")
        : document.getElementById("hp2");

    hp.style.width = player.hp + "%";

    if (player.hp <= 30) {
        hp.style.background = "red";
    }
}

function endGame(text) {

    gameOver = true;

    document.getElementById("winner").textContent = text;
    document.getElementById("message").style.display = "flex";
}

function restart() {

    gameOver = false;

    p1.hp = 100;
    p2.hp = 100;

    p1.x = 180;
    p1.y = window.innerHeight / 2 - 27;

    p2.x = window.innerWidth - 235;
    p2.y = window.innerHeight / 2 - 27;

    document.getElementById("hp1").style.width = "100%";
    document.getElementById("hp2").style.width = "100%";

    document.getElementById("hp1").style.background = "#26e63b";
    document.getElementById("hp2").style.background = "#26e63b";

    document.getElementById("message").style.display = "none";
}

function gameLoop() {

    if (!gameOver) {
        updatePlayer(p1);
        updatePlayer(p2);
    }

    requestAnimationFrame(gameLoop);
}

gameLoop();
</script>

</body>
</html>

function setup() {
  createCanvas(400, 400);
}

function draw() {
  background(220);
}
// Variáveis globais do cenário e do jogo
let cityBuildings = []; // Prédios da cidade
let isNight = false;      // Indicador de modo: diurno ou noturno

let player;             // Objeto do carro do jogador
let items = [];         // Colecionáveis (itens de campo e de cidade)
let obstacles = [];     // Obstáculos que devem ser evitados
let score = 0;          // Pontuação do jogador
let lives = 3;          // Vidas do jogador

let cloudPositions = []; // Nuvens (apenas em modo diurno)

function setup() {
  createCanvas(800, 600);
  
  // Inicializa alguns prédios para a cidade (lado direito)
  for (let i = 0; i < 8; i++) {
    let bWidth = random(50, 100);
    let bHeight = random(100, 300);
    let x = random(width / 2 + 20, width - bWidth - 20);
    // Os prédios ficarão acima da estrada (que ocupa os 150 px inferiores)
    let y = height - bHeight - 150; 
    cityBuildings.push({ x: x, y: y, w: bWidth, h: bHeight });
  }
  
  // Inicializa o carro do jogador
  player = {
    x: width / 2 - 20,
    y: height - 75,   // Centralizado na estrada (de y = 450 a y = 600)
    w: 40,
    h: 20,
    speed: 5
  };
  
  // Criação de algumas nuvens para o cenário diurno
  for (let i = 0; i < 5; i++) {
    cloudPositions.push({
      x: random(0, width),
      y: random(20, 150),
      speed: random(0.3, 1)
    });
  }
  
  frameRate(30);
}

function draw() {
  // Desenha o fundo: céu com gradiente e sol/lua
  drawSky();
  
  // Se for dia, desenha as nuvens em movimento
  drawClouds();
  
  // Desenha o campo (lado esquerdo) com suas colinas ondulantes e árvores que balançam
  drawField();
  
  // Desenha a cidade (lado direito) com prédios e janelas dinâmicas
  drawCity();
  
  // Desenha a estrada que conecta os dois ambientes
  drawRoad();
  
  // Atualiza e desenha o carro do jogador
  updatePlayer();
  drawPlayer();
  
  // Atualiza e desenha os itens colecionáveis
  updateItems();
  
  // Atualiza e desenha os obstáculos
  updateObstacles();
  
  // Exibe a pontuação e as vidas na tela
  displayScoreLives();
  
  // A cada 300 frames (aproximadamente a cada 10 segundos), cria um novo obstáculo
  if (frameCount % 300 === 0) {
    createObstacle();
  }
  
  // Se as vidas chegarem a zero, encerra o jogo
  if (lives <= 0) {
    gameOver();
    noLoop();
  }
}

// Função para desenhar o céu com gradiente e o corpo celeste (sol ou lua)
function drawSky() {
  for (let y = 0; y < height; y++) {
    let inter = map(y, 0, height, 0, 1);
    let c;
    if (isNight) {
      // Céu noturno: azul escuro a preto
      c = lerpColor(color(25, 25, 112), color(0, 0, 0), inter);
    } else {
      // Céu diurno: azul claro a branco
      c = lerpColor(color(135, 206, 235), color(255, 255, 255), inter);
    }
    stroke(c);
    line(0, y, width, y);
  }
  noStroke();
  
  // Desenha o sol ou a lua no canto superior direito
  if (isNight) {
    fill(255, 255, 204);
    ellipse(width - 80, 80, 60, 60);
  } else {
    fill(255, 204, 0);
    ellipse(width - 80, 80, 80, 80);
  }
}

// Função para desenhar nuvens que se movem (apenas no modo diurno)
function drawClouds() {
  if (!isNight) {
    fill(255, 255, 255, 200);
    noStroke();
    for (let cloud of cloudPositions) {
      ellipse(cloud.x, cloud.y, 50, 30);
      ellipse(cloud.x + 20, cloud.y - 10, 40, 25);
      ellipse(cloud.x - 20, cloud.y - 10, 40, 25);
      cloud.x += cloud.speed;
      if (cloud.x > width + 50) {
        cloud.x = -50;
      }
    }
  }
}

// Desenha o campo com uma colina ondulante e árvores que balançam suavemente
function drawField() {
  fill(34, 139, 34);
  let baseHillY = height * 0.75;
  beginShape();
    vertex(0, height - 150);  // topo da estrada
    vertex(0, baseHillY);
    for (let x = 0; x <= width / 2; x += 10) {
      let y = baseHillY - 30 * sin(x * 0.02 + frameCount * 0.05);
      vertex(x, y);
    }
    vertex(width / 2, height - 150);
  endShape(CLOSE);
  
  // Desenha árvores de forma distribuída com uma leve oscilação
  for (let i = 0; i < 5; i++) {
    let xPos = 30 + i * (width / 2 - 60) / 5 + noise(frameCount * 0.01 + i) * 10;
    drawTree(xPos, baseHillY);
  }
}

// Função para desenhar uma árvore com leve balanço
function drawTree(x, groundY) {
  let sway = sin(frameCount * 0.05 + x) * 2;
  // Tronco
  fill(101, 67, 33);
  rect(x + sway, groundY - 40, 6, 40);
  // Copa
  fill(34, 139, 34);
  ellipse(x + 3 + sway, groundY - 50, 30, 30);
}

// Desenha a cidade com prédios e janelas
function drawCity() {
  for (let building of cityBuildings) {
    fill(169, 169, 169);
    rect(building.x, building.y, building.w, building.h);
    
    // Cria janelas que acendem (no modo noturno) ou permanecem escuras (no dia)
    let cols = floor(building.w / 20);
    let rows = floor(building.h / 20);
    for (let i = 0; i < cols; i++) {
      for (let j = 0; j < rows; j++) {
        let windowX = building.x + i * 20 + 5;
        let windowY = building.y + j * 20 + 5;
        if (isNight) {
          fill(random() > 0.3 ? color(255, 255, 102) : color(0));
        } else {
          fill(30, 30, 30);
        }
        rect(windowX, windowY, 10, 10);
      }
    }
  }
}

// Desenha a estrada que conecta o campo e a cidade, com linha central animada
function drawRoad() {
  // A estrada ocupa a parte inferior com 150 pixels de altura
  fill(50);
  rect(0, height - 150, width, 150);
  
  // Linha tracejada com efeito de movimento (simulando deslocamento)
  stroke(255, 255, 0);
  strokeWeight(4);
  let dashWidth = 20;
  let gap = 20;
  let centerY = height - 75;
  for (let x = 0; x < width; x += dashWidth + gap) {
    let phase = frameCount % (dashWidth + gap);
    line(x - phase, centerY, x - phase + dashWidth, centerY);
  }
  noStroke();
}

// Atualiza a posição do carro com base nas teclas esquerda e direita
function updatePlayer() {
  if (keyIsDown(LEFT_ARROW)) {
    player.x -= player.speed;
  }
  if (keyIsDown(RIGHT_ARROW)) {
    player.x += player.speed;
  }
  player.x = constrain(player.x, 0, width - player.w);
}

// Desenha o carro do jogador com rodas que giram (simulando movimento)
function drawPlayer() {
  fill(255, 0, 0);
  rect(player.x, player.y, player.w, player.h, 5);
  
  fill(0);
  // Efeito de rotação nas rodas baseado no frameCount
  let wheelAngle = frameCount % 360;
  push();
  translate(player.x + 10, player.y + player.h);
  rotate(radians(wheelAngle));
  ellipse(0, 0, 15, 15);
  pop();
  
  push();
  translate(player.x + player.w - 10, player.y + player.h);
  rotate(radians(wheelAngle));
  ellipse(0, 0, 15, 15);
  pop();
}

// Atualiza a posição dos itens, verifica colisões e remove os coletados ou os que saíram da tela
function updateItems() {
  // Cria novos itens com certa probabilidade; itens do tipo "campo" e "cidade" são diferenciados
  if (random() < 0.03) {
    let itemType = random() < 0.5 ? "campo" : "cidade";
    let newItem = {
      x: random(20, width - 20),
      y: height - 150, // Os itens surgem no topo da estrada
      size: 12,
      type: itemType,
      speed: random(2, 4)
    };
    items.push(newItem);
  }
  
  // Atualiza cada item e verifica colisões com o jogador
  for (let i = items.length - 1; i >= 0; i--) {
    items[i].y += items[i].speed;
    
    if (items[i].type === "campo") {
      if (rectCircleColliding(player.x, player.y, player.w, player.h, items[i].x, items[i].y, items[i].size)) {
        score += 10;
        items.splice(i, 1);
        continue;
      }
    } else {
      if (rectRectColliding(
        player.x, player.y, player.w, player.h,
        items[i].x - items[i].size / 2, items[i].y - items[i].size / 2, items[i].size, items[i].size)
      ) {
        score += 15;
        items.splice(i, 1);
        continue;
      }
    }
    
    // Remove itens que saíram da tela
    if (items[i].y > height) {
      items.splice(i, 1);
    }
  }
  
  // Desenha os itens
  for (let itm of items) {
    if (itm.type === "campo") {
      fill(34, 139, 34);
      ellipse(itm.x, itm.y, itm.size);
    } else {
      fill(169, 169, 169);
      rectMode(CENTER);
      rect(itm.x, itm.y, itm.size, itm.size);
      rectMode(CORNER);
    }
  }
}

// Atualiza a posição dos obstáculos, verifica colisões (reduzindo vidas) e remove os que saem da tela
function updateObstacles() {
  for (let i = obstacles.length - 1; i >= 0; i--) {
    obstacles[i].y += obstacles[i].speed;
    
    if (rectRectColliding(
          player.x, player.y, player.w, player.h,
          obstacles[i].x, obstacles[i].y, obstacles[i].w, obstacles[i].h)
       ) {
      lives -= 1;
      obstacles.splice(i, 1);
      continue;
    }
    
    if (obstacles[i].y > height) {
      obstacles.splice(i, 1);
    }
  }
  
  // Desenha os obstáculos (representados como “buracos” ou barreiras na estrada)
  fill(100);
  for (let obs of obstacles) {
    rect(obs.x, obs.y, obs.w, obs.h, 3);
  }
}

// Cria um novo obstáculo na estrada
function createObstacle() {
  let obsWidth = random(30, 60);
  let obsHeight = 15;
  let obsX = random(0, width - obsWidth);
  let obsY = height - 150; // Aparecem no topo da estrada
  let speed = random(3, 5);
  obstacles.push({ x: obsX, y: obsY, w: obsWidth, h: obsHeight, speed: speed });
}

// Verifica colisão entre um retângulo e um círculo
function rectCircleColliding(rx, ry, rw, rh, cx, cy, diameter) {
  let r = diameter / 2;
  let closestX = constrain(cx, rx, rx + rw);
  let closestY = constrain(cy, ry, ry + rh);
  let distX = cx - closestX;
  let distY = cy - closestY;
  return (distX * distX + distY * distY) <= (r * r);
}

// Verifica colisão entre dois retângulos
function rectRectColliding(x1, y1, w1, h1, x2, y2, w2, h2) {
  return (x1 < x2 + w2 &&
          x1 + w1 > x2 &&
          y1 < y2 + h2 &&
          y1 + h1 > y2);
}

// Exibe a pontuação e as vidas na tela
function displayScoreLives() {
  fill(0);
  textSize(20);
  text("Score: " + score, 10, 30);
  text("Lives: " + lives, 10, 60);
}

// Exibe a tela de game over com pontuação final
function gameOver() {
  background(0, 0, 0, 150);
  fill(255, 0, 0);
  textSize(40);
  textAlign(CENTER, CENTER);
  text("Game Over", width / 2, height / 2 - 20);
  textSize(25);
  text("Pontuação Final: " + score, width / 2, height / 2 + 20);
}

// Permite alternar entre modo diurno e noturno ao pressionar a tecla "N"
function keyPressed() {
  if (key === 'n' || key === 'N') {
    isNight = !isNight;
  }
}

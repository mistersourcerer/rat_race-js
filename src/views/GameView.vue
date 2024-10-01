<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { Application, Sprite, Texture } from 'pixi.js'

// Variáveis globais
let player,
  ground,
  groundTiles = [],
  speed = 5,
  gravity = 0.5,
  isJumping = false,
  jumpVelocity = 0,
  maxJump = -12,
  lanesCount = 3,
  laneWidth = 0,
  currentLane = 1,
  isMoving = false,
  pos = null,
  moveVelocity = 30,
  currentDir = null

const app = new Application()

// Função para inicializar o jogo
function setup() {
  // Cria o chão (movimento de baixo para cima)
  for (let i = 0; i < 7; i++) {
    const groundTile = new Sprite(Texture.WHITE)
    groundTile.width = app.screen.width
    groundTile.height = 100
    groundTile.tint = 0xbfa4a7 // Cor verde para o chão
    groundTile.x = 0
    groundTile.y = app.screen.height - i * (groundTile.height + 10)
    groundTiles.push(groundTile)
    app.stage.addChild(groundTile)
  }

  laneWidth = app.screen.width / lanesCount

  // Cria o jogador (um quadrado simples)
  player = new Sprite(Texture.WHITE)
  player.width = 50
  player.height = 50
  player.tint = 0xff0000 // Cor vermelha para o jogador

  player.x = playerPosX(currentLane, player)
  player.y = app.screen.height - player.height * 2

  app.stage.addChild(player)

  // Eventos de tecla
  window.addEventListener('keydown', onKeyDown)

  // Inicia o loop do jogo
  app.ticker.add(gameLoop)
}

function playerPosX(lane, player) {
  return laneWidth * (lane + 1) - laneWidth / 2 - player.width / 2
}

// Função para manipular eventos de tecla
function onKeyDown(e) {
  if (e.code === 'KeyW' && !isJumping) {
    isJumping = true
    jumpVelocity = maxJump
  }

  if (e.code === 'KeyA') {
    currentDir = keyMapping.KeyA
  }

  if (e.code === 'KeyD') {
    currentDir = keyMapping.KeyD
  }
}

const directions = {
  UP: 'up',
  DOWN: 'down',
  RIGHT: 'right',
  LEFT: 'left'
}

const keyMapping = {
  KeyA: directions.LEFT,
  KeyD: directions.RIGHT
}

// Função principal do jogo
function gameLoop(delta) {
  moveGround()
  handleJump()
  move()
}

// Função para mover o chão de baixo para cima
function moveGround() {
  for (let i = 0; i < groundTiles.length; i++) {
    groundTiles[i].y += speed

    // Reposiciona o tile do chão para a parte de baixo se sair da tela
    if (groundTiles[i].y >= app.screen.height + groundTiles[i].height) {
      groundTiles[i].y = -groundTiles[i].height
    }
  }
}

function move() {
  if (currentDir === directions.LEFT && currentLane >= 1) {
    if (!isMoving) {
      pos = playerPosX(currentLane - 1, player)
      isMoving = true
    }
    if (player.x != pos) {
      if (player.x > pos) {
        player.x -= moveVelocity
      } else if (player.x < pos) {
        player.x = pos
      }
    } else {
      currentLane -= 1
      pos = null
      isMoving = false
      currentDir = null
    }
  }

  if (currentDir == directions.RIGHT && currentLane < lanesCount - 1) {
    if (!isMoving) {
      pos = playerPosX(currentLane + 1, player)
      isMoving = true
    }
    if (player.x != pos) {
      if (player.x < pos) {
        player.x += 30
      } else if (player.x > pos) {
        player.x = pos
      }
    } else {
      currentLane += 1
      pos = null
      isMoving = false
      currentDir = null
    }
  }
}

// Função para controlar o pulo do jogador
function handleJump() {
  if (isJumping) {
    player.y += jumpVelocity
    jumpVelocity += gravity

    // Verifica se o jogador atingiu o chão
   
    if (player.y >= (app.screen.height - player.height * 2)) {
      player.y = app.screen.height - player.height * 2
      isJumping = false
    }
  }
}

onMounted(async () => {
  // debugging
  globalThis.__PIXI_APP__ = app

  await app.init({ background: '#ffffff', width: 800, height: 600 })
  document.getElementById('game').appendChild(app.canvas)
  document.body.appendChild(app.canvas)
  // Inicia o jogo
  setup()
})
</script>

<template>
  <main id="game" />
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { Application, Sprite, Texture, Ticker } from 'pixi.js'

abstract class InGame {
  sprite: Sprite

  constructor(sprite: Sprite) {
    this.sprite = sprite
  }

  set x(value: number) {
    this.sprite.x = value
  }

  get x() {
    return this.sprite.x
  }

  set y(value: number) {
    this.sprite.y = value
  }

  get y() {
    return this.sprite.y
  }

  set width(value: number) {
    this.sprite.width = value
  }

  get width() {
    return this.sprite.width
  }

  set height(value: number) {
    this.sprite.height = value
  }

  get height() {
    return this.sprite.height
  }
}

enum Direction {
  UP = 'up',
  DOWN = 'down',
  RIGHT = 'right',
  LEFT = 'left'
}

type KeyMapping = {
  [key: string]: Direction
}

const keyMapping: KeyMapping = {
  KeyA: Direction.LEFT,
  KeyD: Direction.RIGHT
}

class Character extends InGame {}

class Player extends Character {}

class Tile extends InGame {}

let player: Player,
  ground,
  pos: number = Number.NEGATIVE_INFINITY,
  currentDir: Direction | null = null,
  groundTiles: Tile[] = [],
  speed = 5,
  gravity = 0.5,
  isJumping = false,
  jumpVelocity = 0,
  maxJump = -12,
  lanesCount = 3,
  laneWidth = 0,
  currentLane = 1,
  isMoving = false,
  moveVelocity = 30,
  playerGroundLevel = Number.NEGATIVE_INFINITY

const app = new Application()

let groundPos = 0

function setup() {
  // tiles for moving ground
  for (let i = 0; i < 7; i++) {
    const sprite = new Sprite(Texture.WHITE)
    sprite.tint = 0xbfa4a7

    const groundTile = new Tile(sprite)
    groundTile.width = app.screen.width
    groundTile.height = 100
    groundTile.x = 0
    groundTile.y = app.screen.height - i * (groundTile.height + 10)
    groundTiles.push(groundTile)

    app.stage.addChild(sprite)
  }

  laneWidth = app.screen.width / lanesCount

  // squared player
  const sprite = new Sprite(Texture.WHITE)
  sprite.tint = 0xff0000
  player = new Player(sprite)

  player.width = 50
  player.height = 50
  player.x = playerPosX(currentLane, player)
  player.y = app.screen.height - player.height * 2
  playerGroundLevel = player.y

  app.stage.addChild(player.sprite)
  window.addEventListener('keydown', onKeyDown)

  app.ticker.add(gameLoop)
}

function playerPosX(lane: number, player: Player) {
  return laneWidth * (lane + 1) - laneWidth / 2 - player.width / 2
}

function onKeyDown(e: KeyboardEvent) {
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

function moveGround() {
  for (let i = 0; i < groundTiles.length; i++) {
    groundTiles[i].y += speed

    // tile exiting the screen
    if (groundTiles[i].y >= app.screen.height + groundTiles[i].height) {
      groundTiles[i].y = -groundTiles[i].height
    }
  }
}

let isAlsoJumping = false

function repositioningAt(options: {
  newPos: () => number
  onlyIf: () => boolean
  using: () => void
  stopWhen: () => boolean
  newLane: () => number
}): void {
  const { newPos, onlyIf, using, stopWhen, newLane } = options

  if (!isMoving) {
    pos = newPos()
    isMoving = true
    isAlsoJumping = true
  }

  if (player.x != pos) {
    if (onlyIf()) {
      using()
    } else if (stopWhen()) {
      player.x = pos
    }
  } else {
    currentLane = newLane()
    pos = Number.NEGATIVE_INFINITY
    isMoving = false
    isAlsoJumping = false
    currentDir = null
    moveToSideVelocity = 5
  }
}

function handleJump() {
  if (isJumping) {
    player.y += jumpVelocity
    jumpVelocity += gravity

    // checking if player has landed
    if (player.y >= app.screen.height - player.height * 2) {
      player.y = app.screen.height - player.height * 2
      isJumping = false
    }
  }
}

let moveUpVelocity = -7
let moveToSideVelocity = 5
let sideGravity = 0.9

function jumpToSide() {
    moveToSideVelocity += sideGravity

    // checking if player has landed
    if (player.y > playerGroundLevel) {
      player.y = playerGroundLevel
      isAlsoJumping = false
      moveUpVelocity = -7
    } else if(isAlsoJumping) {
      player.y += moveUpVelocity
      moveUpVelocity += sideGravity
    }
}

function moveLeft() {
  if(isMoving) {
    player.x -= moveToSideVelocity
    jumpToSide()
  }
}

function moveRight() {
  if(isMoving) {
    player.x += moveToSideVelocity
    jumpToSide()
  }
}

function handleMove() {
  let options = null

  if (currentDir === Direction.LEFT && currentLane >= 1) {
    options = {
      newPos: () => playerPosX(currentLane - 1, player),
      onlyIf: () => player.x > pos,
      using: moveLeft,
      stopWhen: () => player.x < pos,
      newLane: () => currentLane - 1
    }
  } else if (currentDir === Direction.RIGHT && currentLane < lanesCount - 1) {
    options = {
      newPos: () => playerPosX(currentLane + 1, player),
      onlyIf: () => player.x < pos,
      using: moveRight,
      stopWhen: () => player.x > pos,
      newLane: () => currentLane + 1
    }
  }

  if (options !== null) {
    repositioningAt(options)
  }
}

function gameLoop(delta: Ticker) {
  moveGround()
  handleJump()
  handleMove()
}

onMounted(async () => {
  // debugging
  globalThis.__PIXI_APP__ = app

  await app.init({ background: '#ffffff', width: 800, height: 600 })
  document!.getElementById('game')!.appendChild(app.canvas)
  document!.body!.appendChild(app.canvas)

  setup()
})

declare global {
  var __PIXI_APP__: Application
}
</script>

<template>
  <main id="game" />
</template>

<template>
  <div class="header">
    <div class="merry">
      Merry Christmas
      <img src="./assets/hat.png" alt="" class="santa-hat" />
    </div>
    <img ref="animatedImg" src="./assets/cat_santa_new.png" alt="" class="santa" />
    <div v-if="!gameStarted && !gameOver" class="startBTN" @click="startGame">START</div>
  </div>
  <div v-if="gameStarted">
    <div class="timer">TIMER : {{ formattedTime }}</div>
    <div class="container">
      <div class="card" v-for="card in cards" :key="card.id" @click="flipCard(card)">
        <img v-if="card.flipped || matchedCards.includes(card)" :src="card.image" alt="" />
        <div v-else class="card-back"></div>
      </div>
    </div>
  </div>
  <div v-if="gameOver" class="game-over">
    <div class="retryBTN" @click="startGame">RETRY</div>
    <div class="result">{{ gameOverMessage }}</div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      cards: [],
      flippedCards: [],
      matchedCards: [],
      gameStarted: false,
      gameOver: false,
      lockBoard: false,
      timer: null,
      gameOverMessage: '',
      timeLeft: 3 * 60 // 3 分鐘
    }
  },
  created() {
    this.initializeCards()
  },
  computed: {
    formattedTime() {
      const minutes = Math.floor(this.timeLeft / 60)
      const seconds = this.timeLeft % 60
      return `${minutes}:${seconds.toString().padStart(2, '0')}`
    }
  },
  methods: {
    startGame() {
      this.startAnimation()
      this.initializeCards()
      this.gameStarted = true
      this.gameOver = false
      this.startTimer()
    },
    startAnimation() {
      const img = this.$refs.animatedImg
      img.classList.add('animate')
    },
    // initializeCards() {
    //   const images = [
    //     'image1.jpg',
    //     'image2.jpg',
    //     'image3.jpg',
    //     'image4.jpg',
    //     'image5.jpg',
    //     'image6.jpg',
    //     'image7.jpg',
    //     'image8.jpg',
    //     'image9.jpg',
    //     'image10.jpg'
    //   ]
    //   this.cards = [...images, ...images]
    //     .sort(() => Math.random() - 0.5)
    //     .map((image, index) => ({
    //       id: index,
    //       image: new URL(`@/assets/cards/${image}`, import.meta.url).href,
    //       flipped: false
    //     }))
    // },
    initializeCards() {
      const images = import.meta.glob('@/assets/cards/*.jpg', { eager: true })
      const imagePaths = Object.values(images).map((image) => image.default)
      this.cards = [...imagePaths, ...imagePaths]
        .sort(() => Math.random() - 0.5)
        .map((image, index) => ({
          id: index,
          image,
          flipped: false
        }))
    },
    flipCard(card) {
      if (this.lockBoard || card.flipped || this.matchedCards.includes(card)) return

      card.flipped = true
      this.flippedCards.push(card)

      if (this.flippedCards.length === 2) {
        this.checkForMatch()
      }
    },
    checkForMatch() {
      this.lockBoard = true
      const [card1, card2] = this.flippedCards

      if (card1.image === card2.image) {
        this.matchedCards.push(card1, card2)
        this.resetBoard()
        if (this.matchedCards.length === this.cards.length) {
          this.winGame()
        }
      } else {
        setTimeout(() => {
          card1.flipped = false
          card2.flipped = false
          this.resetBoard()
        }, 1000)
      }
    },
    resetBoard() {
      this.flippedCards = []
      this.lockBoard = false
    },
    startTimer() {
      this.timeLeft = 3 * 60 // 重置時間
      this.timer = setInterval(() => {
        if (this.timeLeft > 0) {
          this.timeLeft--
        } else {
          this.endGame(false)
        }
      }, 1000)
    },
    endGame(win) {
      clearInterval(this.timer)
      this.gameOver = true
      this.gameStarted = false
      this.gameOverMessage = win ? 'You Win!' : 'Game Over!'
    },
    winGame() {
      this.endGame(true)
    }
  }
}
</script>

<style lang="scss">
@import './assets/all.scss';
@import url('https://fonts.googleapis.com/css2?family=Open+Sans:ital,wght@0,300..800;1,300..800&family=Ubuntu:ital,wght@0,300;0,400;0,500;0,700;1,300;1,400;1,500;1,700&display=swap');

body {
  background-color: #efe9e7;
}

.merry {
  font-family: 'Ubuntu', serif;
  font-weight: 700;
  font-size: 60px;
  font-style: normal;
  position: absolute;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  position: absolute;
}
.santa-hat {
  position: absolute;
  top: 10px;
  right: -20px;
  width: 35px;
  height: 35px;
  animation: bell-shake 1s infinite;
}

@keyframes bell-shake {
  0% {
    transform: rotate(0deg);
  }
  25% {
    transform: rotate(15deg);
  }
  50% {
    transform: rotate(0deg);
  }
  75% {
    transform: rotate(-15deg);
  }
  100% {
    transform: rotate(0deg);
  }
}
.santa {
  position: relative;
  left: 70%; /* 確保貓咪一開始在右邊 */
}

.startBTN {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  padding: 15px 30px;
  background-color: #e4b549;
  color: #fff;
  border-radius: 20px;
  font-family: 'Ubuntu', serif;
  font-weight: 700;
  font-style: normal;
  font-size: 24px;
  font-weight: bold;
  cursor: pointer;
}
.retryBTN {
  position: absolute;
  top: 37%;
  left: 50%;
  transform: translate(-50%, -50%);
  padding: 15px 30px;
  background-color: #e4b549;
  color: #fff;
  border-radius: 20px;
  font-family: 'Ubuntu', serif;
  font-weight: 700;
  font-style: normal;
  font-size: 24px;
  font-weight: bold;
  cursor: pointer;
}

@keyframes moveArc {
  0% {
    left: 70%;
    top: 0;
  }
  50% {
    left: 50%;
    top: -30px; /* 調整頂點位置以形成半圓弧 */
  }
  100% {
    left: 30%;
    top: 0;
  }
}

.animate {
  animation: moveArc 2s cubic-bezier(0.42, 0, 0.58, 1) forwards; /* 使用 cubic-bezier 使動畫更順暢 */
}
.container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

.card {
  width: 110px;
  height: 150px;
  margin: 10px;
  perspective: 1000px;
  cursor: pointer;
  border-radius: 10px; /* 添加邊框半徑 */
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1); /* 添加陰影 */
}

.card img,
.card-back {
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  position: absolute;
  object-fit: cover;
  top: 0;
  left: 0;
  border: 2px solid #000; /* 添加邊框 */
  border-radius: 10px; /* 添加邊框半徑 */
}

.card-back {
  background-image: url('@/assets/snowman.jpg');
  background-size: 80%; /* 縮小背景圖片 */
  background-position: center;
  background-repeat: no-repeat;
}
.timer {
  text-align: center; /* 水平置中 */
  font-family: 'Ubuntu', serif;
  font-weight: 700;
  font-style: normal;
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 20px;
}
.result {
  text-align: center; /* 水平置中 */
  font-family: 'Ubuntu', serif;
  font-weight: 700;
  font-style: normal;
  font-size: 48px;
  font-weight: bold;
  margin-bottom: 20px;
}
@media (max-width: 768px) {
  .santa-hat {
    position: absolute;
    top: 10px;
    right: 110px;
  }
  .santa {
    position: relative;
    left: 40%; /* 確保貓咪一開始在右邊 */
  }
  @keyframes moveArc {
    0% {
      left: 40%;
      top: 0;
    }
    100% {
      left: 10%;
      top: 80px;
    }
  }
}
</style>

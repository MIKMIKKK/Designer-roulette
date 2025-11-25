<template>
  <div id="home-root">
    <header>
      <h1>PIRATE'S DESTINY</h1>
      <button id="rules-btn" class="btn-secondary" @click="showRules = true">Captain's Rules</button>
    </header>

    <main>
      <section class="game-area">
        <div class="dice-container">
          <div class="scene" :class="{ 'stop-anim': isRolling }">
            <div class="cube" id="dice" ref="diceRef">
              <div class="face face-1"></div>
              <div class="face face-2"></div>
              <div class="face face-3"></div>
              <div class="face face-4"></div>
              <div class="face face-5"></div>
              <div class="face face-6"></div>
            </div>
          </div>
          <button id="roll-btn" class="btn-primary" @click="rollDice" :disabled="isRolling">
            {{ isRolling ? 'ROLLING...' : 'Roll the Dice' }}
          </button>
          <div id="dice-result" class="dice-result" :class="{ 'show-result': showResult }">{{ diceResult }}</div>
          
          <!-- Added Start Game button to navigate to the board -->
          <button class="btn-primary" style="margin-top: 20px;" @click="startGame">Start Game</button>
        </div>

        <div class="timer-container">
          <div class="timer-display" id="timer" :style="{ color: isTimerRunning ? 'var(--primary-color)' : 'white' }">{{ formattedTime }}</div>
          <div class="timer-controls">
            <button id="start-timer" class="btn-icon" @click="startTimer">▶</button>
            <button id="stop-timer" class="btn-icon" @click="stopTimer">⏸</button>
            <button id="reset-timer" class="btn-icon" @click="resetTimer">↺</button>
          </div>
        </div>
      </section>

      <section class="cards-area">
        <div class="deck-container" @click="drawCard('question')">
          <h2>Questions</h2>
          <div class="card-slot" id="deck-question">
            <div class="card-back piraterie-back">
              <div class="logo-92i">92i</div>
            </div>
          </div>
        </div>

        <div class="active-card-display" id="active-card-area">
          <div v-if="!activeCard" class="card-placeholder">Draw a card</div>
          <div v-else class="active-card">
            <div class="card-face-back" :class="activeCard.backClass" v-html="activeCard.backHtml"></div>
            <div class="card-face-front" :style="{ backgroundImage: `url('${activeCard.frontUrl}')`, border: '4px solid var(--primary-color)' }"></div>
          </div>
        </div>

        <div class="deck-container" @click="drawCard('beneficial')">
          <h2>Beneficial</h2>
          <div class="card-slot" id="deck-beneficial">
            <div class="card-back futur-back">
              <div class="logo-futur">FUTUR</div>
            </div>
          </div>
        </div>
      </section>
    </main>

    <!-- Rules Modal -->
    <div id="rules-modal" class="modal" :class="{ hidden: !showRules }" @click.self="showRules = false">
      <div class="modal-content">
        <span class="close-modal" @click="showRules = false">&times;</span>
        <h2>Captain's Rules</h2>
        <div class="rules-container">
          <h3>OBJECTIVE</h3>
          <p>Be the first to reach the <strong>Final Square</strong>.</p>

          <h3>TURN SEQUENCE</h3>
          <ol>
            <li>Roll the <strong>D6</strong>.</li>
            <li>Move forward.</li>
            <li>Apply the square's effect.</li>
          </ol>

          <h3>SQUARE TYPES</h3>
          <ul>
            <li><strong>EMPTY:</strong> No effect.</li>
            <li><strong>QUIZ:</strong> Answer a question (Marketing, Digital Creation, Web Dev).
              <ul>
                <li><strong>Correct:</strong> Draw 1 Bonus Card.</li>
                <li><strong>Wrong:</strong> Move back 1 square.</li>
              </ul>
            </li>
          </ul>

          <h3>BONUS CARDS</h3>
          <ul>
            <li><strong>BOOST:</strong> Move forward, roll again, etc.</li>
            <li><strong>PROTECTION:</strong> Cancel penalties.</li>
            <li><strong>ASSET:</strong> Strategic advantages.</li>
            <li><strong>PENALTY:</strong> Attack opponents.</li>
            <li><strong>DUEL:</strong> Defend against penalties.</li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onUnmounted } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();

// --- Game State ---
const showRules = ref(false);

// --- Dice Logic ---
const diceRef = ref(null);
const isRolling = ref(false);
const diceResult = ref(null);
const showResult = ref(false);
let currentRotationX = 0;
let currentRotationY = 0;

const faceRotations = {
  1: { x: 0, y: 0 },       // Front
  2: { x: 0, y: -90 },     // Right
  3: { x: 0, y: -180 },    // Back
  4: { x: 0, y: 90 },      // Left
  5: { x: -90, y: 0 },     // Top
  6: { x: 90, y: 0 },      // Bottom
  7: { x: 0, y: 0 },       // Fallback
  8: { x: 0, y: -90 }      // Fallback
};

function rollDice() {
  if (isRolling.value) return;
  isRolling.value = true;
  showResult.value = false;
  
  const result = Math.floor(Math.random() * 6) + 1; // D6 for board game, but code had D8 fallback? sticking to D6 as per rules
  const target = faceRotations[result];
  
  const minSpin = 720;
  
  let kX = Math.ceil((currentRotationX + minSpin - target.x) / 360);
  let nextX = target.x + (kX * 360);
  
  let kY = Math.ceil((currentRotationY + minSpin - target.y) / 360);
  let nextY = target.y + (kY * 360);
  
  if (diceRef.value) {
    diceRef.value.style.transition = 'transform 1.5s cubic-bezier(0.2, 0.8, 0.2, 1)';
    diceRef.value.style.transform = `rotateX(${nextX}deg) rotateY(${nextY}deg)`;
  }
  
  currentRotationX = nextX;
  currentRotationY = nextY;
  
  setTimeout(() => {
    isRolling.value = false;
    diceResult.value = result;
    showResult.value = true;
  }, 1500);
}

// --- Timer Logic ---
const startTime = ref(0);
const elapsedTime = ref(0);
const isTimerRunning = ref(false);
const timerInterval = ref(null);
const currentTime = ref(0); // For reactivity

const formattedTime = computed(() => {
  const time = currentTime.value;
  const seconds = Math.floor(time / 1000);
  const milliseconds = Math.floor((time % 1000) / 10);
  
  const sStr = seconds.toString().padStart(2, '0');
  const msStr = milliseconds.toString().padStart(2, '0');
  
  return `${sStr}:${msStr}`;
});

function updateTimerDisplay() {
  const now = Date.now();
  currentTime.value = now - startTime.value + elapsedTime.value;
}

function startTimer() {
  if (isTimerRunning.value) return;
  isTimerRunning.value = true;
  startTime.value = Date.now();
  
  timerInterval.value = setInterval(() => {
    updateTimerDisplay();
  }, 10);
}

function stopTimer() {
  if (!isTimerRunning.value) return;
  isTimerRunning.value = false;
  clearInterval(timerInterval.value);
  elapsedTime.value += Date.now() - startTime.value;
}

function resetTimer() {
  isTimerRunning.value = false;
  clearInterval(timerInterval.value);
  elapsedTime.value = 0;
  currentTime.value = 0;
}

onUnmounted(() => {
  clearInterval(timerInterval.value);
});

// --- Card Logic ---
const activeCard = ref(null);

const questionFaces = [
  '/assets/card_face_1.png',
  '/assets/card_face_2.png',
  '/assets/card_face_3.png'
];

const beneficialFaces = [
  '/assets/card_face_3.png',
  '/assets/card_face_4.png',
  '/assets/card_face_5.png'
];

function drawCard(type) {
  let backClass = '';
  let backHtml = '';
  let frontUrl = '';
  
  if (type === 'question') {
    backClass = 'piraterie-back';
    backHtml = '<div class="logo-92i">92i</div>';
    frontUrl = questionFaces[Math.floor(Math.random() * questionFaces.length)];
  } else {
    backClass = 'futur-back';
    backHtml = '<div class="logo-futur">FUTUR</div>';
    frontUrl = beneficialFaces[Math.floor(Math.random() * beneficialFaces.length)];
  }
  
  // Force re-render to trigger animation if needed, or just replace
  activeCard.value = null;
  setTimeout(() => {
    activeCard.value = {
      backClass,
      backHtml,
      frontUrl
    };
  }, 10);
}

function startGame() {
  router.push('/game');
}
</script>

<style scoped>
/* Scoped styles if any specific overrides needed, but mostly relying on global style.css */
</style>

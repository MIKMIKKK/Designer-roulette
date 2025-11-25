<template>
  <div id="octo-root">
    <!-- Setup Modal -->
    <div v-if="showSetupModal" id="setup-modal" class="modal">
      <div class="modal-content">
        <h2>Configuration de la Partie</h2>
        
        <div v-if="!showNamesStep" class="setup-step" id="step-count">
          <label>Nombre de Joueurs :</label>
          <div class="player-count-selector">
            <button v-for="count in [2, 3, 4]" :key="count" class="count-btn" :class="{ selected: playerCount === count }" @click="playerCount = count">
              {{ count }}
            </button>
          </div>
          <button id="next-step-btn" class="btn-primary" @click="goToNamesStep">Suivant</button>
        </div>

        <div v-else class="setup-step" id="step-names">
          <div id="name-inputs-container">
            <div v-for="(player, index) in tempPlayers" :key="index" class="input-group">
              <div class="color-indicator" :style="{ backgroundColor: playerColors[index] }"></div>
              <input type="text" class="name-input" v-model="player.name" :placeholder="`Nom du Joueur ${index + 1}`">
            </div>
          </div>
          <button id="start-game-btn" class="btn-primary" @click="startGame">Lancer la Partie</button>
        </div>
      </div>
    </div>

    <!-- Restart Confirmation Modal -->
    <div v-if="showRestartModal" id="restart-modal" class="modal">
      <div class="modal-content">
        <h2>Recommencer ?</h2>
        <p style="color: #fff; font-family: 'Cinzel', serif; margin-bottom: 30px; font-size: 1.2em;">Toute progression sera perdue.</p>
        <div style="display: flex; justify-content: center; gap: 20px;">
          <button id="confirm-restart-btn" class="btn-primary" @click="confirmRestart">Oui</button>
          <button id="cancel-restart-btn" class="btn-primary" style="background: #333; border-color: #555; color: #aaa;" @click="showRestartModal = false">Non</button>
        </div>
      </div>
    </div>

    <div id="main-layout">
      <!-- Left Panel -->
      <div id="left-panel">
        <div class="timer-container">
          <div class="timer-display" id="timer" :style="{ color: isTimerRunning ? 'var(--primary-color)' : 'red' }">{{ formattedTime }}</div>
          <div class="timer-controls">
            <button id="start-timer" class="btn-icon" @click="startTimer">▶</button>
            <button id="stop-timer" class="btn-icon" @click="stopTimer">⏸</button>
            <button id="reset-timer" class="btn-icon" @click="resetTimer">↺</button>
          </div>
        </div>
        
        <!-- 3D Dice Area Moved Here -->
        <div id="dice-area" style="position: relative; width: 100%; height: 150px; display: flex; justify-content: center; align-items: center; margin-top: 20px;">
          <div class="dice-container">
            <div class="scene" :class="{ 'stop-anim': isRolling }" @click="rollDice">
              <div class="cube" id="dice" ref="diceRef">
                <div class="face face-1"></div>
                <div class="face face-2"></div>
                <div class="face face-3"></div>
                <div class="face face-4"></div>
                <div class="face face-5"></div>
                <div class="face face-6"></div>
              </div>
            </div>
          </div>
          <div id="dice-result" class="dice-result" :class="{ 'show-result': showResult }" aria-hidden="true">{{ diceResult }}</div>
        </div>

        <div id="player-info" style="color: #d4af37; text-align: center; font-size: 1.2em; margin-top: 20px;">
          <span v-if="players.length > 0 && !winner">Tour de <span :style="{ color: currentPlayer.color, fontWeight: 'bold' }">{{ currentPlayer.name }}</span></span>
          <span v-else-if="winner" style="color: #D4AF37; font-size: 1.5em; text-shadow: 0 0 10px gold;">👑 VICTOIRE ! {{ winner.name }} a gagné ! 👑</span>
          <span v-else>En attente...</span>
        </div>
        
        <button id="restart-btn" class="btn-primary" style="margin-top: 20px; width: 100%; font-size: 0.8em; padding: 10px;" @click="showRestartModal = true">Recommencer</button>
      </div>

      <!-- Center Board -->
      <div id="game-container">
        <div id="board">
          <div id="center-piece">
            <img src="/assets/board/skull.png" alt="Skull Center">
            <div id="message-display" :style="winner ? { color: '#D4AF37', fontSize: '18px' } : {}">{{ winner ? `${winner.name.toUpperCase()} GAGNE !` : "LA PIRATERIE N'EST JAMAIS FINIE" }}</div>
          </div>
          
          <!-- Steps -->
          <div id="steps-container">
            <div v-for="(step, index) in steps" :key="index" class="step" :class="step.type" :style="{ left: `${step.x - 50}px`, top: `${step.y - 50}px`, transform: `rotate(${step.angle}deg)` }">
              <span class="step-number">{{ index + 1 }}</span>
            </div>
          </div>
          
          <!-- Decorative elements -->
          <div class="decoration compass"></div>
          <div class="decoration chain"></div>
          
          <!-- Player Tokens -->
          <div id="tokens-container">
            <div v-for="(player, index) in players" :key="player.id" class="token" :id="`p${player.id}`"
                 :style="{ 
                   left: `${getTokenPosition(player, index).x}px`, 
                   top: `${getTokenPosition(player, index).y}px`, 
                   border: `2px solid ${player.color}`, 
                   borderRadius: '50%' 
                 }">
            </div>
          </div>
        </div>
      </div>

      <!-- Right Panel -->
      <div id="right-panel">
        <div class="deck-container" @click="drawCard('question')">
          <h2 style="color: #d4af37; font-size: 1rem; margin-bottom: 10px;">Questions</h2>
          <div class="card-slot" id="deck-question">
            <div class="card-back piraterie-back">
              <div class="logo-92i">92i</div>
            </div>
          </div>
        </div>
        
        <div class="active-card-display" id="active-card-area" style="margin: 20px 0;">
          <div v-if="!activeCard" class="card-placeholder">Draw a card</div>
          <div v-else class="active-card">
            <div class="card-face-back" :class="activeCard.backClass" v-html="activeCard.backHtml"></div>
            <div class="card-face-front" :style="{ backgroundImage: `url('${activeCard.frontUrl}')` }"></div>
          </div>
        </div>

        <div class="deck-container" @click="drawCard('beneficial')">
          <h2 style="color: #d4af37; font-size: 1rem; margin-bottom: 10px;">Bonus</h2>
          <div class="card-slot" id="deck-beneficial">
            <div class="card-back futur-back">
              <div class="logo-futur">FUTUR</div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue';

// --- Assets Imports (if needed, but using public paths) ---

// --- Game State ---
const showSetupModal = ref(true);
const showNamesStep = ref(false);
const showRestartModal = ref(false);
const playerCount = ref(4);
const playerColors = ['#ff4444', '#4444ff', '#44ff44', '#ffff44'];
const tempPlayers = ref([]);
const players = ref([]);
const currentPlayerIndex = ref(0);
const winner = ref(null);

const currentPlayer = computed(() => players.value[currentPlayerIndex.value]);

// --- Timer Logic ---
const timeLeft = ref(15000);
const isTimerRunning = ref(false);
const timerInterval = ref(null);

const formattedTime = computed(() => {
  const seconds = Math.floor(timeLeft.value / 1000);
  const milliseconds = Math.floor((timeLeft.value % 1000) / 10);
  const sStr = seconds.toString().padStart(2, '0');
  const msStr = milliseconds.toString().padStart(2, '0');
  return `${sStr}:${msStr}`;
});

function updateTimer() {
  if (timeLeft.value > 0) {
    timeLeft.value -= 10;
  } else {
    timeLeft.value = 0;
    stopTimer();
  }
}

function startTimer() {
  stopTimer();
  timeLeft.value = 15000;
  isTimerRunning.value = true;
  timerInterval.value = setInterval(updateTimer, 10);
}

function stopTimer() {
  isTimerRunning.value = false;
  clearInterval(timerInterval.value);
}

function resetTimer() {
  stopTimer();
  timeLeft.value = 15000;
}

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
  6: { x: 90, y: 0 }       // Bottom
};

function rollDice() {
  if (isRolling.value || winner.value) return;
  isRolling.value = true;
  showResult.value = false;
  
  const result = Math.floor(Math.random() * 6) + 1;
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
    movePlayer(result);
  }, 1500);
}

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
  
  activeCard.value = null;
  setTimeout(() => {
    activeCard.value = {
      backClass,
      backHtml,
      frontUrl
    };
  }, 10);
}

// --- Board Logic ---
const steps = [
    { x: 120, y: 680, angle: 15, type: 'normal' },
    { x: 200, y: 720, angle: 5, type: 'question' }, // Blue
    { x: 300, y: 735, angle: -5, type: 'normal' },
    { x: 410, y: 720, angle: -15, type: 'bonus' }, // Green
    { x: 520, y: 680, angle: -30, type: 'normal' },
    
    { x: 610, y: 610, angle: -55, type: 'question' },
    { x: 680, y: 520, angle: -75, type: 'normal' },
    { x: 720, y: 410, angle: -95, type: 'bonus' },
    { x: 735, y: 290, angle: -110, type: 'normal' },
    { x: 720, y: 180, angle: -135, type: 'question' },
    
    { x: 670, y: 100, angle: -165, type: 'normal' },
    { x: 550, y: 60, angle: 175, type: 'bonus' },
    { x: 420, y: 55, angle: 165, type: 'normal' },
    { x: 290, y: 70, angle: 155, type: 'question' },
    { x: 180, y: 110, angle: 135, type: 'normal' },
    
    { x: 100, y: 200, angle: 105, type: 'bonus' },
    { x: 60, y: 310, angle: 85, type: 'normal' },
    { x: 55, y: 430, angle: 75, type: 'question' },
    { x: 75, y: 540, angle: 55, type: 'normal' },
    { x: 105, y: 620, angle: 35, type: 'normal' }
];

function getTokenPosition(player, index) {
  const targetStep = steps[player.position];
  const offset = (index - 1.5) * 10;
  return {
    x: targetStep.x + 30 + offset,
    y: targetStep.y + 30 + offset
  };
}

function movePlayer(roll) {
  const player = players.value[currentPlayerIndex.value];
  let newPos = player.position + roll;
  
  if (newPos >= steps.length) {
    newPos = steps.length - 1;
    player.position = newPos;
    winner.value = player;
    stopTimer();
    localStorage.removeItem('octoGameState');
    return;
  }
  
  player.position = newPos;
  
  const targetStep = steps[newPos];
  
  if (targetStep.type === 'question') {
    setTimeout(() => drawCard('question'), 500);
    startTimer();
  } else if (targetStep.type === 'bonus') {
    setTimeout(() => drawCard('beneficial'), 500);
    resetTimer();
  } else {
    resetTimer();
  }
  
  currentPlayerIndex.value = (currentPlayerIndex.value + 1) % players.value.length;
  saveGameState();
}

// --- Setup & Persistence ---
function goToNamesStep() {
  showNamesStep.value = true;
  tempPlayers.value = [];
  for (let i = 0; i < playerCount.value; i++) {
    tempPlayers.value.push({
      name: `Joueur ${i + 1}`
    });
  }
}

function startGame() {
  players.value = tempPlayers.value.map((p, i) => ({
    id: i + 1,
    name: p.name,
    color: playerColors[i],
    position: 0
  }));
  
  showSetupModal.value = false;
  saveGameState();
}

function saveGameState() {
  const gameState = {
    players: players.value,
    currentPlayerIndex: currentPlayerIndex.value,
    playerCount: playerCount.value
  };
  localStorage.setItem('octoGameState', JSON.stringify(gameState));
}

function loadGameState() {
  const savedState = localStorage.getItem('octoGameState');
  if (savedState) {
    const state = JSON.parse(savedState);
    players.value = state.players;
    currentPlayerIndex.value = state.currentPlayerIndex;
    playerCount.value = state.playerCount;
    showSetupModal.value = false;
    return true;
  }
  return false;
}

function confirmRestart() {
  localStorage.removeItem('octoGameState');
  stopTimer();
  timeLeft.value = 15000;
  players.value = [];
  currentPlayerIndex.value = 0;
  winner.value = null;
  showRestartModal.value = false;
  showSetupModal.value = true;
  showNamesStep.value = false;
  playerCount.value = 4;
}

onMounted(() => {
  if (!loadGameState()) {
    showSetupModal.value = true;
  }
});

onUnmounted(() => {
  stopTimer();
});
</script>

<style scoped>
@import '../assets/board.css';
</style>

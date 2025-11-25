<template>
  <div id="home-root">

    <header>
      <h1>PIRATE'S DESTINY</h1>
      <button class="btn-secondary" @click="openRules">Captain's Rules</button>
    </header>

    <main>
      <section class="game-area">
        <div class="dice-container">
          <div class="scene">
            <div class="cube">
              <div class="face face-1"></div>
              <div class="face face-2"></div>
              <div class="face face-3"></div>
              <div class="face face-4"></div>
              <div class="face face-5"></div>
              <div class="face face-6"></div>
            </div>
          </div>
          <button class="btn-primary">Roll the Dice</button>
          <div class="dice-result">-</div>
        </div>

        <div class="timer-container">
          <div class="timer-display">00:00</div>
          <div class="timer-controls">
            <button class="btn-icon">▶</button>
            <button class="btn-icon">⏸</button>
            <button class="btn-icon">↺</button>
          </div>
        </div>
      </section>

      <section class="cards-area">
        <div class="deck-container">
          <h2>Questions</h2>
          <div class="card-slot">
            <div class="card-back piraterie-back">
              <div class="logo-92i">92i</div>
            </div>
          </div>
        </div>

        <div class="active-card-display">
          <div class="card-placeholder">Draw a card</div>
        </div>

        <div class="deck-container">
          <h2>Beneficial</h2>
          <div class="card-slot">
            <div class="card-back futur-back">
              <div class="logo-futur">FUTUR</div>
            </div>
          </div>
        </div>
      </section>
    </main>

    <!-- Rules Modal -->
    <div v-if="showRules" class="modal">
      <div class="modal-content">
        <span class="close-modal" @click="closeRules">&times;</span>
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
import { ref } from 'vue'
// satisfy lint: multi-word component name
defineOptions({ name: 'HomePage' })
const showRules = ref(false)
function openRules(){ showRules.value = true }
function closeRules(){ showRules.value = false }
</script>

<style scoped>

:root {
  --bg-color: #000000;
  --text-color: #e2e8f0;
  --primary-color: #D4AF37; /* Or */
  --secondary-color: #ffffff;
  --accent-color: #ff0000;
  --glass-bg: rgba(20, 20, 20, 0.8);
  --glass-border: rgba(212, 175, 55, 0.3);
  --card-width: 200px;
  --card-height: 300px;
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: 'Inter', sans-serif;
  background-color: var(--bg-color);
  color: var(--text-color);
  min-height: 100vh;
  display: flex;
  justify-content: center;
  overflow-x: hidden;
  background-image: 
    linear-gradient(rgba(0,0,0,0.9), rgba(0,0,0,0.9)),
    url('https://www.transparenttextures.com/patterns/carbon-fibre.png');
}

#app {
  width: 100%;
  max-width: 1200px;
  padding: 2rem;
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 2rem;
  background: var(--glass-bg);
  backdrop-filter: blur(10px);
  border-radius: 0;
  border: 1px solid var(--primary-color);
  box-shadow: 0 0 20px rgba(212, 175, 55, 0.2);
}

h1 {
  font-family: 'Cinzel', serif;
  font-size: 2.5rem;
  font-weight: 900;
  color: var(--primary-color);
  text-transform: uppercase;
  letter-spacing: 5px;
  text-shadow: 0 0 10px rgba(212, 175, 55, 0.5);
}

/* Buttons */
button {
  cursor: pointer;
  border: none;
  font-family: 'Inter', sans-serif;
  transition: all 0.3s ease;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.btn-primary {
  background: var(--primary-color);
  color: black;
  padding: 1rem 2rem;
  border-radius: 0;
  font-weight: 800;
  box-shadow: 0 0 15px rgba(212, 175, 55, 0.4);
  clip-path: polygon(10% 0, 100% 0, 100% 90%, 90% 100%, 0 100%, 0 10%);
}

.btn-primary:hover {
  transform: scale(1.05);
  box-shadow: 0 0 25px rgba(212, 175, 55, 0.6);
  background: #fff;
}

.btn-secondary {
  background: transparent;
  border: 1px solid var(--primary-color);
  color: var(--primary-color);
  padding: 0.5rem 1rem;
  border-radius: 0;
}

.btn-secondary:hover {
  background: var(--primary-color);
  color: black;
}

.btn-icon {
  background: black;
  color: var(--primary-color);
  width: 50px;
  height: 50px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.5rem;
  border: 2px solid var(--primary-color);
}

.btn-icon:hover {
  background: var(--primary-color);
  color: black;
  transform: rotate(360deg);
}

/* Main Layout */
main {
  display: flex;
  flex-direction: column;
  gap: 4rem;
}

.game-area {
  display: flex;
  justify-content: space-around;
  align-items: center;
  flex-wrap: wrap;
  gap: 2rem;
}

/* Dice and timer styles - make the cube match octo-game look */
.dice-container { display:flex; flex-direction:column; align-items:center; gap:1rem }
.scene { width:120px; height:120px; position: relative; transform-style: preserve-3d; animation: floatDice 6s ease-in-out infinite; cursor: pointer }
.cube { width: 100%; height: 100%; position: absolute; transform-style: preserve-3d; transform: rotateX(-20deg) rotateY(-30deg); transition: transform 1.2s cubic-bezier(0.2,0.8,0.2,1); }
.face { position: absolute; width: 120px; height: 120px; border: 2px solid #8B6914; background-color: #1a1a1a; background-size: cover; background-position: center; box-shadow: inset 0 0 20px rgba(0,0,0,0.5); backface-visibility: hidden; }

/* Cube Faces - TranslateZ = 60px */
.face-1 { transform: rotateY(0deg) translateZ(60px); background-image: url('/assets/dice/face_1.svg'); }
.face-2 { transform: rotateY(90deg) translateZ(60px); background-image: url('/assets/dice/face_2.svg'); }
.face-3 { transform: rotateY(180deg) translateZ(60px); background-image: url('/assets/dice/face_3.svg'); }
.face-4 { transform: rotateY(-90deg) translateZ(60px); background-image: url('/assets/dice/face_4.svg'); }
.face-5 { transform: rotateX(90deg) translateZ(60px); background-image: url('/assets/dice/face_5.svg'); }
.face-6 { transform: rotateX(-90deg) translateZ(60px); background-image: url('/assets/dice/face_6.svg'); }

@keyframes floatDice { 0%,100% { transform: translateY(0) rotateX(-20deg) rotateY(-30deg); } 50% { transform: translateY(-20px) rotateX(-10deg) rotateY(-40deg); } }

.stop-anim { animation: none !important; transform: rotateX(0) rotateY(0) !important; }

.dice-result { color: var(--primary-color); margin-top: 0.5rem; min-width: 44px; height: 44px; display:inline-flex; align-items:center; justify-content:center; background: rgba(0,0,0,0.6); border: 2px solid rgba(212,175,55,0.2); border-radius:6px; font-weight:700 }

.timer-container { background: black; padding:1.5rem; border:2px solid var(--primary-color); text-align:center }
.timer-display { font-family: 'Cinzel', serif; font-size:2.5rem; color:var(--primary-color) }

/* Cards area */
.cards-area { display:flex; gap:2rem; justify-content:center }
.card-slot { width:var(--card-width); height:var(--card-height); border:2px solid var(--primary-color); background:#111 }
.card-back { width:100%; height:100%; display:flex; align-items:center; justify-content:center }

/* Modal */
.modal { position: fixed; top:0; left:0; width:100%; height:100%; background: rgba(0,0,0,0.9); display:flex; justify-content:center; align-items:center; z-index:100; backdrop-filter: blur(5px) }
.modal.hidden { display:none }
.modal-content { background: black; padding: 3rem; border:2px solid var(--primary-color); max-width:600px; width:90%; position:relative }
.close-modal { position:absolute; top:1rem; right:1rem; font-size:2rem; cursor:pointer; color:var(--primary-color) }
.modal-content h2 { color: var(--primary-color); margin-bottom: 2rem; font-family:'Cinzel', serif; text-align:center }
.modal-content ul { margin-left:1.5rem; line-height:2; color:#ccc }
.modal-content li strong { color: white }


</style>

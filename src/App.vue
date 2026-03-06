<template>
  <div class="app-container">
    <!-- Header con animación de degradado -->
    <header class="main-header">
      <h1>Lección 10: Completos en Cristo</h1>
    </header>

    <main class="content">
      <section v-if="!quizFinished" class="quiz-section">
        <div class="progress-bar">
          Pregunta {{ currentQuestionIndex + 1 }} de {{ questions.length }}
          <div class="progress-track">
            <div class="progress-fill" :style="{ width: progressWidth + '%' }"></div>
          </div>
        </div>

        <div class="question-card">
          <h2 class="question-text">{{ currentQuestion.text }}</h2>
          
          <div class="options-grid">
            <button 
              v-for="(option, index) in currentQuestion.options" 
              :key="index"
              @click="handleAnswer(index)"
              :disabled="showFeedback"
              :class="['option-btn', getOptionClass(index)]"
            >
              {{ option }}
            </button>
          </div>

          <transition name="fade">
            <div v-if="showFeedback" class="feedback-box" :class="isCorrect ? 'correct' : 'incorrect'">
              <p class="feedback-title">{{ isCorrect ? '¡Correcto!' : 'Respuesta incorrecta' }}</p>
              <p class="explanation">{{ currentQuestion.explanation }}</p>
              <button @click="nextQuestion" class="next-btn">
                {{ isLastQuestion ? 'Ver Resultados' : 'Siguiente Pregunta' }}
              </button>
            </div>
          </transition>
        </div>
      </section>

      <!-- Pantalla de Resultados -->
      <section v-else class="results-section">
        <div class="results-card">
          <h2>Evaluación Finalizada</h2>
          <div class="score-circle">
            <span class="score-num">{{ score }}</span>
            <span class="score-total">/ {{ questions.length }}</span>
          </div>
          <p class="results-msg">{{ resultsMessage }}</p>
          <button @click="resetQuiz" class="retry-btn">Reintentar Quiz</button>
        </div>
      </section>
    </main>

    <footer class="footer">
      <p>Basado en el estudio de Colosenses 2 - Teología de la Salvación ❤️ </p>
    </footer>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

// --- DATOS DEL QUIZ (Grounding Crítico del PDF) ---
const questions = ref([
  {
    text: "¿Qué significa estar 'completos en Cristo' según Colosenses 2?",
    options: [
      "Que debemos añadir ritos para ser salvos",
      "Que en Él habita toda la plenitud y no necesitamos nada externo para la plenitud espiritual",
      "Que la plenitud depende de nuestro buen desempeño religioso",
      "Que es un estado alcanzado por el esfuerzo humano"
    ],
    correctAnswer: 1,
    explanation: "Estar completo en Él significa que Jesús es la revelación absoluta y que en Él están todos los tesoros de sabiduría; no necesitamos ritos tradicionales."
  },
  {
    text: "¿Cómo se define el proceso de 'morir al yo'?",
    options: [
      "Como la anulación de la personalidad humana",
      "Como un evento único que ocurre solo una vez",
      "Como una entrega completa y continua de la voluntad a Cristo",
      "Como el rechazo a cualquier tipo de emoción"
    ],
    correctAnswer: 2,
    explanation: "Morir al yo es una reorientación radical de la voluntad donde el individuo deja de ser el centro para que la Palabra de Dios guíe su vida."
  },
  {
    text: "¿A qué se refiere Pablo con las 'sombras' en Colosenses 2:16-17?",
    options: [
      "A los Diez Mandamientos",
      "A leyes morales universales",
      "A ritos y festividades ceremoniales que señalaban la obra de Cristo",
      "A filosofías griegas oscuras"
    ],
    correctAnswer: 2,
    explanation: "Las sombras eran celebraciones religiosas y sacrificios del calendario litúrgico que prefiguraban la realidad, la cual es Cristo."
  },
  {
    text: "¿Qué simboliza la 'circuncisión hecha sin mano'?",
    options: [
      "Un procedimiento físico obligatorio",
      "La transformación interior del corazón y la conversión",
      "Una tradición humana de los colosenses",
      "El cumplimiento de leyes de pureza"
    ],
    correctAnswer: 1,
    explanation: "Representa el despojarse del cuerpo pecaminoso mediante el bautismo y la renovación del carácter por la gracia."
  },
  {
    text: "Según el análisis de la conducta, ¿qué es el 'narcisismo espiritual'?",
    options: [
      "Sentirse superior a otros por observar ritos externos o méritos propios",
      "Tener una autoestima saludable en Cristo",
      "Predicar el evangelio con autoridad",
      "Estudiar la Biblia con diligencia"
    ],
    correctAnswer: 0,
    explanation: "Es la tendencia patológica a creer que la observancia de reglamentos o cambios internos son meritorios para la salvación."
  },
  {
    text: "¿Cuál es el fundamento único de la salvación según las fuentes?",
    options: [
      "Nuestras transformaciones internas",
      "La obra objetiva de Jesús (vida, muerte y resurrección) fuera de nosotros",
      "La combinación de fe y tradiciones humanas",
      "Nuestra capacidad de entender misterios ocultos"
    ],
    correctAnswer: 1,
    explanation: "La salvación descansa en lo que Jesús hizo sin nuestra intervención; nuestros cambios son el fruto, no la causa."
  },
  {
    text: "¿Cuál es la diferencia entre el sábado semanal y los sábados ceremoniales?",
    options: [
      "No hay diferencia, todos fueron abolidos",
      "El semanal fue instituido en el Edén; los ceremoniales eran sombras del plan de redención",
      "El semanal es una tradición de hombres",
      "Los ceremoniales son más importantes que el semanal"
    ],
    correctAnswer: 1,
    explanation: "El sábado del cuarto mandamiento precede al pecado, mientras que los ceremoniales estaban ligados al sistema de sacrificios."
  },
  {
    text: "¿Qué significa estar 'arraigados' en Cristo?",
    options: [
      "Seguir filosofías humanas modernas",
      "Tener una fe firme basada en el fundamento bíblico que protege del error",
      "Estar atrapado en tradiciones antiguas",
      "Depender de la opinión de los líderes religiosos"
    ],
    correctAnswer: 1,
    explanation: "Sugiere una estabilidad que protege al individuo de ser llevado cautivo por vanas sutilezas o corrientes humanas."
  },
  {
    text: "¿Qué advierte Pablo sobre los 'mandatos y enseñanzas de hombres'?",
    options: [
      "Que son necesarios para organizar la iglesia",
      "Que tienen apariencia de sabiduría pero no tienen valor contra los apetitos de la carne",
      "Que deben ser seguidos si traen provecho espiritual",
      "Que son la base de la madurez cristiana"
    ],
    correctAnswer: 1,
    explanation: "Estas regulaciones humanas a menudo imponen ascetismo pero no transforman el corazón ni conectan con la Cabeza, que es Cristo."
  },
  {
    text: "¿Por qué es vital considerar la Biblia como la 'voz de Dios que habla directamente'?",
    options: [
      "Para evitar que filosofías como la alta crítica destruyan la fe en la revelación divina",
      "Porque es un libro de historia interesante",
      "Para ganar discusiones teológicas",
      "Porque las tradiciones humanas son insuficientes para escribir libros"
    ],
    correctAnswer: 0,
    explanation: "Dudar de su autoridad deja la mente vulnerable a la cautividad intelectual y a marcos cognitivos erróneos."
  }
]);

// --- LÓGICA DEL ESTADO ---
const currentQuestionIndex = ref(0);
const score = ref(0);
const showFeedback = ref(false);
const isCorrect = ref(false);
const quizFinished = ref(false);
const selectedAnswer = ref(null);

const currentQuestion = computed(() => questions.value[currentQuestionIndex.value]);
const progressWidth = computed(() => ((currentQuestionIndex.value + 1) / questions.length) * 100);
const isLastQuestion = computed(() => currentQuestionIndex.value === questions.length - 1);

const resultsMessage = computed(() => {
  const percentage = (score.value / questions.length) * 100;
  if (percentage === 100) return "¡Plenitud total! Estás profundamente arraigado en la Verdad [3].";
  if (percentage >= 70) return "Gran conocimiento. Sigue creciendo en la gracia y el conocimiento de Cristo [30].";
  return "Es una oportunidad para volver a las fuentes y consolidar tu fundamento en Él [2].";
});

// --- ACCIONES ---
const handleAnswer = (index) => {
  selectedAnswer.value = index;
  isCorrect.value = index === currentQuestion.value.correctAnswer;
  if (isCorrect.value) score.value++;
  showFeedback.value = true;
};

const getOptionClass = (index) => {
  if (!showFeedback.value) return '';
  if (index === currentQuestion.value.correctAnswer) return 'correct-opt';
  if (index === selectedAnswer.value && !isCorrect.value) return 'wrong-opt';
  return 'fade-opt';
};


const nextQuestion = () => {
  showFeedback.value = false
  selectedAnswer.value = null

  if (currentQuestionIndex.value < questions.value.length - 1) {
    currentQuestionIndex.value++
  } else {
    quizFinished.value = true
  }

  console.log("Quiz terminado:", quizFinished.value)
}


const resetQuiz = () => {
  currentQuestionIndex.value = 0;
  score.value = 0;
  showFeedback.value = false;
  quizFinished.value = false;
  selectedAnswer.value = null;
};
</script>

<style scoped>
/* Estilos Base y Accesibilidad */
.app-container {
  font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
  max-width: 1200px;
  margin: 0 auto;
  color: #2c3e50;
  background-color: #f9fbfc;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* Header con Animación */
.main-header {
  padding: 3rem 1rem;
  text-align: center;
  color: white;
  background: linear-gradient(-45deg, #5BB7C6, #6983B4, #5BB7C6);
  background-size: 400% 400%;
  /*animation: gradientBG 15s ease infinite;*/
  box-shadow: 0 4px 15px rgba(0,0,0,0.1);
}

@keyframes gradientBG {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

.main-header h1 {
  margin: 0;
  font-size: 2.5rem;
  letter-spacing: 1px;
}

/* Contenido Principal */
.content {
  flex: 1;
  padding: 2rem 1rem;
  display: flex;
  justify-content: center;
}

.quiz-section, .results-section {
  width: 100%;
  max-width: 800px;
}

/* Barra de Progreso */
.progress-bar {
  margin-bottom: 1.5rem;
  font-weight: 600;
  color: #6983B4;
}

.progress-track {
  height: 8px;
  background: #e0e6ed;
  border-radius: 4px;
  margin-top: 0.5rem;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: #5BB7C6;
  transition: width 0.4s ease;
}

/* Card de Pregunta */
.question-card {
  background: white;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 10px 25px rgba(0,0,0,0.05);
}

.question-text {
  font-size: 1.4rem;
  margin-bottom: 2rem;
  line-height: 1.4;
}

/* Botones de Opción */
.options-grid {
  display: grid;
  gap: 1rem;
}

.option-btn {
  padding: 1.2rem;
  border: 2px solid #e0e6ed;
  border-radius: 8px;
  background: white;
  cursor: pointer;
  font-size: 1.1rem;
  text-align: left;
  transition: all 0.2s;
  color: #34495e;
}

.option-btn:hover:not(:disabled) {
  border-color: #5BB7C6;
  background-color: #f0f9fa;
}

.correct-opt {
  border-color: #27ae60 !important;
  background-color: #eafaf1 !important;
  color: #1e8449;
  font-weight: bold;
}

.wrong-opt {
  border-color: #e74c3c !important;
  background-color: #fdedec !important;
  color: #943126;
}

.fade-opt {
  opacity: 0.6;
}

/* Feedback */
.feedback-box {
  margin-top: 2rem;
  padding: 1.5rem;
  border-radius: 8px;
  animation: slideIn 0.3s ease-out;
}

.feedback-box.correct { background: #eafaf1; border-left: 5px solid #27ae60; }
.feedback-box.incorrect { background: #fdf2f2; border-left: 5px solid #e74c3c; }

.feedback-title {
  font-weight: bold;
  margin-bottom: 0.5rem;
  font-size: 1.2rem;
}

.explanation {
  font-style: italic;
  line-height: 1.5;
  margin-bottom: 1.5rem;
}

.next-btn, .retry-btn {
  background: #6983B4;
  color: white;
  border: none;
  padding: 0.8rem 1.5rem;
  border-radius: 6px;
  cursor: pointer;
  font-size: 1rem;
  transition: background 0.3s;
}

.next-btn:hover, .retry-btn:hover {
  background: #536a91;
}

/* Pantalla de Resultados */
.results-card {
  text-align: center;
  background: white;
  padding: 3rem;
  border-radius: 12px;
  box-shadow: 0 10px 25px rgba(0,0,0,0.05);
}

.score-circle {
  width: 150px;
  height: 150px;
  border: 8px solid #5BB7C6;
  border-radius: 50%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  margin: 2rem auto;
}

.score-num { font-size: 3rem; font-weight: bold; color: #5BB7C6; }
.score-total { color: #7f8c8d; }

.results-msg {
  font-size: 1.2rem;
  margin-bottom: 2rem;
  color: #34495e;
}

/* Footer */
.footer {
  text-align: center;
  padding: 1.5rem;
  font-size: 0.9rem;
  color: #7f8c8d;
}

/* Animaciones */
@keyframes slideIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}

/* Responsividad */
@media (max-width: 768px) {
  .main-header h1 { font-size: 1.8rem; }
  .question-text { font-size: 1.2rem; }
  .option-btn { font-size: 1rem; }
}
</style>





<!-- App.vue -->
<template>
  <div class="quiz-app">
    <div class="quiz-container">
      <!-- Start Screen -->
      <div v-if="!started && !finished" class="start-screen">
        <h1>Awesome Quiz</h1>
        <div class="category-select">
          <label for="category">Select Category:</label>
          <select v-model="selectedCategory" id="category">
            <option v-for="cat in categories" :key="cat.id" :value="cat.id">
              {{ cat.name }}
            </option>
          </select>
        </div>
        <div class="difficulty-select">
          <label for="difficulty">Select Difficulty:</label>
          <select v-model="selectedDifficulty" id="difficulty">
            <option value="easy">Easy</option>
            <option value="medium">Medium</option>
            <option value="hard">Hard</option>
          </select>
        </div>
        <button @click="startQuiz" class="start-button">Start Quiz</button>
      </div>

      <!-- Loading State -->
      <div v-if="loading" class="loading">
        <div class="loader"></div>
        <p>Loading questions...</p>
      </div>

      <!-- Quiz Questions -->
      <div v-if="started && !finished && !loading" class="quiz-content">
        <div class="progress-bar">
          <div :style="{ width: `${progress}%` }" class="progress-fill"></div>
        </div>
        
        <div class="quiz-header">
          <span class="question-counter">Question {{ currentQuestionIndex + 1 }}/{{ questions.length }}</span>
          <span class="score-display">Score: {{ score }}</span>
        </div>

        <div class="question-card">
          <h2 v-html="currentQuestion.question"></h2>
          <div class="options-grid">
            <button 
              v-for="(option, index) in currentQuestion.options" 
              :key="index"
              @click="selectAnswer(option)"
              :class="{
                'option-button': true,
                'selected': selectedAnswer === option,
                'correct': answered && option === currentQuestion.correct_answer,
                'wrong': answered && selectedAnswer === option && option !== currentQuestion.correct_answer
              }"
              :disabled="answered"
              v-html="option"
            ></button>
          </div>
        </div>

        <button 
          v-if="answered"
          @click="nextQuestion" 
          class="next-button"
        >
          {{ currentQuestionIndex === questions.length - 1 ? 'Show Results' : 'Next Question' }}
        </button>
      </div>

      <!-- Results Screen -->
      <div v-if="finished" class="results-screen">
        <h2>Quiz Complete! 🎉</h2>
        <div class="results-card">
          <div class="score-circle">
            <span class="final-score">{{ Math.round(correctAnswers / questions.length * 100) }}%</span>
          </div>
          <div class="statistics">
            <p>Final Score: {{ score }} points</p>
            <p>Correct Answers: {{ correctAnswers }} out of {{ questions.length }}</p>
            <p>Category: {{ categoryName }}</p>
            <p>Difficulty: {{ selectedDifficulty.charAt(0).toUpperCase() + selectedDifficulty.slice(1) }}</p>
          </div>
          <button @click="restartQuiz" class="restart-button">Try Again</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      started: false,
      finished: false,
      loading: false,
      currentQuestionIndex: 0,
      score: 0,
      correctAnswers: 0,
      selectedAnswer: null,
      answered: false,
      questions: [],
      selectedCategory: 9, // Default: General Knowledge
      selectedDifficulty: 'medium',
      categories: [],
      categoryName: ''
    }
  },
  computed: {
    currentQuestion() {
      return this.questions[this.currentQuestionIndex] || {}
    },
    progress() {
      return (this.currentQuestionIndex / this.questions.length) * 100
    }
  },
  methods: {
    async fetchCategories() {
      try {
        const response = await fetch('https://opentdb.com/api_category.php')
        const data = await response.json()
        this.categories = data.trivia_categories
      } catch (error) {
        console.error('Error fetching categories:', error)
      }
    },
    async fetchQuestions() {
      this.loading = true
      try {
        const response = await fetch(
          `https://opentdb.com/api.php?amount=10&category=${this.selectedCategory}&difficulty=${this.selectedDifficulty}&type=multiple`
        )
        const data = await response.json()
        
        this.questions = data.results.map(q => ({
          question: q.question,
          options: this.shuffleArray([...q.incorrect_answers, q.correct_answer]),
          correct_answer: q.correct_answer
        }))
        
        // Store category name
        const category = this.categories.find(c => c.id === this.selectedCategory)
        this.categoryName = category ? category.name : ''
      } catch (error) {
        console.error('Error fetching questions:', error)
      }
      this.loading = false
    },
    shuffleArray(array) {
      for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]]
      }
      return array
    },
    async startQuiz() {
      await this.fetchQuestions()
      this.started = true
      this.resetQuiz()
    },
    selectAnswer(answer) {
      if (!this.answered) {
        this.selectedAnswer = answer
        this.answered = true
        
        if (answer === this.currentQuestion.correct_answer) {
          this.score += 10
          this.correctAnswers++
        }
      }
    },
    nextQuestion() {
      if (this.currentQuestionIndex < this.questions.length - 1) {
        this.currentQuestionIndex++
        this.selectedAnswer = null
        this.answered = false
      } else {
        this.finished = true
      }
    },
    resetQuiz() {
      this.currentQuestionIndex = 0
      this.score = 0
      this.correctAnswers = 0
      this.selectedAnswer = null
      this.answered = false
      this.finished = false
    },
    restartQuiz() {
      this.started = false
      this.resetQuiz()
    }
  },
  mounted() {
    this.fetchCategories()
  }
}
</script>

<style>
@import './styles/quiz.css';
</style>
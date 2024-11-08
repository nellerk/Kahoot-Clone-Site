<template>
<div class="flex items-center justify-center min-h-screen bg-gray-100">
    <div class="bg-white p-8 rounded-lg shadow-lg max-w-md w-full">
      <h1 class="text-center text-2xl font-semibold mb-4">{{ category }} Quiz</h1>
      
      <div v-if="currentQuestionIndex < questions.length">
        <p class="text-lg text-gray-700 mb-6">{{ currentQuestion.text }}</p>
        
        <!-- Display image if available -->
        <div v-if="currentQuestion.imageUrl" class="mb-6">
          <img
            :src="`${currentQuestion.imageUrl}`"
            alt="Question image"
            class="w-full h-auto rounded-lg shadow-sm"
          />
        </div>
        
        <ul class="space-y-4">
  <li v-for="(answer, index) in currentQuestion.answers" :key="index">
    <button
      @click="selectAnswer(index)"
      :disabled="showCorrectAnswer" 
      :class="getAnswerClass(index)"
      class="w-full py-3 px-4 rounded-lg text-left font-medium transition-colors duration-150"
    >
      {{ answer }}
    </button>
  </li>
</ul>

  
        <button
          v-if="selectedAnswer !== null && !showCorrectAnswer"
          @click="revealCorrectAnswer"
          class="mt-6 w-full py-3 px-4 bg-green-500 text-white rounded-lg font-medium hover:bg-green-600 transition-colors duration-150"
        >
          Következő kérdés
        </button>
      </div>
  
      <div v-else class="text-center">
        <h2 class="text-2xl font-semibold text-gray-800 mb-4">Eredmény</h2>
        <p class="text-lg text-gray-700 mb-6">Helyes válaszok száma: {{ score }} / {{ questions.length }}</p>
  
        <div v-for="(question, index) in questions" :key="index" class="mb-4 text-left">
        <p class="font-semibold">{{ index + 1 }}. {{ question.text }}</p>
        <ul class="ml-4 mt-2">
          <li
            v-for="(answer, i) in question.answers"
            :key="i"
            :class="{
              'text-green-600 font-semibold': question.correctAnswers.includes(i),
              'text-red-500': i === userAnswers[index] && !question.correctAnswers.includes(i),
              'text-gray-700': !question.correctAnswers.includes(i) && i !== userAnswers[index]
            }"
            class="ml-2"
          >
            {{ answer }}
          </li>
        </ul>
      </div>
  
        <button
          @click="restartQuiz"
          class="mt-6 w-full py-3 px-4 bg-blue-500 text-white rounded-lg font-medium hover:bg-blue-600 transition-colors duration-150"
        >
          Újrakezdés
        </button>
      </div>
    </div>
</div>
</template>
  
  <script>
  import axios from 'axios';
  
  export default {
    props: ['category'],
    data() {
      return {
        questions: [],
        currentQuestionIndex: 0,
        selectedAnswer: null,
        score: 0,
        userAnswers: [],
        showCorrectAnswer: false,
        apiBaseUrl: 'https://szakmasztarapi.runasp.net/api',
        isAnswerConfirmed: false // Prevent multiple scoring
      };
    },
    computed: {
      currentQuestion() {
        return this.questions[this.currentQuestionIndex];
      },
      isLastQuestion() {
        return this.currentQuestionIndex === this.questions.length - 1;
      }
    },
    methods: {
      async fetchQuestions() {
        try {
          const response = await axios.get(`${this.apiBaseUrl}/questions/category/${this.category}`);
          this.questions = response.data;
        } catch (error) {
          console.error('Error fetching questions:', error);
        }
      },
      selectAnswer(index) {
        this.selectedAnswer = index;
      },
      revealCorrectAnswer() {
        // Prevent this logic from running multiple times
        if (this.isAnswerConfirmed) return;
        this.isAnswerConfirmed = true;

        // Only increment the score if the selected answer is correct
        if (this.selectedAnswer !== null && this.currentQuestion.correctAnswers.includes(this.selectedAnswer)) {
          this.score++;
        }

        this.showCorrectAnswer = true;
        setTimeout(() => {
          this.isAnswerConfirmed = false; // Reset the flag for the next question
          this.nextQuestion();
        }, 1500);
      },
      nextQuestion() {
        this.userAnswers.push(this.selectedAnswer); // Track the user's answer
        this.currentQuestionIndex++;
        this.selectedAnswer = null;
        this.showCorrectAnswer = false;
      },
      getAnswerClass(index) {
    if (this.selectedAnswer === null) {
      return 'bg-gray-200 text-gray-700 hover:bg-blue-100';
    }
    if (this.showCorrectAnswer) {
      if (this.currentQuestion.correctAnswers.includes(index)) {
        return 'bg-green-500 text-white'; // Highlight correct answers
      }
      if (index === this.selectedAnswer && !this.currentQuestion.correctAnswers.includes(index)) {
        return 'bg-red-500 text-white'; // Highlight incorrect selection
      }
    }
    return this.selectedAnswer === index ? 'bg-blue-500 text-white' : 'bg-gray-200 text-gray-700';
  },
      restartQuiz() {
        this.$emit('quizEnded');
        this.currentQuestionIndex = 0;
        this.selectedAnswer = null;
        this.score = 0;
        this.userAnswers = [];
        this.showCorrectAnswer = false;
        this.fetchQuestions(); // Reload questions when restarting
      }
    },
    mounted() {
      this.fetchQuestions();
    }
  };
  </script>
  
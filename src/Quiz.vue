<template>
  
<div class="flex items-center justify-center min-h-screen bg-gray-100">
  <div class="absolute top-4 right-4 bg-gray-200 text-gray-700 rounded-lg px-4 py-2 font-semibold">
      Pontok: {{ totalPoints }}
    </div>
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
      class="w-full py-3 px-4 rounded-lg text-left font-medium transition-colors duration-150 whitespace-normal break-words min-h-[50px] max-h-[150px] overflow-y-auto"
    >
      {{ answer }}
    </button>
  </li>
</ul>

<button
  v-if="selectedAnswers.length > 0 && !showCorrectAnswer"
  @click="revealCorrectAnswer"
  class="mt-6 w-full py-3 px-4 bg-green-500 text-white rounded-lg font-medium hover:bg-green-600 transition-colors duration-150"
>
  Következő kérdés
</button>

      </div>
  
      <div v-else class="text-center">
        <h2 class="text-2xl font-semibold text-gray-800 mb-4">Eredmény</h2>
        <p class="text-lg text-gray-700 mb-6">Helyes válaszok száma: {{ score }} / {{ questions.length }}</p>
        <p class="text-lg text-gray-700 mb-6">Összes pontszám: {{ totalPoints }}</p>
  
        <div v-for="(question, index) in questions" :key="index" class="mb-4 text-left">
  <p class="font-semibold">{{ index + 1 }}. {{ question.text }}</p>
  <ul class="ml-4 mt-2">
    <li
      v-for="(answer, i) in question.answers"
      :key="i"
      :class="{
        'text-green-600 font-semibold': question.correctAnswers.includes(i) && userAnswers[index]?.includes(i),
        'text-yellow-500 font-semibold': question.correctAnswers.includes(i) && !userAnswers[index]?.includes(i),
        'text-red-500 font-semibold': !question.correctAnswers.includes(i) && userAnswers[index]?.includes(i),
        'text-gray-700': !userAnswers[index]?.includes(i) && !question.correctAnswers.includes(i)
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
        selectedAnswers: [],
        score: 0,
        totalPoints: 0, // Total points tracker
        userAnswers: [],
        showCorrectAnswer: false,
        apiBaseUrl: 'http://localhost:5048/api',
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
        const correctAnswerLimit = this.currentQuestion.correctAnswers.length; // Determine the limit

        if (correctAnswerLimit === 1) {
          // For single correct answer, allow free switching
          this.selectedAnswers = [index]; // Replace the selection with the new one
        } else {
          // For multiple correct answers, enforce the selection limit
          if (this.selectedAnswers.includes(index)) {
            // Deselect the answer if already selected
            this.selectedAnswers = this.selectedAnswers.filter(i => i !== index);
          } else if (this.selectedAnswers.length < correctAnswerLimit) {
            // Allow selection only if the limit is not reached
            this.selectedAnswers.push(index);
          } else {
            // Optionally, inform the user of the limit
            alert(`You can select up to ${correctAnswerLimit} answers.`);
          }
        }
      },
    revealCorrectAnswer() {
      const correctAnswers = this.currentQuestion.correctAnswers;
      const numCorrectAnswers = correctAnswers.length;
      const correctSelections = this.selectedAnswers.filter(answer => correctAnswers.includes(answer)).length;

      let points = 0;
      if (numCorrectAnswers === 1) {
        points = correctSelections === 1 ? 2 : 0;
      } else if (numCorrectAnswers === 2) {
        if (correctSelections === 2) points = 4;
        else if (correctSelections === 1) points = 2;
      } else if (numCorrectAnswers === 3) {
        if (correctSelections === 3) points = 6;
        else if (correctSelections === 2) points = 4;
        else if (correctSelections === 1) points = 2;
      }

      this.totalPoints += points; // Add points to total
      if (correctSelections === numCorrectAnswers) this.score++; // Increment the "correctly answered" counter if fully correct

      this.showCorrectAnswer = true;
      setTimeout(this.nextQuestion, 1500); // Automatically move to the next question
    },
    nextQuestion() {
      this.userAnswers.push([...this.selectedAnswers]); // Save user selections
      this.selectedAnswers = []; // Reset selections for the next question
      this.currentQuestionIndex++;
      this.showCorrectAnswer = false;
  },
    getAnswerClass(index) {
      if (!this.showCorrectAnswer) {
        // Before confirmation, highlight all selected answers in blue
        return this.selectedAnswers.includes(index)
          ? 'bg-blue-500 text-white'
          : 'bg-gray-200 text-gray-700 hover:bg-blue-100'; // Default state for unselected answers
      }

      // After confirmation, handle highlighting based on correctness
      const incorrectSelected = this.selectedAnswers.some(
        selected => !this.currentQuestion.correctAnswers.includes(selected)
      );

      if (incorrectSelected) {
        // If an incorrect answer is selected, highlight all correct answers in green
        if (this.currentQuestion.correctAnswers.includes(index)) {
          return 'bg-yellow-500 text-black';
        }

        // Highlight selected incorrect answers in red
        if (this.selectedAnswers.includes(index)) {
          return 'bg-red-500 text-white';
        }

        return 'bg-gray-200 text-gray-700'; // Keep other answers neutral
      }

      // Handle partially selected correct answers
      const selectedCorrectAnswers = this.selectedAnswers.filter(
        selected => this.currentQuestion.correctAnswers.includes(selected)
      );

      if (
        this.currentQuestion.correctAnswers.includes(index) &&
        !this.selectedAnswers.includes(index)
      ) {
        // Highlight unselected correct answers in yellow
        if (
          selectedCorrectAnswers.length > 0 &&
          selectedCorrectAnswers.length < this.currentQuestion.correctAnswers.length
        ) {
          return 'bg-yellow-400 text-black';
        }
      }

      // Highlight selected correct answers in green
      if (this.currentQuestion.correctAnswers.includes(index) && this.selectedAnswers.includes(index)) {
        return 'bg-green-500 text-white';
      }

      // Highlight incorrect answers in red if selected but not in "showCorrectAnswer"
      if (!this.currentQuestion.correctAnswers.includes(index) && this.selectedAnswers.includes(index)) {
        return 'bg-red-500 text-white';
      }

      return 'bg-gray-200 text-gray-700 hover:bg-blue-100'; // Default for unselected answers
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

<style>
.absolute {
  position: absolute;
}
.top-4 {
  top: 1rem;
}
.right-4 {
  right: 1rem;
}
.bg-gray-200 {
  background-color: #e2e8f0;
}
.text-gray-700 {
  color: #4a5568;
}
.rounded-lg {
  border-radius: 0.5rem;
}
.px-4 {
  padding-left: 1rem;
  padding-right: 1rem;
}
.py-2 {
  padding-top: 0.5rem;
  padding-bottom: 0.5rem;
}
.font-semibold {
  font-weight: 600;
}

.bg-yellow-400 {
  background-color: #f6e05e; /* Yellow */
}
.text-black {
  color: #000000; /* Black text for better contrast */
}

</style>
  
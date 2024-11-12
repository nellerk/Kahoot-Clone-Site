<template>
  <div>
    <!-- Only show the menu and title if no quiz is active -->
    <div v-if="!isQuizActive">
      <h1 class="text-center text-3xl font-bold mb-6">Kvíz</h1>
      <div class="flex justify-center space-x-4">
        <button v-for="category in categories" :key="category" @click="selectCategory(category)"
                class="px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600">
          {{ category }}
        </button>
      </div>
    </div>

    <!-- Show Quiz Component if a category is selected -->
    <Quiz v-if="selectedCategory" :category="selectedCategory" @quizEnded="resetQuiz" />
  </div>
</template>

<script>
import Quiz from './Quiz.vue';

export default {
  components: { Quiz },
  data() {
    return {
      categories: [
        'Információtechológiai Alapok',
        'Hálózatok',
        'Programozás és Adatbázis Kezelés',
        'Szoftverfejlesztés és Webfejlesztés'
      ],
      selectedCategory: null,
      isQuizActive: false
    };
  },
  methods: {
    selectCategory(category) {
      this.selectedCategory = category;
      this.isQuizActive = true; // Set quiz active to hide menu and title
    },
    resetQuiz() {
      this.selectedCategory = null;
      this.isQuizActive = false; // Reset quiz active to show menu and title
    }
  }
};
</script>

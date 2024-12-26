<template>
  <div class="container mt-4">
    <h1 class="text-center">Cooking Recipes</h1>
    <hr>

    <!-- Sorting Area -->
    <div class="d-flex justify-content-center align-items-center mb-3 gap-2">
      <template v-for="sortOption in sortOptions" :key="sortOption.key">
        <span 
          class="badge bg-primary" 
          @click="sortRecipes(sortOption.key)"
          style="cursor: pointer;"
        >
          {{ sortOption.label }} 
          <i :class="currentSortKey === sortOption.key ? (sortOrder === 'asc' ? 'fa-solid fa-arrow-up' : 'fa-solid fa-arrow-down') : ''"></i>
        </span>
      </template>
      <span 
        class="badge bg-danger" 
        v-if="currentSortKey" 
        @click="resetSorting"
        style="cursor: pointer;"
      >
        <i class="fa-solid fa-xmark"></i>
      </span>
    </div>
    <hr> 


    <!-- Recipe Cards -->
    <div class="row g-2 mb-5" v-if="displayingRecipes?.length > 0">
      <h5 v-if="currentType" class="text-center">
        Now displaying: {{ currentType }} 
        <span @click="goBackToBasic" class="badge bg-danger p-1 ml-1" style="cursor: pointer;">
          <i class="fa-solid fa-xmark"></i>
        </span>
      </h5>
      <div
        class="col-4"
        v-for="recipe in displayingRecipes"
        :key="recipe.name"
      >
        <div class="card h-100" :class="recipe.times == 0 ? `flashing-border` : `` ">
          <div class="card-body">
            <div class="recipeImg-container mb-2">
              <img :src="recipe.img || '/img/notFound.jpg'" alt="Thumbnail" class="recipeImg">
            </div>
            <div class="d-flex align-items-center mb-1 custom-gap">
              <span class="badge bg-warning p-1 text-secondary"> ¥{{ recipe.price }}</span>
              <span @click="showType(recipe.type)" class="badge bg-success">{{ recipe.type }}</span>
            </div>
            <div class="d-flex align-items-center mb-2 custom-gap">
              <span class="badge bg-secondary p-1">{{ convertMinutes(recipe.duration) }}</span>
              <span class="badge bg-secondary">Lv. {{ recipe.difficulty }}/5</span>
            </div>
            <h5 class="card-title">{{ recipe.name }} <small class="text-secondary">x{{ recipe.times }}</small></h5>
            <!-- <p><strong>Ingredients:</strong> {{ recipe.ingredients }}</p>
            <p v-if="recipe.description"><strong>Description:</strong> {{ recipe.description }}</p> -->
          </div>
        </div>
      </div>
    </div>
    <div v-else><h5 class="text-center">Loading data now...</h5></div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      baseData: [],
      displayingRecipes: [], // To store recipes fetched from the API
      baseApiUrl: 'https://script.google.com/macros/s/AKfycbzWYzucDKrTiwQaFM6WCPUFc5MXcHehZLozXAQJOTopBm0HT3it4TgvbBAsdzRlxScs/exec',
      currentType: null,
      sortOrder: 'desc', 
      sortOptions: [
        { key: 'price', label: '¥' },
        { key: 'duration', label: 'mins' },
        { key: 'difficulty', label: 'Lv.' },
        { key: 'times', label: 'times' },
        { key: 'type', label: 'category' },
      ],

    };
  },
  methods: {
    async fetchRecipes() {
      try {
        const URL = `${this.baseApiUrl}?action=getAllRecipes`
        console.log(URL)
        const response = await fetch(URL);
        const data = await response.json();
        if (data.success) {
          this.baseData = data.data; // Assign fetched data to recipes
          this.displayingRecipes = this.baseData
        } else {
          console.error("Failed to fetch recipes:", data.message);
        }
      } catch (error) {
        console.error("Error fetching recipes:", error);
      }
    },
    sortRecipes(property) {
      if (this.currentSortKey === property) {
        // Flip the sort order if the same property is clicked
        this.sortOrder = this.sortOrder === 'asc' ? 'desc' : 'asc';
      } else {
        // Set new property and default to ascending order
        this.currentSortKey = property;
        this.sortOrder = 'desc';
      }

      // Sort logic
      const orderMultiplier = this.sortOrder === 'asc' ? 1 : -1;
      this.displayingRecipes.sort((a, b) => {
        if (a[property] > b[property]) return orderMultiplier;
        if (a[property] < b[property]) return -orderMultiplier;
        return 0;
      });
    },
    resetSorting() {
      this.currentSortKey = null;
      this.sortOrder = 'desc';
      if(this.currentType){
        return this.displayingRecipes = this.baseData.filter(recipe => recipe.type === this.currentType);
      }else{
        return this.displayingRecipes = [...this.baseData];
      }
    },

    convertMinutes(number){
      if(number < 60) return `${number}分`

      const hours = Math.floor(number / 60);
      const minutes = number % 60;
      return `${hours}時間${minutes > 0 ? `${minutes}分` : ''}`;
    },
    showType(type){
      this.currentType = type;
      this.displayingRecipes = this.baseData.filter(recipe => recipe.type === type);
    },
    goBackToBasic(){
      this.currentType = null;
      this.displayingRecipes = this.baseData;
    },
  },
  mounted() {
    console.clear();
    this.fetchRecipes(); // Fetch data when the component mounts
  }
};
</script>

<style>
  #app {
    height: 100vh; /* Ensures the app takes the full viewport height */
    display: flex; /* Optional: For centering or flex layouts */
    flex-direction: column; /* Optional: Ensures children stack vertically */
  }
  .container {
    max-width: 1200px;
    margin: auto;
  }
  .card {
    border: 1px solid #ddd;
  }

  .flashing-border {
    border: unset;
    outline: 4px solid moccasin; /* Initial border color */
    animation: heartbeat 3s infinite;
  }

  @keyframes heartbeat {
    0% {
      outline: 4px solid moccasin;
    }
    50% {
      outline: 1px solid moccasin;
    }
    100% {
      outline: 4px solid moccasin;
    }
  }
  .card-body {
    padding: .5em;
    font-size: 14px;
  }

  .card-body .recipeImg-container{
    position: relative;
    display: block;
    widows: 100%;
    aspect-ratio: 5/4;
    overflow: hidden;

    border: 1px solid grey;
  }
  
  .card-body .recipeImg-container img{
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%,-50%);
    
    height: 100%;
    width: auto;
  }

  .card-body .card-title{
    font-size: 1.1rem
    /* display: -webkit-box;
    -webkit-line-clamp: 2; 
    -webkit-box-orient: vertical;
    overflow: hidden;
    text-overflow: ellipsis;
    line-height: 1.5;
    min-height: calc(1.5em * 2); 
    max-height: calc(1.5em * 2); */
  }

  .custom-gap{
    gap: .5em;
  }
</style>

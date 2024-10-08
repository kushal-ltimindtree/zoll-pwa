<template>
  <div class="recipe wrapper">
    <div class="recipe__comp">
      <img :src="recipe?.image"/>
      <div>
        <p><PhBowlFood size="20"/> {{ numberOfIngredient }} ingredents</p>
        <p><PhTimer size="20" /> {{ recipe?.readyInMinutes }} mins</p>
        <p><PhUser size="20" /> {{ recipe?.servings }} serves</p>
      </div>
    </div>
    <div class="recipe__content">
      <div class="recipe__info">
        <h1>{{ recipe?.title }}</h1>
        <p v-html="recipe?.summary"></p>
      </div>
      <div class="recipe__steps">
        <h2>Cooking- instruction</h2>
        <router-link to="/">home</router-link>
        <div v-if="numberOfIngredient > 0">
          <div class="recipe__step"
            v-for="step in recipe?.analyzedInstructions[0].steps"
            v-bind:key="step.number"
          >
            <h3>Step {{ step.number }}</h3>
            <p>
              {{ step?.step }}
            </p>
            <h4>Ingredents</h4>
            <ul>
              <li v-for="ingredient in step?.ingredients" v-bind:key="ingredient.name">{{ ingredient.name }}</li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import axios from 'axios';
import {PhBowlFood, PhTimer, PhUser} from '@phosphor-icons/vue'
import { computed, ref } from 'vue';
import {watch} from 'vue'
import { useRoute } from 'vue-router';

const recipe = ref(null);
const route = useRoute();


const checkedPopularRecipes = ref(false);

(async () => {
  if ('caches' in window){
    const cache = await caches.open('popular-recipes')
    const responses = await cache.matchAll()
    if(responses.length) {
      const responsesData = await Promise.all(responses.map(async (response) => 
        await response.json()
      ))
      console.log('ResponseData =>', responsesData)
      const rec = responsesData[0].results.find((recipe) => recipe.id === Number(route.params.id))
      if(rec){
        const cacheRecent = await caches.open('recently-viewed')
        const res = await cacheRecent.matchAll()
        if(res.length) {
          const resData = await Promise.all(res.map(async (response) => 
            await response.json()
          ))
          recipe.value = resData.find((recipe) => recipe.id === Number(route.params.id))
        }
      }
      checkedPopularRecipes.value = true
    }
  }
})(); 
// set recipe state to the recipe from locastorage if it exist else fetch recipe then store it in the state and localstorage
// Remove earliest recipe from storage if recipes in storage equal 8

watch([recipe, checkedPopularRecipes], async ([newRecipe, newCheckPopRecipes]) => {
  if(newCheckPopRecipes && !newRecipe) {
    const res = await axios.get(`https://api.spoonacular.com/recipes/${route.params.id}/information?apiKey=${process.env.VUE_APP_API_KEY}`)
    if(res.data) recipe.value = res.data
  }
})

const numberOfIngredient = computed(() => {
  let num = 0
  if (recipe.value && recipe.value.analyzedInstructions && recipe.value.analyzedInstructions.length > 0)
  {
    recipe.value.analyzedInstructions[0].steps.forEach(step => {
      num += step.ingredients.length
    });
  }
  return num
})
</script>

<style scoped>
.recipe {
  min-height: 100vh;
  background-color: white;
}
.recipe__comp {
  width: 100%;
}

.recipe__comp > div {
  width: calc(100% - 40px);
  margin: 0 auto;
  background-color: rgba(0, 0, 0, 0.5);
  padding: 30px 20px;
  margin-top: -160px;
  position: relative;
  color: white;
  border-radius: 20px;
  margin-bottom: 10px;
  display: flex;
  gap: 10px;
  justify-content: space-between;
  flex-wrap: wrap;
  justify-content: center;
}

.recipe__comp > div p {
  display: flex;
  align-items: center;
}

.recipe__comp > div p > svg {
  margin-right: 5px;
}

.recipe__comp img {
  width: 100%;
  min-height: 350px;
  object-fit: cover;
  max-height: 450px;
}

.recipe__content {
  border-top-left-radius: 30px;
  border-top-right-radius: 30px;
  background-color: white;
  padding: 20px;
  text-align: left;
  color: black;
  z-index: 100;
  position: relative;
}

.recipe__info h1 {
  font-size: 24px;
  margin-bottom: 10px;
}

.recipe__info p {
  color: rgb(77, 78, 78);
  line-height: 20px;
}

.recipe__steps h2 {
  margin-top: 20px;
  font-size: 20px;
  margin-bottom: 10px;
  font-family: unset;
}

.recipe__steps > div {
  background-color: rgba(250, 238, 216, 0.692);
  padding: 20px;
  border-radius: 20px;
  color: rgb(77, 78, 78);
}

.recipe__step h3{
  font-size: 18px;
  margin-bottom: 10px;
  margin-top: 20px;
  color: rgb(236, 171, 51);
}

.recipe__step:first-child h3 {
  margin-top: 0;
}

.recipe__step h4 {
  margin-bottom: 5px;
  margin-top: 10px;
}

.recipe__info a {
  color: gray;
}

.recipe__step ul {
  font-size: 16px;
  list-style-position: inside;
}
</style>
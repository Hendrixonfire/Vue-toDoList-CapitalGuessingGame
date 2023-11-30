<template>
  <div id="app">
   
    <router-link to="/">Home page</router-link>
    <router-link to="/capitals">Guess a capital Game:</router-link>

    <router-view></router-view>
  </div>
</template>

<style scoped>
/* Стили для About.vue */

a{
  display:inline;
  text-decoration:none;
  margin:auto;
  padding:5px;

}

</style>
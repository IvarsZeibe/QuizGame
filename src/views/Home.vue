<template>
  <div class="d-flex flex-column flex-grow-1 justify-center p-1">
    <div id="container">
      <div id="title">Quiz Game</div>
      <router-link
        to="/quiz"
        custom
        v-slot="{ navigate }"
      >
        <v-btn @click="navigate" role="link" id="start-button">Start a new Quiz</v-btn>
      </router-link>
    </div>
  </div>
</template>
  
<style scoped>
#title {
  font-size: 5em;
  text-align: center;
  color: #face09;
  text-shadow: 2px 2px 2px #000000;
  font-weight: bold;
}

#container {
  margin-top: -10vh;
  font-family: 'Courier New', Courier, monospace;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
}
</style>
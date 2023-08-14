<script setup>
import { ref, nextTick } from 'vue'

const maxQuestions = 5;

let hasEnded = ref(false);

const currentQuestionIndex = ref(0);
let correctAnswerIndex = ref();

let currentChoices = ref();
let areChoicesDisabled = ref(false);

let points = ref(0);

let quizes = ref();

await startNewQuizGame();

async function startNewQuizGame() {
  currentQuestionIndex.value = 0;
  quizes.value = await getNewQuizes();
  points.value = 0;

  let randomizedChoices = getRandomizedChoices(quizes.value[currentQuestionIndex.value]);
  currentChoices.value = randomizedChoices.choices;
  correctAnswerIndex.value = randomizedChoices.correctAnswerIndex;

  await nextTick();
  hasEnded.value = false;
}

async function getNewQuizes() {
  let response = await fetch('https://opentdb.com/api.php?amount=5&category=18&difficulty=easy&type=multiple');
  let quizes = (await response.json()).results;
  unescapeQuizes(quizes);
  return quizes;
}

function unescapeQuizes(quizes) {
  let domParser = new DOMParser();
  quizes.forEach(quiz => {
    quiz.question = domParser.parseFromString(quiz.question,'text/html').querySelector('html').textContent;
    quiz.correct_answer = domParser.parseFromString(quiz.correct_answer,'text/html').querySelector('html').textContent;
    quiz.incorrect_answers = quiz.incorrect_answers.map(incorrect_answer => {
      return domParser.parseFromString(incorrect_answer,'text/html').querySelector('html').textContent
    });
  });
}

function getRandomizedChoices(quiz) {
  let choices = quiz.incorrect_answers.slice(0, 3);
  let correctAnswerIndex = Math.floor(Math.random() * 4);
  choices.splice(correctAnswerIndex, 0, quizes.value[currentQuestionIndex.value].correct_answer);
  return {
    choices,
    correctAnswerIndex}
}

function chooseAnswer(index) {
  showCorrectAnswers();

  if (index == correctAnswerIndex.value) {
    points.value++;
  }

  setTimeout(() => {
    hideCorrectAnswers();
    if (currentQuestionIndex.value >= maxQuestions - 1) {
      endQuiz();
    } else {
      currentQuestionIndex.value++;

      let randomizedChoices = getRandomizedChoices(quizes.value[currentQuestionIndex.value]);
      currentChoices.value = randomizedChoices.choices;
      correctAnswerIndex.value = randomizedChoices.correctAnswerIndex;
    }
    
  }, 1000);
}

function showCorrectAnswers() {
  Array.from(document.getElementsByClassName("choice")).forEach((choice, i) => {
    if (i == correctAnswerIndex.value) {
      choice.style.backgroundColor = "green";
    } else {
      choice.style.backgroundColor = "red";
    };
    areChoicesDisabled.value = true;
  });
}

function endQuiz() {
  hasEnded.value = true;
}

function hideCorrectAnswers() {
  Array.from(document.getElementsByClassName("choice")).forEach((choice, i) => {
    choice.style.backgroundColor = "white";
    areChoicesDisabled.value = false;
  });
}

function getCurrentQuestion() {
  return quizes.value[currentQuestionIndex.value].question;
}
</script>

<template>
  <div class="d-flex flex-column flex-grow-1 justify-center align-center p-1 ">
    <div id="container">
      <div id="question">
        <span v-if="hasEnded">Congratulations, you answered {{points}}/5 questions correctly.</span>
        <span v-else>{{getCurrentQuestion()}}</span>
      </div>

      <v-btn v-if="hasEnded" @click="startNewQuizGame" role="button">Play 1 more time</v-btn>

      <v-row no-gutters v-else>
        <v-col
          v-for="n in 4"
          :key="n"
          cols="12"
          sm="3"
        >
          <div class="pa-2">
            <v-btn class="choice flex-wrap" block @click="chooseAnswer(n-1)" :class="{'disable-events': areChoicesDisabled}">
              <span style="white-space: normal;">
                {{currentChoices[n-1]}}
              </span>
            </v-btn>
          </div>
        </v-col>
      </v-row>
    </div>
  </div>
</template>

<style scoped>
#question {
  padding: 5px;
  display: flex;
  flex-wrap: wrap;
  flex-direction: column;
  overflow: auto;
  width: 100%;
  height: 30vh;
  font-size: 3em;
  font-weight: bold;
  color: #face09;
  text-shadow: 1px 1px 2px #000000;
  background-color: #FFFFFF55;
  border-radius: 5px;
}

#question span {
  margin: auto;
}

#container {
  width: 60%;
  font-family: 'Courier New', Courier, monospace;
  text-align: center;
}

.disable-events {
  pointer-events: none;
}

.choice {
  overflow-y: auto;
  overflow-x: hidden;
}
</style>
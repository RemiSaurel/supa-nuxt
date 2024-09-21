<script setup>
const client = useSupabaseClient();

const questions = ref([]);
const currentQuestionIndex = ref(0);
const userAnswers = ref([]);
const timeRemaining = ref(600);
const state = ref("questions");

async function getQuestions() {
  const { data, error } = await client.from("questions").select("*");
  if (error) {
    console.error(error);
  } else {
    questions.value = data;
  }
}

async function submitAnswers() {
  userAnswers.value.forEach(async (answer) => {
    const { error } = await client.from("answers").insert([
      {
        question_id: answer.question_id,
        answer: answer.answer,
      },
    ]);
    if (error) console.error(error);
  });

  state.value = "results";
  // Clear the timer
  localStorage.removeItem("start_time");
}

const calculateRemainingTime = () => {
  const startTime = localStorage.getItem("start_time");
  if (startTime) {
    const elapsedTime = Math.floor((Date.now() - parseInt(startTime)) / 1000);
    timeRemaining.value = Math.max(600 - elapsedTime, 0);
  } else {
    // Store the start time in localStorage
    localStorage.setItem("start_time", Date.now().toString());
    timeRemaining.value = 600;
  }
};

const startTimer = () => {
  calculateRemainingTime();

  const timerInterval = setInterval(() => {
    if (timeRemaining.value > 0) {
      timeRemaining.value--;
    } else {
      clearInterval(timerInterval);
      submitAnswers();
    }
  }, 1000);
};

function submitCurrentAnswer(answer) {
  userAnswers.value.push({
    question_id: questions.value[currentQuestionIndex.value].id,
    answer,
  });
  if (currentQuestionIndex.value < questions.value.length - 1) {
    currentQuestionIndex.value++;
  } else {
    submitAnswers();
  }
}

const timeFormatted = computed(() => {
  const minutes = Math.floor(timeRemaining.value / 60);
  const seconds = timeRemaining.value % 60;
  return `${minutes}:${seconds.toString().padStart(2, "0")}`;
});

onMounted(() => {
  getQuestions();
  startTimer();
});
</script>

<template>
  <UContainer class="max-w-3xl mt-8">
    <div v-if="state === 'questions'">
      <!-- Timer -->
      <div class="text-right text-lg font-bold mb-4">
        Temps restant : {{ timeFormatted }}
      </div>

      <div
        class="border border-gray-200 p-4 rounded"
        v-if="questions.length > 0"
      >
        <h3 class="font-bold">Question n° {{ currentQuestionIndex + 1 }}</h3>
        <p class="text-xl pt-2">
          {{ questions[currentQuestionIndex].question }}
        </p>

        <div
          v-if="questions[currentQuestionIndex].choices"
          class="flex gap-2 w-full pt-6"
        >
          <button
            v-for="(choice, index) in questions[currentQuestionIndex].choices"
            :key="index"
            @click="submitCurrentAnswer(choice)"
            class="bg-blue-500 text-white p-2 rounded flex-1"
          >
            {{ choice }}
          </button>
        </div>

        <div v-else>
          <textarea
            v-model="userAnswer"
            class="w-full p-2 border rounded"
          ></textarea>
          <button
            @click="submitCurrentAnswer(userAnswer)"
            class="bg-green-500 text-white p-2 mt-4 rounded"
          >
            Soumettre la réponse
          </button>
        </div>
      </div>

      <div v-else>
        <p>Chargement des questions...</p>
      </div>
    </div>
    <div class="text-center text-2xl pt-8" v-else>
      <h2>Merci d'avoir répondu à toutes les questions !</h2>
    </div>
  </UContainer>
</template>

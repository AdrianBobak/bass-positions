<template>
  <div class="container">
    <p>Position: {{ activePosition.position }}</p>
    <div v-if="isSubmitted">
      <p>{{ isCorrect ? 'Correct' : 'Incorrect' }}</p>
    </div>
    <BassNeck
      ref="bassNeck"
      :data="activePosition"
      :isSubmitted="isSubmitted"
      :isCorrectAnswerShown="isCorrectAnswerShown"
      @onSubmit="handleSubmit"
      @onShowCorrectAnswer="handleShowCorrectAnswer"
    >
      <button
        v-if="isSubmitted || isCorrectAnswerShown"
        class="control-btn"
        @click="handleNextPosition"
      >
        Next position
      </button>
    </BassNeck>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import BassNeck from './components/BassNeck.vue';
import positionsData from './data/positions.json';

const bassNeck = ref(null);
const isCorrect = ref(false);
const isSubmitted = ref(false);
const isCorrectAnswerShown = ref(false);
const completedPositions = ref([]);

const getRandomPostion = () => {
  let randomIndex = Math.floor(Math.random() * positionsData.length);
  while (completedPositions.value.includes(randomIndex)) {
    randomIndex = Math.floor(Math.random() * positionsData.length);
  }
  return positionsData[randomIndex];
};

const activePosition = ref(getRandomPostion());

const arePositionsEqual = (submittedPositions, expectedNotes) => {
  if (!submittedPositions || !expectedNotes) return false;

  const expectedStrings = Object.keys(expectedNotes);
  const submittedStrings = Object.keys(submittedPositions);

  if (expectedStrings.length !== submittedStrings.length) return false;

  for (const stringName of expectedStrings) {
    const submittedArray = submittedPositions[stringName] || [];
    const expectedArray = expectedNotes[stringName] || [];

    if (submittedArray.length !== expectedArray.length) return false;

    for (let i = 0; i < expectedArray.length; i += 1) {
      const submitted = submittedArray[i];
      const expected = expectedArray[i];

      if (
        !submitted ||
        submitted.fret !== expected.fret ||
        submitted.finger !== expected.finger ||
        submitted.tonicMinor !== expected.tonicMinor ||
        submitted.tonicMajor !== expected.tonicMajor
      ) {
        return false;
      }
    }
  }

  return true;
};

const handleCompletedPositions = () => {
  completedPositions.value.push(activePosition.value);
  if (completedPositions.value.length === positionsData.length) completedPositions.value = [];
};

const handleSubmit = (submittedPositions) => {
  isCorrect.value = arePositionsEqual(submittedPositions, activePosition.value?.notes);
  if (isCorrect.value) handleCompletedPositions();
  isSubmitted.value = true;
};

const handleShowCorrectAnswer = () => {
  isCorrectAnswerShown.value = true;
};

const handleNextPosition = () => {
  activePosition.value = getRandomPostion();
  isSubmitted.value = false;
  isCorrectAnswerShown.value = false;
  isCorrect.value = false;
  bassNeck.value?.onClearData();
};
</script>

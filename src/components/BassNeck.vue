<template>
  <div class="bass-neck-container">
    <div class="bass-neck">
      <div v-for="(string, stringName) in positions" :key="stringName" class="string-row">
        <div class="string-label">{{ stringName }}</div>
        <div class="frets-container">
          <div v-for="(button, index) in string" :key="button" class="fret-cell">
            <button
              class="note-btn"
              :class="{
                'note-btn--minor': button.tonicMinor,
                'note-btn--major': button.tonicMajor,
              }"
              :aria-label="`String ${stringName} Fret ${button.fret}`"
              @click="onFretClick(stringName, button.fret)"
            >
              <span v-if="button.finger > 0">{{ button.finger }}</span>
            </button>
            <div class="string-line"></div>
            <div v-if="index < frets - 1" class="fret-bar"></div>
          </div>
        </div>
      </div>
    </div>

    <div class="controls">
      <button
        class="control-btn minor-btn"
        :class="{ active: activeTonic === 'minor' }"
        @click="onToggleTonic('minor')"
      >
        Tonic Minor
      </button>
      <button
        class="control-btn major-btn"
        :class="{ active: activeTonic === 'major' }"
        @click="onToggleTonic('major')"
      >
        Tonic Major
      </button>
    </div>
    <div class="controls">
      <button v-if="!isSubmitted && !isCorrectAnswerShown" class="control-btn" @click="onSubmit">
        Submit
      </button>
      <button v-if="!isCorrectAnswerShown" class="control-btn" @click="onShowCorrectAnswer">
        Show correct answer
      </button>
      <slot></slot>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { getCleanedData } from '../helpers/getCleanedData';

const props = defineProps({
  data: {
    type: Object,
    default: () => {},
  },
  isSubmitted: {
    type: Boolean,
    default: false,
  },
  isCorrectAnswerShown: {
    type: Boolean,
    default: false,
  },
});

const emit = defineEmits(['onSubmit', 'onShowCorrectAnswer']);

const frets = ref(5);
const maxFingerValue = ref(4);
const positions = ref(getCleanedData());
const activeTonic = ref(null);

const onToggleTonic = (type) => {
  if (activeTonic.value === type) activeTonic.value = null;
  else activeTonic.value = type;
};

const onFretClick = (stringName, fret) => {
  const stringPositions = positions.value[stringName];

  const target = stringPositions.find((position) => position.fret === fret);

  target.tonicMinor = activeTonic.value === 'minor';
  target.tonicMajor = activeTonic.value === 'major';

  const nextFinger = target.finger + 1;
  if (nextFinger > maxFingerValue.value) target.finger = 0;
  else target.finger = nextFinger;
};

const onSubmit = () => {
  emit('onSubmit', positions.value);
};

const onShowCorrectAnswer = () => {
  positions.value = JSON.parse(JSON.stringify(props.data.notes));
  emit('onShowCorrectAnswer');
};

const onClearData = () => {
  positions.value = getCleanedData();
};

defineExpose({
  onClearData,
});
</script>

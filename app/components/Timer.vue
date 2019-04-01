<template>
  <GridLayout columns="100, 5, 100, 5,  100" rows="*">
    <ListPicker
      row="0"
      col="0"
      width="100"
      height="100"
      :items="numbers"
      selectedIndex="0"
      v-model="minutes"
      :isEnabled="!disabled"
    />
    <Label
      row="0"
      col="1"
      height="100"
      :text="':'"
    />
    <ListPicker
      row="0"
      col="2"
      width="100"
      height="100"
      :items="[0,1,2,3,4,5]"
      selectedIndex="0"
      v-model="firstSeconds"
      :isEnabled="!disabled"
    />
    <Label
      row="0"
      col="3"
      height="100"
      :text="':'"
    />
    <ListPicker
      row="0"
      col="4"
      width="100"
      height="100"
      :items="numbers"
      selectedIndex="0"
      v-model="secondSeconds"
      :isEnabled="!disabled"
    />
  </GridLayout>

</template>

<script>
import { TNSPlayer } from 'nativescript-audio';

export default {
  name: 'Timer',
  props: {
    value: {
      type: Number,
      default: 0,
    },
    index: {
      type: [Number, String],
      required: true,
    },
    running: {
      type: Boolean,
      default: false,
    },
    disabled: {
      type: Boolean,
      default: false,
    },
  },
  mounted() {
    this.player = new TNSPlayer();
    this.player.initFromFile({
      audioFile: '~/assets/sounds/beep.mp3',
      loop: false,
    });
  },
  destroyed() {
    this.player.dispose();
  },
  data() {
    return {
      numbers: [0,1,2,3,4,5,6,7,8,9],
      interval: null,
      player: null,
    };
  },
  computed: {
    internalValue: {
      get() {
        return this.value;
      },
      set(value) {
        this.$emit('input', value);
        if (!this.running && this.value > 0) {
          this.$emit('user-changed');
        }
      },
    },
    hasTimeRemaining() {
      return this.internalValue > 1;
    },
    underFiveSecondsLeft() {
      return this.internalValue <= 6;
    },
    minutes: {
      get() {
        return this.getPartOfTime(14, 2)
      },
      set(value) {
        this.internalValue = (value * 60) + this.seconds;
      },
    },
    firstSeconds: {
      get() {
        return this.getPartOfTime(17, 1)
      },
      set(value) {
        this.internalValue = (this.minutes * 60) + parseInt(`${value}${this.secondSeconds}`, 10);
      },
    },
    secondSeconds: {
      get() {
        return this.getPartOfTime(18, 1)
      },
      set(value) {
        this.internalValue = (this.minutes * 60) + parseInt(`${this.firstSeconds}${value}`, 10);
      },
    },
    seconds() {
      return parseInt(`${this.firstSeconds}${this.secondSeconds}`, 10);
    },
  },
  watch: {
    running() {
      this.startStop();
    },
  },
  methods: {
    startStop() {
      if (this.running) {
        this.runTimer();
      } else {
        clearInterval(this.interval);
      }
    },
    runTimer(){
      this.interval = setInterval(() => {
        this.internalValue = this.internalValue - 1;
        if (this.underFiveSecondsLeft && this.hasTimeRemaining) {
          this.warning();
        }
        if(!this.hasTimeRemaining) {
          this.alarm();
          clearInterval(this.interval);
          this.$emit('time-ended');
        }
      }, 1000);
    },
    alarm() {
      this.playFromStart();
    },
    warning() {
      this.playFromStart();
      setTimeout(() => {
        this.player.pause();
      }, 250);
    },
    getPartOfTime(start, take) {
      return parseInt(new Date(this.internalValue * 1000)
        .toISOString()
        .substr(start, take), 10);
    },
    playFromStart() {
      this.player.seekTo(0);
      this.player.resume();
    },
  },
};
</script>

<style scoped>
  Label {
    font-size: 14px;
    font-weight: bold;
    top: -30px;
    position: relative;
  }
</style>

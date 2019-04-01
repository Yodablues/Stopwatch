
<script src="http://localhost:8098"></script>

<template>
    <Page>
        <ActionBar title="Stopwatch"/>
        <ScrollView>
          <GridLayout
            columns="*"
            rows="50, 50, 50, *"
            >
            <Button
              col="0"
              row="0"
              text="Start/Stop"
              @tap="handleStartStopClicked"
            />
            <Button
              col="0"
              row="1"
              text="Add Interval"
              @tap="addInterval"
              :isEnabled="!running"
            />
            <GridLayout
              col="0"
              row="2"
              columns="100, *"
              rows="*"
            >
              <Label
                row="0"
                col="0"
                text="Repeat?"
              />
              <Switch
                row="0"
                col="1"
                v-model="repeat"
                :isEnabled="!running"
              />
            </GridLayout>
            <StackLayout
              col="0"
              row="3"
            >
              <GridLayout
                columns="300, 100"
                rows="*"
                v-for="(timer, index) in timers"
                :key="index"
                :ref="`timer_grid_${index}`"
              >
                <timer
                  col="0"
                  :index="timer.index"
                  :row="timer.index"
                  :ref="`timer_${index}`"
                  :running="timer.running"
                  :disabled="running"
                  v-model="timer.timeRemaining"
                  @time-ended="handleTimeEnded(index)"
                />
                <Button
                  col="1"
                  :row="timer.index"
                  text="Remove"
                  @tap="removeInterval(index)"
                  :isEnabled="!running"
                />
              </GridLayout>
            </StackLayout>
          </GridLayout>
        </ScrollView>
    </Page>
</template>

<script>
  import Timer from '@/components/Timer.vue';
  import Vue from 'vue';

  export default {
    components: {
      Timer,
    },
    data() {
      return {
        timers: [],
        savedTimers: [],
        running: false,
        repeat: false,
      };
    },
    methods: {
      addInterval(key) {
        this.timers.push({ index: this.timers.length, running: false, timeRemaining: 0 });
      },
      removeInterval(index) {
        this.timers.splice(index, 1);
      },
      handleStartStopClicked() {
        if (!this.hasTimeRemaining) {
          return false;
        }

        this.running = !this.running;
        if (this.running) {
          this.saveTimers();
        } else {
          this.resetTimers();
        }
        this.startStop();
      },
      startStop() {
        const index = 0;
        for (let index = 0; index < this.timers.length; index++) {
          if (this.timers[index].timeRemaining > 0) {
            this.timers[index].running = this.running;
            break;
          }
        }
      },
      handleTimeEnded(index) {
        this.timers[index].running = false;
        this.startStop();
        if (this.running && !this.hasTimeRemaining) {
          setTimeout(() => {
            this.resetTimers();
            if (this.repeat) {
              this.startStop();
            } else {
              this.running = false;
            }
          }, 1000);
        }
      },
      saveTimers() {
        this.savedTimers = JSON.parse(JSON.stringify(this.timers));
      },
      resetTimers() {
        this.timers = JSON.parse(JSON.stringify(this.savedTimers));
      },
    },
    computed: {
      rows() {
        const rows = [];
        for (let index = 0; index < this.timers.length; index++) {
          rows.push(50);
        }
        return rows.join(',');
      },
      totalTimeRemaining() {
        let totalTimeRemaining = 0;
        for (let index = 0; index < this.timers.length; index++) {
          totalTimeRemaining = totalTimeRemaining + this.timers[index].timeRemaining;
        }
        return totalTimeRemaining;
      },
      hasTimeRemaining() {
        return this.totalTimeRemaining > 0;
      },
    },
  }
</script>

<style scoped>
    ActionBar {
        background-color: #6d8fff;
        color: #ffffff;
    }

    .message {
        vertical-align: center;
        text-align: center;
        font-size: 20;
        color: #333333;
    }

    Label {
      font-size: 24px;
      font-weight: bold;
    }
</style>

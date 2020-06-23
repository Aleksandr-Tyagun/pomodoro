<template>
  <div
    class="container"
    :class="{
      'container--short-break': isShortBreak,
      'container--long-break': isLongBreak,
    }"
  >
    <transition name="component-fade" mode="out-in">
      <Menu
        v-if="isSettingsOpen"
        class="menu"
        :work-time="workTime"
        :short-break-time="shortBreak"
        :long-break-time="longBreak"
        @save-settings="saveSettings"
        @close-settings="isSettingsOpen = false"
      />
    </transition>
    <main class="main">
      <button
        class="main__close-button"
        @click="isSettingsOpen = !isSettingsOpen"
      />
      <div class="progress" :style="{ width: `${progressWidth}%` }"></div>
      <div class="main__title-buttons">
        <button
          class="main__title-button"
          :class="{ 'main__title-button--active': !isBreak }"
          @click="startWork(workTime)"
        >
          Work
        </button>
        <button
          class="main__title-button"
          :class="{
            'main__title-button--active': isShortBreak,
          }"
          @click="startBreak(shortBreak)"
        >
          Short Break
        </button>
        <button
          class="main__title-button"
          :class="{
            'main__title-button--active': isLongBreak,
          }"
          @click="startBreak(longBreak)"
        >
          Long Break
        </button>
      </div>
      <div class="main__title">
        {{ titleText }}
      </div>
      <div class="main__subtitle">
        <span>{{
          !isBreak ? `Next break is ${nextBreakTime} minutes` : ''
        }}</span>
      </div>
      <div class="main__timer">
        <div class="main__timer-numbers">{{ minutes }}</div>
        <div>:</div>
        <div class="main__timer-numbers">{{ seconds }}</div>
      </div>
      <div class="main__action-buttons">
        <button
          class="main__action-button"
          :class="{
            'main__action-button--short-break': isShortBreak,
            'main__action-button--long-break': isLongBreak,
          }"
          @click="handleTimer"
        >
          {{ buttonText }}
        </button>
        <button
          class="main__action-button"
          :class="{
            'main__action-button--short-break': isShortBreak,
            'main__action-button--long-break': isLongBreak,
          }"
          @click="resetTimer"
        >
          RESET
        </button>
      </div>
    </main>
  </div>
</template>

<script>
import Menu from '@/components/Menu'

export default {
  components: {
    Menu,
  },
  data() {
    return {
      timerId: null,
      timeLeft: 25 * 60,
      timeInterval: 3,
      resetButton: false,
      isStarted: false,
      breakCount: 0,
      isBreak: false,
      workTime: 25,
      shortBreak: 5,
      longBreak: 15,
      isSettingsOpen: false,
    }
  },
  computed: {
    minutes() {
      const minutes = String(Math.floor(this.timeLeft / 60))
      return minutes.padStart(2, 0)
    },
    seconds() {
      const seconds = String(this.timeLeft - this.minutes * 60)
      return seconds.padStart(2, 0)
    },
    buttonText() {
      return this.isStarted ? 'stop' : 'start'
    },
    progressWidth() {
      return (this.timeLeft / this.timeInterval) * 100
    },
    titleText() {
      if (this.isShortBreak) {
        return 'Take a break'
      }
      if (this.isLongBreak) {
        return 'Good job! Time to relax'
      }
      return 'Time to work!'
    },
    titleIcon() {
      if (this.isShortBreak) {
        return 'favicon-green.svg'
      }
      if (this.isLongBreak) {
        return 'favicon-blue.svg'
      }
      return 'favicon-red.svg'
    },
    nextBreakTime() {
      if (this.breakCount === 4) {
        return this.longBreak
      }

      return this.shortBreak
    },
    isShortBreak() {
      return this.isBreak && this.timeInterval === this.shortBreak * 60
    },
    isLongBreak() {
      return this.isBreak && this.timeInterval === this.longBreak * 60
    },
  },
  watch: {
    timeLeft(oldTime, newTime) {
      if (newTime === 1) {
        if (this.isBreak) {
          this.startWork(this.workTime)
          this.workSound()
        } else {
          this.breakCount++
          this.startBreak(
            this.breakCount === 5 ? this.longBreak : this.shortBreak
          )
          this.breakSound()
        }
      }
    },
    breakCount(oldCount, newCount) {
      if (newCount === 5) {
        this.breakCount = 1
      }
    },
  },
  created() {
    this.timeInterval = 25 * 60
  },
  methods: {
    saveSettings({ newWorkTime, newShortTime, newLongTime }) {
      this.workTime = newWorkTime
      this.shortBreak = newShortTime
      this.longBreak = newLongTime
      this.isSettingsOpen = false
      this.timeLeft = newWorkTime * 60
      this.timeInterval = newWorkTime * 60
      if (this.isStarted) {
        this.handleTimer()
      }
    },
    breakSound() {
      const audio = new Audio('/sounds/break-sound.mp3')
      audio.play()
    },
    workSound() {
      const audio = new Audio('/sounds/work-sound.mp3')
      audio.play()
    },
    startBreak(breakTime = 5) {
      this.handleTimer()
      this.isBreak = true
      this.timeLeft = breakTime * 60
      this.timeInterval = breakTime * 60
      this.handleTimer()
    },
    startWork(workTime = 25) {
      this.handleTimer()
      this.isBreak = false
      this.timeLeft = workTime * 60
      this.timeInterval = workTime * 60
      this.handleTimer()
    },
    handleTimer() {
      if (!this.isStarted) {
        this.timerId = setInterval(() => this.countDown(), 1000)
        this.isStarted = true
      } else {
        clearInterval(this.timerId)
        this.timerId = null
        this.isStarted = false
      }
    },
    resetTimer() {
      clearInterval(this.timerId)
      this.timerId = null
      this.isStarted = false
      this.timeLeft = this.timeInterval
    },
    countDown() {
      this.timeLeft--
    },
  },
  head() {
    return {
      title: `${this.minutes}:${this.seconds} - ${this.titleText}`,
      link: [
        {
          rel: 'icon',
          href: `/${this.titleIcon}`,
        },
      ],
    }
  },
}
</script>

<style lang="scss">
$work-color: #ff756b;
$short-break-color: #4bb462;
$long-break-color: #1771f1;

.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  background-color: #f85c50;
  transition: background-color 200ms;

  &--short-break {
    background-color: $short-break-color;
  }
  &--long-break {
    background-color: $long-break-color;
  }
}
.progress {
  position: absolute;
  top: 0;
  left: 0;
  height: 5px;
  background-color: white;
  width: 0;
  transition: width 200ms;
}

.menu {
  z-index: 2;
}

.main {
  position: relative;
  width: 600px;
  background-color: rgba(255, 255, 255, 0.2);
  border-radius: 5px;
  padding: 40px;
  color: white;
  overflow: hidden;

  &__close-button {
    position: absolute;
    top: 20px;
    right: 15px;
    width: 35px;
    height: 35px;
    border: transparent;
    background: url('/settings.svg') no-repeat;
    outline: none;
    cursor: pointer;
    transition: transform 200ms;

    &:hover {
      transform: rotate(30deg);
    }
  }

  &__timer {
    display: flex;
    font-size: 8em;
    font-weight: 700;
    justify-content: center;
  }

  &__title {
    margin-top: 25px;
    font-size: 32px;
    font-weight: 500;
  }

  &__subtitle {
    margin-top: 5px;
    font-size: 14px;
    font-weight: 200;
    height: 20px;
  }

  &__action-buttons {
    width: 350px;
    margin: 0 auto;
    padding-top: 20px;
    display: grid;
    grid-template-columns: 4fr 2fr;
    column-gap: 20px;
  }

  &__action-button {
    font-family: 'Roboto Condensed', sans-serif;
    margin-top: 20px;
    padding: 10px;
    font-size: 24px;
    background-color: white;
    border: 1px solid transparent;
    border-radius: 5px;
    color: $work-color;
    text-transform: uppercase;
    transition: box-shadow 200ms;
    outline: none;
    cursor: pointer;

    &:hover {
      box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1),
        0 4px 6px -2px rgba(0, 0, 0, 0.05);
    }

    &--short-break {
      color: $short-break-color;
    }
    &--long-break {
      color: $long-break-color;
    }
  }

  &__title-button {
    border: 2px solid transparent;
    border-radius: 5px;
    background-color: transparent;
    color: white;
    font-size: 16px;
    padding: 5px 10px;
    outline: none;
    transition: box-shadow 200ms, background-color 200ms;
    cursor: pointer;

    &:hover {
      background-color: rgba(255, 255, 255, 0.4);
    }

    &--active {
      background-color: rgba(255, 255, 255, 0.4);
    }
  }
}
.component-fade-enter-active,
.component-fade-leave-active {
  transition: opacity 0.3s ease;
}
.component-fade-enter,
.component-fade-leave-to {
  opacity: 0;
}
</style>

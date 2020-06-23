<template>
  <div>
    <div class="background"></div>
    <div class="settings">
      <button
        class="settings__close-button"
        @click="$emit('close-settings')"
      ></button>
      <h1 class="settings__title">Settings</h1>
      <ul class="settings__list">
        <li class="settings__item">
          <label for="work">
            <span class="settings__name">Work Time:</span>
          </label>
          <input
            v-model="setWorkTime"
            class="settings__input"
            min="1"
            max="60"
            type="number"
            name="work"
          />
          minutes
        </li>
        <li class="settings__item">
          <label for="short">
            <span class="settings__name">Short Break:</span>
          </label>
          <input
            v-model="setShortBreakTime"
            class="settings__input"
            min="1"
            max="60"
            type="number"
            name="short"
          />
          minutes
        </li>
        <li class="settings__item">
          <label for="long">
            <span class="settings__name">Long Break:</span>
          </label>
          <input
            v-model="setLongBreakTime"
            class="settings__input"
            min="1"
            max="60"
            type="number"
            name="long"
          />
          minutes
        </li>
      </ul>
      <button class="settings__button" @click="handleSave">
        Save
      </button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    workTime: {
      type: Number,
      default: 25,
    },
    shortBreakTime: {
      type: Number,
      default: 5,
    },
    longBreakTime: {
      type: Number,
      default: 15,
    },
  },
  data() {
    return {
      setWorkTime: this.workTime,
      setShortBreakTime: this.shortBreakTime,
      setLongBreakTime: this.longBreakTime,
    }
  },
  methods: {
    handleSave() {
      this.$emit('save-settings', {
        newWorkTime: +this.setWorkTime,
        newShortTime: +this.setShortBreakTime,
        newLongTime: +this.setLongBreakTime,
      })
    },
  },
}
</script>

<style lang="scss" scoped>
.background {
  position: absolute;
  height: 100vh;
  width: 100vw;
  top: 0;
  left: 0;
  background-color: rgba(black, 0.4);
  z-index: 5;
}

.settings {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translateX(-50%) translateY(-50%);
  height: 400px;
  width: 300px;
  background-color: white;
  z-index: 6;
  border-radius: 5px;
  padding: 15px;

  &__close-button {
    position: absolute;
    right: 10px;
    height: 20px;
    width: 20px;
    cursor: pointer;
    background: url('/close.svg') no-repeat;
    transition: transform 200ms;
    outline: none;
    border: 1px solid transparent;

    &:hover {
      transform: scale(1.1);
    }
  }

  &__list {
    display: flex;
    justify-content: space-between;
    list-style: none;
    font-size: 14px;
    margin: 0;
    padding: 20px;
  }

  &__item {
    display: flex;
    flex-direction: column;
    text-align: left;
  }

  &__input {
    margin-top: 5px;
    margin-bottom: 5px;
    font-size: 18px;
    width: 55px;
    border-color: transparent;
    outline: none;
    background-color: #efefef;
    border-radius: 5px;
  }

  &__button {
    background-color: rgb(83, 83, 83);
    color: white;
    border: 1px solid transparent;
    border-radius: 5px;
    padding: 5px;
    cursor: pointer;
    padding: 5px 20px;
    transition: background-color 200ms;

    &:hover {
      background-color: black;
    }
  }
}
</style>

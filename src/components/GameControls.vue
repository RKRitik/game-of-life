<template>
  <div class="container-controls" @mouseover="showControls" @mouseleave="hideControls">
    &nbsp;
    <transition>
      <div v-if="isControls" class="controls">
        <button @click="clearFn">Clear</button>
        <button @click="toggleFn">{{ paused ? 'Resume' : 'Pause' }}</button>
        <button @click="randomFn">Random</button>
        <button @click="spawnFn">Spawn Random Patterns</button>
      </div>
    </transition>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isControls: false
    };
  },
  props: {
    paused: {
      type: Boolean,
      required: true
    }
  },
  methods: {
    clearFn() {
      this.$emit('onClear');
    },
    toggleFn() {
      this.$emit('onToggle');
    },
    randomFn() {
      this.$emit('onRandom');
    },
    spawnFn() {
      this.$emit('onSpawnLife');
    },
    showControls() {
      this.isControls = true;
    },
    hideControls() {
      this.isControls = false;
    }
  }
};
</script>

<style scoped>
/* we will explain what these classes do next! */
.v-enter-active,
.v-leave-active {
  transition: opacity 0.5s ease;
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
}

.container-controls {
  position: absolute;
  top: 0%;
  left: 0;
  width: 20%;
  height: 100vh;
  z-index: 5;
}

.controls {
  position: fixed;
  top: 55%;
  transform: translateY(-50%);
  left: 10px;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  z-index: 2;
  /* background-color: white; */
  padding: 10px;
}

button {
  margin-bottom: 10px;
}
</style>

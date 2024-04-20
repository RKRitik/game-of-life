<template>
  <div class="controls">
    <!-- Your control buttons here -->
    <button @click="clearFn">Clear</button>
    <button @click="toggleFn">{{ paused ? 'Resume' : 'Pause' }}</button>
    <button @click="randomFn">Random</button>
    <button @click="spawnFn">Spawn Random Patterns</button>
  </div>
</template>

<script>
export default {
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
    }
  }
};
</script>

<style scoped>
.controls {
  position: fixed;
  top: 55%;
  transform: translateY(-50%);
  left: 10px;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}

button {
  margin-bottom: 10px;
}
</style>

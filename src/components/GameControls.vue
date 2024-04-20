<template>
  <div class="container">
    <div class="controls-shadow" @mouseover="showControls" @mouseleave="hideControls">&nbsp;</div>
    <div v-if="isControls" @mouseover="test" class="controls">
      <button @click="clearFn">Clear</button>
      <button @click="toggleFn">{{ paused ? 'Resume' : 'Pause' }}</button>
      <button @click="randomFn">Random</button>
      <button @click="spawnFn">Spawn Random Patterns</button>
    </div>
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
    test() {
      console.log('fvbbcbcvbcvbcvbc')
      alert('bruh')
    },
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
.shadow-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 30%;
  /* Adjust width as needed */
  height: 100vh;
  background: rgba(0, 0, 0, 0.2);
  /* Adjust opacity as needed */
  z-index: 1;
  /* Ensure overlay is above other content */
  display: none;
  /* Initially hidden */
}

.controls:hover .shadow-overlay {
  display: block;
  /* Show overlay on hover */
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
  /* Ensure it's above the shadow component */
  background-color: white;
  /* White background */
  padding: 10px;
  /* Add padding to the buttons */
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  /* Add shadow */
}

button {
  margin-bottom: 10px;
}
</style>

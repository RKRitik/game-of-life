<template>
  <div class="controls">
    <!-- Your control buttons here -->
    <button @click="executeFunction">Clear</button>
    <button>Resume</button>
    <button>Random</button>
    <button>Spawn Random Patterns</button>
  </div>
</template>

<script>
export default {
  props: {
    executeFunction: {
      type: Function,
      required: true
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

<template>
  <div id="app" class="min-h-screen bg-gray-50">
    <DriveCompressor />
  </div>
</template>

<script>
import DriveCompressor from './components/DriveCompressor.vue'

export default {
  name: 'App',
  components: {
    DriveCompressor
  }
}
</script>

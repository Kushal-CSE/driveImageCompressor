<template>
  <div class="bg-white rounded-lg shadow p-6">
    <h2 class="text-xl font-semibold text-gray-900 mb-4">Google Drive File Transfer</h2>

    <!-- Source Drive -->
    <div class="mb-6">
      <h3 class="text-lg font-medium text-gray-800 mb-2">1. Select Source Folder (Drive A)</h3>
      <FileExplorer
        :folders="sourceFolders"
        :currentPath="sourcePath"
        :currentFolders="sourceCurrentFolders"
        :selectedFolderId="sourceSelected"
        @refresh="fetchSourceFolders"
        @navigate-to-level="navigateSourceToLevel"
        @select-folder-highlight="highlightSource"
        @navigate-into-folder="navigateSourceInto"
        @select-folder="setSourceFolder"
        @go-back="goBackSource"
      />
    </div>

    <!-- Destination Drive -->
    <div class="mb-6">
      <h3 class="text-lg font-medium text-gray-800 mb-2">2. Select Destination Folder (Drive B)</h3>
      <FileExplorer
        :folders="destFolders"
        :currentPath="destPath"
        :currentFolders="destCurrentFolders"
        :selectedFolderId="destSelected"
        @refresh="fetchDestFolders"
        @navigate-to-level="navigateDestToLevel"
        @select-folder-highlight="highlightDest"
        @navigate-into-folder="navigateDestInto"
        @select-folder="setDestFolder"
        @go-back="goBackDest"
      />
    </div>

    <!-- Transfer Button -->
    <div class="text-center">
      <button
        :disabled="!sourceSelected || !destSelected || loading"
        @click="transferFiles"
        class="px-5 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition disabled:bg-gray-400"
      >
        {{ loading ? "Transferring..." : "Transfer Files" }}
      </button>
    </div>

    <!-- Status -->
    <div v-if="statusMessage" class="mt-4 text-center text-sm text-gray-700">
      {{ statusMessage }}
    </div>
  </div>
</template>

<script>
import FileExplorer from './FileExplorer.vue'

export default {
  name: 'FileTransfer',
  components: { FileExplorer },
  data() {
    return {
      // Drive A
      sourceFolders: [],
      sourcePath: [],
      sourceCurrentFolders: [],
      sourceSelected: null,

      // Drive B
      destFolders: [],
      destPath: [],
      destCurrentFolders: [],
      destSelected: null,

      // State
      loading: false,
      statusMessage: '',
    }
  },
  methods: {
    /* --------- Fetch Source Drive (A) Folders --------- */
    async fetchSourceFolders() {
      const res = await fetch('http://localhost:5000/driveA/folders')
      const data = await res.json()
      this.sourceFolders = data
      this.sourceCurrentFolders = this.sourceFolders.filter((f) => !f.parents)
    },
    navigateSourceToLevel(index) {
      this.sourcePath = this.sourcePath.slice(0, index + 1)
      this.sourceCurrentFolders = this.sourceFolders.filter(
        (f) => f.parents && f.parents[0] === this.sourcePath[index]?.id
      )
    },
    highlightSource(folder) {
      this.sourceSelected = folder.id
    },
    navigateSourceInto(folder) {
      this.sourcePath.push(folder)
      this.sourceCurrentFolders = this.sourceFolders.filter(
        (f) => f.parents && f.parents[0] === folder.id
      )
    },
    setSourceFolder(folderId) {
      this.sourceSelected = folderId
    },
    goBackSource() {
      this.sourcePath.pop()
      this.sourceCurrentFolders =
        this.sourcePath.length === 0
          ? this.sourceFolders.filter((f) => !f.parents)
          : this.sourceFolders.filter((f) => f.parents && f.parents[0] === this.sourcePath.at(-1).id)
    },

    /* --------- Fetch Destination Drive (B) Folders --------- */
    async fetchDestFolders() {
      const res = await fetch('http://localhost:5000/driveB/folders')
      const data = await res.json()
      this.destFolders = data
      this.destCurrentFolders = this.destFolders.filter((f) => !f.parents)
    },
    navigateDestToLevel(index) {
      this.destPath = this.destPath.slice(0, index + 1)
      this.destCurrentFolders = this.destFolders.filter(
        (f) => f.parents && f.parents[0] === this.destPath[index]?.id
      )
    },
    highlightDest(folder) {
      this.destSelected = folder.id
    },
    navigateDestInto(folder) {
      this.destPath.push(folder)
      this.destCurrentFolders = this.destFolders.filter(
        (f) => f.parents && f.parents[0] === folder.id
      )
    },
    setDestFolder(folderId) {
      this.destSelected = folderId
    },
    goBackDest() {
      this.destPath.pop()
      this.destCurrentFolders =
        this.destPath.length === 0
          ? this.destFolders.filter((f) => !f.parents)
          : this.destFolders.filter((f) => f.parents && f.parents[0] === this.destPath.at(-1).id)
    },

    /* --------- Transfer Files --------- */
    async transferFiles() {
      this.loading = true
      this.statusMessage = 'Starting transfer...'

      try {
        const res = await fetch('http://localhost:5000/transfer', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            sourceFolderId: this.sourceSelected,
            destFolderId: this.destSelected,
          }),
        })

        const data = await res.json()
        this.statusMessage = data.message || 'Transfer completed!'
      } catch (err) {
        console.error(err)
        this.statusMessage = 'Error: Transfer failed.'
      } finally {
        this.loading = false
      }
    },
  },
  mounted() {
    this.fetchSourceFolders()
    this.fetchDestFolders()
  },
}
</script>

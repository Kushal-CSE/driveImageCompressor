<template>
  <div class="bg-white rounded-lg shadow p-6">
    <!-- Page 1: Source Drive Login -->
    <div v-if="currentPage === 1" class="text-center">
      <h2 class="text-2xl font-semibold text-gray-900 mb-6">File Transfer Setup</h2>
      <div class="max-w-md mx-auto">
        <div class="mb-6">
          <div class="text-6xl mb-4">🔗</div>
          <h3 class="text-xl font-medium text-gray-800 mb-2">Connect Source Drive</h3>
          <p class="text-gray-600 mb-6">
            First, connect to the Google Drive account that contains the files you want to transfer.
          </p>
        </div>
        
        <button
          @click="connectSourceDrive"
          :disabled="loading"
          class="w-full bg-blue-600 hover:bg-blue-700 text-white font-semibold py-3 px-6 rounded-lg flex items-center justify-center transition duration-200 disabled:bg-gray-400"
        >
          <svg v-if="loading" class="animate-spin -ml-1 mr-3 h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
          </svg>
          <svg v-else class="w-5 h-5 mr-2" viewBox="0 0 24 24" fill="currentColor">
            <path d="M22.56 12.25c0-.78-.07-1.53-.2-2.25H12v4.26h5.92c-.26 1.37-1.04 2.53-2.21 3.31v2.77h3.57c2.08-1.92 3.28-4.74 3.28-8.09z"/>
            <path d="M12 23c2.97 0 5.46-.98 7.28-2.66l-3.57-2.77c-.98.66-2.23 1.06-3.71 1.06-2.86 0-5.29-1.93-6.16-4.53H2.18v2.84C3.99 20.53 7.7 23 12 23z"/>
            <path d="M5.84 14.09c-.22-.66-.35-1.36-.35-2.09s.13-1.43.35-2.09V7.07H2.18C1.43 8.55 1 10.22 1 12s.43 3.45 1.18 4.93l2.85-2.22.81-.62z"/>
            <path d="M12 5.38c1.62 0 3.06.56 4.21 1.64l3.15-3.15C17.45 2.09 14.97 1 12 1 7.7 1 3.99 3.47 2.18 7.07l3.66 2.84c.87-2.6 3.3-4.53 6.16-4.53z"/>
          </svg>
          {{ loading ? 'Connecting...' : 'Connect Source Drive' }}
        </button>
        
        <div v-if="sourceConnected" class="mt-4 p-3 bg-green-50 border border-green-200 rounded-lg">
          <div class="flex items-center">
            <svg class="w-5 h-5 text-green-600 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"></path>
            </svg>
            <span class="text-green-800 font-medium">Source Drive Connected</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Page 2: Destination Drive Login -->
    <div v-if="currentPage === 2" class="text-center">
      <h2 class="text-2xl font-semibold text-gray-900 mb-6">Connect Destination Drive</h2>
      <div class="max-w-md mx-auto">
        <div class="mb-6">
          <div class="text-6xl mb-4">🎯</div>
          <h3 class="text-xl font-medium text-gray-800 mb-2">Connect Destination Drive</h3>
          <p class="text-gray-600 mb-6">
            Now connect to the Google Drive account where you want to transfer the files.
          </p>
        </div>
        
        <button
          @click="connectDestinationDrive"
          :disabled="loading"
          class="w-full bg-green-600 hover:bg-green-700 text-white font-semibold py-3 px-6 rounded-lg flex items-center justify-center transition duration-200 disabled:bg-gray-400"
        >
          <svg v-if="loading" class="animate-spin -ml-1 mr-3 h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
          </svg>
          <svg v-else class="w-5 h-5 mr-2" viewBox="0 0 24 24" fill="currentColor">
            <path d="M22.56 12.25c0-.78-.07-1.53-.2-2.25H12v4.26h5.92c-.26 1.37-1.04 2.53-2.21 3.31v2.77h3.57c2.08-1.92 3.28-4.74 3.28-8.09z"/>
            <path d="M12 23c2.97 0 5.46-.98 7.28-2.66l-3.57-2.77c-.98.66-2.23 1.06-3.71 1.06-2.86 0-5.29-1.93-6.16-4.53H2.18v2.84C3.99 20.53 7.7 23 12 23z"/>
            <path d="M5.84 14.09c-.22-.66-.35-1.36-.35-2.09s.13-1.43.35-2.09V7.07H2.18C1.43 8.55 1 10.22 1 12s.43 3.45 1.18 4.93l2.85-2.22.81-.62z"/>
            <path d="M12 5.38c1.62 0 3.06.56 4.21 1.64l3.15-3.15C17.45 2.09 14.97 1 12 1 7.7 1 3.99 3.47 2.18 7.07l3.66 2.84c.87-2.6 3.3-4.53 6.16-4.53z"/>
          </svg>
          {{ loading ? 'Connecting...' : 'Connect Destination Drive' }}
        </button>
        
        <div v-if="destinationConnected" class="mt-4 p-3 bg-green-50 border border-green-200 rounded-lg">
          <div class="flex items-center">
            <svg class="w-5 h-5 text-green-600 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"></path>
            </svg>
            <span class="text-green-800 font-medium">Destination Drive Connected</span>
          </div>
        </div>

        <div class="mt-6 pt-4 border-t border-gray-200">
          <button
            @click="goToPreviousPage"
            class="text-blue-600 hover:text-blue-800 text-sm font-medium"
          >
            ← Back to Source Drive
          </button>
        </div>
      </div>
    </div>

    <!-- Page 3: File Transfer Interface -->
    <div v-if="currentPage === 3">
      <div class="flex items-center justify-between mb-6">
        <h2 class="text-2xl font-semibold text-gray-900">File Transfer</h2>
        <button
          @click="goToPreviousPage"
          class="text-blue-600 hover:text-blue-800 text-sm font-medium"
        >
          ← Back to Destination Setup
        </button>
      </div>

      <!-- Two File Explorers Side by Side -->
      <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-6">
        <!-- Source Drive Explorer -->
        <div class="border rounded-lg">
          <div class="bg-blue-50 px-4 py-3 border-b">
            <h3 class="font-semibold text-blue-900 flex items-center">
              <svg class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"></path>
              </svg>
              Source Drive (Select files to transfer)
            </h3>
          </div>
          <div class="p-4">
            <FileExplorer
              :folders="sourceFolders"
              :currentPath="sourcePath"
              :currentFolders="sourceCurrentFolders"
              :selectedFolderId="sourceSelectedFolderId"
              @refresh="fetchSourceFolders"
              @navigate-to-level="navigateSourceToLevel"
              @select-folder-highlight="highlightSourceFolder"
              @navigate-into-folder="navigateSourceIntoFolder"
              @select-folder="selectSourceFolder"
              @go-back="goBackSource"
            />
          </div>
        </div>

        <!-- Destination Drive Explorer -->
        <div class="border rounded-lg">
          <div class="bg-green-50 px-4 py-3 border-b">
            <h3 class="font-semibold text-green-900 flex items-center">
              <svg class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 7v10a2 2 0 002 2h14a2 2 0 002-2V9a2 2 0 00-2-2H5a2 2 0 00-2-2z"></path>
              </svg>
              Destination Drive (Select target folder)
            </h3>
          </div>
          <div class="p-4">
            <FileExplorer
              :folders="destinationFolders"
              :currentPath="destinationPath"
              :currentFolders="destinationCurrentFolders"
              :selectedFolderId="destinationSelectedFolderId"
              @refresh="fetchDestinationFolders"
              @navigate-to-level="navigateDestinationToLevel"
              @select-folder-highlight="highlightDestinationFolder"
              @navigate-into-folder="navigateDestinationIntoFolder"
              @select-folder="selectDestinationFolder"
              @go-back="goBackDestination"
            />
          </div>
        </div>
      </div>

      <!-- Transfer Section -->
      <div class="bg-gray-50 rounded-lg p-6">
        <div class="text-center">
          <h3 class="text-lg font-semibold text-gray-900 mb-4">Ready to Transfer</h3>
          
          <!-- Transfer Status -->
          <div v-if="sourceSelectedFolderId && destinationSelectedFolderId" class="mb-4 p-4 bg-white rounded-lg border">
            <div class="flex items-center justify-center space-x-4 text-sm">
              <div class="flex items-center text-blue-600">
                <svg class="w-4 h-4 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"></path>
                </svg>
                Source: {{ getSelectedSourceFolderName() }}
              </div>
              <svg class="w-6 h-6 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 8l4 4m0 0l-4 4m4-4H3"></path>
              </svg>
              <div class="flex items-center text-green-600">
                <svg class="w-4 h-4 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 7v10a2 2 0 002 2h14a2 2 0 002-2V9a2 2 0 00-2-2H5a2 2 0 00-2-2z"></path>
                </svg>
                Destination: {{ getSelectedDestinationFolderName() }}
              </div>
            </div>
          </div>

          <!-- Transfer Button -->
          <button
            @click="transfer"
            :disabled="!sourceSelectedFolderId || !destinationSelectedFolderId || transferring"
            class="px-8 py-3 bg-purple-600 hover:bg-purple-700 text-white font-semibold rounded-lg transition duration-200 disabled:bg-gray-400 disabled:cursor-not-allowed flex items-center justify-center mx-auto"
          >
            <svg v-if="transferring" class="animate-spin -ml-1 mr-3 h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
              <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
              <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
            </svg>
            <svg v-else class="w-5 h-5 mr-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7H5a2 2 0 00-2 2v9a2 2 0 002 2h14a2 2 0 002-2V9a2 2 0 00-2-2h-3m-1 4l-3-3m0 0l-3 3m3-3v12"></path>
            </svg>
            {{ transferring ? 'Transferring Files...' : 'Start Transfer' }}
          </button>

          <!-- Status Message -->
          <div v-if="statusMessage" class="mt-4 p-3 rounded-lg" :class="statusMessage.includes('Error') ? 'bg-red-50 text-red-800 border border-red-200' : 'bg-blue-50 text-blue-800 border border-blue-200'">
            {{ statusMessage }}
          </div>
        </div>
      </div>
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
      // Navigation
      currentPage: 1,
      
      // Connection Status
      sourceConnected: false,
      destinationConnected: false,
      loading: false,
      transferring: false,
      statusMessage: '',

      // Source Drive Data
      sourceFolders: [],
      sourcePath: [],
      sourceCurrentFolders: [],
      sourceSelectedFolderId: null,
      sourceFolderHierarchy: new Map(),

      // Destination Drive Data
      destinationFolders: [],
      destinationPath: [],
      destinationCurrentFolders: [],
      destinationSelectedFolderId: null,
      destinationFolderHierarchy: new Map(),
    }
  },
  computed: {
    sourceCurrentFolders() {
      return this.sourceFolderHierarchy.get(this.getCurrentSourceFolderId()) || []
    },
    destinationCurrentFolders() {
      return this.destinationFolderHierarchy.get(this.getCurrentDestinationFolderId()) || []
    }
  },
  methods: {
    // Navigation Methods
    async connectSourceDrive() {
      this.loading = true
      try {
        // Simulate API call to connect source drive
        await this.simulateConnection()
        await this.fetchSourceFolders()
        this.sourceConnected = true
        this.currentPage = 2
      } catch (error) {
        this.statusMessage = 'Error: Failed to connect to source drive'
      } finally {
        this.loading = false
      }
    },

    async connectDestinationDrive() {
      this.loading = true
      try {
        // Simulate API call to connect destination drive
        await this.simulateConnection()
        await this.fetchDestinationFolders()
        this.destinationConnected = true
        this.currentPage = 3
      } catch (error) {
        this.statusMessage = 'Error: Failed to connect to destination drive'
      } finally {
        this.loading = false
      }
    },

    goToPreviousPage() {
      if (this.currentPage > 1) {
        this.currentPage--
      }
    },

    // Utility Methods
    async simulateConnection() {
      // Simulate network delay
      return new Promise(resolve => setTimeout(resolve, 1500))
    },

    getCurrentSourceFolderId() {
      return this.sourcePath.length > 0 ? this.sourcePath[this.sourcePath.length - 1].id : null
    },

    getCurrentDestinationFolderId() {
      return this.destinationPath.length > 0 ? this.destinationPath[this.destinationPath.length - 1].id : null
    },

    getSelectedSourceFolderName() {
      const folder = this.sourceFolders.find(f => f.id === this.sourceSelectedFolderId)
      return folder ? folder.name : 'None selected'
    },

    getSelectedDestinationFolderName() {
      const folder = this.destinationFolders.find(f => f.id === this.destinationSelectedFolderId)
      return folder ? folder.name : 'None selected'
    },

    // Source Drive Methods
    async fetchSourceFolders() {
      try {
        // Mock data for source drive
        this.sourceFolders = [
          { id: 'src1', name: 'Documents', parents: null },
          { id: 'src2', name: 'Photos', parents: null },
          { id: 'src3', name: 'Work Files', parents: null },
          { id: 'src4', name: 'Vacation 2023', parents: ['src2'] },
          { id: 'src5', name: 'Family Photos', parents: ['src2'] },
          { id: 'src6', name: 'Projects', parents: ['src3'] },
        ]
        this.organizeFolderHierarchy('source')
      } catch (error) {
        this.statusMessage = 'Error: Failed to fetch source folders'
      }
    },

    navigateSourceToLevel(level) {
      if (level === -1) {
        this.sourcePath = []
      } else if (level < this.sourcePath.length) {
        this.sourcePath = this.sourcePath.slice(0, level + 1)
      }
    },

    highlightSourceFolder(folder) {
      this.sourceSelectedFolderId = folder.id
    },

    navigateSourceIntoFolder(folder) {
      if (this.hasSubfolders(folder.id, 'source')) {
        this.sourcePath.push({
          id: folder.id,
          name: folder.name
        })
      }
    },

    selectSourceFolder(folderId) {
      this.sourceSelectedFolderId = folderId
    },

    goBackSource() {
      if (this.sourcePath.length > 0) {
        this.sourcePath.pop()
      }
    },

    // Destination Drive Methods
    async fetchDestinationFolders() {
      try {
        // Mock data for destination drive
        this.destinationFolders = [
          { id: 'dest1', name: 'Backup', parents: null },
          { id: 'dest2', name: 'Archive', parents: null },
          { id: 'dest3', name: 'Shared', parents: null },
          { id: 'dest4', name: '2024 Backup', parents: ['dest1'] },
          { id: 'dest5', name: 'Old Files', parents: ['dest2'] },
        ]
        this.organizeFolderHierarchy('destination')
      } catch (error) {
        this.statusMessage = 'Error: Failed to fetch destination folders'
      }
    },

    navigateDestinationToLevel(level) {
      if (level === -1) {
        this.destinationPath = []
      } else if (level < this.destinationPath.length) {
        this.destinationPath = this.destinationPath.slice(0, level + 1)
      }
    },

    highlightDestinationFolder(folder) {
      this.destinationSelectedFolderId = folder.id
    },

    navigateDestinationIntoFolder(folder) {
      if (this.hasSubfolders(folder.id, 'destination')) {
        this.destinationPath.push({
          id: folder.id,
          name: folder.name
        })
      }
    },

    selectDestinationFolder(folderId) {
      this.destinationSelectedFolderId = folderId
    },

    goBackDestination() {
      if (this.destinationPath.length > 0) {
        this.destinationPath.pop()
      }
    },

    // Folder Organization
    organizeFolderHierarchy(type) {
      const folders = type === 'source' ? this.sourceFolders : this.destinationFolders
      const hierarchy = type === 'source' ? this.sourceFolderHierarchy : this.destinationFolderHierarchy
      
      hierarchy.clear()

      folders.forEach(folder => {
        const parentId = folder.parents && folder.parents.length > 0 ? folder.parents[0] : null
        
        if (!hierarchy.has(parentId)) {
          hierarchy.set(parentId, [])
        }
        
        hierarchy.get(parentId).push(folder)
      })

      // Sort folders alphabetically
      hierarchy.forEach(folders => {
        folders.sort((a, b) => a.name.localeCompare(b.name))
      })
    },

    hasSubfolders(folderId, type) {
      const hierarchy = type === 'source' ? this.sourceFolderHierarchy : this.destinationFolderHierarchy
      return hierarchy.has(folderId) && hierarchy.get(folderId).length > 0
    },

    // Transfer Method
    async transfer() {
      if (!this.sourceSelectedFolderId || !this.destinationSelectedFolderId) {
        this.statusMessage = 'Error: Please select both source and destination folders'
        return
      }

      this.transferring = true
      this.statusMessage = 'Starting transfer...'

      try {
        // Simulate transfer process
        await new Promise(resolve => setTimeout(resolve, 3000))
        
        const sourceFolder = this.getSelectedSourceFolderName()
        const destFolder = this.getSelectedDestinationFolderName()
        
        this.statusMessage = `Transfer completed successfully! Files from "${sourceFolder}" have been transferred to "${destFolder}".`
      } catch (error) {
        this.statusMessage = 'Error: Transfer failed. Please try again.'
      } finally {
        this.transferring = false
      }
    }
  }
}
</script>
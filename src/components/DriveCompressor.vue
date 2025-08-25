<template>
  <div>
    <!-- Loading Screen -->
    <div v-if="loading" class="fixed inset-0 bg-white flex items-center justify-center z-50">
      <div class="text-center">
        <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600 mx-auto"></div>
        <p class="mt-4 text-gray-600">{{ loadingMessage }}</p>
      </div>
    </div>

    <!-- Login Screen -->
    <div v-if="currentScreen === 'login'" class="min-h-screen flex items-center justify-center p-4">
      <div class="max-w-md w-full bg-white rounded-lg shadow-lg p-8">
        <div class="text-center">
          <h1 class="text-3xl font-bold text-gray-900 mb-4">Drive Image Compressor</h1>
          <p class="text-gray-600 mb-8">
            Compress your Google Drive images efficiently and automatically organize them in
            folders.
          </p>

          <div class="grid grid-cols-1 gap-4 mb-8">
            <div class="text-center p-4 bg-gray-50 rounded-lg">
              <div class="text-2xl mb-2">🗂️</div>
              <h3 class="font-semibold mb-1">Folder Selection</h3>
              <p class="text-sm text-gray-600">
                Browse and select any folder from your Google Drive
              </p>
            </div>
            <div class="text-center p-4 bg-gray-50 rounded-lg">
              <div class="text-2xl mb-2">🔧</div>
              <h3 class="font-semibold mb-1">Smart Compression</h3>
              <p class="text-sm text-gray-600">Reduce file sizes while maintaining image quality</p>
            </div>
            <div class="text-center p-4 bg-gray-50 rounded-lg">
              <div class="text-2xl mb-2">📁</div>
              <h3 class="font-semibold mb-1">Auto Organization</h3>
              <p class="text-sm text-gray-600">
                Creates organized "crushed" subfolders automatically
              </p>
            </div>
          </div>

          <button
            @click="signInWithGoogle"
            class="w-full bg-blue-600 hover:bg-blue-700 text-white font-semibold py-3 px-6 rounded-lg flex items-center justify-center transition duration-200"
          >
            <svg class="w-5 h-5 mr-2" viewBox="0 0 24 24" fill="currentColor">
              <path
                d="M22.56 12.25c0-.78-.07-1.53-.2-2.25H12v4.26h5.92c-.26 1.37-1.04 2.53-2.21 3.31v2.77h3.57c2.08-1.92 3.28-4.74 3.28-8.09z"
              />
              <path
                d="M12 23c2.97 0 5.46-.98 7.28-2.66l-3.57-2.77c-.98.66-2.23 1.06-3.71 1.06-2.86 0-5.29-1.93-6.16-4.53H2.18v2.84C3.99 20.53 7.7 23 12 23z"
              />
              <path
                d="M5.84 14.09c-.22-.66-.35-1.36-.35-2.09s.13-1.43.35-2.09V7.07H2.18C1.43 8.55 1 10.22 1 12s.43 3.45 1.18 4.93l2.85-2.22.81-.62z"
              />
              <path
                d="M12 5.38c1.62 0 3.06.56 4.21 1.64l3.15-3.15C17.45 2.09 14.97 1 12 1 7.7 1 3.99 3.47 2.18 7.07l3.66 2.84c.87-2.6 3.3-4.53 6.16-4.53z"
              />
            </svg>
            Sign in with Google
          </button>

          <p class="text-sm text-gray-500 mt-4">
            By signing in, you agree to allow this app to access your Google Drive files for
            compression purposes only.
          </p>
        </div>
      </div>
    </div>

    <!-- Dashboard Screen -->
    <div v-if="currentScreen === 'dashboard'" class="min-h-screen bg-gray-100">
      <!-- Header -->
      <header class="bg-white shadow-sm border-b">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <div class="flex justify-between items-center h-16">
            <h1 class="text-xl font-semibold text-gray-900">Drive Image Compressor</h1>
            <div class="flex items-center space-x-4">
              <span v-if="userInfo" class="text-gray-600">{{ userInfo.name }}</span>
              <button
                @click="signOut"
                class="px-4 py-2 border border-gray-300 rounded-md text-sm font-medium text-gray-700 bg-white hover:bg-gray-50 transition duration-200"
              >
                Logout
              </button>
            </div>
          </div>
        </div>
      </header>

      <!-- Main Content -->
      <main class="max-w-7xl mx-auto p-4 sm:p-6 lg:p-8">
        <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
          <!-- File Explorer Panel -->
          <div class="lg:col-span-2">
            <FileExplorer
              :folders="folders"
              :current-path="currentPath"
              :current-folders="currentFolders"
              :selected-folder-id="selectedFolderId"
              @refresh="loadDriveFolders"
              @navigate-to-level="navigateToLevel"
              @select-folder-highlight="selectFolderForHighlight"
              @navigate-into-folder="navigateIntoFolder"
              @select-folder="selectFolder"
              @go-back="goBack"
            />
          </div>

          <!-- Settings Panel -->
          <div class="space-y-6">
            <!-- Compression Settings -->
            <div class="bg-white rounded-lg shadow p-6">
              <h2 class="text-lg font-semibold text-gray-900 mb-4">Compression Settings</h2>

              <div class="space-y-4">
                <div>
                  <label class="block text-sm font-medium text-gray-700 mb-2">
                    Image Quality: {{ Math.round(settings.compressionQuality * 100) }}%
                  </label>
                  <input
                    type="range"
                    v-model="settings.compressionQuality"
                    min="0.1"
                    max="1"
                    step="0.1"
                    class="w-full h-2 bg-gray-200 rounded-lg cursor-pointer"
                  />
                </div>

                <div>
                  <label class="block text-sm font-medium text-gray-700 mb-2">Max Width (px)</label>
                  <input
                    type="number"
                    v-model="settings.maxWidth"
                    min="100"
                    max="4000"
                    class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                  />
                </div>

                <div>
                  <label class="block text-sm font-medium text-gray-700 mb-2"
                    >Max Height (px)</label
                  >
                  <input
                    type="number"
                    v-model="settings.maxHeight"
                    min="100"
                    max="4000"
                    class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                  />
                </div>

                <div>
                  <label class="block text-sm font-medium text-gray-700 mb-2"
                    >Compressed Folder Name</label
                  >
                  <input
                    type="text"
                    v-model="settings.subfolderName"
                    placeholder="crushed"
                    class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
                  />
                </div>

                <button
                  @click="startCompression"
                  :disabled="!selectedFolder || images.length === 0 || processing"
                  class="w-full bg-blue-600 text-white py-2 px-4 rounded-md hover:bg-blue-700 disabled:bg-gray-400 disabled:cursor-not-allowed transition duration-200"
                >
                  Start Compression
                </button>
              </div>
            </div>

            <!-- Selected Folder Info -->
            <div v-if="selectedFolder && images.length > 0" class="bg-white rounded-lg shadow p-6">
              <h3 class="text-lg font-semibold text-gray-900 mb-2">{{ selectedFolder.name }}</h3>
              <div class="space-y-2 text-sm text-gray-600">
                <div>
                  Images found: <strong class="text-gray-900">{{ images.length }}</strong>
                </div>
                <div>
                  Total size: <strong class="text-gray-900">{{ formatFileSize(totalSize) }}</strong>
                </div>
              </div>
            </div>
          </div>
        </div>
      </main>
    </div>

    <!-- Processing Screen -->
    <div
      v-if="currentScreen === 'processing'"
      class="min-h-screen bg-gray-100 flex items-center justify-center p-4"
    >
      <div class="max-w-2xl w-full bg-white rounded-lg shadow-lg p-8">
        <div class="text-center mb-8">
          <h2 class="text-2xl font-bold text-gray-900 mb-2">Compressing Images</h2>
          <div class="flex justify-between items-center text-sm text-gray-600">
            <span>{{ progressText }}</span>
            <span class="font-semibold">{{ Math.round(progress) }}%</span>
          </div>
        </div>

        <!-- Progress Bar -->
        <div class="w-full bg-gray-200 rounded-full h-3 mb-6">
          <div
            class="bg-blue-600 h-3 rounded-full transition-all duration-300"
            :style="{ width: progress + '%' }"
          ></div>
        </div>

        <!-- Current Image -->
        <div v-if="currentImageName" class="mb-6 p-4 bg-gray-50 rounded-lg">
          <h4 class="font-semibold mb-2">Currently processing:</h4>
          <p class="text-sm text-gray-600">{{ currentImageName }}</p>
        </div>

        <!-- Controls -->
        <div class="flex justify-center space-x-4">
          <button
            @click="togglePause"
            :disabled="!processing"
            class="px-6 py-2 bg-gray-600 text-white rounded-md hover:bg-gray-700 disabled:bg-gray-400 transition duration-200"
          >
            {{ paused ? 'Resume' : 'Pause' }}
          </button>
          <button
            @click="cancelProcessing"
            :disabled="!processing"
            class="px-6 py-2 border border-gray-300 rounded-md text-gray-700 hover:bg-gray-50 disabled:text-gray-400 disabled:cursor-not-allowed transition duration-200"
          >
            Cancel
          </button>
        </div>
      </div>
    </div>

    <!-- Results Screen -->
    <div
      v-if="currentScreen === 'results'"
      class="min-h-screen bg-gray-100 flex items-center justify-center p-4"
    >
      <div class="max-w-4xl w-full bg-white rounded-lg shadow-lg p-8">
        <div class="text-center mb-8">
          <h2 class="text-2xl font-bold text-gray-900 mb-4">Compression Complete!</h2>
        </div>

        <!-- Stats -->
        <div class="grid grid-cols-2 md:grid-cols-4 gap-6 mb-8">
          <div class="text-center p-4 bg-gray-50 rounded-lg">
            <h3 class="text-2xl font-bold text-blue-600">{{ results.processed }}</h3>
            <p class="text-gray-600">Images Processed</p>
          </div>
          <div class="text-center p-4 bg-gray-50 rounded-lg">
            <h3 class="text-2xl font-bold text-green-600">
              {{ formatFileSize(results.totalSaved) }}
            </h3>
            <p class="text-gray-600">Space Saved</p>
          </div>
          <div class="text-center p-4 bg-gray-50 rounded-lg">
            <h3 class="text-2xl font-bold text-purple-600">{{ compressionRatio }}%</h3>
            <p class="text-gray-600">Average Compression</p>
          </div>
          <div class="text-center p-4 bg-gray-50 rounded-lg">
            <h3 class="text-2xl font-bold text-orange-600">{{ processingTime }}</h3>
            <p class="text-gray-600">Total Time</p>
          </div>
        </div>

        <!-- Errors -->
        <div
          v-if="results.errors.length > 0"
          class="mb-8 p-4 bg-red-50 border border-red-200 rounded-lg"
        >
          <h4 class="font-semibold text-red-800 mb-2">Issues Encountered:</h4>
          <ul class="text-sm text-red-700 space-y-1">
            <li v-for="error in results.errors" :key="error.fileName">
              <strong>{{ error.fileName }}</strong
              >: {{ error.error }}
            </li>
          </ul>
        </div>

        <!-- Actions -->
        <div class="flex justify-center space-x-4">
          <button
            @click="resetApp"
            class="px-6 py-3 bg-blue-600 text-white rounded-md hover:bg-blue-700 transition duration-200"
          >
            Process Another Folder
          </button>
        </div>
      </div>
    </div>

    <!-- Error Modal -->
    <div
      v-if="showErrorModal"
      class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4"
    >
      <div class="bg-white rounded-lg shadow-xl max-w-md w-full p-6">
        <div class="flex justify-between items-center mb-4">
          <h3 class="text-lg font-semibold text-gray-900">Error</h3>
          <button @click="hideError" class="text-gray-400 hover:text-gray-600">
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M6 18L18 6M6 6l12 12"
              ></path>
            </svg>
          </button>
        </div>
        <p class="text-gray-600 mb-6">{{ errorMessage }}</p>
        <div class="flex justify-end">
          <button
            @click="hideError"
            class="px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 transition duration-200"
          >
            OK
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import FileExplorer from './FileExplorer.vue'

export default {
  name: 'DriveCompressor',
  components: {
    FileExplorer,
  },
  data() {
    return {
      // Configuration
      config: {
        clientId: import.meta.env.VITE_GOOGLE_CLIENT_ID,
        apiKey: import.meta.env.VITE_GOOGLE_API_KEY,
        discoveryDocs: [import.meta.env.VITE_DISCOVERY_DOCS],
        scopes: import.meta.env.VITE_SCOPES,
      },

      // State
      isAuthenticated: false,
      currentScreen: 'login',
      loading: false,
      loadingMessage: 'Loading...',
      showErrorModal: false,
      errorMessage: '',

      // User info
      accessToken: null,
      userInfo: null,
      tokenClient: null,

      // Folders and hierarchy
      folders: [],
      folderHierarchy: new Map(),
      currentPath: [],
      currentFolderId: null,
      selectedFolderId: null,
      selectedFolder: null,

      // Images and processing
      images: [],
      processing: false,
      paused: false,
      cancelled: false,
      currentImageIndex: 0,
      currentImageName: '',

      // Settings
      settings: {
        compressionQuality: 0.8,
        maxWidth: 1920,
        maxHeight: 1080,
        namingSuffix: '_compressed',
        subfolderName: 'crushed',
      },

      // Results
      results: {
        processed: 0,
        totalSaved: 0,
        errors: [],
        startTime: null,
        endTime: null,
      },
    }
  },

  computed: {
    currentFolders() {
      return this.folderHierarchy.get(this.currentFolderId) || []
    },

    totalSize() {
      return this.images.reduce((sum, img) => sum + parseInt(img.size || 0), 0)
    },

    progress() {
      if (this.images.length === 0) return 0
      return (this.results.processed / this.images.length) * 100
    },

    progressText() {
      return `Processing ${this.results.processed} of ${this.images.length}`
    },

    compressionRatio() {
      if (this.results.totalSaved === 0) return 0
      const originalTotal = this.images.reduce((sum, img) => sum + parseInt(img.size || 0), 0)
      return Math.round((this.results.totalSaved / originalTotal) * 100)
    },

    processingTime() {
      if (!this.results.startTime || !this.results.endTime) return '0s'
      const duration = (this.results.endTime - this.results.startTime) / 1000
      return `${Math.round(duration)}s`
    },
  },

  async mounted() {
    if (
      this.config.clientId !== 'YOUR_CLIENT_ID_HERE' &&
      this.config.apiKey !== 'YOUR_API_KEY_HERE'
    ) {
      await this.initializeGoogleIdentityServices()
    }
  },

  methods: {
    async initializeGoogleIdentityServices() {
      try {
        await this.waitForGoogleIdentityServices()
        await this.initializeGoogleAPIClient()
        this.setupGoogleSignIn()
        console.log('Google Identity Services initialized successfully')
      } catch (error) {
        console.error('Failed to initialize Google Identity Services:', error)
        this.showError('Initialization Failed', error.message)
      }
    },

    waitForGoogleIdentityServices() {
      return new Promise((resolve, reject) => {
        let attempts = 0
        const maxAttempts = 50

        const checkGoogle = () => {
          attempts++
          if (window.google && window.google.accounts && window.gapi) {
            resolve()
          } else if (attempts >= maxAttempts) {
            reject(new Error('Google Identity Services failed to load'))
          } else {
            setTimeout(checkGoogle, 100)
          }
        }
        checkGoogle()
      })
    },

    async initializeGoogleAPIClient() {
      return new Promise((resolve, reject) => {
        gapi.load('client', async () => {
          try {
            await gapi.client.init({
              apiKey: this.config.apiKey,
              discoveryDocs: this.config.discoveryDocs,
            })
            resolve()
          } catch (error) {
            reject(error)
          }
        })
      })
    },

    setupGoogleSignIn() {
      google.accounts.id.initialize({
        client_id: this.config.clientId,
        callback: this.handleCredentialResponse,
        auto_select: false,
        cancel_on_tap_outside: false,
      })

      this.tokenClient = google.accounts.oauth2.initTokenClient({
        client_id: this.config.clientId,
        scope: this.config.scopes,
        callback: (tokenResponse) => {
          if (tokenResponse.error !== undefined) {
            console.error('Token error:', tokenResponse.error)
            this.showError('Authentication Failed', tokenResponse.error)
            return
          }
          this.accessToken = tokenResponse.access_token
          this.handleSuccessfulAuth(tokenResponse)
        },
      })
    },

    signInWithGoogle() {
      if (this.config.clientId === 'YOUR_CLIENT_ID_HERE') {
        this.showError('Configuration Required', 'Please configure your Google API credentials.')
        return
      }
      this.tokenClient.requestAccessToken()
    },

    handleCredentialResponse(response) {
      const credential = response.credential
      const payload = this.parseJwt(credential)

      this.userInfo = {
        email: payload.email,
        name: payload.name,
        picture: payload.picture,
      }
    },

    handleSuccessfulAuth(tokenResponse) {
      this.isAuthenticated = true
      this.accessToken = tokenResponse.access_token

      gapi.client.setToken({
        access_token: this.accessToken,
      })

      this.currentScreen = 'dashboard'
      this.loadDriveFolders()
    },

    parseJwt(token) {
      try {
        const base64Url = token.split('.')[1]
        const base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/')
        const jsonPayload = decodeURIComponent(
          atob(base64)
            .split('')
            .map(function (c) {
              return '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2)
            })
            .join(''),
        )
        return JSON.parse(jsonPayload)
      } catch (error) {
        console.error('Error parsing JWT:', error)
        return {}
      }
    },

    async loadDriveFolders() {
      try {
        this.loading = true
        this.loadingMessage = 'Loading your Drive folders...'

        const response = await gapi.client.drive.files.list({
          q: "mimeType='application/vnd.google-apps.folder' and trashed=false and 'me' in owners",
          pageSize: 1000,
          fields: 'files(id, name, parents)',
          // Explicitly exclude shared drives and shared items
          includeItemsFromAllDrives: false,
          supportsAllDrives: false,
        })

        // Ensure proper reactive assignment - create new array reference
        this.folders = [...(response.result.files || [])] // Spread to new array

        console.log('Owned folders assigned:', this.folders.length)
        this.organizeFolderHierarchy()
        this.showFoldersAtLevel(null)
      } catch (error) {
        console.error('Error loading folders:', error)
        this.showError('Failed to Load Folders', error.message)
      } finally {
        this.loading = false
      }
    },

    organizeFolderHierarchy() {
      console.log('🔍 Starting to organize folder hierarchy with', this.folders.length, 'folders')
      this.folderHierarchy.clear()

      // Create a Set of all owned folder IDs for quick lookup
      const ownedFolderIds = new Set(this.folders.map((f) => f.id))

      this.folders.forEach((folder) => {
        let parentId = null

        // Check if folder has parents and if the parent is also owned by us
        if (folder.parents && Array.isArray(folder.parents) && folder.parents.length > 0) {
          const potentialParentId = folder.parents[0]

          // Only use the parent if it's also in our owned folders list
          if (ownedFolderIds.has(potentialParentId)) {
            parentId = potentialParentId
          }
          // If parent is not owned by us, treat this folder as a root-level folder
        }

        console.log(`Processing folder "${folder.name}" with resolved parentId:`, parentId)

        if (!this.folderHierarchy.has(parentId)) {
          this.folderHierarchy.set(parentId, [])
        }

        this.folderHierarchy.get(parentId).push(folder)
      })

      // Debug: Log the hierarchy map
      console.log('🗂️ Folder hierarchy map:')
      for (let [parentId, folders] of this.folderHierarchy.entries()) {
        console.log(
          `Parent "${parentId || 'ROOT'}":`,
          folders.map((f) => f.name),
        )
      }

      // Sort folders alphabetically
      this.folderHierarchy.forEach((folders) => {
        folders.sort((a, b) => a.name.localeCompare(b.name))
      })

      console.log('✅ Hierarchy organization complete')

      // Debug: Check what's at root level
      const rootFolders = this.folderHierarchy.get(null) || []
      console.log(
        '🌳 Root level folders:',
        rootFolders.length,
        rootFolders.map((f) => f.name),
      )
    },

    showFoldersAtLevel(parentId) {
      this.currentFolderId = parentId
      this.selectedFolderId = null
    },

    selectFolderForHighlight(folder) {
      console.log('🔥 PARENT: selectFolderForHighlight called with:', folder)

      this.selectedFolderId = folder.id
    },

    // In DriveCompressor.vue methods
    navigateIntoFolder(folder) {
      console.log('Parent received navigate-into-folder event for:', folder.name)

      if (!this.hasSubfolders(folder.id)) {
        console.log('Folder has no subfolders, returning')
        return
      }

      console.log('Adding to path:', folder)
      this.currentPath.push({
        id: folder.id,
        name: folder.name,
      })

      this.showFoldersAtLevel(folder.id)
    },

    navigateToLevel(level) {
      if (level === -1) {
        this.currentPath = []
        this.showFoldersAtLevel(null)
      } else if (level < this.currentPath.length) {
        this.currentPath = this.currentPath.slice(0, level + 1)
        const targetFolderId = level >= 0 ? this.currentPath[level].id : null
        this.showFoldersAtLevel(targetFolderId)
      }
    },

    goBack() {
      if (this.currentPath.length > 0) {
        this.currentPath.pop()
        const parentId =
          this.currentPath.length > 0 ? this.currentPath[this.currentPath.length - 1].id : null
        this.showFoldersAtLevel(parentId)
      }
    },

    hasSubfolders(folderId) {
      return this.folderHierarchy.has(folderId) && this.folderHierarchy.get(folderId).length > 0
    },

    async selectFolder(folderId) {
      try {
        this.loading = true
        this.loadingMessage = 'Loading images from folder...'

        const folder = this.folders.find((f) => f.id === folderId)
        this.selectedFolder = folder

        const response = await gapi.client.drive.files.list({
          q: `'${folderId}' in parents and (mimeType contains 'image/') and trashed=false`,
          fields: 'files(id, name, size, mimeType)',
        })

        this.images = response.result.files || []

        if (this.images.length === 0) {
          this.showError('No Images Found', 'The selected folder does not contain any images.')
          this.selectedFolder = null
          return
        }

        console.log(`Found ${this.images.length} images`)
      } catch (error) {
        console.error('Error selecting folder:', error)
        this.showError('Failed to Load Images', error.message)
      } finally {
        this.loading = false
      }
    },

    async startCompression() {
      if (this.processing || !this.selectedFolder || this.images.length === 0) {
        return
      }

      this.processing = true
      this.paused = false
      this.cancelled = false
      this.currentImageIndex = 0
      this.currentImageName = ''
      this.results = {
        processed: 0,
        totalSaved: 0,
        errors: [],
        startTime: Date.now(),
        endTime: null,
      }

      this.currentScreen = 'processing'

      try {
        const crushFolder = await this.createCrushedFolder()

        for (let i = 0; i < this.images.length; i++) {
          if (this.cancelled) break

          while (this.paused && !this.cancelled) {
            await this.sleep(100)
          }

          this.currentImageIndex = i
          this.currentImageName = this.images[i].name
          await this.processImage(this.images[i], crushFolder.id)
        }

        this.results.endTime = Date.now()
        this.currentScreen = 'results'
      } catch (error) {
        console.error('Compression failed:', error)
        this.showError('Compression Failed', error.message)
      } finally {
        this.processing = false
      }
    },

    async createCrushedFolder() {
      const folderMetadata = {
        name: this.settings.subfolderName,
        mimeType: 'application/vnd.google-apps.folder',
        parents: [this.selectedFolder.id],
      }

      const response = await gapi.client.drive.files.create({
        resource: folderMetadata,
      })

      return response.result
    },

    async processImage(image, crushFolderId) {
      try {
        const imageBlob = await this.downloadImage(image.id)
        const compressedBlob = await this.compressImage(imageBlob, image.mimeType)
        const newFileName = this.generateCompressedFileName(image.name)
        await this.uploadImage(compressedBlob, newFileName, crushFolderId)

        const originalSize = imageBlob.size
        const compressedSize = compressedBlob.size
        const saved = originalSize - compressedSize

        this.results.processed++
        this.results.totalSaved += saved
      } catch (error) {
        console.error(`Error processing image ${image.name}:`, error)
        this.results.errors.push({
          fileName: image.name,
          error: error.message,
        })
      }
    },

    async downloadImage(fileId) {
      const response = await fetch(
        `https://www.googleapis.com/drive/v3/files/${fileId}?alt=media`,
        {
          method: 'GET',
          headers: {
            Authorization: `Bearer ${this.accessToken}`,
          },
        },
      )

      if (!response.ok) {
        throw new Error(`Download failed: ${response.status} ${response.statusText}`)
      }

      return await response.blob()
    },

    async compressImage(imageBlob, mimeType) {
      return new Promise((resolve, reject) => {
        const img = new Image()
        const canvas = document.createElement('canvas')
        const ctx = canvas.getContext('2d')

        img.onload = () => {
          const { width, height } = this.calculateNewDimensions(img.width, img.height)

          canvas.width = width
          canvas.height = height
          ctx.drawImage(img, 0, 0, width, height)

          canvas.toBlob(
            (compressedBlob) => {
              if (compressedBlob) {
                resolve(compressedBlob)
              } else {
                reject(new Error('Failed to compress image'))
              }
            },
            mimeType,
            this.settings.compressionQuality,
          )
        }

        img.onerror = () => reject(new Error('Failed to load image for compression'))
        img.src = URL.createObjectURL(imageBlob)
      })
    },

    calculateNewDimensions(originalWidth, originalHeight) {
      const maxWidth = this.settings.maxWidth
      const maxHeight = this.settings.maxHeight

      if (originalWidth <= maxWidth && originalHeight <= maxHeight) {
        return { width: originalWidth, height: originalHeight }
      }

      const aspectRatio = originalWidth / originalHeight

      if (originalWidth > originalHeight) {
        const width = Math.min(maxWidth, originalWidth)
        const height = width / aspectRatio
        return { width: Math.round(width), height: Math.round(height) }
      } else {
        const height = Math.min(maxHeight, originalHeight)
        const width = height * aspectRatio
        return { width: Math.round(width), height: Math.round(height) }
      }
    },

    generateCompressedFileName(originalName) {
      const lastDot = originalName.lastIndexOf('.')
      if (lastDot === -1) {
        return originalName + this.settings.namingSuffix
      }

      const name = originalName.substring(0, lastDot)
      const extension = originalName.substring(lastDot)
      return name + this.settings.namingSuffix + extension
    },

    async uploadImage(imageBlob, fileName, folderId) {
      const metadata = {
        name: fileName,
        parents: [folderId],
      }

      const form = new FormData()
      form.append('metadata', new Blob([JSON.stringify(metadata)], { type: 'application/json' }))
      form.append('file', imageBlob)

      const response = await fetch(
        'https://www.googleapis.com/upload/drive/v3/files?uploadType=multipart',
        {
          method: 'POST',
          headers: {
            Authorization: `Bearer ${this.accessToken}`,
          },
          body: form,
        },
      )

      if (!response.ok) {
        throw new Error(`Upload failed: ${response.statusText}`)
      }

      return await response.json()
    },

    togglePause() {
      this.paused = !this.paused
    },

    cancelProcessing() {
      this.cancelled = true
      this.processing = false
    },

    resetApp() {
      this.selectedFolder = null
      this.images = []
      this.processing = false
      this.paused = false
      this.cancelled = false
      this.currentImageIndex = 0
      this.currentImageName = ''
      this.results = {
        processed: 0,
        totalSaved: 0,
        errors: [],
        startTime: null,
        endTime: null,
      }

      this.currentScreen = 'dashboard'
      this.loadDriveFolders()
    },

    signOut() {
      google.accounts.id.disableAutoSelect()
      this.accessToken = null
      this.userInfo = null
      this.isAuthenticated = false
      gapi.client.setToken(null)
      this.currentScreen = 'login'
    },

    showError(title, message) {
      this.errorMessage = message
      this.showErrorModal = true
    },

    hideError() {
      this.showErrorModal = false
      this.errorMessage = ''
    },

    formatFileSize(bytes) {
      if (bytes === 0) return '0 Bytes'
      const k = 1024
      const sizes = ['Bytes', 'KB', 'MB', 'GB']
      const i = Math.floor(Math.log(bytes) / Math.log(k))
      return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i]
    },

    sleep(ms) {
      return new Promise((resolve) => setTimeout(resolve, ms))
    },
  },
}
</script>

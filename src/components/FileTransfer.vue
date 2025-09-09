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
            <span v-if="sourceUserInfo" class="ml-2 text-sm text-green-700">({{ sourceUserInfo.email }})</span>
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
            <strong>Note:</strong> You may need to sign out and sign in with a different account.
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
            <span v-if="destinationUserInfo" class="ml-2 text-sm text-green-700">({{ destinationUserInfo.email }})</span>
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
              Source Drive ({{ sourceUserInfo?.email || 'Loading...' }})
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
              Destination Drive ({{ destinationUserInfo?.email || 'Loading...' }})
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

    <!-- Error Modal -->
    <div v-if="showErrorModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4">
      <div class="bg-white rounded-lg shadow-xl max-w-md w-full p-6">
        <div class="flex justify-between items-center mb-4">
          <h3 class="text-lg font-semibold text-gray-900">Error</h3>
          <button @click="hideError" class="text-gray-400 hover:text-gray-600">
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
            </svg>
          </button>
        </div>
        <p class="text-gray-600 mb-6">{{ errorMessage }}</p>
        <div class="flex justify-end">
          <button @click="hideError" class="px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 transition duration-200">
            OK
          </button>
        </div>
      </div>
    <!-- Error Modal -->
    <div v-if="showErrorModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 p-4">
      <div class="bg-white rounded-lg shadow-xl max-w-md w-full p-6">
        <div class="flex justify-between items-center mb-4">
          <h3 class="text-lg font-semibold text-gray-900">Error</h3>
          <button @click="hideError" class="text-gray-400 hover:text-gray-600">
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
            </svg>
          </button>
        </div>
        <p class="text-gray-600 mb-6">{{ errorMessage }}</p>
        <div class="flex justify-end">
          <button @click="hideError" class="px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 transition duration-200">
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
  name: 'FileTransfer',
  components: { FileExplorer },
  data() {
    return {
      // Configuration
      config: {
        clientId: import.meta.env.VITE_GOOGLE_CLIENT_ID,
        apiKey: import.meta.env.VITE_GOOGLE_API_KEY,
        discoveryDocs: [import.meta.env.VITE_DISCOVERY_DOCS],
        scopes: import.meta.env.VITE_SCOPES,
      },

      // Navigation
      currentPage: 1,

      // Connection Status
      sourceConnected: false,
      destinationConnected: false,
      loading: false,
      transferring: false,
      statusMessage: '',
      showErrorModal: false,
      errorMessage: '',

      // Authentication tokens and user info
      sourceAccessToken: null,
      destinationAccessToken: null,
      sourceUserInfo: null,
      destinationUserInfo: null,
      sourceTokenClient: null,
      destinationTokenClient: null,

      // Source Drive Data
      sourceFolders: [],
      sourcePath: [],
      sourceSelectedFolderId: null,
      sourceFolderHierarchy: new Map(),
      sourceSelectedFolderId: null,
      sourceFolderHierarchy: new Map(),

      // Destination Drive Data
      destinationFolders: [],
      destinationPath: [],
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

  async mounted() {
    // Initialize Source token client
    this.sourceTokenClient = google.accounts.oauth2.initTokenClient({
      client_id: this.config.clientId,
      scope: "https://www.googleapis.com/auth/drive.metadata.readonly https://www.googleapis.com/auth/drive.file https://www.googleapis.com/auth/userinfo.email",
      callback: (tokenResponse) => {
        this.handleSourceAuth(tokenResponse);
      },
    });

    // Initialize Destination token client
    this.destinationTokenClient = google.accounts.oauth2.initTokenClient({
      client_id: this.config.clientId,
      scope: "https://www.googleapis.com/auth/drive.metadata.readonly https://www.googleapis.com/auth/drive.file https://www.googleapis.com/auth/userinfo.email",
      callback: (tokenResponse) => {
        this.handleDestinationAuth(tokenResponse);
      },
    });
  },


  methods: {
  // =========================
  // AUTHENTICATION
  // =========================
  async connectSourceDrive() {
    if (this.config.clientId === "YOUR_CLIENT_ID_HERE") {
      this.showError(
        "Configuration Required",
        "Please configure your Google API credentials."
      );
      return;
    }

    this.loading = true;
    try {
      this.sourceTokenClient.requestAccessToken({ prompt: "select_account" });
    } catch (error) {
      this.loading = false;
      this.showError("Authentication Error", error.message);
    }
  },

  async connectDestinationDrive() {
    this.loading = true;
    try {
      this.destinationTokenClient.requestAccessToken({ prompt: "select_account" });
    } catch (error) {
      this.loading = false;
      this.showError("Authentication Error", error.message);
    }
  },

  async handleSourceAuth(tokenResponse) {
    try {
      this.sourceAccessToken = tokenResponse.access_token;
      this.sourceUserInfo = await this.getUserInfo(this.sourceAccessToken);
      await this.fetchSourceFolders();
      this.sourceConnected = true;
      this.currentPage = 2;
      console.log("Source drive connected:", this.sourceUserInfo.email);
    } catch (error) {
      console.error("Source auth error:", error);
      this.showError("Source Authentication Failed", error.message);
    } finally {
      this.loading = false;
    }
  },

  async handleDestinationAuth(tokenResponse) {
    try {
      this.destinationAccessToken = tokenResponse.access_token;
      this.destinationUserInfo = await this.getUserInfo(this.destinationAccessToken);
      await this.fetchDestinationFolders();
      this.destinationConnected = true;
      this.currentPage = 3;
      console.log("Destination drive connected:", this.destinationUserInfo.email);
    } catch (error) {
      console.error("Destination auth error:", error);
      this.showError("Destination Authentication Failed", error.message);
    } finally {
      this.loading = false;
    }
  },

  async getUserInfo(token) {
    try {
      const response = await fetch("https://www.googleapis.com/oauth2/v2/userinfo", {
        headers: { Authorization: `Bearer ${token}` },
      });

      if (!response.ok) {
        throw new Error(`HTTP ${response.status} - ${await response.text()}`);
      }

      return await response.json();
    } catch (error) {
      console.error("getUserInfo failed:", error);
      throw new Error("Failed to get user info");
    }
  },

  // =========================
  // DRIVE FOLDER FETCH
  // =========================
  async fetchSourceFolders() {
    try {
      const response = await fetch(
        "https://www.googleapis.com/drive/v3/files?q=mimeType='application/vnd.google-apps.folder' and trashed=false and 'me' in owners&fields=files(id,name,parents)",
        {
          headers: { Authorization: `Bearer ${this.sourceAccessToken}` },
        }
      );

      if (!response.ok) {
        throw new Error(`HTTP ${response.status} - ${await response.text()}`);
      }

      const data = await response.json();
      this.sourceFolders = data.files || [];
      this.organizeFolderHierarchy("source");
      console.log("Source folders loaded:", this.sourceFolders.length);
    } catch (error) {
      console.error("Error loading source folders:", error);
      this.showError("Failed to Load Source Folders", error.message);
    }
  },

  async fetchDestinationFolders() {
    try {
      const response = await fetch(
        "https://www.googleapis.com/drive/v3/files?q=mimeType='application/vnd.google-apps.folder' and trashed=false and 'me' in owners&fields=files(id,name,parents)",
        {
          headers: { Authorization: `Bearer ${this.destinationAccessToken}` },
        }
      );

      if (!response.ok) {
        throw new Error(`HTTP ${response.status} - ${await response.text()}`);
      }

      const data = await response.json();
      this.destinationFolders = data.files || [];
      this.organizeFolderHierarchy("destination");
      console.log("Destination folders loaded:", this.destinationFolders.length);
    } catch (error) {
      console.error("Error loading destination folders:", error);
      this.showError("Failed to Load Destination Folders", error.message);
    }
  },

  // =========================
  // FOLDER NAVIGATION HELPERS
  // =========================
  getCurrentSourceFolderId() {
    return this.sourcePath.length > 0 ? this.sourcePath[this.sourcePath.length - 1].id : null;
  },

  getCurrentDestinationFolderId() {
    return this.destinationPath.length > 0
      ? this.destinationPath[this.destinationPath.length - 1].id
      : null;
  },

  getSelectedSourceFolderName() {
    const folder = this.sourceFolders.find((f) => f.id === this.sourceSelectedFolderId);
    return folder ? folder.name : "None selected";
  },

  getSelectedDestinationFolderName() {
    const folder = this.destinationFolders.find((f) => f.id === this.destinationSelectedFolderId);
    return folder ? folder.name : "None selected";
  },

  navigateSourceToLevel(level) {
    if (level === -1) {
      this.sourcePath = [];
    } else if (level < this.sourcePath.length) {
      this.sourcePath = this.sourcePath.slice(0, level + 1);
    }
  },

  navigateDestinationToLevel(level) {
    if (level === -1) {
      this.destinationPath = [];
    } else if (level < this.destinationPath.length) {
      this.destinationPath = this.destinationPath.slice(0, level + 1);
    }
  },

  highlightSourceFolder(folder) {
    this.sourceSelectedFolderId = folder.id;
  },

  highlightDestinationFolder(folder) {
    this.destinationSelectedFolderId = folder.id;
  },

  navigateSourceIntoFolder(folder) {
    if (this.hasSubfolders(folder.id, "source")) {
      this.sourcePath.push({ id: folder.id, name: folder.name });
    }
  },

  navigateDestinationIntoFolder(folder) {
    if (this.hasSubfolders(folder.id, "destination")) {
      this.destinationPath.push({ id: folder.id, name: folder.name });
    }
  },

  selectSourceFolder(folderId) {
    this.sourceSelectedFolderId = folderId;
  },

  selectDestinationFolder(folderId) {
    this.destinationSelectedFolderId = folderId;
  },

  goBackSource() {
    if (this.sourcePath.length > 0) {
      this.sourcePath.pop();
    }
  },

  goBackDestination() {
    if (this.destinationPath.length > 0) {
      this.destinationPath.pop();
    }
  },

  // =========================
  // FOLDER ORGANIZATION
  // =========================
  organizeFolderHierarchy(type) {
    const folders = type === "source" ? this.sourceFolders : this.destinationFolders;
    const hierarchy = type === "source"
      ? this.sourceFolderHierarchy
      : this.destinationFolderHierarchy;

    hierarchy.clear();

    const ownedFolderIds = new Set(folders.map((f) => f.id));

    folders.forEach((folder) => {
      let parentId = null;
      if (folder.parents && Array.isArray(folder.parents) && folder.parents.length > 0) {
        const potentialParentId = folder.parents[0];
        if (ownedFolderIds.has(potentialParentId)) {
          parentId = potentialParentId;
        }
      }

      if (!hierarchy.has(parentId)) {
        hierarchy.set(parentId, []);
      }

      hierarchy.get(parentId).push(folder);
    });

    hierarchy.forEach((folders) => {
      folders.sort((a, b) => a.name.localeCompare(b.name));
    });
  },

  hasSubfolders(folderId, type) {
    const hierarchy = type === "source"
      ? this.sourceFolderHierarchy
      : this.destinationFolderHierarchy;
    return hierarchy.has(folderId) && hierarchy.get(folderId).length > 0;
  },

  // =========================
  // FILE TRANSFER
  // =========================
  async transfer() {
    if (!this.sourceSelectedFolderId || !this.destinationSelectedFolderId) {
      this.statusMessage = "Error: Please select both source and destination folders";
      return;
    }

    this.transferring = true;
    this.statusMessage = "Starting transfer...";

    try {
      const response = await fetch(
        `https://www.googleapis.com/drive/v3/files?q='${this.sourceSelectedFolderId}' in parents and trashed=false&fields=files(id,name,mimeType,size)`,
        { headers: { Authorization: `Bearer ${this.sourceAccessToken}` } }
      );

      if (!response.ok) {
        throw new Error(`HTTP ${response.status} - ${await response.text()}`);
      }

      const data = await response.json();
      const filesToTransfer = data.files || [];

      if (filesToTransfer.length === 0) {
        this.statusMessage = "No files found in the selected source folder";
        this.transferring = false;
        return;
      }

      this.statusMessage = `Found ${filesToTransfer.length} files to transfer...`;

      let transferred = 0;
      for (const file of filesToTransfer) {
        try {
          await this.transferFile(file);
          transferred++;
          this.statusMessage = `Transferred ${transferred}/${filesToTransfer.length} files...`;
        } catch (error) {
          console.error(`Failed to transfer ${file.name}:`, error);
        }
      }

      const sourceFolder = this.getSelectedSourceFolderName();
      const destFolder = this.getSelectedDestinationFolderName();

      this.statusMessage = `Transfer completed! ${transferred}/${filesToTransfer.length} files transferred from "${sourceFolder}" to "${destFolder}".`;
    } catch (error) {
      console.error("Transfer error:", error);
      this.statusMessage = "Error: Transfer failed. Please try again.";
    } finally {
      this.transferring = false;
    }
  },

  async transferFile(file) {
    // Download from source
    const downloadResponse = await fetch(
      `https://www.googleapis.com/drive/v3/files/${file.id}?alt=media`,
      { headers: { Authorization: `Bearer ${this.sourceAccessToken}` } }
    );

    if (!downloadResponse.ok) {
      throw new Error(`Failed to download ${file.name}`);
    }

    const fileBlob = await downloadResponse.blob();

    // Upload to destination
    const metadata = {
      name: file.name,
      parents: [this.destinationSelectedFolderId],
    };

    const form = new FormData();
    form.append("metadata", new Blob([JSON.stringify(metadata)], { type: "application/json" }));
    form.append("file", fileBlob);

    const uploadResponse = await fetch(
      "https://www.googleapis.com/upload/drive/v3/files?uploadType=multipart",
      {
        method: "POST",
        headers: { Authorization: `Bearer ${this.destinationAccessToken}` },
        body: form,
      }
    );

    if (!uploadResponse.ok) {
      throw new Error(`Failed to upload ${file.name}`);
    }

    return await uploadResponse.json();
  },

  // =========================
  // ERROR HANDLING
  // =========================
  showError(title, message) {
    this.errorMessage = `${title}: ${message}`;
    this.showErrorModal = true;
  },

  hideError() {
    this.showErrorModal = false;
    this.errorMessage = "";
  },

  // =========================
  // NAVIGATION
  // =========================
  goToPreviousPage() {
    if (this.currentPage > 1) {
      this.currentPage--;
    }
  },
}

}
</script>

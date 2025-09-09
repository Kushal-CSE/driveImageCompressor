<template>
  <div class="bg-white rounded-lg">
    <div class="flex items-center justify-between mb-4">
      <h3 class="text-lg font-semibold text-gray-900">Select Files & Folders</h3>
      <button
        @click="$emit('refresh')"
        :disabled="loading"
        class="px-3 py-1 text-sm border border-gray-300 rounded-md hover:bg-gray-50 transition duration-200 disabled:bg-gray-100"
      >
        {{ loading ? 'Loading...' : 'Refresh' }}
      </button>
    </div>

    <!-- File Explorer -->
    <div class="border rounded-lg overflow-hidden">
      <!-- Breadcrumb -->
      <div class="bg-gray-50 px-3 py-2 border-b text-sm">
        <div class="flex items-center flex-wrap">
          <span
            @click="$emit('navigate-to-level', -1)"
            :class="
              currentPath.length > 0
                ? 'cursor-pointer text-blue-600 hover:bg-blue-100 px-1 rounded'
                : 'text-gray-800'
            "
            class="flex items-center"
          >
            📁 My Drive
          </span>
          <template v-for="(folder, index) in currentPath" :key="folder.id">
            <span class="mx-2 text-gray-400">›</span>
            <span
              @click="$emit('navigate-to-level', index)"
              :class="
                index === currentPath.length - 1
                  ? 'text-gray-800 font-medium'
                  : 'cursor-pointer text-blue-600 hover:bg-blue-100 px-1 rounded'
              "
            >
              {{ folder.name }}
            </span>
          </template>
        </div>
      </div>

      <!-- Items List -->
      <div class="max-h-96 overflow-y-auto">
        <div v-if="loading" class="p-8 text-center text-gray-500">
          <div class="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-600 mx-auto mb-2"></div>
          <p>Loading...</p>
        </div>

        <div v-else-if="currentFolders.length === 0 && currentFiles.length === 0" class="p-8 text-center text-gray-500">
          <div class="text-4xl mb-2">📁</div>
          <p>This folder is empty</p>
          <button
            v-if="currentPath.length > 0"
            @click="$emit('go-back')"
            class="mt-2 px-3 py-1 text-sm border border-gray-300 rounded-md hover:bg-gray-50"
          >
            Go Back
          </button>
        </div>

        <div v-else class="divide-y divide-gray-100">
          <div class="px-3 py-2 bg-gray-50 text-sm text-gray-600 font-medium">
            Click to select • Use Open button to browse subfolders
          </div>

          <!-- Folders -->
          <div
            v-for="folder in currentFolders"
            :key="'folder-' + folder.id"
            :class="[
              'flex items-center p-3 transition-colors duration-150 select-none',
              isSelected(folder) ? 'bg-blue-50 border-l-4 border-l-blue-500' : 'hover:bg-gray-50',
            ]"
          >
            <!-- Clickable Folder Info Area -->
            <div
              @click="toggleSelection(folder)"
              class="flex items-center flex-1 cursor-pointer min-w-0"
            >
              <!-- Selection Checkbox -->
              <div class="flex-shrink-0 mr-3">
                <div class="w-5 h-5 border-2 border-gray-300 rounded flex items-center justify-center"
                     :class="isSelected(folder) ? 'bg-blue-600 border-blue-600' : 'bg-white'">
                  <svg v-if="isSelected(folder)" class="w-3 h-3 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"></path>
                  </svg>
                </div>
              </div>

              <!-- Folder Icon -->
              <div class="flex-shrink-0 mr-3 relative">
                <span class="text-xl">📁</span>
                <span
                  v-if="hasSubfolders(folder)"
                  class="absolute -bottom-1 -right-1 text-xs text-green-500"
                  >▶</span
                >
              </div>

              <!-- Folder Info -->
              <div class="flex-1 min-w-0">
                <div class="font-medium text-gray-900 truncate">{{ folder.name }}</div>
                <div class="text-sm text-gray-500">
                  Folder • {{ hasSubfolders(folder) ? 'Contains subfolders' : 'Empty folder' }}
                </div>
              </div>
            </div>

            <!-- Action Buttons -->
            <div class="flex-shrink-0 ml-3 flex space-x-2">
              <!-- Open Button (only for folders with subfolders) -->
              <button
                v-if="hasSubfolders(folder) || hasFiles(folder)"
                @click="$emit('navigate-into-folder', folder)"
                class="px-3 py-1 bg-green-600 text-white text-sm rounded-md hover:bg-green-700 transition duration-200 flex items-center"
              >
                <svg class="w-3 h-3 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                    d="M9 5l7 7-7 7"
                  ></path>
                </svg>
                Open
              </button>
            </div>
          </div>

          <!-- Files -->
          <div
            v-for="file in currentFiles"
            :key="'file-' + file.id"
            :class="[
              'flex items-center p-3 transition-colors duration-150 select-none',
              isSelected(file) ? 'bg-blue-50 border-l-4 border-l-blue-500' : 'hover:bg-gray-50',
            ]"
          >
            <!-- Clickable File Info Area -->
            <div
              @click="toggleSelection(file)"
              class="flex items-center flex-1 cursor-pointer min-w-0"
            >
              <!-- Selection Checkbox -->
              <div class="flex-shrink-0 mr-3">
                <div class="w-5 h-5 border-2 border-gray-300 rounded flex items-center justify-center"
                     :class="isSelected(file) ? 'bg-blue-600 border-blue-600' : 'bg-white'">
                  <svg v-if="isSelected(file)" class="w-3 h-3 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"></path>
                  </svg>
                </div>
              </div>

              <!-- File Icon -->
              <div class="flex-shrink-0 mr-3">
                <span class="text-xl">{{ getFileIcon(file.mimeType) }}</span>
              </div>

              <!-- File Info -->
              <div class="flex-1 min-w-0">
                <div class="font-medium text-gray-900 truncate">{{ file.name }}</div>
                <div class="text-sm text-gray-500">
                  {{ formatFileSize(file.size) }} • {{ getFileType(file.mimeType) }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'SourceFileExplorer',
  props: {
    folders: {
      type: Array,
      default: () => [],
    },
    files: {
      type: Array,
      default: () => [],
    },
    currentPath: {
      type: Array,
      default: () => [],
    },
    currentFolders: {
      type: Array,
      default: () => [],
    },
    currentFiles: {
      type: Array,
      default: () => [],
    },
    selectedItems: {
      type: Array,
      default: () => [],
    },
    loading: {
      type: Boolean,
      default: false,
    },
  },
  emits: [
    'refresh',
    'navigate-to-level',
    'navigate-into-folder',
    'toggle-item-selection',
    'go-back',
  ],
  methods: {
    hasSubfolders(folder) {
      return this.folders.some((f) => f.parents && f.parents[0] === folder.id)
    },

    hasFiles(folder) {
      return this.files.some((f) => f.parents && f.parents[0] === folder.id)
    },

    isSelected(item) {
      return this.selectedItems.some(selected => selected.id === item.id)
    },

    toggleSelection(item) {
      this.$emit('toggle-item-selection', item)
    },

    getFileIcon(mimeType) {
      if (mimeType.startsWith('image/')) return '🖼️'
      if (mimeType.startsWith('video/')) return '🎥'
      if (mimeType.startsWith('audio/')) return '🎵'
      if (mimeType.includes('pdf')) return '📄'
      if (mimeType.includes('document') || mimeType.includes('word')) return '📝'
      if (mimeType.includes('spreadsheet') || mimeType.includes('excel')) return '📊'
      if (mimeType.includes('presentation') || mimeType.includes('powerpoint')) return '📽️'
      if (mimeType.includes('zip') || mimeType.includes('archive')) return '📦'
      return '📄'
    },

    getFileType(mimeType) {
      if (mimeType.startsWith('image/')) return 'Image'
      if (mimeType.startsWith('video/')) return 'Video'
      if (mimeType.startsWith('audio/')) return 'Audio'
      if (mimeType.includes('pdf')) return 'PDF'
      if (mimeType.includes('document') || mimeType.includes('word')) return 'Document'
      if (mimeType.includes('spreadsheet') || mimeType.includes('excel')) return 'Spreadsheet'
      if (mimeType.includes('presentation') || mimeType.includes('powerpoint')) return 'Presentation'
      if (mimeType.includes('zip') || mimeType.includes('archive')) return 'Archive'
      return 'File'
    },

    formatFileSize(bytes) {
      if (!bytes || bytes === 0) return '0 Bytes'
      const k = 1024
      const sizes = ['Bytes', 'KB', 'MB', 'GB']
      const i = Math.floor(Math.log(bytes) / Math.log(k))
      return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i]
    },
  },
}
</script>
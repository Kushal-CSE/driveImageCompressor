<template>
  <div class="bg-white rounded-lg shadow p-6">
    <div class="flex items-center justify-between mb-4">
      <h2 class="text-lg font-semibold text-gray-900">Select Folder</h2>
      <button
        @click="$emit('refresh')"
        class="px-3 py-1 text-sm border border-gray-300 rounded-md hover:bg-gray-50 transition duration-200"
      >
        Refresh
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

      <!-- Folder List -->
      <div class="max-h-96 overflow-y-auto">
        <div v-if="currentFolders.length === 0" class="p-8 text-center text-gray-500">
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
            Click to select folder • Use Open button to browse subfolders
          </div>

          <div
            v-for="folder in currentFolders"
            :key="folder.id"
            :class="[
              'flex items-center p-3 transition-colors duration-150 select-none',
              selectedFolderId === folder.id ? 'folder-item-selected' : 'folder-item-hover',
            ]"
          >
            <!-- Clickable Folder Info Area -->
            <div
              @click="handleFolderClick(folder)"
              class="flex items-center flex-1 cursor-pointer min-w-0"
            >
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
                  {{ hasSubfolders(folder) ? 'Contains subfolders' : 'Empty folder' }}
                </div>
              </div>
            </div>

            <!-- Action Buttons - Separate from clickable area -->
            <div class="flex-shrink-0 ml-3 flex space-x-2">
              <!-- Open Button (only for folders with subfolders) -->
              <button
                v-if="hasSubfolders(folder)"
                @click="handleOpenFolder(folder)"
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

              <!-- Select Button -->
              <button
                @click="handleSelectFolder(folder)"
                class="px-3 py-1 bg-blue-600 text-white text-sm rounded-md hover:bg-blue-700 transition duration-200"
              >
                Select
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'FileExplorer',
  props: {
    folders: {
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
    selectedFolderId: {
      type: String,
      default: null,
    },
  },
  emits: [
    'refresh',
    'navigate-to-level',
    'select-folder-highlight',
    'navigate-into-folder',
    'select-folder',
    'go-back',
  ],
  methods: {
    hasSubfolders(folder) {
      // Only check among owned folders since we're filtering for owned folders now
      return this.folders.some((f) => f.parents && f.parents[0] === folder.id)
    },

    handleFolderClick(folder) {
      console.log('Folder clicked for highlighting:', folder.name)
      this.$emit('select-folder-highlight', folder)
    },

    handleOpenFolder(folder) {
      console.log('Open button clicked for folder:', folder.name)
      this.$emit('navigate-into-folder', folder)
    },

    handleSelectFolder(folder) {
      console.log('Select button clicked for folder:', folder.name)
      this.$emit('select-folder', folder.id)
    },
  },
}
</script>

      
<template>
  <v-layout class="fill-height documents-layout">
    <!-- Navigation Drawer -->
    <v-navigation-drawer v-model="drawerOpen" :permanent="false" location="left" width="300" class="sidebar">
      <div class="sidebar-header">
        <h3 class="text-h6 mb-1">Navigation</h3>
        <p class="text-caption text-medium-emphasis">App Sections</p>
      </div>
      <v-divider></v-divider>
      <v-list nav density="compact" class="sidebar-list">
        <!-- Link to Chat Page -->
        <v-list-item
          title="Chat"
          subtitle="AI Assistant"
          prepend-icon="mdi-message-text-outline"
          to="/"
          link
          class="sidebar-item chat-link"
        ></v-list-item>
        
        <!-- Link to Knowledge Base (Current Page) -->
        <v-list-item
          title="Knowledge Base"
          subtitle="Manage documents"
          prepend-icon="mdi-database"
          to="/knowledge-base"
          link
          class="sidebar-item knowledge-base-link active"
        ></v-list-item>
      </v-list>
      
      <template v-slot:append>
        <div class="sidebar-footer">
          <v-btn 
            block 
            variant="outlined" 
            @click="refreshList" 
            prepend-icon="mdi-refresh"
            class="refresh-docs-btn"
            :loading="documentsStore.isLoading"
          >
            Refresh Documents
          </v-btn>
        </div>
      </template>
    </v-navigation-drawer>

    <!-- App Bar -->
    <v-app-bar class="app-bar" elevation="0" density="compact">
      <v-app-bar-nav-icon @click="drawerOpen = !drawerOpen" class="nav-icon"></v-app-bar-nav-icon>
      <v-app-bar-title class="app-title">
        <div class="d-flex align-center">
          <!-- Add CII Logo -->
          <v-img
            src="/logo.png"  
            alt="CII Logo"
            max-height="32" 
            max-width="100" 
            contain 
            class="mr-3"
          ></v-img>
          <span class="app-name">{{ appTitle }}</span>
        </div>
      </v-app-bar-title>
      <v-spacer></v-spacer>
      
      <!-- Logout button -->
      <v-btn icon="mdi-logout" title="Logout" class="logout-btn" variant="text"></v-btn>
    </v-app-bar>

    <!-- Main Content -->
    <v-main class="documents-main">
      <v-container class="documents-container">
      <!-- File Upload Section -->
      <FileUpload @upload-complete="onUploadComplete" class="mb-8" />
      
      <v-card class="documents-card" elevation="12" rounded="xl">
        <v-card-title class="documents-header">
          <div class="d-flex align-center w-100">
            <div class="d-flex align-center">
              <v-avatar size="44" color="primary" class="mr-4">
                <v-icon color="white" size="24">mdi-database</v-icon>
              </v-avatar>
              <div>
                <h3 class="text-h5 mb-1">Knowledge Base</h3>
                <div class="d-flex align-center">
                  <v-chip 
                    size="small" 
                    color="primary" 
                    variant="outlined"
                    class="mr-2"
                  >
                    {{ documentsStore.collectionName }}
                  </v-chip>
                  <v-chip 
                    size="small" 
                    color="success" 
                    variant="tonal"
                  >
                    {{ filteredDocuments?.length || 0 }} / {{ documentsStore.processedFiles?.length || 0 }} documents
                  </v-chip>
                </div>
              </div>
            </div>
            <v-spacer></v-spacer>
            <v-btn
              icon="mdi-refresh"
              variant="elevated"
              color="primary"
              size="small"
              @click="refreshList"
              :loading="documentsStore.isLoading"
              title="Refresh List"
              class="refresh-btn"
            ></v-btn>
          </div>
        </v-card-title>
        <v-card-subtitle class="documents-subtitle">
          Your indexed documents are ready for intelligent search and retrieval
        </v-card-subtitle>

        <!-- Search Filter -->
        <div class="search-section">
          <v-text-field
            v-model="searchQuery"
            placeholder="Search documents..."
            prepend-inner-icon="mdi-magnify"
            variant="outlined"
            density="compact"
            clearable
            class="search-input"
            hide-details
          ></v-text-field>
        </div>
  
        <v-divider></v-divider>
  
        <!-- Loading State -->
        <div v-if="documentsStore.isLoading" class="loading-state">
          <v-progress-circular 
            indeterminate 
            color="primary" 
            size="48"
            width="4"
          ></v-progress-circular>
          <p class="mt-4 text-h6 text-medium-emphasis">Loading document list...</p>
          <p class="text-caption text-medium-emphasis">Fetching your knowledge base</p>
        </div>
  
        <!-- Error State -->
        <v-alert
          v-else-if="documentsStore.error"
          type="error"
          variant="tonal"
          class="ma-4"
          closable
          @click:close="documentsStore.error = null"
        >
          {{ documentsStore.error }}
        </v-alert>
  
        <!-- Empty State -->
        <div
          v-else-if="!documentsStore.processedFiles || documentsStore.processedFiles.length === 0"
          class="empty-state"
        >
          <v-icon size="80" color="primary" class="mb-4">mdi-folder-open-outline</v-icon>
          <h4 class="text-h5 mb-2">No Documents Yet</h4>
          <p class="text-body-1 text-medium-emphasis mb-4">Your knowledge base is empty. Upload some documents to get started!</p>
          <v-btn 
            color="primary" 
            variant="outlined" 
            rounded="xl"
            @click="$el.querySelector('.upload-card').scrollIntoView({ behavior: 'smooth' })"
          >
            <v-icon left>mdi-upload</v-icon>
            Upload Documents
          </v-btn>
        </div>

        <!-- No Search Results -->
        <div
          v-else-if="searchQuery && filteredDocuments.length === 0"
          class="empty-state"
        >
          <v-icon size="80" color="primary" class="mb-4">mdi-file-search-outline</v-icon>
          <h4 class="text-h5 mb-2">No Documents Found</h4>
          <p class="text-body-1 text-medium-emphasis mb-4">No documents match your search for "{{ searchQuery }}"</p>
          <v-btn 
            color="primary" 
            variant="outlined" 
            rounded="xl"
            @click="searchQuery = ''"
          >
            <v-icon left>mdi-close</v-icon>
            Clear Search
          </v-btn>
        </div>
  
        <!-- Document List -->
        <v-list class="document-list" v-else>
          <transition-group name="list-item" tag="div">
            <v-list-item
              v-for="(filename, index) in filteredDocuments"
              :key="filename"
              :title="filename"
              :prepend-icon="getFileIcon(filename)"
              class="document-item"
              rounded="lg"
            >
              <template v-slot:prepend>
                <v-avatar 
                  size="40" 
                  :color="getFileIcon(filename) === 'mdi-file-pdf-box' ? 'red-lighten-1' : 'blue-lighten-1'"
                  class="mr-3"
                >
                  <v-icon color="white" size="20">{{ getFileIcon(filename) }}</v-icon>
                </v-avatar>
              </template>
              
              <template v-slot:title>
                <span class="document-title">{{ filename }}</span>
              </template>
              
              <template v-slot:subtitle>
                <span class="text-caption">
                  {{ getFileIcon(filename) === 'mdi-file-pdf-box' ? 'PDF Document' : 'Word Document' }}
                </span>
              </template>

              <template v-slot:append>
                <v-btn
                  icon="mdi-delete-outline"
                  variant="elevated"
                  size="small"
                  color="error"
                  @click="confirmDelete(filename)"
                  :loading="deletingFiles[filename]"
                  title="Delete file"
                  class="delete-btn"
                ></v-btn>
              </template>
            </v-list-item>
          </transition-group>
        </v-list>
      </v-card>

      <!-- Delete Confirmation Dialog -->
      <v-dialog v-model="deleteDialog" max-width="500px">
        <v-card class="delete-dialog" rounded="xl" elevation="24">
          <v-card-title class="delete-dialog-header">
            <div class="d-flex align-center">
              <v-avatar size="48" color="error" class="mr-4">
                <v-icon color="white" size="28">mdi-delete-alert</v-icon>
              </v-avatar>
              <div>
                <h3 class="text-h5 mb-1">Delete Document</h3>
                <p class="text-body-2 text-medium-emphasis mb-0">This action cannot be undone</p>
              </div>
            </div>
          </v-card-title>
          <v-card-text class="delete-dialog-content">
            <v-alert
              type="warning"
              variant="tonal"
              rounded="lg"
              class="mb-4"
            >
              <template v-slot:prepend>
                <v-icon>mdi-alert-triangle</v-icon>
              </template>
              You are about to permanently delete <strong>"{{ fileToDelete }}"</strong>
            </v-alert>
            <p class="text-body-2">This will remove the file from:</p>
            <ul class="text-body-2 ml-4">
              <li>The filesystem storage</li>
              <li>The vector database</li>
              <li>All search indexes</li>
            </ul>
          </v-card-text>
          <v-card-actions class="delete-dialog-actions">
            <v-spacer></v-spacer>
            <v-btn
              color="grey"
              variant="outlined"
              rounded="lg"
              @click="deleteDialog = false"
              class="mr-2"
            >
              <v-icon left>mdi-close</v-icon>
              Cancel
            </v-btn>
            <v-btn
              color="error"
              variant="elevated"
              rounded="lg"
              @click="deleteFile"
              :loading="deletingFiles[fileToDelete]"
              class="delete-confirm-btn"
            >
              <v-icon left>mdi-delete</v-icon>
              Delete Forever
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      </v-container>
    </v-main>
  </v-layout>
</template>
  
  <script setup>
  import { onMounted, ref, computed } from 'vue';
  import { useDocumentsStore } from '@/store/documents';
  import FileUpload from '@/components/FileUpload.vue';
  import ragApi from '@/services/ragApi';
  
  const documentsStore = useDocumentsStore();
  
  // Access the environment variable for the app bar title
  const appTitle = import.meta.env.VITE_APP_TITLE || 'GPT';
  const drawerOpen = ref(true); // Sidebar open by default
  
  // Search functionality
  const searchQuery = ref('');
  
  // Computed filtered documents
  const filteredDocuments = computed(() => {
    if (!searchQuery.value || !documentsStore.processedFiles) {
      return documentsStore.processedFiles || [];
    }
    
    const query = searchQuery.value.toLowerCase();
    return documentsStore.processedFiles.filter(filename => 
      filename.toLowerCase().includes(query)
    );
  });
  
  // Delete functionality
  const deleteDialog = ref(false);
  const fileToDelete = ref('');
  const deletingFiles = ref({});
  
  // Fetch documents when the component is mounted
  onMounted(() => {
    // Fetch only if the list hasn't been loaded recently or is empty
    // Or always fetch if you want the latest on every page visit
    if (documentsStore.processedFiles.length === 0 || documentsStore.error) {
        documentsStore.fetchProcessedDocuments();
    }
  });
  
  // Function to trigger a manual refresh
  const refreshList = () => {
      documentsStore.fetchProcessedDocuments();
  }
  
  // Handle upload completion
  const onUploadComplete = (result) => {
    // Refresh the document list after successful upload
    if (result.successful && result.successful.length > 0) {
      documentsStore.fetchProcessedDocuments();
    }
  }
  
  // Delete file functions
  const confirmDelete = (filename) => {
    fileToDelete.value = filename;
    deleteDialog.value = true;
  }
  
  const deleteFile = async () => {
    if (!fileToDelete.value) return;
    
    const filename = fileToDelete.value; // Store filename before clearing
    console.log('Starting delete for file:', filename);
    deletingFiles.value[filename] = true;
    
    try {
      console.log('Calling ragApi.deleteFile...');
      const response = await ragApi.deleteFile(filename);
      console.log('Delete response:', response);
      
      // Refresh the document list
      console.log('Refreshing document list...');
      await documentsStore.fetchProcessedDocuments();
      
      // Close dialog
      deleteDialog.value = false;
      fileToDelete.value = '';
      
    } catch (error) {
      console.error('Failed to delete file:', error);
      console.error('Error details:', error.response?.data || error.message);
      // You might want to show an error message to the user here
      // For now, we'll just log it
    } finally {
      console.log('Cleaning up delete state for:', filename);
      deletingFiles.value[filename] = false;
    }
  }
  
  // Helper function to determine file icon based on extension
  const getFileIcon = (filename) => {
    if (!filename) return 'mdi-file-outline';
    const extension = filename.split('.').pop()?.toLowerCase();
    switch (extension) {
      case 'pdf':
        return 'mdi-file-pdf-box';
      case 'docx':
        return 'mdi-file-word-box-outline';
      default:
        return 'mdi-file-outline';
    }
  };
  </script>
  
  <style scoped>
  /* Layout */
  .documents-layout {
    background: #f8fafc;
    height: 100vh;
    overflow: hidden;
  }

  .documents-main {
    height: calc(100vh - 64px); /* Subtract app bar height */
    background: #f8fafc;
    overflow-y: auto;
  }

  .documents-container {
    background: #f8fafc;
    min-height: 100%;
    padding: 2rem;
  }

  /* Sidebar */
  .sidebar {
    background: #ffffff !important;
    border-right: 1px solid #e5e7eb !important;
  }

  .sidebar-header {
    padding: 1.5rem 1rem 1rem;
    border-bottom: 1px solid #f3f4f6;
  }

  .sidebar-list {
    padding: 0.5rem 0;
  }

  .sidebar-item {
    margin: 0.25rem 0.75rem;
    border-radius: 8px;
    transition: all 0.2s ease;
  }

  .sidebar-item:hover {
    background: #f9fafb !important;
  }

  .chat-link {
    background: #f8fafc !important;
    border: 1px solid #e5e7eb;
  }

  .chat-link:hover {
    background: #f3f4f6 !important;
    border-color: #d1d5db;
  }

  .knowledge-base-link.active {
    background: #eef2ff !important;
    border: 1px solid #c7d2fe;
    color: #3730a3 !important;
  }

  .knowledge-base-link.active:hover {
    background: #e0e7ff !important;
    border-color: #a5b4fc;
  }

  .sidebar-footer {
    padding: 1rem;
  }

  .refresh-docs-btn {
    border-color: #e5e7eb !important;
    color: #6b7280 !important;
    transition: all 0.2s ease;
  }

  .refresh-docs-btn:hover {
    border-color: #6366f1 !important;
    color: #6366f1 !important;
    background: #f8fafc !important;
  }

  /* App Bar */
  .app-bar {
    background: #ffffff !important;
    border-bottom: 1px solid #e5e7eb !important;
    color: #111827 !important;
  }

  .app-title {
    color: #111827 !important;
    font-weight: 600 !important;
  }

  .app-name {
    font-size: 1.25rem;
    font-weight: 700;
    color: #164f9a;
    letter-spacing: 0.025em;
  }

  .nav-icon {
    color: #6b7280 !important;
  }

  .logout-btn {
    color: #6b7280 !important;
    transition: all 0.2s ease;
  }

  .logout-btn:hover {
    background: #f9fafb !important;
    color: #111827 !important;
  }

  .documents-card {
    background: #ffffff;
    border: 1px solid #e5e7eb;
    overflow: hidden;
  }

  .documents-header {
    background: #ffffff;
    color: #1f2937 !important;
    padding: 1.5rem 2rem;
    border-bottom: 1px solid #e5e7eb;
  }

  .documents-header :deep(.v-chip) {
    background: #f3f4f6 !important;
    color: #6b7280 !important;
    border-color: #d1d5db !important;
  }

  .documents-subtitle {
    padding: 1rem 2rem 0.5rem;
    font-size: 1rem;
    color: #6b7280;
  }

  .search-section {
    padding: 1rem 2rem;
    background: #f9fafb;
    border-bottom: 1px solid #e5e7eb;
  }

  .search-input :deep(.v-field) {
    background: #ffffff;
    border-radius: 8px;
  }

  .search-input :deep(.v-field:focus-within) {
    border-color: #6366f1;
    box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.1);
  }

  .refresh-btn {
    background: #6366f1 !important;
    color: white !important;
    border: none;
    transition: all 0.2s ease;
  }

  .refresh-btn:hover {
    background: #4f46e5 !important;
    transform: rotate(90deg);
    box-shadow: 0 2px 8px rgba(99, 102, 241, 0.25);
  }

  .loading-state {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 4rem 2rem;
    background: #ffffff;
  }

  .empty-state {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 4rem 2rem;
    background: #ffffff;
  }

  .document-list {
    padding: 0;
  }

  .document-item {
    margin: 0.5rem 1.5rem;
    background: #ffffff;
    border: 1px solid #f3f4f6;
    border-radius: 8px;
    transition: all 0.2s ease;
  }

  .document-item:hover {
    background: #ffffff;
    border-color: #e5e7eb;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  }

  .document-title {
    font-weight: 600;
    color: #111827;
  }

  .delete-btn {
    opacity: 0.6;
    transition: all 0.2s ease;
  }

  .document-item:hover .delete-btn {
    opacity: 1;
  }

  .delete-btn:hover {
    background: #fef2f2 !important;
    color: #dc2626 !important;
  }

  .delete-dialog {
    background: #ffffff;
    border: 1px solid #e5e7eb;
  }

  .delete-dialog-header {
    background: #ffffff;
    color: #111827 !important;
    padding: 1.5rem 2rem;
    border-bottom: 1px solid #e5e7eb;
  }

  .delete-dialog-content {
    padding: 2rem;
  }

  .delete-dialog-actions {
    padding: 1rem 2rem 1.5rem;
  }

  .delete-confirm-btn {
    background: #dc2626 !important;
    color: white !important;
    transition: all 0.2s ease !important;
    font-weight: 600 !important;
  }

  .delete-confirm-btn:hover {
    background: #b91c1c !important;
    box-shadow: 0 4px 12px rgba(220, 38, 38, 0.25) !important;
  }

  /* List item transitions */
  .list-item-enter-active,
  .list-item-leave-active {
    transition: all 0.5s ease;
  }

  .list-item-enter-from {
    opacity: 0;
    transform: translateX(-30px);
  }

  .list-item-leave-to {
    opacity: 0;
    transform: translateX(30px);
  }

  .list-item-move {
    transition: transform 0.5s ease;
  }
  </style>
  
      
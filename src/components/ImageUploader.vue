<template>
  <div class="image-uploader">
    <div
      class="drop-zone"
      :class="{ 'drag-over': isDragOver, 'has-file': selectedFile }"
      @drop="handleDrop"
      @dragover.prevent="handleDragOver"
      @dragleave="handleDragLeave"
      @click="triggerFileInput"
    >
      <input
        ref="fileInput"
        type="file"
        accept="image/*"
        @change="handleFileSelect"
        style="display: none"
      />
      
      <div v-if="!selectedFile" class="drop-content">
        <div class="upload-icon">📁</div>
        <h3>Drop your image here</h3>
        <p>or <span class="link">click to browse</span></p>
        <small>Supports: JPG, PNG, TIFF, HEIC and more</small>
      </div>
      
      <div v-else class="file-preview">
        <div class="preview-container">
          <img
            v-if="previewUrl"
            :src="previewUrl"
            :alt="selectedFile.name"
            class="preview-image"
          />
          <div v-else class="file-icon">
            <div class="icon-placeholder">
              📷
            </div>
            <small>{{ getFileExtension(selectedFile.name).toUpperCase() }}</small>
          </div>
        </div>
        <div class="file-info">
          <h4>{{ selectedFile.name }}</h4>
          <p>{{ formatFileSize(selectedFile.size) }}</p>
          <p v-if="!previewUrl" class="preview-note">Preview not available for this format</p>
          <button @click.stop="clearFile" class="clear-btn">
            ✕ Remove
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, nextTick } from 'vue'

export default {
  name: 'ImageUploader',
  emits: ['file-selected', 'loading'],
  setup(props, { emit }) {
    const isDragOver = ref(false)
    const selectedFile = ref(null)
    const previewUrl = ref('')
    const fileInput = ref(null)

    const handleDrop = (e) => {
      e.preventDefault()
      isDragOver.value = false
      
      const files = e.dataTransfer.files
      if (files.length > 0) {
        processFile(files[0])
      }
    }

    const handleDragOver = (e) => {
      e.preventDefault()
      isDragOver.value = true
    }

    const handleDragLeave = () => {
      isDragOver.value = false
    }

    const handleFileSelect = (e) => {
      const files = e.target.files
      if (files.length > 0) {
        processFile(files[0])
      }
    }

    const processFile = (file) => {
      const maxFileSize = 50 * 1024 * 1024
      const supportedTypes = ['image/jpeg', 'image/jpg', 'image/png', 'image/tiff', 'image/heic', 'image/webp', 'image/gif']
      
      if (!file.type.startsWith('image/')) {
        alert('Please select an image file')
        return
      }

      if (!supportedTypes.includes(file.type.toLowerCase())) {
        alert('Unsupported image format. Please use JPG, PNG, TIFF, HEIC, WebP, or GIF.')
        return
      }

      if (file.size > maxFileSize) {
        alert('File is too large. Please select an image smaller than 50MB.')
        return
      }

      selectedFile.value = file
      
      // Handle HEIC files differently since browsers can't preview them natively
      if (file.type === 'image/heic' || file.name.toLowerCase().endsWith('.heic')) {
        previewUrl.value = null // Don't try to show preview for HEIC
      } else {
        const reader = new FileReader()
        reader.onload = (e) => {
          previewUrl.value = e.target.result
        }
        reader.onerror = () => {
          alert('Error reading file. Please try again.')
          clearFile()
        }
        reader.readAsDataURL(file)
      }

      emit('file-selected', file)
    }

    const triggerFileInput = () => {
      fileInput.value.click()
    }

    const clearFile = () => {
      selectedFile.value = null
      previewUrl.value = ''
      fileInput.value.value = ''
    }

    const formatFileSize = (bytes) => {
      if (bytes === 0) return '0 Bytes'
      const k = 1024
      const sizes = ['Bytes', 'KB', 'MB', 'GB']
      const i = Math.floor(Math.log(bytes) / Math.log(k))
      return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i]
    }

    const getFileExtension = (filename) => {
      return filename.split('.').pop() || ''
    }

    return {
      isDragOver,
      selectedFile,
      previewUrl,
      fileInput,
      handleDrop,
      handleDragOver,
      handleDragLeave,
      handleFileSelect,
      triggerFileInput,
      clearFile,
      formatFileSize,
      getFileExtension
    }
  }
}
</script>

<style scoped>
.image-uploader {
  margin-bottom: 30px;
}

.drop-zone {
  border: 2px dashed #ccc;
  border-radius: 12px;
  padding: 40px 20px;
  text-align: center;
  cursor: pointer;
  transition: all 0.3s ease;
  background: #fafafa;
}

.drop-zone:hover {
  border-color: #667eea;
  background: #f0f4ff;
}

.drop-zone.drag-over {
  border-color: #667eea;
  background: #e8f0ff;
  transform: scale(1.02);
}

.drop-zone.has-file {
  border-color: #28a745;
  background: #f8fff9;
}

.drop-content .upload-icon {
  font-size: 3rem;
  margin-bottom: 20px;
}

.drop-content h3 {
  color: #333;
  margin-bottom: 10px;
  font-size: 1.5rem;
}

.drop-content p {
  color: #666;
  margin-bottom: 10px;
}

.drop-content .link {
  color: #667eea;
  text-decoration: underline;
}

.drop-content small {
  color: #999;
  font-size: 0.9rem;
}

.file-preview {
  display: flex;
  align-items: center;
  gap: 20px;
  text-align: left;
}

.preview-container {
  flex-shrink: 0;
}

.preview-image {
  width: 100px;
  height: 100px;
  object-fit: cover;
  border-radius: 8px;
  border: 2px solid #ddd;
}

.file-icon {
  width: 100px;
  height: 100px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: #f8f9fa;
  border: 2px solid #ddd;
  border-radius: 8px;
  gap: 8px;
}

.icon-placeholder {
  font-size: 2rem;
}

.file-icon small {
  font-size: 0.7rem;
  font-weight: 600;
  color: #666;
  text-transform: uppercase;
}

.file-info h4 {
  color: #333;
  margin-bottom: 5px;
  font-size: 1.2rem;
}

.file-info p {
  color: #666;
  margin-bottom: 10px;
}

.preview-note {
  color: #999 !important;
  font-size: 0.9rem !important;
  font-style: italic;
}

.clear-btn {
  background: #dc3545;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: background-color 0.2s;
}

.clear-btn:hover {
  background: #c82333;
}

/* Mobile optimizations */
@media (max-width: 768px) {
  .drop-zone {
    padding: 30px 15px;
  }
  
  .drop-content .upload-icon {
    font-size: 2.5rem;
    margin-bottom: 15px;
  }
  
  .drop-content h3 {
    font-size: 1.3rem;
    margin-bottom: 8px;
  }
  
  .drop-content p {
    font-size: 0.95rem;
  }
  
  .file-preview {
    flex-direction: column;
    align-items: center;
    gap: 15px;
    text-align: center;
  }
  
  .preview-container {
    flex-shrink: 0;
  }
  
  .preview-image,
  .file-icon {
    width: 80px;
    height: 80px;
  }
  
  .file-info {
    width: 100%;
  }
  
  .file-info h4 {
    font-size: 1.1rem;
    word-break: break-all;
  }
  
  .file-info p {
    font-size: 0.9rem;
  }
  
  .clear-btn {
    padding: 10px 20px;
    font-size: 0.95rem;
  }
}
</style>
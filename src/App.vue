<template>
  <div id="app">
    <header>
      <h1>Image Meta Extractor</h1>
      <p>Upload an image to extract metadata information</p>
    </header>
    
    <main>
      <ImageUploader 
        @file-selected="handleFileSelected"
        @loading="isLoading = $event"
      />
      
      <MetaDisplay 
        v-if="metaData"
        :meta-data="metaData"
      />
      
      <div v-if="isLoading" class="loading">
        Extracting metadata...
      </div>
      
      <div v-if="error" class="error">
        {{ error }}
      </div>
    </main>
  </div>
</template>

<script>
import { ref } from 'vue'
import exifr from 'exifr'
import ImageUploader from './components/ImageUploader.vue'
import MetaDisplay from './components/MetaDisplay.vue'

export default {
  name: 'App',
  components: {
    ImageUploader,
    MetaDisplay
  },
  setup() {
    const metaData = ref(null)
    const isLoading = ref(false)
    const error = ref('')

    const handleFileSelected = async (file) => {
      error.value = ''
      metaData.value = null
      isLoading.value = true

      try {
        const meta = await exifr.parse(file, {
          exif: true,
          gps: true,
          xmp: true,
          iptc: true,
          icc: true
        })
        
        const basicInfo = {
          fileName: file.name,
          fileSize: formatFileSize(file.size),
          fileType: file.type,
          lastModified: new Date(file.lastModified).toLocaleString()
        }

        metaData.value = {
          basicInfo,
          exifData: meta || {}
        }
      } catch (err) {
        error.value = 'Failed to extract metadata: ' + err.message
      } finally {
        isLoading.value = false
      }
    }

    const formatFileSize = (bytes) => {
      if (bytes === 0) return '0 Bytes'
      const k = 1024
      const sizes = ['Bytes', 'KB', 'MB', 'GB']
      const i = Math.floor(Math.log(bytes) / Math.log(k))
      return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i]
    }

    return {
      metaData,
      isLoading,
      error,
      handleFileSelected
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
}

#app {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  min-height: 100vh;
}

header {
  text-align: center;
  margin-bottom: 40px;
  color: white;
}

header h1 {
  font-size: 2.5rem;
  margin-bottom: 10px;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
}

header p {
  font-size: 1.2rem;
  opacity: 0.9;
}

main {
  background: white;
  border-radius: 12px;
  padding: 30px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.2);
}

.loading {
  text-align: center;
  padding: 40px;
  font-size: 1.2rem;
  color: #667eea;
}

.error {
  background: #fee;
  color: #c33;
  padding: 15px;
  border-radius: 8px;
  margin: 20px 0;
  border: 1px solid #fcc;
}

/* Mobile optimizations */
@media (max-width: 768px) {
  #app {
    padding: 10px;
  }
  
  header {
    margin-bottom: 20px;
  }
  
  header h1 {
    font-size: 1.8rem;
    margin-bottom: 8px;
  }
  
  header p {
    font-size: 1rem;
  }
  
  main {
    padding: 20px 15px;
    border-radius: 8px;
  }
  
  .loading {
    padding: 30px 15px;
    font-size: 1.1rem;
  }
}
</style>
<template>
  <div class="meta-display">
    <h2>📊 Metadata Information</h2>
    
    <!-- Basic File Info -->
    <div class="meta-section">
      <h3>📁 File Information</h3>
      <div class="meta-grid">
        <div class="meta-item">
          <span class="label">File Name:</span>
          <span class="value">{{ metaData.basicInfo.fileName }}</span>
        </div>
        <div class="meta-item">
          <span class="label">File Size:</span>
          <span class="value">{{ metaData.basicInfo.fileSize }}</span>
        </div>
        <div class="meta-item">
          <span class="label">File Type:</span>
          <span class="value">{{ metaData.basicInfo.fileType }}</span>
        </div>
        <div class="meta-item">
          <span class="label">Last Modified:</span>
          <span class="value">{{ metaData.basicInfo.lastModified }}</span>
        </div>
      </div>
    </div>

    <!-- EXIF Data Sections -->
    <div v-if="cameraInfo.length" class="meta-section">
      <h3>📷 Camera Information</h3>
      <div class="meta-grid">
        <div v-for="item in cameraInfo" :key="item.key" class="meta-item">
          <span class="label">{{ item.label }}:</span>
          <span class="value">{{ item.value }}</span>
        </div>
      </div>
    </div>

    <div v-if="imageInfo.length" class="meta-section">
      <h3>🖼️ Image Properties</h3>
      <div class="meta-grid">
        <div v-for="item in imageInfo" :key="item.key" class="meta-item">
          <span class="label">{{ item.label }}:</span>
          <span class="value">{{ item.value }}</span>
        </div>
      </div>
    </div>

    <div v-if="locationInfo.length" class="meta-section">
      <h3>📍 Location Information</h3>
      <div class="meta-grid">
        <div v-for="item in locationInfo" :key="item.key" class="meta-item">
          <span class="label">{{ item.label }}:</span>
          <span class="value">{{ item.value }}</span>
        </div>
      </div>
      <div v-if="metaData.exifData.latitude && metaData.exifData.longitude" class="map-link">
        <a 
          :href="`https://www.google.com/maps?q=${metaData.exifData.latitude},${metaData.exifData.longitude}`"
          target="_blank"
          class="btn-link"
        >
          🗺️ View on Google Maps
        </a>
      </div>
    </div>

    <div v-if="dateInfo.length" class="meta-section">
      <h3>📅 Date Information</h3>
      <div class="meta-grid">
        <div v-for="item in dateInfo" :key="item.key" class="meta-item">
          <span class="label">{{ item.label }}:</span>
          <span class="value">{{ item.value }}</span>
        </div>
      </div>
    </div>

    <div v-if="otherInfo.length" class="meta-section">
      <h3>🔧 Technical Details</h3>
      <div class="meta-grid">
        <div v-for="item in otherInfo" :key="item.key" class="meta-item">
          <span class="label">{{ item.label }}:</span>
          <span class="value">{{ item.value }}</span>
        </div>
      </div>
    </div>

    <!-- Raw Data Toggle -->
    <div class="raw-data-section">
      <button @click="showRawData = !showRawData" class="toggle-btn">
        {{ showRawData ? '🔽 Hide' : '🔼 Show' }} Raw Data
      </button>
      
      <div v-if="showRawData" class="raw-data">
        <pre>{{ JSON.stringify(metaData.exifData, null, 2) }}</pre>
      </div>
    </div>
  </div>
</template>

<script>
import { computed, ref } from 'vue'

export default {
  name: 'MetaDisplay',
  props: {
    metaData: {
      type: Object,
      required: true
    }
  },
  setup(props) {
    const showRawData = ref(false)

    const formatValue = (value) => {
      if (value === null || value === undefined) return 'N/A'
      if (typeof value === 'number') return value.toString()
      if (typeof value === 'boolean') return value ? 'Yes' : 'No'
      if (Array.isArray(value)) return value.join(', ')
      return value.toString()
    }

    const cameraInfo = computed(() => {
      const exif = props.metaData.exifData
      const items = []
      
      if (exif.Make) items.push({ key: 'make', label: 'Make', value: exif.Make })
      if (exif.Model) items.push({ key: 'model', label: 'Model', value: exif.Model })
      if (exif.LensModel) items.push({ key: 'lens', label: 'Lens', value: exif.LensModel })
      if (exif.Software) items.push({ key: 'software', label: 'Software', value: exif.Software })
      if (exif.FocalLength) items.push({ key: 'focal', label: 'Focal Length', value: `${exif.FocalLength}mm` })
      if (exif.FNumber) items.push({ key: 'aperture', label: 'Aperture', value: `f/${exif.FNumber}` })
      if (exif.ExposureTime) items.push({ key: 'shutter', label: 'Shutter Speed', value: `1/${Math.round(1/exif.ExposureTime)}s` })
      if (exif.ISO) items.push({ key: 'iso', label: 'ISO', value: exif.ISO })
      if (exif.Flash) items.push({ key: 'flash', label: 'Flash', value: exif.Flash === 16 ? 'Off' : 'On' })
      
      return items
    })

    const imageInfo = computed(() => {
      const exif = props.metaData.exifData
      const items = []
      
      if (exif.ExifImageWidth) items.push({ key: 'width', label: 'Width', value: `${exif.ExifImageWidth}px` })
      if (exif.ExifImageHeight) items.push({ key: 'height', label: 'Height', value: `${exif.ExifImageHeight}px` })
      if (exif.XResolution) items.push({ key: 'xres', label: 'X Resolution', value: `${exif.XResolution} dpi` })
      if (exif.YResolution) items.push({ key: 'yres', label: 'Y Resolution', value: `${exif.YResolution} dpi` })
      if (exif.ColorSpace) items.push({ key: 'colorspace', label: 'Color Space', value: exif.ColorSpace === 1 ? 'sRGB' : 'Other' })
      if (exif.Orientation) items.push({ key: 'orientation', label: 'Orientation', value: exif.Orientation })
      
      return items
    })

    const locationInfo = computed(() => {
      const exif = props.metaData.exifData
      const items = []
      
      if (exif.latitude) items.push({ key: 'lat', label: 'Latitude', value: exif.latitude.toFixed(6) })
      if (exif.longitude) items.push({ key: 'lng', label: 'Longitude', value: exif.longitude.toFixed(6) })
      if (exif.GPSAltitude) items.push({ key: 'alt', label: 'Altitude', value: `${exif.GPSAltitude}m` })
      
      return items
    })

    const dateInfo = computed(() => {
      const exif = props.metaData.exifData
      const items = []
      
      if (exif.DateTimeOriginal) items.push({ key: 'taken', label: 'Date Taken', value: new Date(exif.DateTimeOriginal).toLocaleString() })
      if (exif.DateTime) items.push({ key: 'modified', label: 'Date Modified', value: new Date(exif.DateTime).toLocaleString() })
      if (exif.CreateDate) items.push({ key: 'created', label: 'Date Created', value: new Date(exif.CreateDate).toLocaleString() })
      
      return items
    })

    const otherInfo = computed(() => {
      const exif = props.metaData.exifData
      const items = []
      
      if (exif.WhiteBalance) items.push({ key: 'wb', label: 'White Balance', value: exif.WhiteBalance === 0 ? 'Auto' : 'Manual' })
      if (exif.MeteringMode) items.push({ key: 'metering', label: 'Metering Mode', value: exif.MeteringMode })
      if (exif.ExposureMode) items.push({ key: 'exposure', label: 'Exposure Mode', value: exif.ExposureMode })
      if (exif.SceneCaptureType) items.push({ key: 'scene', label: 'Scene Type', value: exif.SceneCaptureType })
      
      return items
    })

    return {
      showRawData,
      cameraInfo,
      imageInfo,
      locationInfo,
      dateInfo,
      otherInfo,
      formatValue
    }
  }
}
</script>

<style scoped>
.meta-display {
  margin-top: 30px;
}

.meta-display h2 {
  color: #333;
  margin-bottom: 30px;
  font-size: 1.8rem;
  text-align: center;
}

.meta-section {
  background: #f8f9fa;
  border-radius: 8px;
  padding: 20px;
  margin-bottom: 20px;
  border: 1px solid #e9ecef;
}

.meta-section h3 {
  color: #495057;
  margin-bottom: 15px;
  font-size: 1.3rem;
  border-bottom: 2px solid #dee2e6;
  padding-bottom: 8px;
}

.meta-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 12px;
}

.meta-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  background: white;
  border-radius: 6px;
  border: 1px solid #e9ecef;
}

.label {
  font-weight: 600;
  color: #495057;
  flex: 0 0 auto;
  margin-right: 15px;
}

.value {
  color: #212529;
  text-align: right;
  flex: 1;
  word-break: break-word;
}

.map-link {
  margin-top: 15px;
  text-align: center;
}

.btn-link {
  display: inline-block;
  padding: 10px 20px;
  background: #667eea;
  color: white;
  text-decoration: none;
  border-radius: 6px;
  font-weight: 500;
  transition: background-color 0.2s;
}

.btn-link:hover {
  background: #5a67d8;
}

.raw-data-section {
  margin-top: 30px;
  border-top: 2px solid #dee2e6;
  padding-top: 20px;
}

.toggle-btn {
  background: #6c757d;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 1rem;
  transition: background-color 0.2s;
  width: 100%;
}

.toggle-btn:hover {
  background: #5a6268;
}

.raw-data {
  margin-top: 15px;
  background: #f8f9fa;
  border: 1px solid #dee2e6;
  border-radius: 6px;
  padding: 15px;
  max-height: 400px;
  overflow-y: auto;
}

.raw-data pre {
  margin: 0;
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
  font-size: 0.9rem;
  line-height: 1.4;
  color: #495057;
}

@media (max-width: 768px) {
  .meta-grid {
    grid-template-columns: 1fr;
  }
  
  .meta-item {
    flex-direction: column;
    align-items: flex-start;
  }
  
  .value {
    text-align: left;
    margin-top: 5px;
  }
}
</style>
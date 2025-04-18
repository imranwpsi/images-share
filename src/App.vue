<template>
  <div class="app">
    <h1>Image Combiner & WhatsApp Share</h1>

    <!-- Image Upload -->
    <input type="file" accept="image/*" multiple @change="handleImageUpload" />

    <!-- Image Preview with Textareas -->
    <div v-if="images.length > 0" class="preview">
      <div v-for="(image, index) in images" :key="index" class="image-wrapper">
        <img :src="image" alt="Preview" />
        <textarea
          v-model="texts[index]"
          rows="2"
          maxlength="120"
          :placeholder="`Enter text for Image ${index + 1}`"
        ></textarea>
      </div>
    </div>

    <!-- Generate & Share Buttons -->
    <button @click="generateImage">Generate Image</button>
    <button v-if="combinedImage" @click="shareViaWhatsApp">Share on WhatsApp</button>

    <!-- Result Image -->
    <div v-if="combinedImage">
      <h2>Combined Image:</h2>
      <img :src="combinedImage" alt="Combined Image" />
    </div>

    <!-- Hidden Canvas -->
    <canvas ref="canvas" style="display: none"></canvas>
  </div>
</template>

<script>
export default {
  data() {
    return {
      images: [],
      texts: ['', '', '', ''],
      combinedImage: null,
    }
  },
  methods: {
    handleImageUpload(event) {
      const files = event.target.files
      if (files.length + this.images.length > 4) {
        alert('You can upload up to 4 images.')
        return
      }

      for (let file of files) {
        const reader = new FileReader()
        reader.onload = (e) => {
          if (this.images.length < 4) {
            this.images.push(e.target.result)
          }
        }
        reader.readAsDataURL(file)
      }
    },

    async generateImage() {
      if (this.images.length !== 4) {
        alert('Please upload exactly 4 images.')
        return
      }

      const canvas = this.$refs.canvas
      const ctx = canvas.getContext('2d')

      const imgSize = 600
      const textAreaHeight = 80

      canvas.width = imgSize * 2
      canvas.height = imgSize * 2

      ctx.font = '24px Arial'
      ctx.textAlign = 'center'
      ctx.fillStyle = 'white'

      for (let i = 0; i < this.images.length; i++) {
        const img = new Image()
        img.src = this.images[i]

        await new Promise((resolve) => {
          img.onload = () => {
            const x = (i % 2) * imgSize
            const y = Math.floor(i / 2) * imgSize

            // Draw image
            ctx.drawImage(img, x, y, imgSize, imgSize)

            // Draw semi-transparent background
            ctx.fillStyle = 'rgba(0, 0, 0, 0.5)'
            ctx.fillRect(x, y + imgSize - textAreaHeight, imgSize, textAreaHeight)

            // Set font and text style
            ctx.font = '22px Arial'
            ctx.textAlign = 'center'
            ctx.textBaseline = 'middle'
            ctx.fillStyle = 'white' // 👈 Ensure white text

            const text = this.texts[i]
            const rawLines = text
              .split('\n')
              .flatMap((line) => this.wrapText(ctx, line, imgSize - 40))
            const lines = rawLines.slice(0, 2) // Limit to 2 lines

            lines.forEach((line, idx) => {
              ctx.fillText(line, x + imgSize / 2, y + imgSize - textAreaHeight / 2 + idx * 24 - 12)
            })

            resolve()
          }
        })
      }

      this.combinedImage = canvas.toDataURL('image/png')
    },

    wrapText(ctx, text, maxWidth) {
      const words = text.split(' ')
      const lines = []
      let currentLine = words[0]

      for (let i = 1; i < words.length; i++) {
        const word = words[i]
        const width = ctx.measureText(currentLine + ' ' + word).width
        if (width < maxWidth) {
          currentLine += ' ' + word
        } else {
          lines.push(currentLine)
          currentLine = word
        }
      }
      lines.push(currentLine)
      return lines
    },

    shareViaWhatsApp() {
      const canvas = this.$refs.canvas
      this.copyImageToClipboard(canvas)
      setTimeout(() => {
        window.open(`https://web.whatsapp.com/send?text=`, '_blank')
      }, 1000)
    },

    async copyImageToClipboard(canvas) {
      try {
        canvas.toBlob(async (blob) => {
          await navigator.clipboard.write([new ClipboardItem({ 'image/png': blob })])
        }, 'image/png')
      } catch (error) {
        console.error('Failed to copy image:', error)
      }
    },
  },
}
</script>

<style>
.app {
  text-align: center;
  margin: auto;
  padding: 20px;
  width: 1280px;
  max-width: 100%;
}

input[type='file'] {
  margin: 10px 0;
  padding: 10px;
}

.preview {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 10px;
}

.image-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 24%;
}

.image-wrapper img {
  width: 120px;
  height: 120px;
  object-fit: cover;
  border-radius: 8px;
  margin-bottom: 5px;
}

.image-wrapper textarea {
  width: 100%;
  resize: none;
  padding: 5px;
  font-size: 14px;
  border-radius: 4px;
}

button {
  margin: 10px 5px;
  padding: 10px 15px;
  font-size: 16px;
}
</style>

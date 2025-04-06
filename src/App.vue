<template>
  <div class="app">
    <h1>Image Combiner & WhatsApp Share</h1>

    <!-- Image Upload -->
    <input type="file" accept="image/*" multiple @change="handleImageUpload" />

    <div v-if="images.length > 0" class="preview">
      <div v-for="(image, index) in images" :key="index">
        <img :src="image" alt="Preview" />
      </div>
    </div>

    <!-- Text Input -->
    <input v-model="customText" placeholder="Enter your text" />

    <!-- Generate & Share Buttons -->
    <button @click="generateImage">Generate Image</button>
    <button v-if="combinedImage" @click="shareViaWhatsApp">Share on WhatsApp</button>

    <!-- Result Image -->
    <div v-if="combinedImage">
      <h2>Combined Image:</h2>
      <img :src="combinedImage" alt="Combined Image" />
    </div>

    <canvas ref="canvas" style="display: none"></canvas>
  </div>
</template>

<script>
export default {
  data() {
    return {
      images: [],
      customText: '',
      combinedImage: null
    };
  },
  methods: {
    handleImageUpload(event) {
      const files = event.target.files;
      if (files.length + this.images.length > 4) {
        alert('You can upload up to 4 images.');
        return;
      }

      for (let file of files) {
        const reader = new FileReader();
        reader.onload = (e) => this.images.push(e.target.result);
        reader.readAsDataURL(file);
      }
    },

    async generateImage() {
      if (this.images.length < 4) {
        alert('Please upload exactly 4 images.');
        return;
      }

      const canvas = this.$refs.canvas;
      const ctx = canvas.getContext('2d');

      canvas.width = 1200;
      canvas.height = 1200;

      // Draw images in a 2x2 grid
      const imgSize = 600;
      for (let i = 0; i < this.images.length; i++) {
        const img = new Image();
        img.src = this.images[i];
        await new Promise((resolve) => {
          img.onload = () => {
            const x = (i % 2) * imgSize;
            const y = Math.floor(i / 2) * imgSize;
            ctx.drawImage(img, x, y, imgSize, imgSize);
            resolve();
          };
        });
      }

      // Text settings
      const backgroundHeight = 60;

      ctx.font = '40px Arial';
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle'; // important for vertical centering

      // Draw semi-transparent black rectangle as background (overlay on bottom part of canvas)
      ctx.fillStyle = 'rgba(0, 0, 0, 0.5)';
      ctx.fillRect(0, canvas.height - backgroundHeight, canvas.width, backgroundHeight);

      // Draw white centered text
      ctx.fillStyle = 'white';
      ctx.fillText(
        this.customText,
        canvas.width / 2,
        canvas.height - backgroundHeight / 2
      );

      this.combinedImage = canvas.toDataURL('image/png');
    },

    shareViaWhatsApp() {
      const canvas = this.$refs.canvas;

      // Copy image to clipboard
      this.copyImageToClipboard(canvas);

      setTimeout(() => {
        window.open(`https://web.whatsapp.com/send?text=`, '_blank');
      }, 1000);

      // const url = encodeURIComponent(this.combinedImage);
      // window.open(`https://wa.me/?text=https://admin.bdbou.com/uploads/product/images/thumbnail/1725273843598bdbou-blue-this-one%20(2).jpg`, '_blank');
      // window.open(`https://web.whatsapp.com/send?text=`, '_blank');
    },

    async copyImageToClipboard(canvas) {
      try {
        canvas.toBlob(async (blob) => {
          await navigator.clipboard.write([new ClipboardItem({ 'image/png': blob })]);
          // alert('Image copied to clipboard!');
        }, 'image/png');
      } catch (error) {
        console.error('Failed to copy image:', error);
      }
    },
  }
};
</script>

<style>
.app {
  text-align: center;
}
input {
  margin: 10px;
  width: 100%;
  padding: 15px;
}
.preview {
  display: flex;
  justify-content: center;
}
.preview img {
  width: 100px;
  height: 100px;
  margin: 5px;
}
button {
  margin: 10px;
  padding: 10px;
}
</style>

<template>
    <div>
      <div class="d-flex justify-content-center">
      <b-icon icon="camera" font-scale="2" class="my-5"></b-icon>
      <input type="file" class="my-5 ml-1" @change="uploadImage" accept=".jpg, .jpeg, .bmp, .png, .gif" />
      </div>
      <div class="d-flex justify-content-center">
      <p v-if="uploadStatus === 'success'" class="my-5 mx-5 text-success">Gambar berhasil diunggah</p>
      <p v-if="uploadStatus === 'error'" class="my-5 mx-5 text-danger">Upload error</p>
      <p v-if="uploadStatus === 'invalid'" class="my-5 mx-5 text-danger">Hanya boleh file gambar</p>
      <p v-if="uploadStatus === 'toolarge'" class="my-5 mx-5 text-danger">File gambar terlalu besar</p>
      </div>
      <div class="image-row">
      <div v-for="(image, index) in uploadedImages" :key="index" class="image-container">
        <img :src="image.url" alt="Uploaded Image" class="border border-5 border-black mx-2 " style="width: 20rem; height: 20rem;"/>
        <div v-if="index % 5 === 4" class="new-row" ></div>
      </div>
    </div>
    </div>
  </template>
  
  <script>
  import axios from 'axios'
  export default {
    name : 'AppImage',
    data() {
      return {
        uploadStatus: null,
        uploadedImages: [],
      };
    },
    methods: {
      async uploadImage(event) {
        const file = event.target.files[0];
        if (!file) return;
  
        if (!['image/jpeg', 'image/jpg', 'image/bmp', 'image/png', 'image/gif'].includes(file.type)) {
          this.uploadStatus = 'invalid';
          return;
        }
  
        if (file.size > 2 * 1024 * 1024) {
          this.uploadStatus = 'toolarge';
          return;
        }
  
        const formData = new FormData();
        formData.append('image', file);
  
        try {
          const response = await this.uploadToImgBB(formData);
          if (response.status === 200) {
            this.uploadStatus = 'success';
            const imageUrl = response.data.data.url;
            this.saveImageUrl(imageUrl);
            this.uploadedImages.push({ id: Date.now(), url: imageUrl });
          } else {
            this.uploadStatus = 'error';
          }
        } catch (error) {
          this.uploadStatus = 'error';
        }
      },
      uploadToImgBB(formData) {
        const apiKey = '28ca2677ebac7480f5af291fc8393af6';
        return axios.post(`https://api.imgbb.com/1/upload?key=${apiKey}`, formData);
      },
      saveImageUrl(imageUrl) {
        const storedImages = JSON.parse(localStorage.getItem('uploadedImages') || '[]');
        storedImages.push(imageUrl);
        localStorage.setItem('uploadedImages', JSON.stringify(storedImages));
      },
    },
    created() {
      const storedImages = JSON.parse(localStorage.getItem('uploadedImages') || '[]');
      this.uploadedImages = storedImages.map((url, id) => ({ id, url }));
    },
  };
  
  </script>
  
  <style scoped>
  .image-row {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
  }
  .image-container {
    margin: 5px;
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  .new-row {
    width: 100%;
  }
  </style>
 
  
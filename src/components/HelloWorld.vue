<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12">
        <v-img
          :src="require('../assets/logo.svg')"
          class="my-3"
          contain
          height="200"
        />
      </v-col>

      <v-col class="mb-4" cols="12">
        <h1 class="display-2 font-weight-bold mb-3">
          CV Read Image/Document To Text
        </h1>
      </v-col>

      <v-col
        class="mb-5"
        cols="12"
      >
        <v-row justify="center">
          <v-file-input ref="file1"   placeholder="Upload image"
            prepend-icon="mdi-camera"
            label="Image"></v-file-input>
        </v-row>
        <v-row justify="center">
          <v-col>
            <v-btn @click="handleUpload">Upload Image</v-btn>
          </v-col>
         
          <v-col>
            <v-btn @click="handleUploadDoc" color="info">Upload Docs</v-btn>
          </v-col>
        
        </v-row>
      </v-col>
      <v-col cols="12">
        <v-card
            elevation="5"
            shaped
          >
            <v-card-title>
              Result
            </v-card-title>
            <v-card-text v-html="Description">
            </v-card-text>
          </v-card>
      </v-col>
      <v-col cols="12">
        <v-list>
          <v-list-item v-for="item, index in response" :key="index">
            <v-list-item-content>
              {{ item }}
            </v-list-item-content>
            <v-divider></v-divider>
          </v-list-item>
        </v-list>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from 'axios';
export default {
  name: 'HelloWorld',

  data: () => {
    return {
      response: {},
      Description: '',
      previewImage: null
    }
  },
  methods: {
    async handleUploadDoc() {
      var formData = new FormData();
      var imagefile = this.$refs.file1
      formData.append("doc", imagefile.files[0]);
      await axios.post("https://localhost:44314/api/vision/document", formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      }).then(resp => {
        console.log('docs', resp.data);
      })
    },
    async handleUpload() {
      var formData = new FormData();
      var imagefile = this.$refs.file1
      formData.append("image", imagefile.files[0]);
      await axios.post("https://localhost:44314/api/vision/image", formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      }).then(resp => {
        resp.data.forEach((element,index) => {
          if (index == 0) this.Description = element.Description;
        });
        this.response = resp.data;
      })
    }
  }
}
</script>

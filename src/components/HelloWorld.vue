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
          Computer Vision Read Image/Document To Text
        </h1>
      </v-col>
      <v-col cols="12" align-self="center" v-if="previewFile != ''">
        <v-img ref="output" :src="previewFile" max-height="400" />
      </v-col>
      <v-col
        class="mb-5"
        cols="12"
      >
        <v-row justify="center">
          <v-file-input ref="file1"   placeholder="Upload file"
          @click:clear="previewFile = ''"
          @change="handlePreviewFile"
            counter
            show-size
            accept="image/*,.pdf, .tiff"
            label="Select .jpg, .jpeg, .png, .pdf, .tiff file"></v-file-input>
        </v-row>
        <v-row justify="center">
          <v-col>
            <v-btn @click="handleUpload">Upload Image</v-btn>
          </v-col>
         <v-col>
          <v-btn @click="handleUploadDocImage" color="error">Upload Docs/Image</v-btn>
         </v-col>
          <v-col>
            <v-btn @click="handleUploadDoc" color="info">Upload Docs</v-btn>
          </v-col>
        
        </v-row>
      </v-col>
      <v-col cols="12" v-if="respPdf.length == 0 && response.length > 0">
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
          <v-table>
            <thead>
              <tr>
                <th v-for="col in imageColumns" :key="col.value">{{ col.text }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="row,index in imageRows" :key="index">
                <td v-for="col in imageColumns" :key="col.value" style="text-align: left;">
                  {{ row[col.value] }}
                </td>
              </tr>
            </tbody>
          </v-table>
      </v-col>
      <v-col cols="12" v-if="respPdf.length > 0">
        <v-list>
          <v-list-item v-for="item, index in respPdf" :key="index">
              <v-card variant="outlined" class="mx-auto">
                <v-card-title>{{ item.context }}</v-card-title>
                <v-card-text>
                  {{ item.text }}
                </v-card-text>
                <v-divider></v-divider>
                <v-expansion-panels v-if="pdfRows.length > 0">
                  <v-expansion-panel elevation="20">
                  <v-expansion-panel-title>Paragraphs of page {{ item.context.pageNumber }}</v-expansion-panel-title>
                  <v-expansion-panel-text>
                    <v-table density="comfortable">
                      <thead>
                        <tr>
                          <th>Paragraphs</th>
                          <th>Word</th>
                        </tr>
                      </thead>
                      <tbody>
                        <tr v-for="row,rindex in pdfRows[index].para" :key="rindex">
                          <td>{{ rindex + 1 }}</td>
                          <td style="text-align: left;">
                            {{ row.word }}
                          </td>
                        </tr>
                      </tbody>
                    </v-table>
                  </v-expansion-panel-text>
                </v-expansion-panel>
                </v-expansion-panels>
              </v-card>
          </v-list-item>
        </v-list>
      </v-col>
    </v-row>
  </v-container>
  <v-dialog
      v-model="dialog"
      hide-overlay
      persistent
      width="300"
    >
      <v-card
        color="primary"
        dark
      >
        <v-card-text>
          Loading...
          <v-progress-linear
            indeterminate
            color="white"
            class="mb-0"
          ></v-progress-linear>
        </v-card-text>
      </v-card>
    </v-dialog>
</template>

<script>
import axios from 'axios';
export default {
  name: 'HelloWorld',

  data: () => {
    return {
      response: [],
      respPdf: [],
      Description: '',
      previewImage: null,
      dialog: false,
      previewFile: '',
      imageColumns: [
        // { text: 'Block (x,y)', value: 'blockValue'},
        // { text: 'Paragraph (x,y)', value: 'paragraph'},
        { text: 'Word', value: 'word'},
        { text: 'confidence', value: 'confidence'}
      ],
      imageRows: [],
      pdfRows: []
    }
  },
  methods: {
    handlePreviewFile() {
      this.previewFile = '';
      var reader = new FileReader();
      let self = this;
      reader.onload = function(){
        // var output = document.getElementById('output');
        self.previewFile = reader.result;
        // output.src = reader.result;
      };
      reader.readAsDataURL(this.$refs.file1.files[0]);
    },
    async handleUploadDoc() {
      this.respPdf = [];
      this.dialog = true;
      var formData = new FormData();
      var imagefile = this.$refs.file1
      formData.append("doc", imagefile.files[0]);
      await axios.post("https://localhost:44314/api/vision/document", formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      }).then(resp => {
        let data = resp.data.responses;
        console.log('docs', data);
        data.forEach(x => {
          let context = x.context;
          let text = x.fullTextAnnotation.text;
          let obj = {
            "context": context,
            "text": text
          }
          this.respPdf.push(obj);
          // split each page & paragraph
          x.fullTextAnnotation.pages.forEach(page => {
            let row = {
              page: context.pageNumber,
              para: []
            };
            page.blocks.forEach((block) => {
              block.paragraphs.forEach(p => {
                let paraRow = {};
                let wordArr = [];
                p.words.forEach(w => {
                  // compose Symbols into one word
                  wordArr.push(w.symbols.map(s => s.text).join(''));
                })
                paraRow.word = wordArr;
                paraRow.confidence = p.confidence;
                row.para.push(paraRow);
              }); // Paragraphs
            }); // block
            // page
            console.log('Paragraphs each page: ', row);
            this.pdfRows.push(row);
          });
        });
        this.dialog = false;
      }).catch(err => {  console.error(err); this.dialog = false;})
    },
    async handleUpload() {
      this.dialog = true;
      this.response = [];
      this.respPdf = [];
      var formData = new FormData();
      var imagefile = this.$refs.file1
      formData.append("image", imagefile.files[0]);
      await axios.post("https://localhost:44314/api/vision/image", formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      }).then(resp => {
        console.log('--> resp-img', resp.data);
        let arr = [];
        resp.data.DetectDocumentText.Pages.forEach(page => {
          page.Blocks.forEach((block) => {
            let box = block.BoundingBox.Vertices.map(v => {
              return `${v.X}, ${v.Y}`
            })
            // console.log(`(Block ${block.BlockType} at ${box}`);
            block.Paragraphs.forEach(p => {
              let obj = { paragraph: null, word: null, confidence: null};
              box = p.BoundingBox.Vertices.map(v => {
                return `${v.X}, ${v.Y}`
              })
              // console.log(`Paragraph at ${box}`);
              obj.paragraph = box;
              let wordArr = [];
              p.Words.forEach(w => {
                // compose Symbols into one word
                wordArr.push(w.Symbols.map(s => s.Text).join(''));
              })
              obj.word = wordArr;
              obj.confidence = p.Confidence;
              arr.push(obj);
            }); // Paragraphs
          }); // block
          // console.log('-->', arr);
          this.imageRows = arr;
        });
        // resp.data.data1.forEach((element,index) => {
        //   if (index == 0) this.Description = element.Description;
        // });
        this.Description = resp.data.DetectDocumentText.Text
        this.response = resp.data.DetectText.slice(1);
        this.dialog = false;
      }).catch(err => {  console.error(err); this.dialog = false;})
    },
    async handleUploadDocImage() {
      this.dialog = true;
      this.respPdf = [];
      this.dialog = true;
      var formData = new FormData();
      var imagefile = this.$refs.file1
      formData.append("image", imagefile.files[0]);
      await axios.post("https://localhost:44314/api/vision/batch/image", formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      }).then(resp => {
        console.log('--> resp-doc-to-img', resp.data);
        resp.data.responses.forEach((x, index) => {
          let obj = {
            "text": x.fullTextAnnotation.text,
            "context": {...x.context, pageNumber: (index +1)}
          }
          this.respPdf.push(obj);
          // split each page & paragraph
          x.fullTextAnnotation.pages.forEach(page => {
            let row = {
              page:  (index + 1),
              para: []
            };
            page.blocks.forEach((block) => {
              block.paragraphs.forEach(p => {
                let paraRow = {};
                let wordArr = [];
                p.words.forEach(w => {
                  // compose Symbols into one word
                  let symbols = w.symbols.map(s => s.text).join('');
                  wordArr.push(symbols);
                })
                paraRow.word = wordArr.join(" ");
                paraRow.confidence = p.confidence;
                row.para.push(paraRow);
              }); // Paragraphs
            }); // block
            // page
            // console.log('Paragraphs each page: ', row);
            this.pdfRows.push(row);
          });
        });
        this.dialog = false;
      }).catch(err => {  console.error(err); this.dialog = false;})
    }
  }
}
</script>

<style>
  .pre-formatted {
    white-space: pre-wrap;
  }
</style>
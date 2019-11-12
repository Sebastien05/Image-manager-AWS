<template>
  <v-app>
    <v-toolbar
      dark
      height="75"
      prominent
      src="https://cdn.vuetifyjs.com/images/backgrounds/vbanner.jpg"
    >
      <v-toolbar-title class="pb-4">
        AWS Manager 
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <div class="mt-5">
        Connected to: {{currBucket}}
      </div>
    </v-toolbar>
    <v-container style="height:100%;">
      <v-row
        style="height:100%;"
        align="center"
        justify="center"
      >
        <v-col cols="auto">
          <div class="text-center pb-10">
            <v-card
              shaped
              width="600"
              max-height="600"
            >
              <template v-if="loadingReady">
                <v-toolbar height="64" color="secondary" dark>
                  <v-toolbar-title class="pl-4">
                    Research your bucket
                  </v-toolbar-title>
                  <div class="flex-grow-1"></div>

                  <v-text-field
                    class="mt-6 pr-4"
                    prepend-icon="search"
                    label="Recherche"
                    dense
                    v-model="query"
                    style="width: 100px;"
                  ></v-text-field>
                </v-toolbar>
                <v-btn dark class="mt-12 mb-12" color="blue"
                  @click="getListObjects"
                  >
                  Open Bucket
                </v-btn>
              </template>

              <div v-else>
                <v-overlay
                  :absolute="absolute"
                  :opacity="0.3"
                  :value="loadingFiles"
                >
                  <v-container>
                    <v-row justify="center" align="center">
                      <v-progress-circular
                        :size="70"
                        :width="7"
                        color="white"
                        indeterminate
                      ></v-progress-circular>
                    </v-row>
                  </v-container>
                </v-overlay>
                <v-scale-transition style="width: 100%;" class="d-flex flex-wrap">
                  <v-list dense flat>
                    <v-subheader class="mt-4 mb-7 mr-7 subtitle-1 ml-5 d-flex "> 
                      List files ~ bucket: {{currBucket}}
                      <v-spacer></v-spacer>
                      <v-card
                        style="font-weight:italic;"
                        class="px-3 py-1">
                        File Number : {{listObjects.length}} 
                      </v-card>
                    </v-subheader>
                    <v-list-item 
                      class="file-list-item"
                      v-for="(file, i) in listObjects"
                      :key="i"
                    >
                      
                        <FileRow
                          :fileName="file"
                          :path='"https://n2mvpkp1y0.execute-api.eu-central-1.amazonaws.com/api/test?bucket=sebs3bucket"' 
                          v-on:delete-file-row='deleteFileRow'
                          v-on:read-raw-file='readRawFile'
                          :key="i"/>

                        <v-dialog v-model="dialog" max-width="700" persistent no-click-animation>
                          <v-card
                            min-height="100"
                          >
                            <div v-if="loadingFile">
                              <v-overlay
                                :absolute="absolute"
                                :opacity="0.4"
                                :value="loadingFile"
                              >
                                <v-container>
                                  <v-row justify="center" align="center">
                                    <v-progress-circular
                                      :size="70"
                                      :width="7"
                                      color="white"
                                      indeterminate
                                    ></v-progress-circular>
                                  </v-row>
                                </v-container>
                              </v-overlay>
                            </div>
                            <div v-else>  
                              <v-card-title class="headline">RawFile</v-card-title>
                              <v-card-text>
                                <div v-if="image">
                                 <img :src="image" class="ml-8" style="display:block;width:100%;max-width:600px">
                                </div>
                                <div v-else>
                                  {{currRawFile}}
                                </div>
                              </v-card-text>
                              <v-card-actions>
                                <v-spacer></v-spacer>
                                <v-btn color="green darken-1" text @click="clearImage">Exit</v-btn>
                              </v-card-actions>
                            </div>
                          </v-card>
                      </v-dialog>
                    </v-list-item>
                  </v-list>
                </v-scale-transition>
                <v-card-actions>
                  <template>
                    <v-file-input class="mt-5 ml-4" accept="image/*,audio/*,text/*" filled label="File input" v-model="fileToUpload" dense></v-file-input>
                    <!-- <v-text-field label="Select Image" @click='pickFile' v-model='imageName' prepend-icon='attach_file'></v-text-field>
                    <input
                      type="file"
                      style="display: none"
                      ref="image"
                      accept="image/*"
                      @change="newUpload"
                    > -->
                  </template>
                  <v-spacer></v-spacer>
                  <v-btn color="primary"
                    
                    text @click="imageRekognition">
                    Upload
                  </v-btn>
                  <v-btn color="primary"
                  @click="leaveCurrBucket"
                    class="mr-5"
                    text>
                    Exit
                  </v-btn>
                </v-card-actions>
              </div>
              <div>
                <v-dialog 
                  v-model="categoryDialog"
                  max-width="800" 
                  persistent 
                  no-click-animation
                >
                  <v-card>
                    <v-card-title class="subtitle-1">Upload to..</v-card-title>
                    <CategoryModal
                    :listLabels="listCategory"
                    :image="image"
                    v-on:leave-labels-modal="leaveLabelsModal"
                    v-on:validate-image-labels="uploadImageWithLabels"/>
                  </v-card>
                
                </v-dialog>
              </div>
            </v-card>
          </div>
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>

<script>

import FileRow from '@/components/FileRow';
import CategoryModal from '@/components/CategoryModal';

export default {
  components: {
      FileRow,
      CategoryModal
  },
  data () {
    return {
      dialog : false,
      currBucket : "sebs3bucket",
      listObjects : [],
      query : '', 
      currRawFile : null,
      isValid : false,
      loadingFiles : true,
      loadingFile : false,
      loadingReady : true,

      image: null,
      fileToUpload: null,
      imageName: null,
      listCategory: [],
      categoryDialog: false,
      
      absolute: true,
      format : require('./fileFormat.json'),
      path : "https://n2mvpkp1y0.execute-api.eu-central-1.amazonaws.com/api"
    }
  },
  methods: {
    getListObjects () {
      this.loadingReady = false;
      let link = this.path + '/test?bucket=' + this.currBucket;
      fetch (link).then(response => response.json())
      .then(data => {
        this.listObjects = data;
        this.loadingFiles = false
      })
    },
    leaveCurrBucket () {
      this.listObjects = [];
      this.loadingReady = true;
      this.loadingFiles = true;
    },
    readRawFile (fileName) {
      this.dialog = true;
      this.currRawFile = null;
      const extn = fileName.match(/\.[0-9a-z]+$/i)[0].slice(1);
      if( extn === "txt" || extn ==="png" || extn ==="jpg") {
        this.loadingFile = true;
        let link = this.path + '/readfile?bucket=' + this.currBucket +"&fileKey=" + fileName;
        fetch (link).then(response => response.json())
        .then(data => {
          this.currRawFile = data;
          if (extn === "png" || extn === "jpg") {
            this.image = data;
            this.currRawFile = ""
          }
          this.loadingFile = false
        })
      }
      else {
        if (extn === "png") ;//this.download (fileName);
        else
          this.currRawFile = "Read Raw file not available to "+extn+" files"
      }
    },
    deleteFileRow (fileName) {
      console.log(fileName);
      let link = this.path + '/deletion';
      fetch (link, {
        method: 'DELETE',
        body: JSON.stringify({'fileKey': fileName, 'bucket':this.currBucket})
      }).then(res => res.json())
      .then(data => {
        this.listObjects.splice(this.listObjects.indexOf(data.fileKey), 1)
      })
    },
    getBase64 (file) {
      var reader = new FileReader();
      return new Promise ((resolve, reject) => {
        reader.readAsDataURL(file);
        reader.onload = function () {
          resolve(reader.result);
        };
        reader.onerror = function (error) {
          reject(error);
        };
      })
    },

    leaveLabelsModal (){
      this.categoryDialog = false;
      this.listCategory = [];
    },

    async imageRekognition () {
      const extFile = this.fileToUpload.name.match(/\.[0-9a-z]+$/i)[0];

      if (this.fileToUpload && 
      ( extFile == ".jpg" ||
        extFile == ".png" ||
        extFile == ".jpeg")) {
        
        const base64Image  = await (this.getBase64(this.fileToUpload));
        const base64ImageF = base64Image.replace(/^data:image\/jpeg;base64,/,"");
        const link = this.path + '/base64-recognition';
        
        //console.log(base64Image)
        
        fetch (link, {
            method: 'POST',
            body: JSON.stringify(base64ImageF)
        })
        .then(res => res.json())
        .then(data => {
          console.log(data);
          this.image = base64Image;
          this.categoryDialog = true;
          this.listCategory = data.Labels;
        });
      }  
    },

    uploadImageWithLabels (labelsSelected) {
      console.log(labelsSelected);
    },

    async uploadFile () {

      //const base64Format = await (this.getBase64 (this.fileToUpload));
      //console.log(base64Format);
      const link = this.path + '/upload-up' +"?bucket=" + this.currBucket + "&fileName=" +  + this.fileToUpload.name;
      console.log(this.fileToUpload)
    
      if (this.fileToUpload) {  
        fetch (link, {
          method: 'POST',
          body: this.fileToUpload,
          headers: {'content-type': this.fileToUpload.type}
        })
        /*  method: 'POST',
          body: JSON.stringify({
            'key': this.fileToUpload.name,
            'bucket':this.currBucket,
            'body': this.fileToUpload.data})
        */
        .then(res => res.json())
        .then(data => {
          console.log(data);
          this.listObjects.push(this.fileToUpload.name);
          this.listObjects.sort()
          console.log(this.listObjects)
          })
      }
    },
    clearImage () {
      this.dialog = false;
      this.image = null;
    }
  }
};
</script>

<style scoped>

.file-list-item:hover {
  background-color: #e3e3e3;
  transition: all 0.4s;
}

</style>
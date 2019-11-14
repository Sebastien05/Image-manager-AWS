<template>
  <v-app>
    <v-toolbar
      dark
      height="75"
      prominent
      src="https://cdn.vuetifyjs.com/images/backgrounds/vbanner.jpg"
    >
      <v-toolbar-title class="pb-4 ml-3 title">
        e-PhotoClassifier
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <div class="mt-5">
        Connected to: {{currBucket}}
      </div>
    </v-toolbar>
    <v-snackbar
      v-model="snackbar"
    >
      {{ notificationMessage }}
      <v-btn
        color="primary"
        text
        @click="snackbar = false"
      >
        Close
      </v-btn>
    </v-snackbar>
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
                <div class="pt-1">
                    <v-subheader v-if="inFolder" class="mt-4 mb-7 mr-7 subtitle-1 ml-5 d-flex "> 
                      List files ~ folder: {{currFolder}}
                      <v-spacer></v-spacer>
                      <v-card
                        style="font-weight:italic;"
                        class="px-3 py-1">
                        File Number : {{listObjects.length}} 
                      </v-card>
                    </v-subheader>
                    <v-subheader v-else class="mt-4 mb-7 mr-7 subtitle-1 ml-5 d-flex "> 
                      List folder ~ bucket: {{currBucket}}
                      <v-spacer></v-spacer>
                      <v-card
                        style="font-weight:italic;"
                        class="px-3 py-1">
                        Folder Number : {{listFolder.length}} 
                      </v-card>
                    </v-subheader>
                </div>
                <v-scale-transition style="width: 100%;" class="d-flex flex-wrap">
                  <div class="object-list-container">
                    <v-list dense flat v-if="inFolder">
                      <v-list-item 
                        class="file-list-item"
                        v-for="(file, i) in listObjects"
                        :key="i"
                      >
                          <FileRow
                            :fileName="file"
                            :path="currFolder"
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
                                <v-card-title class="headline">Display Image</v-card-title>
                                <v-card-text>
                                  <div v-if="image">
                                  <img :src="image" class="ml-8" style="display:block;width:100%;max-width:600px">
                                  </div>
                                  <div v-else>
                                    {{currRawTextFile}}
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
                    <v-list flat v-else>
                      <v-list-item 
                        class="file-list-item"
                        v-for="(folder, i) in listFolder"
                        :key="i"
                      >
                        <FolderRow
                          :folderName="folder"
                          :fileNumber="treeFile[folder].length"
                          v-on:select-folder="selectFolder"
                          :key="i"
                        />
                      </v-list-item>
                    </v-list>
                  </div>
                </v-scale-transition>
                <v-card-actions v-if="!inFolder">
                  <template>
                    <v-file-input class="pt-5 ml-2" style="width: 190px;" 
                      accept="image/*" filled label="File input"
                      v-model="fileToUpload" dense
                    >
                    </v-file-input>
                  </template>
                  <v-spacer></v-spacer>
                  <v-switch
                    dense
                    class="mr-7"
                    color="primary"
                    v-model="autoTidyingActivator" 
                    label="Auto Tidying" 
                  >
                  </v-switch>
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
                <v-card-actions v-else>
                  <v-spacer></v-spacer>
                  <v-btn color="primary"
                  @click="leaveCurrFolder"
                    class="mr-5"
                    text>
                    Back
                  </v-btn>
                </v-card-actions>
              </div>
            </v-card>
          </div>
        </v-col>
      </v-row>
    </v-container>
    <v-dialog 
      v-model="categoryDialog"
      max-width="800" 
      persistent
    >
        <CategoryModal
          :listLabels="listCategory"
          :image="image"
          :allLabels="listFolder"
          v-on:leave-labels-modal="leaveLabelsModal"
          v-on:validate-image-labels="uploadImageWithLabels"
        />
    
    </v-dialog>
  </v-app>
</template>

<script>

import FileRow from '@/components/FileRow';
import CategoryModal from '@/components/CategoryModal';
import FolderRow from '@/components/FolderRow';

export default {
  components: {
      FileRow,
      CategoryModal,
      FolderRow
  },
  data () {
    return {
      dialog : false,
      currBucket : "sebs3bucket",
      listObjects : [],
      treeFile: null,
      listFolder: [],
      inFolder: false,
      currFolder: null,
      
      query : '',
      currRawTextFile : null,
      isValid : false,
      loadingFiles : true,
      loadingFile : false,
      loadingReady : true,

      image: null,
      fileToUpload: null,
      listCategory: [],
      autoSelectedLabels: [],
      categoryDialog: false,

      autoTidyingActivator: false,
      notificationMessage: "",
      snackbar: false,
      
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
        this.listObjects  = data.lfn;
        // Array of all bucket file   :> Key = filePathName : Value = File
        this.treeFile     = data.ft;
        // formatted object           :> Key = folderName   : Value = Array [File]
        this.listFolder   = Object.keys(data.ft);
        // Array of all folder Name
        this.loadingFiles = false
      })
    },
    leaveCurrBucket () {
      this.loadingReady = true;
      this.loadingFiles = true;
    },
    leaveCurrFolder () {
      // When leaving a folder, need to reset properties and reload all data in case there has been a change elsewhere
      this.listObjects = [];
      this.inFolder = false;
      this.loadingFiles = true;
      this.getListObjects();

    },
    selectFolder (folderName) {
      // build the file array to display when entering a folder
      let arrImg = [];
      const folder= this.treeFile[folderName];
      for(let i in folder){
        arrImg.push(folder[i].Key);
      }
      this.listObjects = arrImg;
      this.inFolder = true;
      this.currFolder = folderName;
    },
    readRawFile (fileName) {
      // Read a txt or image file
      this.dialog = true;
      this.currRawTextFile = null;
      const extn = fileName.match(/\.[0-9a-z]+$/i)[0].slice(1);
      // check the file extension
      if( extn === "txt" || extn ==="png" || extn ==="jpg" || extn ==="jpeg") {
        this.loadingFile = true;
        let link = this.path + '/readfile?bucket=' + this.currBucket +"&fileKey=" + fileName;
        fetch (link).then(response => response.json())
        .then(data => {
          this.currRawTextFile = data;
          if (extn === "png" || extn === "jpg" || extn ==="jpeg") {
            this.image = data;
            this.currRawTextFile = ""
          }
          this.loadingFile = false
        })
      }
      else {
        this.currRawTextFile = "Read Raw file not available to "+extn+" files"
      }
    },
    deleteFileRow (fileName) {
      // Delete a file from a folder
      const pathFile = this.currFolder+'/'+fileName
      let link = this.path + '/deletion';
      fetch (link, {
        method: 'DELETE',
        body: JSON.stringify({'fileKey': pathFile, 'bucket':this.currBucket})
      }).then(res => res.json())
      .then(() => {
        // Reactivity of deletion
        this.listObjects.splice(this.listObjects.indexOf(fileName), 1)
        const arrTreeFile = this.treeFile[this.currFolder]
        for(let i in arrTreeFile){
          if (arrTreeFile[i].Key == fileName){
            arrTreeFile.splice(i,1);
            break;
          }
        }
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

    leaveLabelsModal () {
      // Reset properties when leaving label modal 
      this.categoryDialog = false;
      this.listCategory = [];
    },

    async imageRekognition () {
      // fetch recognition method to get differents labels
      if (!this.fileToUpload) return
      const extFile = this.fileToUpload.name.match(/\.[0-9a-z]+$/i)[0];

      if (this.fileToUpload && 
      ( extFile == ".jpg" ||
        extFile == ".png" ||
        extFile == ".jpeg")) {
        
        const base64Image  = await (this.getBase64(this.fileToUpload));
        // Need to delete the base64 header in order to be able to use recognition
        const base64ImageF = base64Image.replace(/^data:image\/jpeg;base64,/,"");
        const link = this.path + '/base64-recognition';

        this.loadingFiles = true;
        
        fetch (link, {
            method: 'POST',
            body: JSON.stringify(base64ImageF)
        })
        .then(res => res.json())
        .then(data => {
          this.image = base64Image;
          this.listCategory = data.Labels;
          if (this.autoTidyingActivator) {
            this.directAutoUpload();
          }
          else {
            this.categoryDialog = true;
          }
          this.loadingFiles = false;
        });
      }  
    },
    async directAutoUpload () {
      let similareLabels = []
      if (this.listCategory !== undefined){
        await this.listCategory.forEach(label => {
            if (this.listFolder.includes(label.Name) && label.Confidence>80)
                    similareLabels.push({Name:label.Name});
        });
        if (similareLabels.length>0){
          // console.log(similareLabels)
          this.uploadImageWithLabels(similareLabels)
        }else {
          if(this.listCategory.length>0){
            this.notificationMessage = "No existing folder corresponding, pass to manual";
            this.snackbar = true;
          }
        }
      }
      this.notificationMessage = "No label detected";
      this.snackbar = true;
      
    },
    async uploadImageWithLabels (labels) {
      // Upload one image in several category chosen 
      if (this.fileToUpload) {
        let base64Image  = await (this.getBase64(this.fileToUpload));
        for(let i in labels) {
          let link = this.path  + '/upload';
          fetch (link, {
            method: 'POST',
            body: JSON.stringify({
              'key': labels[i].Name + '/' + this.fileToUpload.name,
              'bucket':this.currBucket,
              'body': base64Image
            })
          })
          .then(res => res.json())
          .then(() => {
            // Reactivity of addition
            // this.treeFile[labels[i].Name].push(this.fileToUpload.name);
            // this.treeFile[labels[i].Name].sort()
            })
        }
        this.loadingFiles = true;
        this.getListObjects();
        let arrLabels = [];
        for(const i in labels){
          arrLabels.push(labels[i].Name)
        }
        this.notificationMessage = `Image upload in ${arrLabels}` ;
        this.snackbar = true,
        this.listCategory = [];
        this.fileToUpload = null;
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
  .object-list-container {
    max-height: 300px;
    overflow-y: auto;
  }
</style>
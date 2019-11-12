<template>
    <v-hover v-slot:default="{ hover }">
        <v-list-item :class="{ 'file-row':true}">
            <div style="width: 45px; margin-right: 25px; ">
                <v-icon>
                    <!-- {{format[fileName.match(/\.[0-9a-z]+$/i)[0].slice(1)]}} -->
                </v-icon>
            </div>
            
            <v-list-item-content cols="12" sm="6">
                <v-list-item-title v-html="fileName"></v-list-item-title>
            </v-list-item-content>

            <v-menu :offset-x="offset" bottom right transition="scale-transition">
                <template v-slot:activator="{ on }">
                <v-btn
                    color="primary"
                    icon
                    v-on="on"
                >
                    <v-icon> mdi-dots-vertical</v-icon>
                </v-btn>
                </template>

                <v-list flat dense>
                    <v-list-item @click="readFile">
                        <v-list-item-icon><v-icon color="secondary">open_in_new</v-icon></v-list-item-icon>
                        <v-list-item-title>Read in raw format</v-list-item-title>
                    </v-list-item>
                    <v-list-item @click="downloadFile">
                        <v-list-item-icon><v-icon color="blue">cloud_download</v-icon></v-list-item-icon>
                        <v-list-item-title>Download this file</v-list-item-title>
                    </v-list-item>
                    <v-list-item @click="showDeleteModal = true">
                        <v-list-item-icon><v-icon color="#b30000">close</v-icon></v-list-item-icon>
                        <v-list-item-title>Delete this file</v-list-item-title>
                    </v-list-item>
                </v-list>
            </v-menu>

            <v-dialog v-model="showDeleteModal" max-width="290">
                <v-card>
                    <v-card-title class="subtitle-1">Do you really want to delete this file ?</v-card-title>
                    <v-card-actions>
                        <div class="flex-grow-1"></div>
                        <v-btn color="secondary" text @click="showDeleteModal = false">Annuler</v-btn>
                        <v-btn color="secondary" text @click="deleteFile">Valider</v-btn>
                    </v-card-actions>
                </v-card>
            </v-dialog>

        </v-list-item>
    </v-hover>
</template>

<script>
export default {
    props: ['fileName','path'],
    data () {
        return {
            format : require('../fileFormat.json'),
            showDeleteModal: false,
            offset: true,

        }
    },
    methods: {
        readFile () {
            this.$emit('read-raw-file',this.path+'/'+this.fileName);
            console.log("read :: "+this.path+'/'+this.fileName);
        },
        downloadFile () {
            console.log("download :: "+this.fileName);
        },
        deleteFile () {
            this.$emit('delete-file-row',this.fileName);
            this.showDeleteModal = false
        }
    }
}
</script>

<style>

    .file-row {
        transition: 0.3s all;
    }

</style>
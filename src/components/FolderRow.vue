<template>
    <v-hover v-slot:default="{ hover }">
        <v-list-item :class="{ 'file-row':true}" style="cursor: pointer;" @click.native="selectFolder">
            <div style="width: 45px; margin-right: 25px; "/>
            
            <v-list-item-content cols="12" sm="6">
                <v-list-item-title v-html="folderName"></v-list-item-title>
            </v-list-item-content>

        </v-list-item>
    </v-hover>
</template>

<script>
export default {
    props: ['folderName'],
    methods: {
        selectFolder () {
            this.$emit('select-folder',this.folderName);
        }
    }
}
</script>
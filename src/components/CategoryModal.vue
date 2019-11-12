<template>
  <div>
    <div class="d-flex">
        <v-card-text style="max-height: 300px; max-width: 300px; overflow: auto;">
            <v-checkbox
                v-for="(label, i) in listLabels"
                :key="i"
                v-model="selectedLabels" 
                class="ml-4" 
                :label="label.Name+' / '+ label.Confidence.toFixed(2)+' %'"
                :value="label"
            >    
            </v-checkbox>
        </v-card-text>
        <img :src="image" class="ml-8" style="display:block;width:100%;max-width:440px;max-height:300px;">
    </div>

    <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn
            @click="addImageLabels"
            class=" py-5"
            text>
            Submit 
        </v-btn>
        <v-btn
            @click="leaveLabelsModal"
            class="mr-1 py-5"
            text>
            Exit 
        </v-btn>
    </v-card-actions>
  </div>
</template>

<script>

export default {

    props: ['listLabels','image'],

    data () {
        return {
            selectedLabels: []
        }
    },
    methods: {
        leaveLabelsModal () {
            this.$emit('leave-labels-modal');
        },
        addImageLabels () {
            this.$emit('validate-image-labels', this.selectedLabels)
            this.$emit('leave-labels-modal');
        }
    }
}
</script>
<template>
    <div>
        <template v-if="(picture == null)">
            <v-btn variant="plain" flat icon @click="showIconFormDialog">
                <v-icon>mdi-image-plus</v-icon>
            </v-btn>
        </template>
        <template v-else>
            <v-img class="bg-white" width="50" :aspect-ratio="1" :src="picture" cover @click="showIconFormDialog"></v-img>
        </template>
        
        <v-dialog v-model="icon_dialog" persistent max-width="500px">
            <v-card>
                <v-card-title>
                    <span class="text-h5">Upload Icon</span>
                </v-card-title>
                <v-card-text>
                    <v-container>
                        <v-row>
                            <v-col cols="12">
                                <v-file-input accept="image/*" label="Icon" v-model="files"
                                    :error="v$.files.$dirty && v$.files.$error"
                                    :rules="[
                                        !v$.files.required.$invalid || 'This field is required',
                                    ]"></v-file-input>
                            </v-col>
                        </v-row>
                    </v-container>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue-darken-1" variant="text" @click="closeIconFormDialog">
                        Close
                    </v-btn>
                    <v-btn color="blue-darken-1" variant="text" @click="handleIconFormSubmit">
                        Save
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
    </div>
</template>

<script setup lang="ts">
    import type { Ref } from 'vue'
    import { ref, computed } from 'vue'
    import { useVuelidate, ValidationRule } from '@vuelidate/core'
    import { required, requiredIf } from '@vuelidate/validators'

    const props = defineProps({
        url: { type: String, required: true },
        picture: { type: String, default: null },
        loading: { type: Boolean, default: false },
    })
    
    const emit = defineEmits(['update'])

    let icon_dialog = ref(false)
    let component_loading = ref(false)
    let files: Ref<File[]> = ref([])
    const rules = computed(() => {
        return {
            files: { required: requiredIf(() => {
                return files.value.length == 0;
            }) }
        }
    })
    const v$ = useVuelidate(rules, { files })

    let showIconFormDialog = function (vision: any) {
        icon_dialog.value = true
        files.value = []
        v$.value.files.$reset()
    }
    let closeIconFormDialog = function () {
        icon_dialog.value = false
    }
    let handleIconFormSubmit = async function() {
        v$.value.files.$validate()
        if(!v$.value.files.$invalid){
            icon_dialog.value = false
            component_loading.value = true
            const response = await window.api.post(`${props.url}`, { file: files.value[0] }, {
                headers: {
                    'Content-Type': 'multipart/form-data'
                }
            })
            component_loading.value = false
            emit('update', response.data.data)
        }
    }
</script>
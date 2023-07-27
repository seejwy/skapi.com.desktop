<template lang="pug">
.overlayContainer(v-if="service" :loading="isDisabled || null")
    form(@submit.prevent="save404" @keydown.enter.prevent="" action="")
        .overlayContainerTitle Additional Settings
        .input
            label 404 file
            sui-input(type="text" placeholder="Enter path eg. /404.html" :value="errorFile" @input="(e) => errorFile = e.target.value")
            .error {{ errorMessage }}
        sui-button.textButton(type="button" style="margin-right: 16px;" @click="emit('close', '')") Cancel
        SubmitButton(:loading="isDisabled") Save
</template>
<!-- script below -->
<script setup>
import { inject, ref, watch } from 'vue';
import { state, skapi } from '@/main';

import Icon from '@/components/Icon.vue';
import SubmitButton from '@/components/SubmitButton.vue';

const emit = defineEmits(['close']);

let service = inject('service');
let isDisabled = ref(false);
const errorFile = ref(service.value[404]);
const errorMessage = ref('');

watch(() => service.value[404], () => {
    errorFile.value = service.value[404];
});

const save404 = () => {
    errorMessage.value = '';
    isDisabled.value = true;
    let filePath = service.value.subdomain + '/' + errorFile.value;
    if (!errorFile.value) {
        filePath = null;
    }

    skapi.set404(service.value.service, filePath).then((res) => {
        if (service.value[404]) {
            skapi.deleteHostFile({
                keys: [service.value.subdomain + '/.cfacdb7c8270a90aba6011585793dfc3/'],
                service: service.value.service
            });
        }
        if (filePath) {
            const blob = new Blob([errorFile.value], { type: 'text/plain' });
            const file = new File([blob], '.cfacdb7c8270a90aba6011585793dfc3');

            let formData = new FormData();
            formData.append('.cfacdb7c8270a90aba6011585793dfc3/' + errorFile.value, file, '.cfacdb7c8270a90aba6011585793dfc3/' + errorFile.value);

            skapi.uploadFiles(formData, {
                service: service.value.service,
                request: 'host',
                progress: (e) => {
                    if (e.progress === 100) {
                        service.value[404] = errorFile.value;
                        isDisabled.value = false;
                    }
                }
            });
        } else {
            delete service.value[404];
            isDisabled.value = false;
        }
        emit('close', '');
    }).catch((err) => {
        console.log(err);
        isDisabled.value = false;
        errorMessage.value = "File does not exist!";
        throw err;
    })
}
</script>

<style lang="less" scoped>
.overlayContainer {
    background-color: #505050;
    border: 1px solid #808080;
    box-shadow: 4px 4px 12px rgba(0, 0, 0, 0.25);
    color: #fff;
    text-align: center;
    padding: 40px;
    width: 500px;
    max-width: 100%;
    border-radius: 8px;

    &Title {
        font-weight: bold;
        font-size: 28px;
        margin-bottom: 40px;
    }

    .input {
        margin: 20px 0 40px 0;

        label {
            display: block;
            text-align: left;
            font-weight: bold;
            color: rgba(255, 255, 255, .6);
            margin-bottom: 8px;
            // font-size: 0;
        }

        sui-input {
            width: 100%;
        }

        .error {
            margin-top: 10px;
            text-align: left;
            color: red;
        }
    }
}

.toggle {
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-weight: bold;
    color: rgba(255, 255, 255, .6);

    &Bar {
        position: relative;
        height: 8px;
        width: 40px;
        background: rgba(0, 0, 0, 0.6);
        border: 0.3px solid #595959;
        box-shadow: inset 0.5px 0.5px 1px rgba(0, 0, 0, 0.25);
        border-radius: 2px;
        margin-right: 10px;
    }

    &Ball {
        position: absolute;
        cursor: pointer;
        height: 20px;
        width: 20px;
        left: -10px;
        top: -7px;
        border-radius: 10px;
        background: #D9D9D9;
        border: 0.3px solid #595959;
        box-shadow: 0.5px 0.5px 1px rgba(0, 0, 0, 0.25), inset -0.5px -0.5px 1px rgba(0, 0, 0, 0.25), inset 1px 1px 1px #FFFFFF;
        transition: left .3s ease-in-out, background-color .3s ease-in-out;

        &.active {
            left: calc(100% - 10px);
            background: #5AD858;
        }
    }
}

hr {
    opacity: 8%;
}</style>
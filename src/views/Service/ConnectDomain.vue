<template lang="pug">
.overlayContainer(v-if="service" :loading="isDisabled || null")
    form(@submit.prevent="func")
        .overlayContainerTitle Connect Subdomain
        .input
            label Name
            sui-input(type="text" placeholder="Name of Service" :value="domainName" @input="(e) => domainName = e.target.value" required)
        .input
            label Value
            sui-input(type="text" :value="value" @input="(e) => value = e.target.value" required @change="validateCors")
        sui-button.textButton(type="button" style="margin-right: 16px;" @click="emit('close', '')") Cancel
        SubmitButton(:loading="isDisabled") Save
</template>
<!-- script below -->
<script setup>
import { ref } from 'vue';

const emit = defineEmits(['close']);
let isDisabled = ref(false);

const func = () => {
    return new Promise((res, rej) => {
        setTimeout(() => res(true), 1000);
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
        margin: 20px auto;

        label {
            display: block;
            text-align: left;
            font-weight: bold;
            color: rgba(255, 255, 255, .6);
            margin-bottom: 8px;
        }
        sui-input {
            width: 100%;
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
}
</style>
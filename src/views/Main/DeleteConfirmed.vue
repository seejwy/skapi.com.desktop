<template lang="pug">
.wrapper
    .container
        h1 Account Deleted
        p We are sad to see you go. Your account has been deleted at your request. Thank you again for being a part of Skapi.
        p We hope to see you again!
        RouterLink(to="/") Return to Home
        .logo
</template>

<script setup>
import { inject } from 'vue';
import { useRouter } from 'vue-router';

let router = useRouter();
// set page title
let pageTitle = inject('pageTitle');
pageTitle.value = 'skapi';
</script>

<style lang="less" scoped>
.wrapper {
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 60px 0;
    min-height: calc(100vh - 140px);
}
.container {
    text-align: center;
    padding: 40px;
    background: #FAFAFA;
    border-radius: 8px;
    width: 542px;
    max-width: 100%;
    border: 1px solid #808080;
    box-shadow: 4px 4px 12px rgba(0, 0, 0, 0.25);
    border-radius: 8px;
    margin-top: 60px;

    h1 {
        font-size: 32px;
        margin: 20px 0 40px 0;
    }

    p {
        margin: 40px auto;
        line-height: 1.5;
    }
}
.logo {
    -webkit-mask: url(@/assets/img/logo.svg) no-repeat center;
    mask: url(@/assets/img/logo.svg) no-repeat center;
    background-color: var(--primary-color);
}
a {
    display: block;
    color: var(--primary-color);
    font-weight: bold;
    text-decoration: none;
    margin: 40px auto;
}
</style>
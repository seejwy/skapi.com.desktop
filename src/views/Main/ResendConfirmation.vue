<template lang="pug">
.wrapper
    .container
        h1 Confirm Your Email
        p Please check your inbox at #[span(style="color: var(--primary-color)") {{ email }}] for a confirmation email. Click the link in the email to confirm your email address. 
        p(style="color: var(--primary-color)") Continue to login after confirmation.
        p(style="text-align: left; ") Haven't got any code?
        sui-button.lineButton(type="button" @click="resendSignupConfirmation" :disabled="secondsTillReady || null") 
            template(v-if="secondsTillReady") Email has been sent
            template(v-else) Re-send Confirmation Email
</template>

<script setup>
import { inject, ref } from 'vue';
import { skapi } from '@/main';
import { useRouter } from 'vue-router';

let router = useRouter();
const secondsTillReady = ref(null);

let urlParams = new URLSearchParams(window.location.search);
let email = ref(urlParams.get('email'));

if(urlParams.size && email.value) {
    let uri = window.location.toString();
    if (uri.indexOf("?") > 0) {
        let clean_uri = uri.substring(0, uri.indexOf("?"));
        window.history.replaceState({}, document.title, clean_uri);
    }
}

// set page title
let pageTitle = inject('pageTitle');
pageTitle.value = 'skapi';

async function resendSignupConfirmation() {
    if(secondsTillReady.value !== null) return false;

    secondsTillReady.value = 30;
    let countDown = setInterval(() => {
        if(secondsTillReady.value > 0) secondsTillReady.value--;
        else {
            secondsTillReady.value = null;
            clearInterval(countDown);
        }

    }, 1000);
    try {
        let x = await skapi.resendSignupConfirmation('/success');
    } catch(e) {
        console.log({e: e});
        if(e.code === 'INVALID_REQUEST') {
            router.replace('/admin');
        }
    }
}
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

    & > * {
        width: 100%;
    }

    h1 {
        font-size: 32px;
        margin: 0 0 20px 0;
    }

    P {
        margin: 40px auto 8px auto;
        line-height: 1.5;
    }

    .input {
        margin: 20px auto 12px;

        label {
            display: block;
            text-align: left;
            font-weight: bold;
            color: rgba(0, 0, 0, 0.65);
            margin-bottom: 8px;
        }
        sui-input {
            width: 100%;
        }
    }
    .action {
        display: flex;
        justify-content: space-between;
        margin-bottom: 20px;

        label {
            cursor: pointer;

            sui-input {
                vertical-align: middle;
                margin-right: 4px;
            }

            span {
                vertical-align: middle;
                color: rgba(0, 0, 0, 0.65);
            }
        }

        a {
            font-weight: normal;
        }
    }

    .error {
        text-align: left;
        color: #F04E4E;
        margin-bottom: 27px;

        svg {
            margin-right: 4px;
        }
    }

    sui-input[type=submit] {
        margin-bottom: 24px;
    }

    a {
        color: #293FE6;
        text-decoration: none;
        font-weight: bold;
    }
}
</style>
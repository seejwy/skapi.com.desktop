<template lang="pug">
div(v-if='!state?.connection')
    // is loading...
div(v-else-if="state?.user" :loading="isSaving || null")
    .pageHeader.headSpaceHelper
        h1.fixed Account Settings
    .settingsWrapper
        form.settings(@submit.prevent="updateUserSettings" @keydown.enter.prevent="" action="")
            .title Name
            .value
                sui-input(v-if="isEdit" type="text" @input="(e) => settings.name = e.target.value" :value="settings.name")
                span(v-else) {{  state.user.name }}
            .actions
            hr
            .title(style="margin-bottom: 0;") Email
            .value
                sui-input(
                    v-if="isEdit" 
                    type="text" 
                    :value="settings.email" 
                    @input="(e)=> { settings.email = e.target.value; e.target.setCustomValidity(''); }" 
                    @change="validateEmail"
                    inputmode="email")
                template(v-else)
                    span {{  state.user.email }}
                    .emailStatus(:class="{'unverified': !state.user.email_verified ? true : null, 'verified': state.user.email_verified ? true : null}")
                        Icon warning
                        span(v-if="state.user.email_verified") Verified
                        span(v-else) Unverified
            .actions(v-if="!state.user.email_verified" @click="openVerifyEmail")
                span Verify Email
            hr   
            .title Newsletter Subscription
            .value
                template(v-if="isEdit")
                    label
                        sui-input(type="checkbox" :disabled="!state.user.email_verified ? true : null" :checked="settings.email_subscription ? true : null" @change="(e) => settings.email_subscription = e.target.checked")
                        span I agree to receive information and news letters from Skapi via Email.
                template(v-else-if="settings.email_subscription !== ''")
                    template(v-if="settings.email_subscription") Subscribed
                        span(v-if="!state.user.email_verified" style="margin-left: 10px; color: #FF8D3B;")   ( Needs Verification )
                    template(v-else) Not Subscribed
            hr
            .title(style="margin-bottom: 0;") Password
            .value
            .actions
                span(@click="() => { if(isSaving) return false; passwordOverlay.open()}") Change Password
            hr
            .submit
                template(v-if="isEdit")
                    sui-button.lineButton(type="button" @click="cancelEdit") Cancel
                    SubmitButton(:loading="isSaving") Save
                sui-button(v-else type="button" @click="isEdit = true") Edit Account

    .settingsWrapper.delete     
        div(@click="openDeletePopup") Delete Your Account
            Icon(style="height: 20px; width: 20px; margin-left: 8px;") trash
    Transition(name="toast")
        .toast(v-if="state.user && !state.user.email_verified && state.showVerificationNotification")
            Icon warning_bell
            .title Email Verfication is Needed
            div
            .body Please verify your email to prevent your services from shutting down.
            Icon.close(@click="state.setVerificationDelay") X2
    sui-overlay(ref="passwordOverlay" style="background: rgba(0, 0, 0, 0.6);")
        ChangePassword(@close="passwordOverlay.close()")
    sui-overlay(ref="emailOverlay" style="background: rgba(0, 0, 0, 0.6);")
        VerifyEmail(@close="emailOverlay.close()")
    sui-overlay(v-if="isDelete" ref="deleteAccountOverlay" style="background: rgba(0, 0, 0, 0.6);")
        DeleteAccount(@close="deleteAccountOverlay.close(() => isDelete = false)")
</template>

<script setup>
import { inject, ref, onMounted, watch, nextTick } from 'vue';
import { state, awaitConnection } from '@/main';
import { useRoute, useRouter } from 'vue-router';

import Icon from '@/components/Icon.vue';
import SubmitButton from '@/components/SubmitButton.vue';
import ChangePassword from '@/views/Main/ChangePassword.vue';
import VerifyEmail from '@/views/Main/VerifyEmail.vue';
import DeleteAccount from '@/views/Main/DeleteAccount.vue';

import { skapi } from '@/main';

let router = useRouter();
let route = useRoute();

let pageTitle = inject('pageTitle');
pageTitle.value = 'skapi';

let serviceList = ref(null);
let overlay = ref(null);
const isEdit = ref(false);
const isDelete = ref(false);
let settings = ref({
    email_subscription: '',
    name: '',
    email: ''
});

const deleteAccountOverlay = ref(null);
const passwordOverlay = ref(null);
const emailOverlay = ref(null);
const isSaving = ref(false);

const openDeletePopup = async () => {
    if (isSaving.value) return false;

    isDelete.value = true;
    await nextTick();
    deleteAccountOverlay.value.open();
};
const openVerifyEmail = async () => {
    skapi.verifyEmail().catch(err => console.log(err));
    emailOverlay.value.open();
};

const cancelEdit = () => {
    isEdit.value = false;
    settings.value.name = state.user.name;
    settings.value.email = state.user.email;
    settings.value.email_subscription = state.user.email_subscription;
};

const validateEmail = (event) => {
    if (skapi.validate.email(event.target.value)) {
        event.target.setCustomValidity('');
    } else {
        event.target.setCustomValidity('Invalid Email');
        event.target.reportValidity();
    }
};

const updateUserSettings = async () => {
    if (state.user.name === settings.value.name && state.user.email === settings.value.email && state.user.email_subscription === settings.value.email_subscription) {
        isEdit.value = false;
        isSaving.value = false;
        return true;
    }

    try {
        isSaving.value = true;
        let isSubscribe = settings.value.email !== state.user.email ? false : settings.value.email_subscription;
        let res = await skapi.updateProfile({
            name: settings.value.name,
            email: settings.value.email
        });

        if (state.user.email_subscription !== settings.value.email_subscription) {
            if (settings.value.email_subscription) {
                await skapi.subscribeNewsletter({ group: 1 });
                settings.value.email_subscription = true;
            } else {
                await skapi.unsubscribeNewsletter({ group: 1 });
                settings.value.email_subscription = false;
            }
        }

        state.user = res;
        if (!res.email_verified) state.showVerificationNotification = true;

    } catch (e) {
        console.log({ e });
        settings.value.name = state.user.name;
        settings.value.email = state.user.email;
        settings.value.email_subscription = state.user.email_subscription;
    } finally {
        isEdit.value = false;
        isSaving.value = false;
    }
};

const deleteAccount = async () => {
    let res = await skapi.postRecord(null, {
        table: {
            name: 'reason'
        },
        index: {
            name: 'NR',
            value: true
        }
    });
};

onMounted(() => {
    awaitConnection.then(async () => {
        if (state.user) {
            if (!state.user.hasOwnProperty('email_subscription')) {
                let subscriptions = await skapi.getNewsletterSubscription();
                state.user.email_subscription = subscriptions.length && subscriptions[0].group === 1 ? true : false;
                settings.value.email_subscription = subscriptions.length && subscriptions[0].group === 1 ? true : false;
            } else {
                settings.value.email_subscription = state.user.email_subscription;
            }
            settings.value.name = state.user.name;
            settings.value.email = state.user.email;
        }
    });
});

watch(() => state.user, async (user) => {
    await nextTick();
    if (!user) {
        overlay.value.open();
    }
});
</script>
        
<style lang="less" scoped>
.pageHeader {
    h1 {
        display: block;
    }
}

.settingsWrapper {
    background-color: #fafafa;
    padding: 28px 37px;
    border-radius: 8px;

    &.delete {
        line-height: 52px;
        margin-top: 24px;
        padding: 0 37px;
        user-select: none;
        font-weight: bold;
        color: rgba(0, 0, 0, 0.65);

        &> {
            cursor: pointer;
        }
    }
}

.settings {
    display: grid;
    grid-template-columns: auto 1fr min-content;
    align-items: center;
    column-gap: 80px;

    &>.title {
        font-weight: bold;
        padding: 28px 0;
        color: rgba(0, 0, 0, .65);
        white-space: nowrap;
    }

    .value {
        &>span {
            display: inline-block;
            margin-right: 8px;
        }
    }

    .emailStatus {
        display: inline-block;

        span {
            vertical-align: middle;
        }

        svg {
            margin-right: 4px;
        }

        &.verified {
            color: #5AD858;
        }

        &.unverified {
            color: #FF8D3B;
        }
    }

    .actions {
        justify-self: end;
        color: var(--primary-color);
        font-weight: bold;
        cursor: pointer;
        white-space: nowrap;

        .save,
        .cancel {
            display: inline-block;
            font-weight: bold;
        }

        .cancel {
            margin-right: 28px;
            color: #F04E4E;
        }
    }

    .value {
        label {
            display: flex;
            gap: 7px;

            sui-input {
                flex-shrink: 0;
                margin-top: calc((1.5em - 16px) / 2);
            }

            span {
                line-height: 1.5;
            }
        }

        .actions {
            text-align: right;
            margin-top: 28px;
        }

        &>sui-input {
            width: 100%;
        }
    }

    hr {
        grid-column: span 3;
        width: 100%;
        border: 1px solid rgba(0, 0, 0, 0.04);
        box-shadow: 0px 1px 3px rgba(0, 0, 0, 0.06);
    }

    .submit {
        grid-column: span 3;
        justify-self: end;
        padding-top: 28px;
    }
}

.error {
    text-align: left;
    margin-top: 4px;
    color: #F04E4E;
}

.lineButton {
    &~sui-button {
        margin-left: 16px;
    }
}
</style> 
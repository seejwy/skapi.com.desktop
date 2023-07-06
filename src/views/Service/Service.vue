<template lang="pug">
EditService(v-if="state?.user && route.query.edit === 'service'" @close="router.replace({query: null})")
template(v-else)
    .pageHeader.headSpaceHelper
        h2 Service
        p.
            A service is a collection of serverless resources working together to form your backend.
            Simply connect to your service and start building your website.      

        div.action
            a(href="https://docs.skapi.com/the-basics/#connecting-to-your-service" target="_blank")
                sui-button.lineButton(type="button") Find out More
    .container
        .innerContainer 
            .titleActionsWrapper
                .titleWrapper
                    Icon information
                    h2 Service Information
                .actions(@click="deleteServiceAsk" :class="{'disabled': !state.user.email_verified ? true : null}")
                    Icon(style="width: 20px; height: 20px;") trash
                    span Delete Service
            .informationGrid
                .informationGridItem(v-for="info in informationGrid" :class="[info.span ? `span-${info?.span}` : '']")
                    .name {{ info.name }}
                    .value(v-if="info.filter") {{ info.filter(service[info.key]) }}
                    .value(v-else) {{ service[info.key] }}

    .container
        .innerContainer 
            .titleActionsWrapper
                .titleWrapper
                    Icon setting
                    h2 Service Setting 
                .actions(@click="edit" :class="{'disabled': !state.user.email_verified ? true : null}")
                    Icon pencil
                    span Edit
            .settingGrid 
                .settingGridItem(v-for="setting in settingGrid")
                    .name 
                        span {{ setting.name }}
                        sui-tooltip(v-if="setting.tip")
                            Icon(slot="tool") question
                            div(slot="tip") {{ setting.tip }}
                    .value(v-if="setting.key === 'active'")
                        .indicator(:class="{'active': service[setting.key] > 0}")
                        span(v-if="service[setting.key] > 0") Enabled 
                        span(v-else) Disabled
                    .value(v-else) {{  service[setting.key] || '-' }}

    .container
        .innerContainer.services
            .titleActionsWrapper
                .titleWrapper
                    h2 Manage your Service 
            .serviceGrid 
                RouterLink(:to="{name: 'users'}").serviceGridItem
                    .content
                        .title
                            Icon users
                            span Users
                        .body Within your service, users are individuals who have successfully created an account and logged in at least once. You can search for and apply access control using our easy to use user database management system.
                    .goto Go to Users >
                RouterLink(:to="{name: 'records'}").serviceGridItem  
                    .content
                        .title
                            Icon folder_open
                            span Record
                        .body Records are data objects created by you or your users and stored within your database. You can efficiently search, modify, or create new records using our database management system.
                    .goto Go to Records >
                //- RouterLink(to="/").serviceGridItem 
                    .content
                        .title
                            Icon mail
                            span Email System
                        .body Users are data that your service user's will store and read from your service database. 
                    .goto Go to Mail >

    .container 
        .innerContainer 
            .titleActionsWrapper(v-if="!service.subdomain" style="margin-bottom: 0;")
                .titleWrapper
                    Icon domain
                    h2 Subdomain
                .actions(@click="create" :class="{'disabled': !state.user.email_verified ? true : null}")
                    Icon pencil
                    span Create
            .titleActionsWrapper(v-else)
                .titleWrapper
                    Icon domain
                    h2 Subdomain
                .actions(v-if="service.subdomain && !deleting" @click="deleteSubdomainAsk" :class="{'disabled': !state.user.email_verified ? true : null}")
                    Icon(style="width: 20px; height: 20px;") trash
                    span Delete Subdomain
            .domainGrid(v-if="domain")
                .domainGridItem
                    .name
                        span Subdomain
                    .value
                        span {{ service.subdomain }}.skapi.com
                    a(:href="`http://${service.subdomain}.skapi.com`" target="_blank")
                        Icon link
            .domainGrid.deleting(v-else-if="deleting") 
                h3 Deleting subdomain ...
                span It may take a few minutes for a subdomain to be deleted.

    .container(v-if="domain")
        .innerContainer 
            .titleActionsWrapper(style="margin-bottom: 0;")
                .titleWrapper
                    Icon list
                    h2 Upload Files List
                .actions(@click="modify" :class="{'disabled': !state.user.email_verified ? true : null}")
                    Icon pencil
                    span Modify

    sui-overlay(v-if="isEdit" ref="settingWindow" style="background: rgba(0, 0, 0, 0.6)" @mousedown="async()=>{await state.blockingPromise; settingWindow.close(()=>isEdit = false)}")
        div.overlay
            EditService(@close="async()=>{await state.blockingPromise; settingWindow.close(()=>isEdit = false)}")
    sui-overlay(v-if="isCreate" ref="subdomainWindow" style="background: rgba(0, 0, 0, 0.6)" @mousedown="async()=>{await state.blockingPromise; subdomainWindow.close(()=>isCreate = false)}")
        div.overlay
            Subdomain(@close="async()=>{await state.blockingPromise; subdomainWindow.close(()=>isCreate = false)}")
sui-overlay(ref="deleteConfirmOverlay")
    form.popup(@submit.prevent="deleteService" action="" :loading="isDisabled || null")
        .title
            Icon warning
            div Deleting Service?
        .body 
            p Are you sure you want to delete "{{ service.name }}" permanently? #[br] You will not be able to undo this action.
            p To confirm deletion, enter Service ID #[br] #[span(style="font-weight: bold") {{ service.service }}]
            sui-input(:placeholder="service.service" :value="confirmationCode" @input="(e) => confirmationCode = e.target.value")
        .foot
            sui-button(type="button" @click="()=> { deleteConfirmOverlay.close(); confirmationCode = ''}").textButton Cancel
            SubmitButton(:loading="isDisabled" class="textButton" backgroundColor="51, 51, 51") Delete
sui-overlay(ref="deleteSubdomainOverlay")
    form.popup(@submit.prevent="deleteSubdomain" action="" :loading="isDisabled || null")
        .title
            Icon warning
            div Deleting Subdomain?
        .body 
            p Are you sure you want to delete "{{ service.subdomain }}" permanently? #[br] You will not be able to undo this action. #[br] And all relevant data will be deleted.
            p To confirm deletion, enter Service ID #[br] #[span(style="font-weight: bold") {{ service.service }}]
            sui-input(:placeholder="service.service" :value="confirmationCode" @input="(e) => confirmationCode = e.target.value")
        .foot
            sui-button(type="button" @click="()=> { deleteSubdomainOverlay.close(); confirmationCode = ''}").textButton Cancel
            SubmitButton(:loading="isDisabled" class="textButton" backgroundColor="51, 51, 51") Delete
sui-overlay(ref="deleteErrorOverlay")
    .popup
        .title
            Icon warning
            div Something went wrong!
        .body {{ deleteErrorMessage }}
        .foot
            sui-button.lineButton(type="button" @click="()=> { deleteErrorOverlay.close(); }") Ok
</template>

<script setup>
import { inject, reactive, ref, watch, nextTick, onBeforeMount } from 'vue';
import { state, skapi } from '@/main';
import { localeName, dateFormat, getSize } from '@/helper/common';
import { useRoute, useRouter } from 'vue-router';

import EditService from '@/views/Service/EditService.vue';
import Subdomain from '@/views/Service/Subdomain.vue';
import Icon from '@/components/Icon.vue';
import SubmitButton from '@/components/SubmitButton.vue';

const route = useRoute();
const router = useRouter();

let service = inject('service');
let pageTitle = inject('pageTitle');
pageTitle.value = 'Service "' + service.value.name + '"'

const settingWindow = ref(null);
const subdomainWindow = ref(null);
const deleteConfirmOverlay = ref(null);
const deleteSubdomainOverlay = ref(null);
const deleteErrorOverlay = ref(null);
const confirmationCode = ref('');
const deleteErrorMessage = ref('');
const isEdit = ref(false);
const isCreate = ref(false);
const isDisabled = ref(false);
const domain = ref(false);
const deleting = ref(false);

const informationGrid = reactive([
    {
        name: 'Service ID',
        key: 'service',
        span: 2
    },
    {
        name: 'Owner\'s ID',
        key: 'owner',
        span: 2
    },
    // {
    //     name: 'Group',
    //     key: 'group',
    //     filter: (value) => {
    //         return value == 1 ? 'Basic' : 'Premium'
    //     }
    // },
    {
        name: 'Service Location',
        key: 'region',
        filter: (value) => {
            return localeName(value);
        }
    },
    {
        name: 'Date Created',
        key: 'timestamp',
        filter: (value) => {
            return dateFormat(value).split(' ')[0];
        }
    },
    {
        name: 'Storage Use',
        key: 'storage',
        filter: (value) => {
            let val = value || 0;
            return getSize(val);
        }
    },
    {
        name: '# of Users',
        key: 'users'
    },
    // {
    //     name: '# of Newsletter Sub',
    //     key: 'newsletter_subscribers'
    // },
]);

const settingGrid = reactive([
    {
        name: 'Enable/Disable',
        key: 'active',
        filter: () => {
            return 1
            // return .indicator(:class="{'active': service.active > 0}")
        }
    },
    {
        name: 'Name of Service',
        key: 'name',
    },
    {
        name: 'CORS',
        key: 'cors',
        tip: 'When CORS is set, your website will not be able to connect to your service unless the request comes from a valid host.',
    },
    {
        name: 'API Key',
        key: 'api_key',
        tip: 'You can set your own private API key if you wish to integrate your users\' secure requests to your external backend server.',
    },
]);

const edit = () => {
    if (!state.user.email_verified) return false;
    isEdit.value = true;
}

const create = () => {
    if (!state.user.email_verified) return false;
    isCreate.value = true;
}

const deleteServiceAsk = () => {
    if (!state.user.email_verified) return;
    deleteConfirmOverlay.value.open();
}

const deleteSubdomainAsk = () => {
    if (!state.user.email_verified) return;
    deleteSubdomainOverlay.value.open();
}

const deleteService = () => {
    isDisabled.value = true;
    if (confirmationCode.value !== service.value.service) {
        confirmationCode.value = '';
        deleteErrorMessage.value = "Your service code did not match.";
        if (deleteConfirmOverlay.value) deleteConfirmOverlay.value.close();
        deleteErrorOverlay.value.open();
        isDisabled.value = false;
        return;
    }

    skapi.deleteService(service.value.service).then(() => {
        if (deleteConfirmOverlay.value) deleteConfirmOverlay.value.close();
        router.replace('/admin');
    }).catch(() => {
        deleteErrorMessage.value = "Please disable your service before deleting it.";
        if (deleteConfirmOverlay.value) deleteConfirmOverlay.value.close();
        deleteErrorOverlay.value.open();
    }).finally(() => {
        confirmationCode.value = '';
        isDisabled.value = false;
    });
}

const deleteSubdomain = async() => {
    isDisabled.value = true;
    if (confirmationCode.value !== service.value.service) {
        confirmationCode.value = '';
        deleteErrorMessage.value = "Your service code did not match.";
        if (deleteSubdomainOverlay.value) deleteSubdomainOverlay.value.close();
        deleteErrorOverlay.value.open();
        isDisabled.value = false;
        return;
    }

    try {
        await skapi.registerSubdomain({
            service: service.value.service,
            subdomain: service.value.subdomain,
            exec: 'remove'
        }).then(() => {
            if (deleteSubdomainOverlay.value) deleteSubdomainOverlay.value.close();
            if(domain.value) {
                if(service.value.subdomain.includes('*')) {
                    deleting.value = true;
                    domain.value = false;
                    isDisabled.value = false;
                } else {
                    deleting.value = false;
                }
            }
        })
    } catch(e) {
        deleteErrorMessage.value = e.message;
        if (deleteSubdomainOverlay.value) deleteSubdomainOverlay.value.close();
        deleteErrorOverlay.value.open();
        isDisabled.value = false;
    }
}

if (!service.value.hasOwnProperty('storage')) {
    skapi.storageInformation(service.value.service).then((storage) => {
        service.value.storage = storage.cloud + storage.database + storage.email;
    });
}

watch(() => isEdit.value, async () => {
    await nextTick();
    if (isEdit.value) {
        settingWindow.value.open();
    }
});

watch(() => isCreate.value, async () => {
    await nextTick();
    if (isCreate.value) {
        subdomainWindow.value.open();
    } else {
        if (service.value.subdomain) {
            domain.value = true;
        }
    }
});

watch(() => service.value.subdomain, async () => {
    await nextTick();
    if (service.value.subdomain.includes('*')) {
        deleting.value = true;
        domain.value = false;
    } else {
        deleting.value = false;
    }
});

onBeforeMount(() => {
    if (service.value.subdomain) {
        domain.value = true;

        if (service.value.subdomain.includes('*')) {
            deleting.value = true;
            domain.value = false;
        } else {
            deleting.value = false;
        }
    }
})
</script>

<style lang="less" scoped>
.container {
    margin: 0 0 40px 0;

    .innerContainer {
        padding: 40px;
        background: #434343;
        border-radius: 12px;

        .titleActionsWrapper {
            margin-bottom: 32px;

            h2 {
                font-size: 20px;
                font-weight: normal;
            }
        }
    }

    h2,
    p {
        color: rgba(255, 255, 255, .85);
        margin: 0;
    }

    h2 {
        display: inline-block;
        vertical-align: middle;
        font-size: 24px;
        margin-bottom: 50px;
        font-weight: bold;
    }

    p {
        color: rgba(255, 255, 255, .85);
        line-height: 1.5;
    }

    .titleActionsWrapper {
        display: flex;
        justify-content: space-between;
        margin-bottom: 16px;

        h2 {
            font-size: 20px;
            font-weight: normal;
        }
    }

    .titleWrapper {
        h2 {
            margin: 0;
        }

        svg {
            margin-right: 8px;
        }
    }

    .actions {
        cursor: pointer;
        user-select: none;

        svg {
            margin-right: 4px;
        }

        span {
            vertical-align: middle;
        }

        &.disabled {
            opacity: 0.4;
        }
    }

    &:last-child {
        margin-bottom: 0;
    }
}

.informationGrid {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
    column-gap: 20px;
    row-gap: 28px;

    &Item {
        min-width: 0;

        .name {
            font-size: 14px;
            color: rgba(255, 255, 255, 0.6);
            margin-bottom: 8px;
        }

        .value {
            font-weight: bold;
            color: rgba(255, 255, 255, 0.85);
            word-break: break-all;
        }
    }
}

.settingGrid {
    display: flex;
    justify-content: space-between;

    &Item {
        .name {
            font-size: 14px;
            color: rgba(255, 255, 255, 0.6);
            margin-bottom: 8px;
        }

        .value {
            font-weight: bold;
            color: rgba(255, 255, 255, 0.85);
        }

        &.span2 {
            grid-column: span 2;
        }
    }
}

.settingGrid {
    display: grid;
    column-gap: 12px;
    row-gap: 28px;
    grid-template-columns: repeat(4, calc(25% - 30px)) 72px;

    &Item {
        .name {
            font-size: 14px;
            line-height: 1;
            color: rgba(255, 255, 255, 0.6);
            margin-bottom: 8px;

            span {
                vertical-align: middle;
            }
        }

        .value {
            font-weight: bold;
            color: rgba(255, 255, 255, 0.85);
            word-break: break-all;

            span {
                vertical-align: middle;
            }
        }

        &.actions {
            align-self: flex-end;
            justify-self: flex-end;
        }
    }
}

.noDomains {
    color: rgba(255, 255, 255, 0.4);
    padding-bottom: 35px;
    margin: 0 -20px -24px -20px;
    border-radius: 0 0 8px 8px;
    text-align: center;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-wrap: wrap;
    opacity: 0.6;

    .title {
        font-size: 28px;
    }

    p {
        font-size: 14px;
        margin: 20px 0 0 0;
        color: rgba(255, 255, 255, 0.4);
    }
}

.domainGrid {
    &.deleting {
        text-align: center;
        color:rgba(255,255,255,0.4);
        padding-bottom: 30px;

        h3 {
            font-size:28px;
            font-weight:500;
            margin: 0 0 20px 0;
        }
        span {
            font-size: 14px;
        }
    }
    &Item {
        position: relative;
        width: 100%;
        background-color: rgba(255, 255, 255, 0.1);
        padding: 24px;
        border-radius: 8px;

        .name {
            font-size: 14px;
            line-height: 1;
            color: rgba(255, 255, 255, 0.6);
            margin-bottom: 8px;
        }

        .value {
            font-weight: bold;
            color: rgba(255, 255, 255, 0.85);
            word-break: break-all;
        }

        a {
            position: absolute;
            top: 50%;
            right: 24px;
            transform: translateY(-50%);
            color: #fff;
        }
    }
}

.serviceGrid {
    display: flex;
    justify-content: space-between;
    gap: 20px;

    &Item {
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        width: 100%;
        background: rgba(255, 255, 255, 0.1);
        padding: 24px;
        border-radius: 8px;

        .content {
            display: flex;
            flex-direction: column;
            justify-content: flex-start;

            .title {
                display: inline-block;
                margin-bottom: 28px;
                font-size: 20px;

                span {
                    margin-left: 8px;
                    vertical-align: middle;
                }
            }

            .body {
                color: rgba(255, 255, 255, 0.85);
                line-height: 1.5;
            }
        }

        .goto {
            text-align: left;
            margin-top: 40px;
            color: rgba(255, 255, 255, 0.6);
            font-size: 14px;
            text-decoration: none;
        }
    }

    a.serviceGridItem {
        text-align: left;
        color: rgba(255, 255, 255, 0.85);
        font-size: 14px;
        text-decoration: none;
    }
}

sui-tooltip {
    margin-top: -7px;
    margin-left: 8px;
}

.indicator {
    position: relative;
    display: inline-block;
    vertical-align: middle;
    height: 20px;
    width: 20px;
    border-radius: 50%;
    background: #D9D9D9;
    border: 0.3px solid #595959;
    box-shadow: inset -1px -1px 2px rgba(0, 0, 0, 0.25), inset 1px 1px 2px rgba(255, 255, 255, 0.65);
    margin-right: 8px;

    &.active {
        background: #5AD858;
    }
}

.overlay {
    padding: 16px;

    .close {
        position: absolute;
        top: 0;
        right: 0;
        width: 32px;
        height: 32px;
        background-color: #BFBFBF;
        border-radius: 50%;
        display: flex;
        align-items: center;
        justify-content: center;
        cursor: pointer;

        svg {
            color: #434343;
        }

        a {
            text-align: left;
            margin-top: 40px;
            color: rgba(255, 255, 255, 0.6);
            font-size: 14px;
            text-decoration: none;
        }
    }
}
</style>
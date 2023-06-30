<template lang="pug">
div(v-if='!state?.connection')
    // is loading...
NewService(v-else-if="state?.user && route.query.new === 'service'")
FeedBackForm(v-else-if="state?.user && route.query.new === 'feedback'")
div(v-else-if="state?.user")
    .pageHeader.headSpaceHelper
        h1.fixed Admin
        p.
            You can see a list of all the services you are running.
            To create a new service, click on "New Service".

        .action
            sui-button.withIcon(type="button" @click="NewServiceConditions" :disabled="!state.user.email_verified || null")
                Icon plus2
                span New Service
    .container(v-if="serviceList?.length")
        template(v-for="service in serviceList")
            router-link.service(:to='"/admin/" + service.service')
                .settings
                    .name 
                        .indicator(:class="{'active': service.active > 0}")
                        span {{ service.name }}
                    Icon right
                .details
                    .item(v-for="(value, key) in filterServiceDetails(service)")
                        .title {{  key }}
                        .value {{ value || '-' }}
    .container.empty(v-else-if="isFetchingServices")
        Icon.animationRotation(style="position: absolute; right: 24px; top: 24px; fill: var(--primary-color)") refresh
    .container.empty.noService(v-else)
        div(style="position: absolute; width: 100%;")
            .title No Services
            span Get started by creating a new service.
    sui-overlay(v-if="isOpen" ref="newServiceWindow" style="background: rgba(0, 0, 0, 0.6)" @mousedown="closeNewServiceWindow")
        div.overlay
            NewService(@close="closeNewServiceWindow")
    sui-overlay(v-if="feedBackOpen" ref="feedBackWindow" style="background: rgba(0, 0, 0, 0.6)" @mousedown="closeFeedBackWindow")
        div.overlay
            FeedBackForm(@close="closeFeedBackWindow" @closeFeedBack="feedBackOpen=false; isOpen=true;")
</template>

<script setup>
import { inject, ref, watch, nextTick } from 'vue';
import { state, skapi } from '@/main';
import { dateFormat, localeName } from '@/helper/common';
import { useRoute, useRouter } from 'vue-router';

import NewService from '@/components/NewService.vue';
import FeedBackForm from '@/views/Main/FeedBackForm.vue';
import Icon from '@/components/Icon.vue';

let router = useRouter();
let route = useRoute();

let pageTitle = inject('pageTitle');
pageTitle.value = 'skapi';

let serviceList = ref(null);
let overlay = ref(null);
const newServiceWindow = ref(null);
const feedBackWindow = ref(null);
const isOpen = ref(false);
const feedBackOpen = ref(false);
const isFetchingServices = ref(true);

const filterServiceDetails = (service) => {
    return {
        'Service Location': localeName(service.region),
        'CORS': service.cors,
        'Date Created': dateFormat(service.timestamp).split(' ')[0]
    };
};

const closeNewServiceWindow = async () => {
    await state.blockingPromise;
    newServiceWindow.value.close(() => isOpen.value = false);
};

const closeFeedBackWindow = async () => {
    await state.blockingPromise;
    feedBackWindow.value.close(() => feedBackOpen.value = false);
};

const NewServiceConditions = async () => {
    await getServices(state.getServices);

    skapi.getProfile().then((r) => {
        if (r.misc === 'feedback complete') {
            if (state.user.email_verified) {
                isOpen.value = true;
            } else {
                return null;
            }
        } else {
            let services = skapi.services;
            let services_count = Object.keys(services).length;
            let users_count = 0;

            if (services_count > 0) {
                // for(let i=0; i<services_count; i++){
                //     console.log(services[Object.keys(services)[i]])
                // }

                let value = services[Object.keys(services)[0]];

                for (let i = 0; i < value.length; i++) {
                    let users = value[i].users;
                    users_count += users;
                }

                if (users_count > 0) {
                    if (state.user.email_verified) {
                        feedBackOpen.value = true;
                    } else {
                        return null;
                    }
                } else {
                    if (state.user.email_verified) {
                        isOpen.value = true;
                    } else {
                        return null;
                    }
                }
            } else {
                if (state.user.email_verified) {
                    isOpen.value = true;
                } else {
                    return null;
                }
            }
        }
    });


};

async function getServices(gs) {
    if (!(gs instanceof Promise) || !state.user) {
        return;
    }

    try {
        let services = await gs;
        isFetchingServices.value = false;
        if (serviceList.value === null) {
            serviceList.value = [];
            for (let region in services) {
                serviceList.value = [...serviceList.value, ...services[region]];
            }

            serviceList.value.sort((a, b) => a.timestamp > b.timestamp ? -1 : a.timestamp < b.timestamp ? 1 : 0);
        }

        return services;

    } catch (err) {
        serviceList.value = null;
        throw err;
    }
}

getServices(state.getServices);

// watch is for users visiting the page directly
watch(() => state.getServices, getServices);
watch(() => isOpen.value, async () => {
    await nextTick();
    if (isOpen.value) {
        newServiceWindow.value.open();
    }
});
watch(() => feedBackOpen.value, async () => {
    await nextTick();
    if (feedBackOpen.value) {
        feedBackWindow.value.open();
    }
});
</script>
    
<style lang="less" scoped>
.container {
    &.empty {
        position: relative;
        display: inline-flex;
        width: 100%;
        align-items: center;
        justify-content: center;
        background: #F5F5F5;
        border-radius: 8px;
        text-align: center;

        .title {
            font-size: 28px;
            font-weight: bold;
            color: rgba(0, 0, 0, 0.6);
            margin-bottom: 12px;
        }

        &,
        &.noService {
            height: 175.33px;
        }
    }

    .service {
        display: block;
        background: #595959;
        padding: 32px 40px;
        border-radius: 8px;
        color: unset;
        text-decoration: unset;

        &:hover {
            background: #8C8C8C;
        }

        &:not(:last-child) {
            margin-bottom: 24px;
        }

        .settings {
            color: #fff;
            display: flex;
            align-items: center;
            justify-content: space-between;
            margin-bottom: 32px;
        }

        .name span {
            margin-left: 12px;
            color: #fff;
            font-size: 24px;
            vertical-align: middle;
        }

        .details {
            display: flex;
            justify-content: flex-start;
            column-gap: 33px;
            flex-wrap: wrap;
            overflow: hidden;

            .item {
                width: 200px;
            }

            .title {
                color: rgba(255, 255, 255, .4);
                margin-bottom: 12px;
            }

            .value {
                font-weight: bold;
                color: rgba(255, 255, 255, .85);
                overflow: hidden;
                text-overflow: ellipsis;
                white-space: nowrap;
            }
        }
    }
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
    }
}
</style>
    
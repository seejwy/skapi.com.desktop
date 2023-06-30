<template lang="pug">
.servicePageShell
    .sideScreen
        NavBar(v-if="pageTitle" style='background-color: #505050;z-index: 2;')
            ul.inlineVerticalMiddle
                li
                    a(href="https://docs.skapi.com" target="_blank") Documentation

                li
                    router-link(to="/admin" :class="{'router-link-active': route.path.split('/')[1] === 'admin'}") Admin

                li
                    router-link(to="/account-settings" tag="li") Account Settings

                li
                    a.clickable(@click="logout") Logout

        main.app#appMain(v-if='state.connection')
            NotExists(v-if='service === 404')
            template(v-else-if='service')
                router-view
            sui-overlay(v-else ref="overlay" style="background: rgba(0, 0, 0, 0.6);")
                Login
    .sidebarHzolder(v-if="state.user")
        .sidebar
            img.logo(src="@/assets/img/logo-small.svg" @click="router.push({name: 'home'})" alt="Skapi")
            img.hoverLogo(src="@/assets/img/logo.svg" @click="router.push({name: 'home'})")

            router-link(:to="{name: 'service'}" :class="{'routerLinkActive': !route.path.split('/')[3]}")
                Icon home
                span Service Home

            router-link(:to="{name: 'users'}" :class="{'routerLinkActive': route.path.split('/')[3] === 'users'}") 
                Icon users
                span Users

            router-link(:to="{name: 'records'}" :class="{'routerLinkActive': route.path.split('/')[3] === 'records'}")
                Icon folder_open
                span Records

            //- router-link(to='/')
                //(:to="{name: 'mail'}")
            //- Icon mail
</template>

<style lang="less">
.servicePageShell {
    display: flex;
    flex-direction: row-reverse;

    // navbar title text
    .titleText {
        color: rgba(255 255 255 / 60%);
    }

    .sidebarHzolder {
        width: 52px;
        flex-shrink: 0;
    }

    .sidebar {
        z-index: 10;
        background: var(--primary-color);
        overflow: hidden;
        height: 100%;
        min-height: 100vh;
        width: 52px;
        position: fixed;
        top: 0;
        transition: width .2s cubic-bezier(1, 0, 0, 1);
        max-width: 170px;
        
        &>* {
            display: inline-block;
            color: #fff;
        }

        & .logo {
            display: block;
            width: 32px;
            margin: 11px 10px;
            cursor: pointer;
        }

        & .hoverLogo {
            display: none;
            cursor: pointer;
        }

        &>a {
            display: block;
            width: min-content;
            margin: 28px 8px;
            border-radius: 4px;

            &.routerLinkActive,
            &:hover {
                background: rgba(255, 255, 255, 0.2);
            }

            svg {
                margin: 6px;
            }

            span {
                display: none;
                margin: 6px;
                font-weight: normal;
                font-size: 16px;
                white-space: nowrap;
                line-height: 24px;
            }
        }
            
        a {
            white-space: nowrap;
            span {    
                vertical-align: middle;
            }
        }
        &:hover {
            flex-shrink: 0;
            width: 170px;

            .logo {
                display: none;
            }
            .hoverLogo {
                display: block;
                height: 35px;
                margin: 14px 14px 11px 14px;
            }

            a {
                background: transparent;
                display: block;
                width: 100%;
                margin: 0;
                border-radius: 4px;
                padding: 14px 8px;
                border-radius: 0;
                &:hover {

                    background: rgba(255, 255, 255, .2);
                    span{
                        font-weight: bold;
                    }
                }
            }

            a span {
                display: inline-block;
            }
        }
        
    }

    .sideScreen {
        flex-grow: 1;
    }
}
</style>
<script setup>
import NavBar from '@/components/Navbar.vue';
import NotExists from '@/views/Main/404.vue';
import Login from '../Main/Login.vue';
import Icon from '@/components/Icon.vue';

import { provide, inject, watch, ref, onMounted, onBeforeUnmount } from 'vue';
import { skapi, state, awaitConnection } from '@/main';
import { useRoute, useRouter } from 'vue-router';
import { recordTables } from '@/helper/records.js';

let router = useRouter();

// sets pageTitle for immediate effect
// this does not trigger again when nested routes change
let route = useRoute();
let serviceId = route.params.service;
let service = ref(null);

provide('service', service);
provide('serviceUsers', ref(null));
provide('fetchingData', ref(false));

let pageTitle = inject('pageTitle');
pageTitle.value = ' ';

let overlay = ref(null);

onMounted(() => {
    awaitConnection.then(()=>{
        if(!state.user) {
            overlay.value.open();
        }
        recordTables.value = null;
    });
});

const logout = async () => {
    await router.push({ name: 'home' });
    skapi.AdminLogout().then(() => { state.user = null; })
}

function getServices(gs) {
    if (!(gs instanceof Promise)) {
        return;
    }

    gs.then(services => {
        let region = skapi.region_list?.[serviceId.substring(0, 4)];
        if (!region) {
            // region does not exists
            service.value = 404
            return;
        }
        if(services[region]) {
            for (let s of services[region]) {
                if (s.service === serviceId) {
                    service.value = s;
                    return s;
                }
            }
        }

        // no matching service id
        service.value = 404;
    });
}

getServices(state.getServices);

// watch is for users visiting the page directly
watch(() => state.getServices, getServices);
watch(() => state.user, u => {
    if (!u) {
        // throw user to login page if not logged in
        router.push('/');
    }
});

document.body.classList.add('admin');
onBeforeUnmount(() => {
    document.body.classList.remove('admin');
})
</script>
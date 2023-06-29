<template lang="pug">
.pageHeader.headSpaceHelper
    h1 Users
    p.
        Users are individuals who have successfully created an account and logged in at least once. 
        You can perform searches and apply access control on this page.
        Find out how you can easily perform authentication and manage your users.

    .action
        a(href="https://docs.skapi.com/authentication" target="_blank")
            sui-button.lineButton(type="button") Find out More
    div(style="clear:both;")
.actionsWrapper(:loading="promiseRunning || null")
    form(@submit.prevent="search" action="")
        .searchInputWrapper
            sui-select(name='search_type' style="width: 150px;" :value="searchParams.searchFor" @change="(e) => { searchParams.searchFor = e.target.value; changeSearchType(e.target.value); searchParams.value = '';}")
                option(value="timestamp") Date Created
                option(value="user_id") User ID
                option(value="email") Email
                option(value="phone_number") Phone
                option(value="address") Address
                option(value="gender") Gender
                option(value="name") Name
                option(value="locale") Locale
                option(value="birthdate") Birth Date
        
            .selectInput.noBorder(@click.stop)
                .inputField
                    sui-input(
                        v-if="searchParams.searchFor === 'locale'"
                        ref="searchField" 
                        type="search" 
                        autocomplete="off" 
                        :placeholder="`      ${placeholder(searchParams.searchFor)}`" 
                        :value="searchParams.value" 
                        @input="(e) => { searchParams.value = e.target.value; e.target.setCustomValidity(''); }"
                        :inputmode="searchParams.searchFor === 'email' ? searchParams.searchFor : null"
                        pattern="[A-Z]{2}"
                        required)
                    sui-input(
                        v-else
                        ref="searchField" 
                        type="search" 
                        autocomplete="off" 
                        :placeholder="`      ${placeholder(searchParams.searchFor)}`" 
                        :value="searchParams.value" 
                        @input="(e) => { searchParams.value = e.target.value; e.target.setCustomValidity(''); }"
                        :inputmode="searchParams.searchFor === 'email' ? searchParams.searchFor : null"
                        required)
                .selectField(v-if="searchParams.searchFor === 'timestamp'")
                    sui-select(style="width: 70px; text-align: center;" :value="searchParams.condition" name='search_condition' @change="(e) => searchParams.condition = e.target.value")
                        option(value=">=") &gt;=
                        option(value="<=") &lt;=
                        option(value="=") =
                .selectField(v-else-if="searchParams.searchFor === 'birthdate'")
                    sui-select(style="width: 70px; text-align: center;" :value="searchParams.condition" name='search_condition' @change="(e) => searchParams.condition = e.target.value")
                        option(value=">") &gt;
                        option(value="<") &lt;
                        option(value="=") =
                    
                        //- option(value=">") &gt;
                        //- option(value=">=") &gt;=
                        //- option(value="<") &lt;
                        //- option(value="<=") &lt;=
                        //- option(value="=") =
    
    .actions
        sui-button.textButton(type="button" @click="blockUsers" :disabled="((selectedUnblockedUsers.length === 0 || selectedBlockedUsers.length > 0) || !state.user.email_verified) || null")
            Icon block
            span block
        sui-button.textButton(type="button" @click="unblockUsers" :disabled="((selectedBlockedUsers.length === 0 || selectedUnblockedUsers.length > 0) || !state.user.email_verified) || null")
            Icon unblock
            span unblock
        sui-button.textButton(type="button" @click="deleteUsers" :disabled="(selectedUsers.length === 0 || !state.user.email_verified) || null")
            Icon trash
            span delete
.tableOuterWrapper(:loading="promiseRunning || null")
    .searchQuery(v-if="route.query.search")
        span Result of {{ visibleFields[route.query.search].text }} : "{{ route.query.value }}" {{ route.query.condition }}
        .clickable(@click="()=>{ searchResult=null; currentSelectedRecordPage=0; currentSelectedRecordBatch=0; router.push({name:'users'})}")
            span(style="vertical-align:middle;") Clear
            Icon X2
    .tableActions(v-if="!route.query.search || fetchingData" :class="{'roundedBorder' : !groupedUserList?.length && fetchingData}")
        .headerActionsBefore(v-if="showSetting" @click="toggleSetting(false)")
        .headerActions(v-if="!route.query.search || groupedUserList?.length" @click="toggleSetting(true)")
            div.dropdown
                span Headers
                Icon down2
            .filterWrapper
                .filter(ref="filterEl" v-if="showSetting")
                    .label(v-for="(field, key) in visibleFields")
                        label
                            sui-input(type="checkbox" :checked="field.show || null" @input="field.show = !field.show" :disabled="computedVisibleFields.length === 1 && field.show ? true : null")
                            span {{  field.text }}
        .headerActions(v-else)
        Icon.refresh(v-if="!route.query.search || fetchingData" :class="{'animationRotation': fetchingData}" @click="getUsers") refresh

    .tableWrapper
        table
            thead(v-if="groupedUserList?.length && (!fetchingData || groupedUserList?.length)")
                tr(:class="{rounded: fetchingData || null}")
                    th
                        sui-input(type="checkbox" :checked="selectedUsers.length === groupedUserList?.[currentSelectedUsersBatch][currentSelectedUsersPage].length || null" @change="selectAllHandler")
                    th(v-for="key in computedVisibleFields" :class="{'iconTd': key === 'block' || key === 'status', 'userId': key === 'user_id'}") {{ visibleFields[key].text }}
                    th(v-if="computedVisibleFields.length <= 2")
            tbody(v-if="groupedUserList?.length")
                tr(v-for="(user, userIndex) in groupedUserList?.[currentSelectedUsersBatch][currentSelectedUsersPage]" :key="user['user_id']" :id="user['user_id']")
                    td
                        sui-input(type="checkbox" :value="user.user_id" :checked="selectedUsers.includes(user.user_id) || null" @change="userSelectionHandler")
                    td(v-for="(key, index) in computedVisibleFields" :class="{'iconTd' : key === 'block' || key === 'status'}") 
                        template(v-if="key === 'approved'")
                            Icon(v-if="user[key]?.includes('suspended')" style="opacity: 40%;") block
                            Icon(v-else) unblock
                        template(v-else-if="key === 'group'")                     
                            Icon(v-if="user[key] > 0") check_circle
                            Icon(v-else) x
                        template(v-else-if="key === 'access_group'")
                            span(v-if="user['group'] === 99") Admin
                            span(v-else) User
                        template(v-else-if="key === 'timestamp'")
                            span {{ dateFormat(user[key]) }}
                        template(v-else) {{ user[key] || '-' }}
                    td(v-if="computedVisibleFields.length <= 2")
                //- Below code needs to change to page list not full users list
                template(v-if="groupedUserList?.[currentSelectedUsersBatch][currentSelectedUsersPage].length < 10")
                    tr(v-for="num in numberOfUsersPerPage - groupedUserList?.[currentSelectedUsersBatch][currentSelectedUsersPage].length")
                        td  
                        td(v-for="(key, index) in computedVisibleFields")
                        td(v-if="computedVisibleFields.length <= 2")

    .noUsersFound(v-if="!groupedUserList?.length && !fetchingData")
        template(v-if="!route.query.value && !groupedUserList?.length")     
            .title No Users
            p You have no existing users yet
        template(v-else) 
            .title No Users Found
            p There were no users matching the query.
    .paginator(v-if="groupedUserList?.length")
        Icon(
            :class="{active: currentSelectedUsersPage || currentSelectedUsersBatch}"
            @click="()=>{ if(currentSelectedUsersPage) currentSelectedUsersPage--; else if(currentSelectedUsersBatch) { currentSelectedUsersPage = numberOfPagePerBatch - 1; currentSelectedUsersBatch--; } }"
            ) left
        span.morePage(
            :class="{active: currentSelectedUsersBatch}"
            @click="()=>{ if(currentSelectedUsersBatch > 0) {currentSelectedUsersBatch--; currentSelectedUsersPage = numberOfPagePerBatch - 1} }"
            ) ...
        span.page(
            v-for="(i, idx) in groupedUserList?.[currentSelectedUsersBatch].length"
            :class="{active: idx === currentSelectedUsersPage}"
            @click="currentSelectedUsersPage = idx"
            ) {{ currentSelectedUsersBatch * numberOfPagePerBatch + i }}
        span.morePage(
            :class="{active: !isEndOfList || !isLastBatch }"
            @click="getMoreUsers") ...

        Icon(
            :class="{active: !isEndOfList || !isLastBatch || !isLastPage }"
            @click="() => { if(!isLastPage) currentSelectedUsersPage++; else if(isLastPage && !isLastBatch) { currentSelectedUsersBatch++; currentSelectedUsersPage = 0;} else if(isLastBatch && !isEndOfList) getMoreUsers() }"
            ) right

sui-overlay(ref="confirmOverlay")
    .popup
        .title
            Icon warning
            div(v-if="actionType === 'delete'") Deleting Users
            div(v-else-if="actionType === 'block'") Blocking Users
            div(v-else-if="actionType === 'unblock'") Unblocking Users
        .body 
            p Do you wish to continue?
        .foot
            sui-button.textButton(type="button") No 
            sui-button.textButton(type="button") Yes
</template>
<script setup>
import { inject, ref, reactive, computed, watch, onMounted, onBeforeUnmount, nextTick } from 'vue';
import { changeSearchCondition, getValidationMessage, placeholder } from '@/helper/users';
import { skapi, state } from '@/main';
import { groupArray, dateFormat } from '@/helper/common';
import { useRoute, useRouter, onBeforeRouteLeave } from 'vue-router';

import Icon from '@/components/Icon.vue';
import SearchNavBar from '@/components/SearchNavBar.vue';

let route = useRoute();
let router = useRouter();
let serviceId = route.params.service;
let searchValue = ref('');
const searchField = ref(null);
const service = inject('service');

let fetchLimit = 50;
let numberOfUsersPerPage = 10;
let numberOfPagePerBatch = fetchLimit / numberOfUsersPerPage;

const filterEl = ref(null);

const currentSelectedUsersBatch = ref(0);
const currentSelectedUsersPage = ref(0);

const visibleFields = reactive({
    approved: {
        text: 'Block',
        show: true,
    },
    group: {
        text: 'Active',
        show: true,
    },
    access_group: {
        text: 'Access',
        show: false,
    },
    user_id: {
        text: 'User ID',
        show: true,
    },
    name: {
        text: 'Name',
        show: true,
    },
    email: {
        text: 'Email',
        show: true,
    },
    address: {
        text: 'Address',
        show: false,
    },
    phone_number: {
        text: 'Phone',
        show: false
    },
    gender: {
        text: 'Gender',
        show: false,
    },
    birthdate: {
        text: 'Birthdate',
        show: false,
    },
    timestamp: {
        text: 'Date Created',
        show: false,
    },
    locale: {
        text: 'Locale',
        show: false,
    }
});

const isLastBatch = computed(() => {
    return currentSelectedUsersBatch.value === groupedUserList?.value?.length - 1;
});

const isLastPage = computed(() => {
    return currentSelectedUsersPage.value === groupedUserList?.value?.[currentSelectedUsersBatch.value]?.length - 1;
});

const isEndOfList = computed(() => {
    return serviceUsers.value?.endOfList;
});

const searchParams = reactive({
    service: serviceId,
    searchFor: 'timestamp',
    condition: '>=',
    value: ''
});
const navbarBackDestination = inject('navbarBackDestination');
const promiseRunning = ref(false);
const confirmOverlay = ref(null);
const actionType = ref('');

const toggleSetting = async (state) => {
    showSetting.value = state;
    if(state) {
        await nextTick();
        window.addEventListener('scroll', setBodyHeight, true);
    } else {
        document.querySelector('.servicePageShell').style.height = null;
    }
}

const setBodyHeight = () => {
    let neededHeight = window.pageYOffset + filterEl.value?.getBoundingClientRect().top + filterEl.value.offsetHeight;
    if(document.querySelector('.servicePageShell').offsetHeight < neededHeight) {
        document.querySelector('.servicePageShell').style.height = neededHeight + 20 + 'px';
        window.removeEventListener('scroll', setBodyHeight, true);
    }
}

const openUser = (user_id) => {
    router.push({name: 'userView', params: {user_id}})
}

const changeSearchType = (value) => {
    let field = searchField.value.children[0];
    field.setCustomValidity('');
    
    changeSearchCondition(value, searchParams);
}

const groupedUserList = computed(() => {
    if (!serviceUsers.value || !serviceUsers.value.list.length) {
        currentSelectedUsersBatch.value = 0;
        return null;
    }

    return groupArray(serviceUsers.value.list, numberOfUsersPerPage, numberOfPagePerBatch);
});

const search = () => {
    let field = searchField.value.children[0];
    
    let errorMessage = getValidationMessage(searchParams);
    if(errorMessage) {
        field.setCustomValidity(errorMessage);
        field.reportValidity();
    }

    if(!field.checkValidity()) {
        return;
    }

    router.push({
        name:"users",
        query: {
            search: searchParams.searchFor, 
            condition: searchParams.condition, 
            value: searchParams.value
        }
    });

    callSearch();
}

const getCleanSearchParams = () => {
    let params = {
        ...searchParams
    }

    if(params.searchFor === 'timestamp') {
        if(params.value === '') params.value = 0;
        else {
            params.value = new Date(params.value).getTime();
        }
    }

    return params;
}
const callSearch = () => {
    fetchingData.value = true;
    serviceUsers.value = null;

    let params = getCleanSearchParams();

    skapi.getUsers(params, {
        fetchMore: false,
        limit: fetchLimit 
    }).then((res) => {
        fetchingData.value = false;
        serviceUsers.value = {
            endOfList: res.endOfList,
            list: res.list
        };
    });
}

let showSetting = ref(false);

const computedVisibleFields = computed(() => {
    return Object.entries(visibleFields).filter(field => field[1].show).map(field => field[0]);
});
const selectedBlockedUsers = ref([]);
const selectedUnblockedUsers = ref([]);

const selectedUsers = computed(() => {
    return [...selectedBlockedUsers.value, ...selectedUnblockedUsers.value];
});

const userSelectionHandler = (e) => {
    let user = groupedUserList.value[currentSelectedUsersBatch.value][currentSelectedUsersPage.value].find((user) => {
        return user.user_id === e.target.value;
    });
    if (e.target.checked) {
        if(user.approved.includes('suspended')) {
            selectedBlockedUsers.value.push(e.target.value);
        } else {
            selectedUnblockedUsers.value.push(e.target.value);
        }
    } else {
        if(user.approved.includes('suspended')) {
            selectedBlockedUsers.value.splice(selectedBlockedUsers.value.indexOf(e.target.value), 1);
        } else {
            selectedUnblockedUsers.value.splice(selectedUnblockedUsers.value.indexOf(e.target.value), 1);
        }
    }
};

const selectAllHandler = (e) => {
    selectedBlockedUsers.value = [];
    selectedUnblockedUsers.value = [];
    
    if (e.target.checked) {
        groupedUserList.value[currentSelectedUsersBatch.value][currentSelectedUsersPage.value].map(user => {
            if(user.approved.includes('suspended')) {
                selectedBlockedUsers.value.push(user.user_id);
            } else {
                selectedUnblockedUsers.value.push(user.user_id);
            }
        });
    }
};

let pageTitle = inject('pageTitle');

// flag
let fetchingData = inject('fetchingData');
let isFabOpen = ref(false);

// data
let serviceUsers = inject('serviceUsers');
let searchResult = inject('searchResult');

let getMoreUsersQueue = null;

async function getMoreUsers() {
    if (serviceUsers.value?.endOfList && groupedUserList.value.length - 1 === currentSelectedUsersBatch.value) {
        return;
    }
    fetchingData.value = true;

    if (groupedUserList.value.length - 1 > currentSelectedUsersBatch.value) {
        currentSelectedUsersBatch.value += 1;
        currentSelectedUsersPage.value = 0;
        fetchingData.value = false;
        return;
    }

    if (getMoreUsersQueue instanceof Promise) {
        return;
    }
    
    let params = getCleanSearchParams();

    getMoreUsersQueue = skapi.getUsers(
        route.query.search ? params :
        {
            service: serviceId,
            searchFor: 'timestamp',
            condition: '>=',
            value: 0
        }, { fetchMore: true, limit: fetchLimit }).catch(err => {
            fetchingData.value = false;
            throw err;
        });

    let result = await getMoreUsersQueue;
    serviceUsers.value.endOfList = result.endOfList;

    serviceUsers.value.list = serviceUsers.value.list.concat(result.list);

    getMoreUsersQueue = null;
    currentSelectedUsersBatch.value++;
    currentSelectedUsersPage.value = 0;
    fetchingData.value = false;
}

function getUsers(refresh = false) {
    // initial table fetch
    if (!refresh && serviceUsers.value) {
        // bypass if already fetched
        fetchingData.value = false;
        return;
    }

    serviceUsers.value = null;
    fetchingData.value = true;

    let params = {
        service: serviceId,
        searchFor: 'timestamp',
        condition: '>=',
        value: 0
    };

    skapi.getUsers(route.query.search ? searchParams : params, { limit: fetchLimit })
        .then(t => {
            serviceUsers.value = {
                endOfList: t.endOfList,
                list: t.list,
                params
            };

            fetchingData.value = false;
        }).catch(err => {
            fetchingData.value = false;
            throw err;
        });

    return;
}

// get users on created
if(route.query.search) {
    searchParams.searchFor = route.query.search;
    searchParams.condition = route.query.condition;
    searchParams.value = route.query.value;
}

function numberOfSkeletons() {
    // calculated by available vertical space
    return parseInt((window.innerHeight / 2) / 48);
}

const confirmActionDialog = async () => {
    return new Promise((res, rej) => {
        confirmOverlay.value.open();
        const noButton = confirmOverlay.value.querySelectorAll('sui-button')[0];
        const yesButton = confirmOverlay.value.querySelectorAll('sui-button')[1];

        noButton.addEventListener('click', () => {
            confirmOverlay.value.close();
            rej();
        });

        yesButton.addEventListener('click', async () => {
            confirmOverlay.value.close();
            res();
        });

    });
}

const blockUsers = async () => {
    if(promiseRunning.value || !selectedUnblockedUsers.value.length || selectedBlockedUsers.value.length || !state.user.email_verified) return false;
    try {
        actionType.value = 'block';
        await confirmActionDialog();
    } catch(e) {
        return false;
    }
    
    promiseRunning.value = true;
    let blockPromise = selectedUsers.value.map((user) => {
        return skapi.blockAccount({service: serviceId, userId: user});
    });

    await Promise.all(blockPromise);
    selectedUnblockedUsers.value.forEach((sel) => {
        let idx = groupedUserList.value[currentSelectedUsersBatch.value][currentSelectedUsersPage.value].findIndex((item) => {
            return item.user_id === sel
        });
        groupedUserList.value[currentSelectedUsersBatch.value][currentSelectedUsersPage.value][idx].approved = 'admin:suspended';
    });

    selectedBlockedUsers.value = [...selectedUnblockedUsers.value];
    selectedUnblockedUsers.value = [];
    
    promiseRunning.value = false;
}

const unblockUsers = async () => {
    if(promiseRunning.value || !selectedBlockedUsers.value.length || selectedUnblockedUsers.value.length || !state.user.email_verified) return false;
    try {
        actionType.value = 'unblock';
        await confirmActionDialog();
    } catch(e) {
        return false;
    }

    promiseRunning.value = true;
    let unblockPromise = selectedUsers.value.map((user) => {
        return skapi.unblockAccount({service: serviceId, userId: user});
    });

    await Promise.all(unblockPromise);
    selectedBlockedUsers.value.forEach((sel) => {
        let idx = groupedUserList.value[currentSelectedUsersBatch.value][currentSelectedUsersPage.value].findIndex((item) => {
            return item.user_id === sel
        });
        groupedUserList.value[currentSelectedUsersBatch.value][currentSelectedUsersPage.value][idx].approved = 'admin:approved';
    });
    selectedUnblockedUsers.value = [...selectedBlockedUsers.value];
    selectedBlockedUsers.value = [];

    promiseRunning.value = false;
}

const deleteUsers = async () => {
    if(promiseRunning.value || !selectedUsers.value.length || !state.user.email_verified) return false;
    try {
        actionType.value = 'delete';
        await confirmActionDialog();
    } catch(e) {
        return false;
    }

    promiseRunning.value = true;
    if(selectedUsers.value.length === 0 || !state.user.email_verified) return false;

    let deletePromise = selectedUsers.value.map((userId) => {
        document.getElementById(userId).classList.add('deleting');
        return skapi.deleteAccount({service: serviceId, userId});
    });

    try {
        let result = await Promise.all(deletePromise);
        
        selectedUsers.value.forEach((user) => {
            let idx = serviceUsers.value.list.findIndex((res) => res.user_id === user);
            serviceUsers.value.list.splice(idx, 1);
        });
        selectedBlockedUsers.value = [];
        selectedUnblockedUsers.value = [];
    } catch(e) {
        console.log({e});
        
        selectedUsers.value.forEach((user) => {
            document.getElementById(user).classList.remove('deleting');
        });
    }

    promiseRunning.value = false;
}

onMounted(() => {    
    if(route.query.search) {
        navbarBackDestination.value = () => {
            pageTitle.value = "Users";
            router.push({name: 'users'});
        };
        callSearch();
    } else {
        navbarBackDestination.value = null;
        if(!serviceUsers.value?.list) {
            getUsers(true);
        }
    }
});

watch([currentSelectedUsersBatch, currentSelectedUsersPage], () => {
    selectedBlockedUsers.value = [];
    selectedUnblockedUsers.value = [];
});

watch(() => route.query, () => {
    if(route.query.search) {
        searchParams.searchFor = route.query.search;
        searchParams.condition = route.query.condition;
        searchParams.value = route.query.value;
    } 
    else {
        getUsers(true);
    }
})

document.body.classList.add('table');
onBeforeUnmount(() => {
    document.body.classList.remove('table');
});

onBeforeRouteLeave((to, from) => {
    if(from.params.search && !to.params.search) {
        getUsers(true);
    }
});
</script>
<style lang="less" scoped>
.actionsWrapper {
    display: flex;
    align-items: center;
    justify-content: space-between;

    .actions {
        flex-shrink: 0;

        &>* {
            display: inline-block;
            margin-left: 16px;
            padding: 9px 12px;
        }

        svg {
            width: 20px;
            height: 20px;
            margin-right: 4px;
        }

    }

    .inputField {
        .icon {
            padding-right: 12px;

            svg {
                color: rgba(255, 255, 255, 0.4);
            }
        }

        sui-input {
            input::placeholder {
                background-image: url(/src/assets/img/icons/search.svg);
                background-size: 20px;
                background-position: 10px center;
                background-repeat: no-repeat;
            }
            input::-webkit-input-placeholder {
                background-position: 0 center;
            }
        }
    }
}

.tableOuterWrapper {
    position: relative;
    margin-top: 24px;
    background-color: #434343;
    border-radius: 8px;
    border: 1px solid rgba(255, 255, 255, 0.2);
    box-shadow: -1px -1px 1px rgba(0, 0, 0, 0.25), inset 1px 1px 1px rgba(0, 0, 0, 0.5);

    .searchQuery {
        display: flex;
        justify-content: space-between;
        font-weight: bold;
        padding: 24px 36px 0 20px;
        background: #434343;
        border-radius: 8px 8px 0 0;

        & + .tableActions {
            border-radius: 0;
        }

        svg {
            margin-left: 4px;
        }
    }

    .tableActions {
        display: flex;
        justify-content: space-between;
        align-items: center;
        background: #434343;
        padding: 24px 36px 24px 20px;
        border-radius: 8px 8px 0 0;
        color: rgba(255, 255, 255, 0.6);

        &>* {
            cursor: pointer;
        }

        &.roundedBorder {
            border-radius: 8px;
        }

        .refresh {
            color: #fff;
        }

        .headerActions {
            position: relative;

            &Before {
                position: fixed;
                top: 0;
                bottom: 0;
                right: 0;
                left: 0;
                z-index: 8;
            }

            .dropdown>* {
                vertical-align: middle;
            }

            sui-select {
                position: absolute;
                top: 0;
                opacity: 0;
            }
        }
    }

    .filter {
        position: absolute;
        z-index: 9;
        padding: 12px;
        left: 14px;
        background: #595959;
        min-width: 200px;
        box-shadow: 2px 2px 4px rgba(0, 0, 0, 0.25);
        border-radius: 4px;
        z-index: 9;
        margin-top: 12px;

        .label:not(:last-child) {
            margin-bottom: 16px;
        }

        label {
            cursor: pointer;

            span {

                margin-left: 12px;
            }
        }
    }
}

.tableWrapper {
    max-height: calc(52px * 11 + 52px);
    overflow: auto;

    table {
        min-width: 100%;
        border-collapse: collapse;
        
        thead,
        tbody {
            tr {
                background-color: #434343;

                td,
                th {
                    padding: 12px;
                    height: 52px;
                    white-space: nowrap;

                    &:first-child {
                        padding-left: 20px;
                        width: 48px;
                        min-width: 48px;
                    }

                    sui-input {
                        font-size: 16px;
                    }
                }

                &.rounded,
                &.rounded th {
                    border-radius: 0 0 8px 8px;
                }
            }
        }

        thead {
            th {
                position: sticky;
                background-color: #434343;
                color: rgba(255, 255, 255, 0.6);
                top: 0;
                text-align: left;

                &.iconTd {
                    width: 48px;
                    text-align: center;
                }

                &.userId {
                    min-width: 330px;
                }

                &:last-child:not(.iconTd) {
                    width: 100%;
                }
            }

            .actions {
                cursor: pointer;
                text-align: right;

                svg {
                    margin-left: 4px;
                }
            }
        }

        tbody {
            tr {
                &:nth-child(odd) {
                    background: #4a4a4a;
                }

                &.deleting td {
                    opacity: 0.3;
                }

                td {
                    font-size: 14px;

                    &.iconTd {
                        width: 48px;
                        text-align: center;
                    }

                    &:last-child:not(.iconTd) {
                        width: 100%;
                    }
                }
            }
        }

        tr {
            height: 52px;

            td {
                white-space: nowrap;
            }
        }
    }
}

.noUsersFound {
    text-align: center;
    padding: 40px 0 60px 0;
    border-radius: 0 0 8px 8px;
    color: rgba(255, 255, 255, .4);
    background: #434343;

    .title {
        font-size: 28px;
    }
    
    p {
        margin: 20px 0 0 0;
    }
}

.paginator {
    padding: 24px 0;
    text-align: center;
    color: rgba(255 255 255 / 60%);
    background: #434343;
    user-select: none;
    border-radius: 0 0 8px 8px;

    span {
        padding: 4px 8px;
        box-sizing: content-box;

        &.page {
            cursor: pointer;

            &.active {
                cursor: default;
                color: #fff;
                font-weight: bold;
            }
        }

        &.morePage {
            visibility: hidden;

            &.active {
                cursor: pointer;
                visibility: visible;
            }
        }
    }

    svg {
        color: rgba(255, 255, 255, .15);
        vertical-align: middle;

        &.active {
            cursor: pointer;
            color: #fff;
        }
    }
}

.pageAction {
    position: fixed;
    bottom: 76px;
    right: 16px;
    overflow: hidden;

    &>sui-button {
        z-index: 2;
    }

    &,
    &>div {
        display: flex;
        flex-direction: column-reverse;
        align-items: center;
        gap: 12px;
        width: 48px;
    }

    .v-enter-active,
    .v-leave-active {
        transition: opacity .1s ease, transform .1s ease;
        opacity: 1;
    }

    .v-enter-from,
    .v-leave-to {
        opacity: 0;
        transform: translateY(100px);
    }
}

.searchInputWrapper {
    & > sui-select {
        background: rgba(255, 255, 255, 0.08);
    }
}
.selectInput {
    width: 300px;
    margin-left: 16px;
    .inputField {
        width: 100%;
    }

    &>*:first-child:not(:only-child) {
        &::after {
            content: '';
            display: inline-block;
            width: 1px;
            height: 1em;
            vertical-align: middle;
            background-color: rgba(255, 255, 255, .2);
            position: absolute;
            top: 50%;
            transform: translateY(-50%);
        }
    }
}

input {
    -webkit-appearance: none;
}
</style>
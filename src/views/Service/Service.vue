<template lang="pug">
EditService(v-if="state?.user && route.query.edit === 'service'" @close="router.replace({query: null})")
template(v-else)
    .pageHeader.headSpaceHelper
        h2 Service
        p.
            A service is a collection of serverless resources working together to form your backend.
            Simply connect to your service and start building your website.      

        div.action
            a(href="https://docs.skapi.com/getting-started/" target="_blank")
                sui-button.lineButton(type="button") Find out More

    // service information
    .container
        .innerContainer 
            .titleActionsWrapper
                .titleWrapper
                    Icon information
                    h2 Service Information
                .actions(@click="deleteServiceAsk" :class="{'disabled': !state.user.email_verified ? true : null}")
                    Icon(style="width: 20px; height: 20px;") trash
                    span Delete
            .informationGrid
                .informationGridItem(v-for="info in informationGrid" :class="[info.span ? `span-${info?.span}` : '']")
                    .name {{ info.name }}
                    .value(v-if="info.filter") {{ info.filter(service[info.key]) }}
                    .value(v-else) {{ service[info.key] }}
    // service setting
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

    // email_triggers_basic
    .container 
        .innerContainer 
            .titleActionsWrapper
                .titleWrapper
                    Icon mail
                    h2 Automated Emails
            .emailGrid
                p Automated emails are sent to your users when certain events occur.
                p You can customize the content of these emails by sending your customized version of the email to the addresses below.
                p
                    |For more information refer the 
                    a(href='https://docs.skapi.com/email') Documentation
                br

                template(v-if="emailGrid")
                    .emailGridItem
                        .name Welcome
                        .value {{ service?.email_triggers?.template_setters?.welcome }}
                        .actions(@click="copy")
                            Icon copy   
                    .emailGridItem
                        .name Verification
                        .value {{ service?.email_triggers?.template_setters?.verification }}
                        .actions(@click="copy")
                            Icon copy  
                    .emailGridItem
                        .name Signup Confirmation
                        .value {{ service?.email_triggers?.template_setters?.signup_confirmation }}
                        .actions(@click="copy")
                            Icon copy  
                    .emailGridItem
                        .name Newsletter Subscription
                        .value {{ service?.email_triggers?.template_setters?.newsletter_subscription }}
                        .actions(@click="copy")
                            Icon copy
                .fetching(v-else-if="fetchingEmail" style="text-align:center;")
                    Icon.animationRotation refresh

    // email_triggers_newsletter
    .container 
        .innerContainer 
            .titleActionsWrapper
                .titleWrapper
                    Icon mail
                    h2 Newsletters
            .emailGrid  
                p By sending the newsletter email to the addresses below,
                p You can send newsletters either to your public subscribers or your service users.
                p
                    | For more information refer the 
                    a(href='https://docs.skapi.com/email') Documentation
                br
                .emailGridItem
                    .name Public Newsletter
                    .value
                        template(v-if="service?.newsletter_triggers?.[0]") {{service?.newsletter_triggers?.[0]}}
                        template(v-else) Loading...
                    .actions(@click="copy")
                        Icon copy 
                .emailGridItem
                    .name Service User Newsletter
                    .value
                        template(v-if="service?.newsletter_triggers?.[1]") {{service?.newsletter_triggers?.[1]}}
                        template(v-else) Loading...
                    .actions(@click="copy")
                        Icon copy 

    // user & record
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

    // subdomain setting
    .container 
        .innerContainer 
            .titleActionsWrapper(v-if="!service.subdomain" style="margin-bottom: 0;")
                .titleWrapper
                    Icon domain
                    h2 Subdomain
                .actions(@click="create")
                    Icon pencil
                    span Create
            .titleActionsWrapper(v-else)
                .titleWrapper
                    Icon domain
                    h2 Subdomain
                .actions(v-if="service.subdomain && !deleting" @click="deleteSubdomainAsk" :class="{'disabled': !state.user.email_verified || null}")
                    Icon(style="width: 20px; height: 20px;") trash
                    span Delete
            .domainGrid(v-if="domain")
                .domainGridItem
                    .name
                        span Subdomain
                    .value
                        span {{ service.subdomain }}.skapi.com
                    a(:href="`http://${service.subdomain}.skapi.com`" target="_blank")
                        Icon link
                .domainGridItem.btn(@click="setting404")
                    Icon setting
                    span Setting
                .domainGridItem.btn(@click="upload")
                    Icon file
                    span Upload file
                .domainGridItem.btn(@click="refreshCDN")
                    Icon(:class="{'animationRotation': isCDNRefreshing}") refresh
                    span Refresh CDN
            .domainGrid.deleting(v-else-if="deleting") 
                h3 Deleting subdomain ...
                span It may take a few minutes for a subdomain to be deleted.

    // uploaded list
    .container(v-if="domain")
        .innerContainer    
            .titleActionsWrapper
                .titleWrapper
                    Icon list
                    h2 File List
                .actions(@click="deleteFiles" :class="{'disabled': (selectedFiles.length === 0 || !state.user.email_verified) || null}")
                    Icon(style="width: 20px; height: 20px;") trash
                    span Delete
            .filesContainer
                .fetching(v-if="isFetching && !service?.files?.[service.subdomain+currentDirectory]?.list?.length" style="text-align:center;")
                    Icon.animationRotation refresh
                template(v-else-if="isEmpty")
                    div.noFiles
                        div.title No Files
                        p You have not uploaded any files
                template(v-else-if="service?.files[service.subdomain+currentDirectory].list.length")
                    .directoryName
                        Icon(@click="goUpDirectory") upload
                        .pathWrapper
                            span.path(v-for="(folder, index) in currentDirectoryArray" @click="jumpto(currentDirectoryArray.length - index)")
                                span {{ folder }}/
                            span /
                    .directoryFiles(@scroll="scrollEvent")
                        template(v-for="(file) in service?.files[service.subdomain+currentDirectory].list")
                            .fileWrapper(v-if="!file.file")
                                .file(:class="{fade: isDeleting && selectedFiles.includes(service.subdomain + currentDirectory + file.name)}")
                                    sui-input(type="checkbox" :checked="selectedFiles.includes(service.subdomain + currentDirectory + file.name) || null" @change="checkboxHandler" :value="file.name")
                                    Icon folder2
                                    .pathWrapper(@click="getDirectory(currentDirectory+=file.name)")
                                        span.path {{ file.name }}
                            .fileWrapper(v-else)
                                .file(:class="{fade: isDeleting && selectedFiles.includes(service.subdomain + currentDirectory + file.name)}")
                                    sui-input(type="checkbox" :checked="selectedFiles.includes(service.subdomain + currentDirectory + file.name) || null" @change="checkboxHandler" :value="file.name")
                                    Icon file
                                    .pathWrapper(@click="download(`https://${service.subdomain}.skapi.com${currentDirectory}${file.name}`);")
                                        span.path {{ file.name }}

    // overlay window
    sui-overlay(v-if="isEdit" ref="settingWindow" style="background: rgba(0, 0, 0, 0.6)" @mousedown="async()=>{await state.blockingPromise; settingWindow.close(()=>isEdit = false)}")
        div.overlay
            EditService(@close="async()=>{await state.blockingPromise; settingWindow.close(()=>isEdit = false)}")

    sui-overlay(v-if="isCreate" ref="subdomainWindow" style="background: rgba(0, 0, 0, 0.6)" @mousedown="async()=>{await state.blockingPromise; subdomainWindow.close(()=>isCreate = false)}")
        div.overlay
            Subdomain(@close="async()=>{await state.blockingPromise; subdomainWindow.close(()=>isCreate = false)}")

    sui-overlay(v-if="isUpload" ref="uploadWindow" style="background: rgba(0, 0, 0, 0.6)" @mousedown="async()=>{await state.blockingPromise; uploadWindow.close(()=>isUpload = false)}")
        div.overlay
            AddFiles(@close="async()=>{await state.blockingPromise; uploadWindow.close(()=>isUpload = false)}" :currentDirectory="currentDirectory")

    sui-overlay(v-if="isSetting404" ref="setting404Window" style="background: rgba(0, 0, 0, 0.6)" @mousedown="async()=>{await state.blockingPromise; setting404Window.close(()=>isSetting404 = false)}")
        div.overlay
            Setting404(@close="async()=>{await state.blockingPromise; setting404Window.close(()=>isSetting404 = false)}")

// delete window
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
            p Are you sure you want to delete "{{ service.subdomain }}" permanently? #[br] All uploaded files will be deleted along with your subdomain. #[br] You will not be able to undo this action.
            p To confirm deletion, enter Subdomain name #[br] #[span(style="font-weight: bold") {{ service.subdomain }}]
            sui-input(:placeholder="service.subdomain" :value="confirmationCode" @input="(e) => confirmationCode = e.target.value")
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
import { inject, reactive, ref, watch, nextTick, onBeforeMount, computed, onBeforeUnmount, onMounted } from "vue";
import { state, skapi } from "@/main";
import { localeName, dateFormat, getSize } from "@/helper/common";
import { useRoute, useRouter } from "vue-router";

import EditService from "@/views/Service/EditService.vue";
import Subdomain from "@/views/Service/Subdomain.vue";
import Icon from "@/components/Icon.vue";
import SubmitButton from "@/components/SubmitButton.vue";
import AddFiles from "@/views/Service/AddFiles.vue";
import Setting404 from "@/views/Service/Setting404.vue"

const route = useRoute();
const router = useRouter();

let service = inject("service");
let pageTitle = inject("pageTitle");
pageTitle.value = 'Service "' + service.value.name + '"';

const settingWindow = ref(null);
const subdomainWindow = ref(null);
const uploadWindow = ref(null);
const setting404Window = ref(null);
const deleteConfirmOverlay = ref(null);
const deleteSubdomainOverlay = ref(null);
const deleteErrorOverlay = ref(null);
const confirmationCode = ref("");
const deleteErrorMessage = ref("");
const isEdit = ref(false);
const isCreate = ref(false);
const isUpload = ref(false);
const isSetting404 = ref(false);
const isDisabled = ref(false);
const isFetching = ref(false);
const isDeleting = ref(false);
const isEmpty = ref(false);
const isCDNRefreshing = ref(false);
const domain = ref(false);
const deleting = ref(false);
const showEmail = ref(false);
const fetchingEmail = ref(false);

const folderUpload = ref(null);
const fileUpload = ref(null);
const currentDirectory = ref("/");
const selectedFiles = ref([]);

const informationGrid = reactive([
    {
        name: "Service ID",
        key: "service",
        span: 2,
    },
    {
        name: "Owner's ID",
        key: "owner",
        span: 2,
    },
    // {
    //     name: 'Group',
    //     key: 'group',
    //     filter: (value) => {
    //         return value == 1 ? 'Basic' : 'Premium'
    //     }
    // },
    {
        name: "Service Location",
        key: "region",
        filter: (value) => {
            return localeName(value);
        },
    },
    {
        name: "Date Created",
        key: "timestamp",
        filter: (value) => {
            return dateFormat(value).split(" ")[0];
        },
    },
    // {
    //     name: "Storage Use",
    //     key: "storage",
    //     filter: (value) => {
    //         let val = value || 0;
    //         return getSize(val);
    //     },
    // },
    {
        name: "# of Users",
        key: "users",
    },
    // {
    //     name: '# of Newsletter Sub',
    //     key: 'newsletter_subscribers'
    // },
]);

const settingGrid = reactive([
    {
        name: "Enable/Disable",
        key: "active",
        filter: () => {
            return 1;
            // return .indicator(:class="{'active': service.active > 0}")
        },
    },
    {
        name: "Name of Service",
        key: "name",
    },
    {
        name: "CORS",
        key: "cors",
        tip: "When CORS is set, your website will not be able to connect to your service unless the request comes from a valid host.",
    },
    {
        name: "API Key",
        key: "api_key",
        tip: "You can set your own private API key if you wish to integrate your users' secure requests to your external backend server.",
    },
]);

const emailGrid = computed(() => { return service?.value?.email_triggers?.template_setters });

const edit = () => {
    if (!state.user.email_verified) return false;
    isEdit.value = true;
};

const create = () => {
    if (!state.user.email_verified) return false;
    isCreate.value = true;
};

const upload = () => {
    if (!state.user.email_verified) return false;
    isUpload.value = true;
};

const setting404 = () => {
    if (!state.user.email_verified) return false;
    isSetting404.value = true;
}

const copy = (e) => {
    let doc = document.createElement('textarea');

    doc.textContent = e.target.parentNode.parentNode.previousElementSibling.innerText;
    document.body.append(doc);
    doc.select();
    document.execCommand('copy');
    doc.remove();

    alert('The code has been copied.');
}

const deleteServiceAsk = () => {
    if (!state.user.email_verified) return;
    deleteConfirmOverlay.value.open();
};

const deleteSubdomainAsk = () => {
    if (!state.user.email_verified) return;
    deleteSubdomainOverlay.value.open();
};

//upload File
const addFileButtonHandler = () => {
    const parent = fileUpload.value.click();
};

const addFolderButtonHandler = () => {
    const parent = folderUpload.value.click();
};

const goto = (name) => {
    currentDirectory.value = name;
    getDirectory(name.slice(0, -1).split("/"));
};

const getDirectory = (directory = '/') => {
    let findingDirectory = service.value.subdomain + (directory ? directory : "/");
    if (service.value.files?.[findingDirectory]) {
        return service.value.files[findingDirectory];
    }

    let params = {
        service: service.value.service,
    };

    if (directory) {
        params.dir = directory;
    }

    isFetching.value = true;

    skapi.listHostDirectory(params).then((files) => {
        if (!service.value.hasOwnProperty("files")) {
            service.value.files = {};
            isEmpty.value = true;
        } else {
            isEmpty.value = false;
        }
        if (
            !service.value.files[
            `${service.value.subdomain}${currentDirectory.value}`
            ]
        ) {
            service.value.files[
                `${service.value.subdomain}${currentDirectory.value}`
            ] = {
                endOfList: files.endOfList,
                list: [],
            };
        }

        if (files.list.length == 0) {
            isEmpty.value = true;
        } else {
            isEmpty.value = false;
        }

        files.list.forEach((file) => {
            if (file.name !== service.value.subdomain + '/.cfacdb7c8270a90aba6011585793dfc3/' && file.name !== service.value.subdomain + '/cfacdb7c8270a90aba6011585793dfc3.html') {
                let filename = extractFileName(file.name);

                if (file.type === "folder") {
                    service.value.files[
                        `${service.value.subdomain}${currentDirectory.value}`
                    ].list.push({
                        name: filename,
                        type: "folder",
                    });
                } else {
                    service.value.files[
                        `${service.value.subdomain}${currentDirectory.value}`
                    ].list.push({
                        type: "file",
                        file, // url: `https://${service.value.subdomain}.skapi.com${currentDirectory.value}${filename}`,
                        name: filename,
                    });
                }
            }
        });

        isFetching.value = false;
    });
};

const goUpDirectory = () => {
    let directoryArray = [...currentDirectoryArray.value];
    directoryArray.reverse();
    directoryArray.pop();

    let newDirectory = "/";

    if (directoryArray.length) {
        newDirectory = `/${directoryArray.join("/")}/`;
        getDirectory(newDirectory);
    } else {
        getDirectory();
    }

    currentDirectory.value = newDirectory;
};

const deleteFiles = () => {
    isDeleting.value = true;

    skapi.deleteHostFile({
        keys: selectedFiles.value,
        service: service.value.service,
    })
        .then(async (res) => {
            selectedFiles.value.forEach((path) => {
                const regex = /^.*?\//;
                let result = path.replace(regex, "");
                let pathArray = path.split("/");
                let subdomain = pathArray[0];
                let index;

                if (result[result.length - 1] === "/") {
                    index = service.value.files[`${subdomain}${currentDirectory.value}`].list.findIndex((path) => {
                        return path.name === pathArray[pathArray.length - 2] + "/";
                    });
                } else {
                    index = service.value.files[`${subdomain}${currentDirectory.value}`].list.findIndex((path) => {
                        return path.name === extractFileName(result);
                    });
                }

                service.value.files[`${subdomain}${currentDirectory.value}`].list.splice(index, 1);

                if (service.value.files[`${subdomain}${currentDirectory.value}`].list.length <= 0) {
                    let oldDirectory = currentDirectory.value;
                    let newDirectory = currentDirectory.value.split("/");
                    newDirectory.splice(-2);
                    currentDirectory.value = newDirectory.join("/") + "/";
                    let folderToDelete = oldDirectory.replace(currentDirectory.value, "");
                    index = service.value.files[`${subdomain}${currentDirectory.value}`].list.findIndex((path) => {
                        return path.name === folderToDelete;
                    });

                    service.value.files[`${subdomain}${currentDirectory.value}`].list.splice(index, 1);
                }
            });

            isDeleting.value = false;
            selectedFiles.value = [];
            await getMoreDirectory();
        });
};

const jumpto = (index) => {
    getDirectory(`/${currentDirectoryArray.value.slice(index * -1).reverse().join("/")}/`);
};

function extractFileName(file) {
    const regex = /\/([^/]+)\/?$|([^/]+)$/;
    const match = file.match(regex);

    if (match && match.length > 0) {
        return match[0].replace("/", "");
    }

    return null;
}

const checkboxHandler = (e) => {
    if (e.target.checked) {
        selectedFiles.value.push(`${service.value.subdomain}/${e.target.value}`);
    } else {
        selectedFiles.value.splice(selectedFiles.value.indexOf(`${service.value.subdomain}/${e.target.value}`), 1);
    }
};

const currentDirectoryArray = computed(() => {
    selectedFiles.value = [];
    return currentDirectory.value.split("/").reverse().filter((value) => { return value; });
});

const deleteService = () => {
    isDisabled.value = true;
    if (confirmationCode.value !== service.value.service) {
        confirmationCode.value = "";
        deleteErrorMessage.value = "Your service code did not match.";
        if (deleteConfirmOverlay.value) deleteConfirmOverlay.value.close();
        deleteErrorOverlay.value.open();
        isDisabled.value = false;
        return;
    }

    skapi.deleteService(service.value.service).then(() => {
        if (deleteConfirmOverlay.value) deleteConfirmOverlay.value.close();
        router.replace("/admin");
    })
        .catch(() => {
            deleteErrorMessage.value = "Please disable your service before deleting it.";
            if (deleteConfirmOverlay.value) deleteConfirmOverlay.value.close();
            deleteErrorOverlay.value.open();
        })
        .finally(() => {
            confirmationCode.value = "";
            isDisabled.value = false;
        });
};

let intervalId;

async function checkServiceStatus() {
    console.log('start');
    let serviceId = service.value.service;

    let services = await skapi.getServices(service.value.service);
    let region = skapi.region_list?.[serviceId.substring(0, 4)];

    if (services[region]) {
        for (let s of services[region]) {
            if (s.service === serviceId) {
                service.value = s;
                break;
            }
        }
    }

    if (!service.value.subdomain) {
        console.log('no subdomain');
        domain.value = false;
        deleting.value = false;
        clearInterval(intervalId);
    }
}

const deleteSubdomain = async () => {
    isDisabled.value = true;
    if (confirmationCode.value !== service.value.subdomain) {
        confirmationCode.value = "";
        deleteErrorMessage.value = "Your subdomain name did not match.";
        if (deleteSubdomainOverlay.value) deleteSubdomainOverlay.value.close();
        deleteErrorOverlay.value.open();
        isDisabled.value = false;
        return;
    }

    try {
        await skapi.registerSubdomain({
            service: service.value.service,
            subdomain: service.value.subdomain,
            exec: "remove",
        }).then(() => {
            // deleting.value = true;
        });
    } catch (e) {
        deleteErrorMessage.value = e.message;
        if (deleteSubdomainOverlay.value) deleteSubdomainOverlay.value.close();
        deleteErrorOverlay.value.open();
        isDisabled.value = false;
    } finally {
        deleting.value = true;
        domain.value = false;
        isDisabled.value = false;
        deleteSubdomainOverlay.value.close();
        intervalId = setInterval(checkServiceStatus, 3000);
    }
};

const getMoreDirectory = async (directory) => {
    let findingDirectory = service.value.subdomain + (directory ? directory : '/');
    if (isFetching.value || service.value.files[findingDirectory].endOfList) return false;

    let params = {
        service: service.value.service,
    };

    let fetchOptions = {
        fetchMore: true,
    }

    if (directory) {
        params.dir = directory;
    }

    isFetching.value = true;

    skapi.listHostDirectory(params, fetchOptions).then((files) => {
        files.list.forEach((file) => {
            let filename = extractFileName(file.name);

            if (file.type === "folder") {
                service.value.files[`${service.value.subdomain}${currentDirectory.value}`].list.push({
                    name: filename,
                    type: "folder",
                });
            } else {
                service.value.files[`${service.value.subdomain}${currentDirectory.value}`].list.push({
                    type: "file",
                    file, // url: `https://${service.value.subdomain}.skapi.com${currentDirectory.value}${filename}`,
                    name: filename,
                });
            }
        });

        isFetching.value = false;
    });
};

const scrollEvent = (e) => {
    const container = e.target;
    const scrollPosition = container.scrollHeight - container.clientHeight;
    if (scrollPosition <= container.scrollTop + 50) {
        getMoreDirectory(currentDirectory.value);
    }
}

const download = async (url) => {
    let file = await skapi.getFile(
        url,
        {
            dataType: 'download',
            noCdn: true,
            service: service.value.service
        }
    );
}

const refreshCDN = () => {
    if (!isCDNRefreshing.value) {
        isCDNRefreshing.value = true;
        skapi.refreshCDN({
            service: service.value.service,
            subdomain: service.value.subdomain
        }).catch((e) => {
            console.log({ e });
        }).finally(() => {
            isCDNRefreshing.value = false;
        });
    }
}

if (!service.value.hasOwnProperty("storage")) {
    skapi.storageInformation(service.value.service).then((storage) => {
        service.value.storage = storage.cloud + storage.database + storage.email;
    });
}

watch(() => isEdit.value,
    async () => {
        await nextTick();
        if (isEdit.value) {
            settingWindow.value.open();
        }
    }
);

watch(() => isCreate.value,
    async () => {
        await nextTick();
        if (isCreate.value) {
            subdomainWindow.value.open();
        }
    }
);

watch(() => isUpload.value,
    async () => {
        await nextTick();
        if (isUpload.value) {
            uploadWindow.value.open();
        }
    }
);

watch(() => isSetting404.value,
    async () => {
        await nextTick();
        if (isSetting404.value) {
            setting404Window.value.open();
        }
    }
);

watch(() => service.value.subdomain,
    () => {
        if ("subdomain" in service.value) {
            if (service.value.subdomain.includes("*")) {
                domain.value = false;
                deleting.value = true;
            } else {
                domain.value = true;
                deleting.value = false;
                getDirectory();
            }
        } else if (!service.value.hasOwnProperty("subdomain")) {
            domain.value = false;
            deleting.value = false;
        }
    }
);

onBeforeMount(async () => {
    if ("subdomain" in service.value) {
        if (service.value.subdomain.includes("*")) {
            domain.value = false;
            deleting.value = true;
        } else {
            domain.value = true;
            deleting.value = false;
            getDirectory();
        }
    } else {
        domain.value = false;
        deleting.value = false;
    }
});

onMounted(() => {
    if (!service.value.hasOwnProperty('newsletter_triggers')) {
        service.value.newsletter_triggers = [];
        for (let i = 0; i < 2; i++) {
            skapi.requestNewsletterSender(service.value.service, i).then((e) => {
                service.value.newsletter_triggers.push(e);
            });
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
            display: flex;
            justify-content: space-between;
            margin-bottom: 32px;

            h2 {
                font-size: 20px;
                font-weight: normal;
            }

            .titleWrapper {
                h2 {
                    margin: 0;
                }

                svg {
                    margin-right: 8px;
                }
            }

            .actionWrapper {
                display: flex;
                flex-wrap: nowrap;

                .actions {
                    opacity: 1;

                    &:first-child {
                        margin-right: 20px;
                    }

                    &.disabled {
                        opacity: 0.4;
                    }
                }
            }
        }
    }

    h2 {
        color: rgba(255, 255, 255, 0.85);
        display: inline-block;
        vertical-align: middle;
        font-size: 24px;
        margin-bottom: 50px;
        font-weight: bold;
    }

    p {
        color: rgba(255, 255, 255, 0.85);
        line-height: 1.5;
        margin: 0;
    }

    .actions {
        opacity: 1;
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

.emailGrid {
    p {
        color: rgba(255, 255, 255, 0.7);

        a {
            color: #fff;
        }
    }

    &Item {
        margin-bottom: 20px;
        overflow: hidden;
        white-space: nowrap;

        &:last-child {
            margin-bottom: 0;
        }

        .value {
            width: calc(100% - 30px);
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
        }
    }

    .actions {
        position: absolute;
        right: 15px;
        top: 50%;
        transform: translate(0, -50%);
    }
}

.domainGrid {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    gap: 20px;

    &Item {
        &.btn {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            width: calc(33% - 13px);
            cursor: pointer;

            svg {
                margin-bottom: 10px;
            }

            span {
                width: 100%;
                text-align: center;
            }
        }

    }

    &.deleting {
        display: block;
        justify-content: unset;
        text-align: center;
        color: rgba(255, 255, 255, 0.4);
        padding-bottom: 30px;

        h3 {
            font-size: 28px;
            font-weight: 500;
            margin: 0 0 20px 0;
        }

        span {
            font-size: 14px;
        }
    }
}

.domainGrid,
.emailGrid {
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

.fileUploadArea {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    margin-bottom: 28px;
    border: 1px dashed #ffffff;
    border-radius: 8px;
    height: 100px;

    sui-button {
        vertical-align: middle;
    }

    &:only-child {
        margin-bottom: 0;
    }

    &>div>* {
        display: inline-block;

        &:first-child {
            margin-right: 6px;
        }

        &:last-child {
            margin-left: 6px;
        }
    }

    svg {
        height: 57px;
        width: 57px;
        color: rgba(255, 255, 255, 0.6);
    }

    .error {
        color: #ff8d3b;

        svg {
            height: 24px;
            width: 24px;
            fill: #ff8d3b;
        }
    }
}

.directoryName {
    display: flex;
    align-items: center;
    padding: 20px;
    margin-bottom: 40px;

    background: rgba(255, 255, 255, 0.1);
    border-radius: 8px;

    svg {
        margin-right: 16px;
        cursor: pointer;
    }

    &>* {
        flex-shrink: 0;
        flex-grow: 0;
    }

    .pathWrapper {
        display: inline-block;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
        width: 100%;
        direction: rtl;
        text-align: left;

        .path {
            unicode-bidi: plaintext;
            cursor: pointer;
        }
    }
}

.directoryFiles {
    max-height: 500px;
    overflow: scroll;
}

.filesContainer {
    .fileWrapper {
        border-radius: 8px;
        cursor: pointer;

        &:nth-child(even) {
            background: #4a4a4a;
        }
    }

    .file {
        display: flex;
        align-items: center;
        height: 52px;
        padding: 8px 20px;

        &.fade {
            opacity: 0.5;
        }

        .pathWrapper {
            display: inline-block;
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
            width: 100%;
            direction: rtl;
            text-align: left;
            cursor: pointer;

            a {
                color: #fff;
                text-decoration: none;
            }

            .path {
                unicode-bidi: plaintext;
                cursor: pointer;
            }
        }

        &>*:not(.pathWrapper) {
            flex-shrink: 0;
            flex-grow: 0;
        }

        a {
            display: inline-block;
            vertical-align: middle;
            width: calc(100% - 32px);
            color: #fff;
            text-decoration: none;
            font-weight: normal;
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
            direction: rtl;
            text-align: left;
        }

        &>sui-input,
        &>svg {
            margin-right: 16px;
        }

        &>sui-input {
            opacity: 0.5;
        }
    }

    .noFiles {
        padding: 12px 16px;
        text-align: center;
        border-radius: 8px;

        .title {
            font-size: 28px;
        }

        .title,
        p {
            opacity: 0.4;
        }
    }
}

.uploadFilesContainer {
    margin: 28px -20px;

    .file {
        display: flex;
        align-items: center;
        height: 52px;
        padding: 8px 20px;

        .progressBar {
            display: inline-block;
            vertical-align: middle;
            width: 20px;
            height: 20px;
            background: var(--progress);
            border-radius: 50%;
            position: relative;
            margin-right: 16px;

            .circle {
                border: 1px solid white;
                height: 20px;
                width: 20px;
                border-radius: 50%;
                position: absolute;
                opacity: 0;

                &~.circle {
                    border: 1px solid white;
                    opacity: 0;
                }
            }

            &::before {
                content: "";
                position: absolute;
                top: calc(50% - (15px / 2));
                left: calc(50% - (15px / 2));
                display: block;
                width: 15px;
                height: 15px;
                border-radius: 50%;
                background-color: #333;
            }

            &.started {
                &::after {
                    position: absolute;
                    content: url("data:image/svg+xml,%3Csvg width='10' height='10' viewBox='0 0 20 20' fill='black' xmlns='http://www.w3.org/2000/svg'%3E%3Cpolygon points='20,5.78 18.22,4 12,10.22 5.78,4 4,5.78 10.22,12 4,18.22 5.78,20 12,13.78 18.22,20 20,18.22 13.78,12 ' fill='white'/%3E%3C/svg%3E");
                    top: 50%;
                    left: 50%;
                    transform: translate(-6px, -11px);
                }
            }

            &.complete {
                .circle {
                    animation: ripple 0.5s ease;

                    &~.circle {
                        animation: smallRipple 0.3s ease;
                    }
                }

                &::before {
                    top: 0;
                    left: 0;
                    width: 20px;
                    height: 20px;
                    background-color: #5ad858;
                    animation: bounce 0.2s ease;
                }

                &::after {
                    position: absolute;
                    content: url("data:image/svg+xml,%3Csvg width='15' height='15' viewBox='0 0 20 20' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='M9.31,18.6L3,12.29l1.54-1.51l4.77,4.77L19.46,5.4L21,6.91L9.31,18.6z' fill='white'/%3E%3C/svg%3E");
                    top: 50%;
                    left: 50%;
                    transform: translate(-9px, -9px);
                }
            }

            &.failed {
                &::before {
                    top: 0;
                    left: 0;
                    width: 20px;
                    height: 20px;
                    background-color: #f04e4e;
                    animation: bounce 0.2s ease;
                }

                &::after {
                    position: absolute;
                    content: url("data:image/svg+xml,%3Csvg width='15' height='15' viewBox='0 0 20 20' fill='black' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath d='m10.88,3.45h2.17v11.95h-2.17V3.45Zm2.17,15.21v2.17h-2.17v-2.17h2.17Z' fill='white'/%3E%3C/svg%3E");
                    top: 50%;
                    left: 50%;
                    transform: translate(-9px, -9px);
                }
            }
        }

        &:nth-child(even) {
            background: #4a4a4a;

            .progressBar::before {
                background-color: #4a4a4a;
            }

            .progressBar.complete::before {
                background-color: #5ad858;
            }

            .progressBar.failed::before {
                background-color: #f04e4e;
            }
        }

        &>sui-input {
            margin-right: 16px;
        }
    }

    .noFiles {
        padding: 12px 16px;
        text-align: center;
        border-radius: 8px;

        .title {
            font-size: 28px;
        }

        .title,
        p {
            opacity: 0.4;
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
    background: #d9d9d9;
    border: 0.3px solid #595959;
    box-shadow: inset -1px -1px 2px rgba(0, 0, 0, 0.25),
        inset 1px 1px 2px rgba(255, 255, 255, 0.65);
    margin-right: 8px;

    &.active {
        background: #5ad858;
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
        background-color: #bfbfbf;
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
        
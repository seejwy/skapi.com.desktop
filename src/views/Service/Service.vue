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
    // service information
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
    // email_triggers
    .container 
        .innerContainer 
            .titleActionsWrapper
                .titleWrapper
                    Icon setting
                    h2 Email Triggers
            .emailGrid
                .emailGridItem(v-for="(email, name) in emailGrid")
                    .name
                        span {{ name }}
                    .value
                        span {{ email }}
                    .actions(@click="copy" :class="{'disabled': !state.user.email_verified ? true : null}")
                        Icon copy                    
    // subdomain setting
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
    // uploaded list
    .container(v-if="domain")
        .innerContainer    
            .titleActionsWrapper
                .titleWrapper
                    Icon list
                    h2 File List
                .actionWrapper
                    .actions(@click="upload")
                        Icon pencil
                        span Upload
                    .actions(@click="deleteFiles" :class="{'active': !isEmpty}")
                        Icon trash
                        span Delete
            .filesContainer
                .fetching(v-if="isFetching" style="text-align:center;")
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
                    .directoryFiles
                        template(v-for="(file) in service?.files[service.subdomain+currentDirectory].list")
                            .fileWrapper(v-if="!file.file")
                                .file(:class="{fade: isDeleting && selectedFiles.includes(service.subdomain + currentDirectory + file.name)}")
                                    sui-input(type="checkbox" :checked="selectedFiles.includes(service.subdomain + currentDirectory + file.name) || null" @change="checkboxHandler" :value="file.name")
                                    Icon folder2
                                    .path-wrapper(@click="getDirectory(currentDirectory+=file.name)")
                                        span.path {{ file.name }}
                            .fileWrapper(v-else)
                                .file(:class="{fade: isDeleting && selectedFiles.includes(service.subdomain + currentDirectory + file.name)}")
                                    sui-input(type="checkbox" :checked="selectedFiles.includes(service.subdomain + currentDirectory + file.name) || null" @change="checkboxHandler" :value="file.name")
                                    Icon file
                                    a(:href="`https://${service.subdomain}.skapi.com${currentDirectory}${file.name}`" download).path-wrapper
                                        span.path {{ file.name }}
                    //- .paginator(style="text-align:center")
                    //-     Icon.arrow(@click="prevPage" :disabled="currentPage === 1") left
                    //-     span.page(v-for="page in visiblePages" :key="page" @click="gotoPage(page)" :class="{ active: page === currentPage }") {{ page }}
                    //-     Icon.arrow(@click="nextPage" :disabled="currentPage === totalPages") right

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
            p To confirm deletion, enter Service ID #[br] #[span(style="font-weight: bold") {{ service.subdomain }}]
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
import { inject, reactive, ref, watch, nextTick, onBeforeMount, computed } from "vue";
import { state, skapi } from "@/main";
import { localeName, dateFormat, getSize } from "@/helper/common";
import { useRoute, useRouter } from "vue-router";

import EditService from "@/views/Service/EditService.vue";
import Subdomain from "@/views/Service/Subdomain.vue";
import Icon from "@/components/Icon.vue";
import SubmitButton from "@/components/SubmitButton.vue";
import AddFiles from "@/views/Service/AddFiles.vue";

const route = useRoute();
const router = useRouter();

let service = inject("service");
let pageTitle = inject("pageTitle");
pageTitle.value = 'Service "' + service.value.name + '"';

const settingWindow = ref(null);
const subdomainWindow = ref(null);
const uploadWindow = ref(null);
const deleteConfirmOverlay = ref(null);
const deleteSubdomainOverlay = ref(null);
const deleteErrorOverlay = ref(null);
const confirmationCode = ref("");
const deleteErrorMessage = ref("");
const isEdit = ref(false);
const isCreate = ref(false);
const isUpload = ref(false);
const isDisabled = ref(false);
const isFetching = ref(false);
const isEmpty = ref(false);
const domain = ref(false);
const deleting = ref(false);

const filesToUpload = ref(0);
const folderUpload = ref(null);
const fileUpload = ref(null);
const fileList = ref({});
const isComplete = ref(false);
const directoryFiles = ref({});
const isSaving = ref(false);
const numberOfFailedUploads = ref(-1);
const currentDirectory = ref("/");
let abortUpload = "";
const selectedFiles = ref([]);

const isDeleting = ref(false);

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
  {
    name: "Storage Use",
    key: "storage",
    filter: (value) => {
      let val = value || 0;
      return getSize(val);
    },
  },
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

const emailGrid = computed(() => {return service.value.email_triggers.template_setters});

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

const copy = (e) => {
    let doc = document.createElement('textarea');
    
    doc.textContent = e.target.parentNode.parentNode.previousElementSibling.childNodes[0].innerText;
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

const getDirectory = (directory) => {
  let findingDirectory =
    service.value.subdomain + (directory ? directory : "/");
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
    console.log(files);
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

    files.list.forEach((file) => {
      let filename = extractFileName(file.name);

      if (files.list.length == 0) {
        isEmpty.value = true;
      } else {
        isEmpty.value = false;
      }

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

  skapi
    .deleteHostFile({
      keys: selectedFiles.value,
      service: service.value.service,
    })
    .then((res) => {
      selectedFiles.value.forEach((path) => {
        const regex = /^.*?\//;
        let result = path.replace(regex, "");
        let pathArray = path.split("/");
        let subdomain = pathArray[0];
        let index;

        if (result[result.length - 1] === "/") {
          index = service.value.files[
            `${subdomain}${currentDirectory.value}`
          ].list.findIndex((path) => {
            return path.name === pathArray[pathArray.length - 2] + "/";
          });
        } else {
          index = service.value.files[
            `${subdomain}${currentDirectory.value}`
          ].list.findIndex((path) => {
            return path.name === extractFileName(result);
          });
        }

        service.value.files[
          `${subdomain}${currentDirectory.value}`
        ].list.splice(index, 1);

        if (
          service.value.files[`${subdomain}${currentDirectory.value}`].list
            .length <= 0
        ) {
          let oldDirectory = currentDirectory.value;
          let newDirectory = currentDirectory.value.split("/");
          newDirectory.splice(-2);
          currentDirectory.value = newDirectory.join("/") + "/";
          let folderToDelete = oldDirectory.replace(currentDirectory.value, "");
          index = service.value.files[
            `${subdomain}${currentDirectory.value}`
          ].list.findIndex((path) => {
            return path.name === folderToDelete;
          });

          service.value.files[
            `${subdomain}${currentDirectory.value}`
          ].list.splice(index, 1);
        }
      });
      isDeleting.value = false;
      selectedFiles.value = [];
    });
};

const jumpto = (index) => {
  getDirectory(
    `/${currentDirectoryArray.value
      .slice(index * -1)
      .reverse()
      .join("/")}/`
  );
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
    selectedFiles.value.splice(
      selectedFiles.value.indexOf(
        `${service.value.subdomain}/${e.target.value}`
      ),
      1
    );
  }
};

const currentDirectoryArray = computed(() => {
  selectedFiles.value = [];
  return currentDirectory.value
    .split("/")
    .reverse()
    .filter((value) => {
      return value;
    });
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

  skapi
    .deleteService(service.value.service)
    .then(() => {
      if (deleteConfirmOverlay.value) deleteConfirmOverlay.value.close();
      router.replace("/admin");
    })
    .catch(() => {
      deleteErrorMessage.value =
        "Please disable your service before deleting it.";
      if (deleteConfirmOverlay.value) deleteConfirmOverlay.value.close();
      deleteErrorOverlay.value.open();
    })
    .finally(() => {
      confirmationCode.value = "";
      isDisabled.value = false;
    });
};

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
    await skapi
      .registerSubdomain({
        service: service.value.service,
        subdomain: service.value.subdomain,
        exec: "remove",
      })
      .then(() => {
        if (service.value.subdomain.includes("*")) {
          deleting.value = true;
        } else {
          deleting.value = false;
        }
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
  }
};

if (!service.value.hasOwnProperty("storage")) {
  skapi.storageInformation(service.value.service).then((storage) => {
    service.value.storage = storage.cloud + storage.database + storage.email;
  });
}

watch(
  () => isEdit.value,
  async () => {
    await nextTick();
    if (isEdit.value) {
      settingWindow.value.open();
    }
  }
);

watch(
  () => isCreate.value,
  async () => {
    await nextTick();
    if (isCreate.value) {
      subdomainWindow.value.open();
    }
  }
);

watch(
  () => isUpload.value,
  async () => {
    await nextTick();
    if (isUpload.value) {
      uploadWindow.value.open();
    }
  }
);

watch(
  () => service.value.subdomain,
  () => {
    // if (service.value.subdomain) {
    //     domain.value = true;

    //     if (service.value.subdomain.includes('*')) {
    //         domain.value = false;
    //         deleting.value = true;
    //     } else {
    //         deleting.value = false;
    //     }
    // } else {
    //     domain.value = false;
    // }

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
  }
);

onBeforeMount(() => {
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
  // if (service.value.subdomain) {
  //     domain.value = true;

  //     if (service.value.subdomain.includes('*')) {
  //         domain.value = false;
  //         deleting.value = true;
  //     } else {
  //         deleting.value = false;
  //     }
  // } else {
  //     domain.value = false;
  // }
});
// console.log(service.value.subdomain, service.value.service)
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

      .actionWrapper {
        display: flex;
        flex-wrap: nowrap;

        .actions {
          &:first-child {
            margin-right: 20px;
          }
          &:last-child {
            opacity: 0.5;

            &.active {
              opacity: 1;
            }
          }
        }
      }
    }
  }

  h2,
  p {
    color: rgba(255, 255, 255, 0.85);
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
    color: rgba(255, 255, 255, 0.85);
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
.emailGrid {
    &Item {
        &:last-child {
            margin-bottom: 0;
        }
        margin-bottom: 20px;
    }
    .actions {
        position: absolute;
        right: 15px;
        top: 50%;
        transform: translate(0, -50%);
    }
}

.domainGrid, .emailGrid {
  &.deleting {
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

  & > div > * {
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
  }

  & > * {
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
  max-height: 400px;
  overflow: scroll;
}

.filesContainer {
  .fileWrapper {
    border-radius: 8px;

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

      a {
        color: #fff;
        text-decoration: none;
      }

      .path {
        unicode-bidi: plaintext;
        cursor: pointer;
      }
    }

    & > *:not(.pathWrapper) {
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

    & > sui-input,
    & > svg {
      margin-right: 16px;
    }

    & > sui-input {
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

        & ~ .circle {
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

          & ~ .circle {
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

    & > sui-input {
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

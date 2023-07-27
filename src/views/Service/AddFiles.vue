<template lang="pug">
.overlayContainer(:loading="isDisabled || null")
    .uploadBtn
        sui-button.lineButton(@click="addFileButtonHandler" :disabled="isSaving")  + Files
        sui-button.lineButton(@click="addFolderButtonHandler" :disabled="isSaving")  + Folders
        .delete(@click="deleteFiles" :disabled="!isSaving || !selectedFiles?.length")
            Icon trash
        input(ref="folderUpload" type="file" webkitdirectory multiple hidden @change="e => addFolders(e)")
        input(ref="fileUpload" type="file" multiple hidden @change="e => addFiles(e)")
    .fileUploadArea(@dragenter.stop.prevent="" @dragover.stop.prevent="" @drop.stop.prevent="e=>onDrop(e, recordIndex)")
        input(hidden type="file" @change="e=>addFiles(e, recordIndex, index)" multiple :disabled="isSaving")
        div
            Icon attached
            span(style="margin-right: 6px;") Drag and Drop Folders or Files here
        template(v-if="service.files")
            label(v-for="(url, name) in service?.files" style="border-radius: 12px; margin-top: 8px; padding: 8px 12px; background: #656565")
                sui-input(type="checkbox")
                    a(:href="url" download) {{ name }}{{ name }}
        template(v-else)
            div.noFiles
                div.title No Files
                p You have not uploaded any files
        br
        br
    .filesContainer
        template(v-if="Object.keys(fileList).length")
            label.file(v-for="(file, path) in fileList")
                sui-input(v-if="!isSaving && !isComplete" type="checkbox" :value="path" :disabled="isSaving" :checked="selectedFiles.includes(path)" @change="selectionHandler")
                .progressBar(v-else @click="()=>abortUpload=path" :class="{'started': file.progress !== 100 && file.currentProgress > 0 && file.currentProgress < 100, 'complete': file.currentProgress === 100, 'failed': file.progress === false}" :style="{'--progress': 'conic-gradient(#5AD858 ' + (file.currentProgress ? file.currentProgress * 3.6 : 0) + 'deg, rgba(255,255,255,.1) 0deg)'}")
                    .circle
                    .circle
                .pathWrapper
                    span.path {{ path }}
        template(v-else)
            div.noFiles
                div.title No Files
                p You have not uploaded any files
    .uploadBtn
        sui-button.textButton(type="button" style="margin-right: 16px;" @click="emit('close', '')") Cancel
        sui-button(:disabled="!Object.keys(fileList).length" @click="uploadFiles") Upload
</template>
<!-- script below -->
<script setup>
import { inject, ref } from "vue";
import { state, skapi } from "@/main";
import { useRouter } from "vue-router";

import LoadingCircle from "@/components/LoadingCircle.vue";
import Icon from "@/components/Icon.vue";

const emit = defineEmits(["close"]);

let router = useRouter();
let service = inject("service");
const errorMessage = ref("");

let fileList = ref({});
const fileUpload = ref(null);
const folderUpload = ref(null);
const selectedFiles = ref([]);
const filesToUpload = ref(0);
const isSaving = ref(false);
const isComplete = ref(false);
const numberOfFailedUploads = ref(-1);

const props = defineProps(["currentDirectory"]);

function extractFileName(file) {
  const regex = /\/([^/]+)\/?$|([^/]+)$/;
  const match = file.match(regex);

  if (match && match.length > 0) {
    return match[0].replace("/", "");
  }

  return null;
}

const addFileButtonHandler = () => {
  const parent = fileUpload.value.click();
};

const addFolderButtonHandler = () => {
  const parent = folderUpload.value.click();
};

const deleteFiles = () => {
  for (let file of selectedFiles.value) {
    delete fileList.value[file];
  }

  selectedFiles.value = [];
};

const selectionHandler = (e) => {
  if (e.target.checked) {
    selectedFiles.value.push(e.target.value);
  } else {
    selectedFiles.value.splice(selectedFiles.value.indexOf(e.target.value), 1);
  }
};

let abortUpload = "";

const addFolders = (event) => {
  if (isComplete.value) {
    fileList.value = {};
    isComplete.value = false;
  }
  const files = event.target.files;

  for (let file of files) {
    let folderName = props.currentDirectory.slice(1) + file.webkitRelativePath;
    fileList.value[folderName] = {
      file,
      progress: 0,
    };
    filesToUpload.value++;
  }

  folderUpload.value.value = "";
};

const addFiles = (event) => {
  if (isComplete.value) {
    fileList.value = {};
    filesToUpload.value = 0;
    isComplete.value = false;
  }

  const files = event.target.files;

  for (let file of files) {
    let fileName = props.currentDirectory.slice(1) + file.name;

    fileList.value[fileName] = {
      file,
      progress: 0,
    };
    filesToUpload.value++;
  }

  fileUpload.value.value = "";
};

const uploadFiles = async () => {
  if (filesToUpload.value <= 0) return false;

  function saveToServiceFiles(file) {
    // function binarySearch(fileObj) {
    //     let low = 0;
    //     directory.list.forEach((file) => {
    //         if(file.type === 'folder' && fileObj.type === 'file') low++;
    //     });
    //     let high = directory.list.length - 1;
    //     let mid = Math.floor((low + high) / 2);
    //     let name = directory.list[mid]?.name;
    //     if(!name) {
    //         return 0;
    //     }

    //     while (low <= high) {
    //         mid = Math.floor((low + high) / 2);
    //         name = directory.list[mid].name;

    //         if (name < fileObj.name && name[name.length - 1] !== '/') {
    //             low = mid + 1;
    //         } else if (name > fileObj.name && name[name.length - 1] !== '/') {
    //             high = mid - 1;
    //         } else {
    //             return mid;
    //         }
    //     }

    //     if (fileObj.name > name) {
    //         return mid + 1;
    //     } else if (fileObj.name < name) {
    //         return mid;
    //     }
    // }

    let fileName = extractFileName(file.name);
    let fileObj = {
      name: fileName,
      type: "file",
      file: file,
    };

    let fileDirectoryTree = fileObj.file.name.split("/");
    fileDirectoryTree.pop();
    fileDirectoryTree = "/" + fileDirectoryTree.join("/");
    if (props.currentDirectory !== "/") {
      fileDirectoryTree += "/";
    }
    let directory =
      service.value.files[service.value.subdomain + fileDirectoryTree];
    if (!directory) {
      directory = service.value.files[
        service.value.subdomain + fileDirectoryTree
      ] = {
        endOfList: false,
        list: [],
      };

      let folderToCreate = service.value.subdomain + fileDirectoryTree;
      folderToCreate = folderToCreate.replace(
        service.value.subdomain + props.currentDirectory,
        ""
      );
      let folderToCreateArray = folderToCreate.split("/");
      if (folderToCreateArray.length === 1) {
        let folderObj = {
          name: folderToCreateArray[0] + "/",
          type: "folder",
        };
        service.value.files[
          service.value.subdomain + props.currentDirectory
        ].list.push(folderObj); // let index = binarySearch(folderObj); // console.log({index}); // let searchResult = service.value.files[service.value.subdomain + props.currentDirectory].list.find((item) => item.name === folderObj.name) // if (!searchResult) { //     service.value.files[service.value.subdomain + props.currentDirectory].list.splice(index, 0, folderObj); // }
      }
      return false;
    }

    directory.list.push(fileObj); // if (directory.endOfList) { // let index = binarySearch(fileObj); // if (directory.list[index]?.name === fileObj.name) { //     directory.list[index] = fileObj; // } else { //     if(directory.list[index].type === 'folder') { //         index++; //     } //     directory.list.splice(index, 0, fileObj); // } // } else { //     let index = binarySearch(fileObj); //     if(index < directory.list.length) { //         if (directory.list[index]?.name === fileObj.name) { //             directory.list[index] = fileObj; //         } else { //             directory.list.splice(index, 0, fileObj); //         } //     } else if(directory.list[index]?.name === fileObj.name) { //         directory.list[index] = fileObj; //     } // }
  }
  let formData = new FormData();

  for (let key in fileList.value) {
    formData.append(key, fileList.value[key].file, key);
  }
  isSaving.value = true;

  let interval = null;
  let progress = 0;
  try {
    state.blockingPromise = await skapi.uploadFiles(formData, {
      service: service.value.service,
      request: "host",
      progress: (e) => {
        console.log({ e });
        console.log(fileList.value[e.currentFile.name]);
        if (abortUpload === e.currentFile.name && e.progress !== 100) {
          e.abort();
          fileList.value[e.currentFile.name].progress = false;
          abortUpload = "";
        } else {
          if (e.failed.length > numberOfFailedUploads.value) {
            numberOfFailedUploads.value++;
            filesToUpload.value--;
          }
          progress = e.progress;
          fileList.value[e.currentFile.name].abort = e.abort;
          fileList.value[e.currentFile.name].progress = e.progress;
          if (!fileList.value[e.currentFile.name].currentProgress)
            fileList.value[e.currentFile.name].currentProgress = 0;
          if (!interval) {
            interval = setInterval(() => {
              // console.log({
              //   fileName: fileList.value[e.currentFile.name].file.name,
              //   progress: fileList.value[e.currentFile.name].currentProgress,
              // });
              try {
                if (
                  fileList.value[e.currentFile.name].currentProgress < progress
                ) {
                  fileList.value[e.currentFile.name].currentProgress += 1;
                }

                if (
                  fileList.value[e.currentFile.name].currentProgress ===
                  progress
                ) {
                  filesToUpload.value--; // service.value.files[e.currentFile.name] = `https://${service.value.subdomain}.skapi.com/${e.currentFile.name}`
                  saveToServiceFiles(e.currentFile);
                  clearInterval(interval);
                  interval = null;
                }
              } catch (e) {
                clearInterval(interval);
                throw e;
              }
            }, 5);
          }
        }
      },
    }).then(() => {
      getMoreDirectory(currentDirectory.value);
    });
  } catch (e) {
    console.log({ e });
  } finally {
    isSaving.value = false;
    isComplete.value = true;
    emit('close', '');
  }
};

const onDrop = (event) => {
  const readEntriesAsync = (item) => {
    let reader = item.createReader();
    reader.readEntries((contents) => {
      for (let content of contents) {
        if (content.isDirectory) {
          readEntriesAsync(content);
        } else {
          getFileAsync(content, content.fullPath);
        }
      }
    });
  };

  const getFileAsync = (item, path) => {
    console.log(item, path);

    item.file((file) => {
      if (path) {
        fileList.value[path?.substring(1)] = {
          file,
          progress: 0,
        };
      } else {
        // fileList[f.name] = file;
        fileList.value[file.name] = {
          file,
          progress: 0,
        };
      }
      filesToUpload.value++;
    });
  };

  let items = event.dataTransfer.items;
  event.preventDefault();

  for (let item of items) {
    let content = item.webkitGetAsEntry();
    if (content.isDirectory) {
      readEntriesAsync(content);
    } else {
      getFileAsync(content);
    }
  }
};
</script>

<style lang="less" scoped>
.overlayContainer {
  display: flex;
  flex-wrap: wrap;
  background-color: #505050;
  border: 1px solid #808080;
  box-shadow: 4px 4px 12px rgba(0, 0, 0, 0.25);
  color: #fff;
  text-align: center;
  padding: 40px;
  //   width: 500px;
  //   max-width: 100%;
  max-width: 800px;
  border-radius: 8px;
  .input {
    margin: 20px auto;

    label {
      display: block;
      text-align: left;
      font-weight: bold;
      color: rgba(255, 255, 255, 0.6);
      margin-bottom: 8px;
    }

    sui-input {
      width: 100%;
    }
  }
}

.uploadBtn {
  width: 100%;
  text-align: right;

  sui-button {
    vertical-align: middle;

    &:not(:last-child) {
      margin-right: 12px;
    }

    &.withIcon {
      padding: 8px 8px;
    }
  }
  .delete {
    display: inline-block;
  }
}

.fileUploadArea {
  width: 49%;
  margin-right: 2%;
  padding: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px dashed #ffffff;
  border-radius: 8px;

  sui-button {
    vertical-align: middle;
  }

  &:only-child {
    margin-bottom: 0;
  }

  & > div > * {
    display: inline-block;

    &:nth-child(2) {
      margin-top: 20px;
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

  label {
    display: none;
  }
}

.filesContainer {
  width: 49%;
  max-height: 300px;
  overflow-y: auto;
  margin: 28px 0px;

  .file {
    display: flex;
    align-items: center;
    height: 52px;
    padding: 8px 20px;

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
      }
    }

    .progressBar {
      flex-shrink: 0;
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
    padding: 90px 16px;
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

.pageAction {
  position: fixed;
  bottom: 76px;
  right: 16px;
  overflow: hidden;

  & > sui-button {
    z-index: 2;
  }

  &,
  & > div {
    display: flex;
    flex-direction: column-reverse;
    align-items: center;
    gap: 12px;
    width: 48px;
  }

  .v-enter-active,
  .v-leave-active {
    transition: opacity 0.1s ease, transform 0.1s ease;
    opacity: 1;
  }

  .v-enter-from,
  .v-leave-to {
    opacity: 0;
    transform: translateY(100px);
  }
}

@keyframes bounce {
  0% {
    transform: scale(1);
  }

  50% {
    transform: scale(1.1);
  }

  100% {
    transform: scale(1);
  }
}

@keyframes smallRipple {
  0% {
    opacity: 0.5;
  }

  100% {
    opacity: 0;
    transform: scale(1.5);
  }
}

@keyframes ripple {
  0% {
    opacity: 0.5;
  }

  100% {
    opacity: 0;
    transform: scale(1.7);
  }
}
</style>

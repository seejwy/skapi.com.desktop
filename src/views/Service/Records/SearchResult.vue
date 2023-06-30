<template lang="pug">
.pageHeader.headSpaceHelper
    h1 Record
    p Lorem ipsum dolor sit amet, consectetur adipiscing elit. Suspendisse porta sed metus eget auctor. Nulla quis nulla a lorem consequat gravida viverra ac nisi. Donec rutrum mauris orci. Sed a velit sed magna aliquet gravida rutrum et magna.
    .action
        a(href="https://docs.skapi.com/database" target="_blank")
            sui-button.lineButton(type="button") Read Doc

// search form
RecordSearch#recordSearch
div(style="clear:both;")

sui-overlay(ref='openRecord' @click="close" style="background-color:rgba(0 0 0 / 60%)")
    .viewRecordOverlay
        ViewRecord(v-if="recordToOpen" :record='recordToOpen' ref='viewRecord' @close="()=>openRecord.close(() => { recordToOpen = null })")

.recordContainer#dataContainer
    .header
        span.notClickable(v-html="searchTitle")
        Icon.notClickable.animationRotation(style='opacity:0.6;' v-if="fetchingData") refresh
        .clickable(v-else @click="()=>{ searchResult=null; currentSelectedRecordPage=0; currentSelectedRecordBatch=0; router.push({name:'records'})}")
            span(style="vertical-align:middle;") Clear
            Icon X2

    .searchPoints(v-if="route.query?.access_group")
        span(v-if='route.query?.access_group') Access Group: {{ route.query.access_group === '0' ? 'Public' : route.query.access_group === '1' ? 'Registered' : route.query.access_group }}
        span(v-if='route.query?.subscription') Subscription: {{ route.query.subscription === 'null' ? 'None' : route.query.access_group === 'true' ? 'Subscribed' : 'Public' }}
        span(v-if='route.query.search_type === "table" && route.query?.reference') Reference: {{ route.query.reference }}
        span(v-if="route.query?.index_name") Index Name: {{ route.query.index_name }}
        span(v-if='route.query?.index_name') Index Value: [ {{ route.query.index_type }} ] {{ route.query.index_condition }} {{ route.query.index_value }}
        span(v-if="route.query?.tag") Tag: {{ route.query.tag }}

    template(v-if='searchResult !== null')
        div(v-if='!searchResult.list.length')
            .noRecordsFound
                .title No Records Found
                p There was no record matching the query:
                .query.showOnTablet(v-if='route.query?.access_group')
                    span Access Group: {{ route.query.access_group === '0' ? 'Public' : route.query.access_group === '1' ? 'Registered' : route.query.access_group }}
                    template(v-if='route.query.search_type === "user" && route.query?.table')
                        br
                        span Table: {{ route.query.table }}
                    template(v-if='route.query?.subscription')
                        br
                        span Subscription: {{ route.query.subscription === 'null' ? 'None' : route.query.access_group === 'true' ? 'Subscribed' : 'Public' }}
                    template(v-if='route.query.search_type === "table" && route.query?.reference')
                        br
                        span Reference: {{ route.query.reference }}
                    template(v-if='route.query?.index_name')
                        br
                        span Index: {{ route.query.index_type === 'string' ? '"' + route.query.index_value + '"' : route.query.index_value }} {{ route.query.index_condition }} [{{ route.query.index_name }}]
                    template(v-if="route.query?.tag")
                        br
                        span Tag: {{ route.query.tag }}

        template(v-else-if="groupedRecordList && groupedRecordList[currentSelectedRecordBatch]")
            .recordWrapper
                template(v-for="batchIdx in [currentSelectedRecordBatch + 1]")
                    template(v-for="pageIdx in [currentSelectedRecordPage + 1]")
                        // when v-for by number, it starts with 1
                        .records(v-for="r in groupedRecordList[batchIdx - 1][pageIdx - 1]" style="cursor:pointer;" @click="()=>displayRecord(r)" :style="{opacity: r.deleting ? '0.3' : null}")
                            div
                                span.label RECORD:
                                span {{ r.record_id }}
                            div
                                span.label USER:
                                span {{ r.user_id }}
                            div
                                span.label UPLOADED: 
                                span {{ dateFormat(r.uploaded) }}
                                .recordWrapper.animation-skeleton.showOnTablet(v-if='searchResult === null')

            .paginator
                Icon.arrow(
                    :class="{active: currentSelectedRecordBatch || currentSelectedRecordPage}"
                    @click="()=>{ if(currentSelectedRecordPage) currentSelectedRecordPage--; else { currentSelectedRecordPage = numberOfPagePerBatch - 1; currentSelectedRecordBatch--; } }") left
                span.morePage(
                    :class="{active: currentSelectedRecordBatch}"
                    @click="()=>{ if(currentSelectedRecordBatch > 0) {currentSelectedRecordBatch--; currentSelectedRecordPage = numberOfPagePerBatch - 1} }") ...

                span.page(
                    v-for="(i, idx) in groupedRecordList[currentSelectedRecordBatch].length"
                    :class="{active: idx === currentSelectedRecordPage}"
                    @click="()=>currentSelectedRecordPage = idx") {{ currentSelectedRecordBatch * numberOfPagePerBatch + i }}

                span.morePage(
                    :class="{active: !searchResult.endOfList || groupedRecordList.length - 1 > currentSelectedRecordBatch }"
                    @click="fetchMoreRecords") ...

                Icon.arrow(
                    :class="{active: currentSelectedRecordPage < groupedRecordList[currentSelectedRecordBatch].length - 1 || !searchResult.endOfList && currentSelectedRecordPage === groupedRecordList[currentSelectedRecordBatch].length - 1 }"
                    @click="()=>{ if(currentSelectedRecordPage < groupedRecordList[currentSelectedRecordBatch].length - 1 ) currentSelectedRecordPage++; else if(!searchResult.endOfList && currentSelectedRecordPage === groupedRecordList[currentSelectedRecordBatch].length - 1) fetchMoreRecords() }") right

</template>
<!-- script below -->
<script setup>
import { inject, ref, watch, computed, nextTick } from 'vue';
import { state, skapi } from '@/main';
import { dateFormat, groupArray } from '@/helper/common';
import { useRoute, useRouter } from 'vue-router';

import RecordSearch from '@/views/Service/Records/RecordSearch.vue';
import ViewRecord from '@/views/Service/Records/ViewRecord.vue';
import Icon from '@/components/Icon.vue';

let route = useRoute();
let router = useRouter();
let serviceId = route.params.service;
let viewRecord = ref(null);

// record page has darker background in mobile mode
let record = inject('recordToOpen');
record.value = null;

// flag
let fetchingData = inject('fetchingData');

// page title

let pageTitle = inject('pageTitle');
let searchTitle = computed(() => {
    let s = `${fetchingData.value ? "Searching" : "Result of"}${fetchingData.value ? '' : ' ' + route.query.search_type.replace('_', ' ')}: "${route.query[route.query.search_type === 'user' ? 'reference' : route.query.search_type === 'record' ? 'record_id' : route.query.search_type]}"${fetchingData.value ? ' ...' : ''}`;
    let capitalized = s.trim().replace(/^\w/, c => c.toUpperCase());
    pageTitle.value = 'Records';
    return capitalized;
});

// data
let searchResult = inject('searchResult');

// for paginators
let fetchLimit = 50;
let numberOfRecordPerPage = 10;
let numberOfPagePerBatch = fetchLimit / numberOfRecordPerPage;
let currentSelectedRecordPage = ref(0);
let currentSelectedRecordBatch = ref(0);

let groupedRecordList = computed(() => {
    if (!searchResult.value || !searchResult.value.list.length) {
        currentSelectedRecordBatch.value = 0;
        return null;
    }
    let recList = groupArray(searchResult.value.list, numberOfRecordPerPage, numberOfPagePerBatch);
    return recList;
});

const close = async() => {
    await state.blockingPromise;
    openRecord.value.close(() => {
        recordToOpen.value = null;
    });
}

watch(currentSelectedRecordPage, n => {
    // close opened table on page change
    for (let t of groupedRecordList.value[currentSelectedRecordBatch.value][n]) {
        t.opened = false;
    }
    nextTick(() => {
        window.document.getElementById('dataContainer').scrollIntoView({ behavior: 'smooth', block: 'center' });
    });
});

function numberOfSkeletons() {
    // calculated by available vertical space
    return parseInt(window.innerHeight / 2 / 48);
}

let openRecord = ref(null);

let promiseQueue = null;
async function fetchMoreRecords() {
    if (!groupedRecordList.value || searchResult.value.endOfList && groupedRecordList.value.length - 1 === currentSelectedRecordBatch.value) {
        return;
    }

    fetchingData.value = true;

    if (groupedRecordList.value.length - 1 > currentSelectedRecordBatch.value) {
        currentSelectedRecordBatch.value += 1;
        currentSelectedRecordPage.value = 0;
        fetchingData.value = false;
        return;
    }

    if (promiseQueue instanceof Promise) {
        return;
    }

    fetchingData.value = true;
    promiseQueue = skapi.getRecords(searchResult.value.params, { fetchMore: true, limit: fetchLimit });

    promiseQueue = await promiseQueue;

    for (let rec of promiseQueue.list) {
        searchResult.value.list.push(rec);
    }

    fetchingData.value = false;
    searchResult.value.endOfList = promiseQueue.endOfList;
    currentSelectedRecordBatch.value += 1;
    currentSelectedRecordPage.value = 0;
    fetchingData.value = false;
    promiseQueue = null;
}

let recordToOpen = inject('recordToOpen');
function displayRecord(r) {
    recordToOpen.value = r;
    openRecord.value.open();
}

</script>

<style lang="less" scoped>
.recordPageHead {
    padding: 50px 0;
}

#recordSearch {
    z-index: 1;
    display: inline-block;
    position: relative;
    max-width: 100%;
}

.recordContainer {
    svg {
        color: white;
        margin-left: 4px;
    }

    background-color: #434343;
    position: relative;
    border: 1px solid rgba(255, 255, 255, 0.2);
    box-shadow: -1px -1px 1px rgba(0, 0, 0, 0.25),
    inset 1px 1px 1px rgba(0, 0, 0, 0.5);
    border-radius: 8px;
    margin: 0;
    margin-top: 20px;
    padding: 24px 20px;

    .searchPoints {
        padding-top: 16px;
        font-weight: bold;

        &>span {
            display: inline-block;
            background: rgba(255, 255, 255, 0.12);
            color: rgba(255, 255, 255, 0.6);
            border-radius: 12px;
            padding: .7em;
            margin: 2px;
        }
    }

    .header {
        padding: 0 16px;
        color: rgba(255, 255, 255, 0.6);
        display: flex;
        align-items: center;
        flex-wrap: wrap;
        justify-content: space-between;
        vertical-align: middle;

        span {
            color: white;
            font-weight: bold;
        }
    }

    .recordWrapper {
        background-color: #333333;
        border-radius: 8px;
        margin-top: 1.5em;

        .records {
            display: flex;
            flex-wrap: wrap;
            justify-content: space-between;
            flex-direction: column;
            font-size: 14px;
            padding: 16px 20px;
            flex-direction: row;

            &:nth-child(odd) {
                background-color: rgba(255, 255, 255, 0.04);
            }

            &:first-child {
                border-radius: 8px 8px 0 0;
            }

            &:last-child {
                border-radius: 0 0 8px 8px;
            }

            &:only-child {
                border-radius: 8px;
            }

            &:hover {
                background: rgba(255, 255, 255, 0.1);
            }

            div {
                &:not(:last-child) {
                    margin-right: .75em;
                }

                span {
                    font-family: Courier;
                    display: inline-block;

                    &.label {
                        color: rgba(255, 255, 255, 0.6);
                    }
                }
            }
        }
    }
        
    .noRecordsFound {
        text-align: center;
        padding: 60px 0 36px 0;
        border-radius: 0 0 8px 8px;
        color: rgba(255, 255, 255, .4);
        align-items: center;
        text-align: center;
        
        .title {
            font-size: 28px;
        }
        
        p {
            margin: 20px 0 0 0;
        }
        
        .query {
            margin-top: 16px;
            text-align: center;
            color: rgba(255, 255, 255, .6);
        }
    }



    .paginator {
        margin-top: 24px;
        text-align: center;
        color: rgba(255 255 255 / 60%);
        user-select: none;

        &>div {
            display: inline-block;
        }

        &>span {
            padding: 4px 8px;
            width: 24px;
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

        .arrow {
            color: rgba(255, 255, 255, .15);

            &.active {
                cursor: pointer;
                color: #fff;
            }
        }
    }
}
</style>
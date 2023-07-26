<template lang="pug">
.pageHeader.headSpaceHelper
    h1 Record
    p.
        Records are data objects created by you or your users and stored within your database.
        skapi's database is designed for flexibility and features automatic indexing.
        Find out how you make use of our advanced features to create a database that fits your needs.

    .action
        a(href="https://docs.skapi.com/database" target="_blank")
            sui-button.lineButton(type="button") Find out More
// search form
RecordSearch#recordSearch

sui-button(type="button" style='float:right;' @click='()=>addRecord()') + Add Record
div(style="clear:both;")

// record view
sui-overlay(ref='openRecord' @mousedown="close" style="background-color:rgba(0 0 0 / 60%)")
    .viewRecordOverlay
        ViewRecord(v-if='recordToOpen && typeof recordToOpen === "object"' ref="viewRecord" :record='recordToOpen' @close="()=>openRecord.close(() => { recordToOpen = null; })")

.tableContainer#dataContainer
    .header.labelHead
        span.notClickable Table name
        div.notClickable
            span Size
            span # of records
        Icon.clickable(:class="{'animationRotation': fetchingData}" @click="()=>{ if(!fetchingData) getTables(); }") refresh

    // table list
    template(v-if='recordTables !== null')
        .noRecords(v-if='!recordTables.list?.length')
            div
                .title No Record Tables
                p List of tables will show when there is data

        template(v-else-if="groupedTableList && groupedTableList[currentSelectedTableBatch] && !fetchingData")
            template(v-for="batchIdx in [currentSelectedTableBatch + 1]")
                template(v-for="pageIdx in [currentSelectedTablePage + 1]")
                    // when v-for by number, it starts with 1
                    template(v-for="t in groupedTableList[batchIdx - 1][pageIdx - 1]")
                        .tableWrapper(v-if="t.number_of_records")
                            .tableHead.labelHead.clickable(@click='()=>{t.opened = !t.opened;}')
                                span {{ t.table }}
                                div
                                    span {{getSize(t.size)}}
                                    span {{t.number_of_records}}

                                template(v-if='t.records')
                                    template
                                        Icon.clickable(v-if="!t.opened") plus
                                        Icon.clickable(v-else) minus

                                Icon.animationRotation(v-else) refresh

                            div(v-if="t.opened && t.records" style="max-height: 60vh;overflow-y: auto;" @scroll.passive="(e)=>getMoreRecords(e, t, serviceId)")
                                .noRecords(v-if='!t.records.list?.length')
                                    div
                                        sui-flextext(min-size='14' max-size='24') No Records
                                        br
                                        p This table will be automatically removed.

                                .records(v-else v-for="r in t.records.list" style="cursor:pointer;" @click="()=>displayRecord(r)" :class="{'deleting': r.deleting ? true : null}" :loading="r.deleting || null")
                                    div
                                        span.labelHead RECORD:
                                        span {{ r.record_id }}
                                    div
                                        span.labelHead USER:
                                        span {{ r.user_id }}
                                    div
                                        span.labelHead UPLOADED:
                                        span {{ dateFormat(r.uploaded) }}

                                .loadMore(v-if="!t.records.endOfList")
                                    Icon.animationRotation refresh
                .paginator
                    Icon.arrow(
                        :class="{active: currentSelectedTableBatch || currentSelectedTablePage}"
                        @click="()=>{ if(currentSelectedTablePage) currentSelectedTablePage--; else if(currentSelectedTableBatch) { currentSelectedTablePage = numberOfPagePerBatch - 1; currentSelectedTableBatch--; } }") left
                    span.morePage(
                        :class="{active: currentSelectedTableBatch}"
                        @click="()=>{ if(currentSelectedTableBatch > 0) {currentSelectedTableBatch--; currentSelectedTablePage = numberOfPagePerBatch - 1} }") ...

                    span.page(
                        v-for="(i, idx) in groupedTableList[currentSelectedTableBatch].length"
                        :class="{active: idx === currentSelectedTablePage}"
                        @click="()=>currentSelectedTablePage = idx") {{ currentSelectedTableBatch * numberOfPagePerBatch + i }}

                    span.morePage(
                        :class="{active: !isEndOfList || !isLastBatch }"
                        @click="getMoreTables") ...
                    Icon.arrow(
                        :class="{active: !isEndOfList || !isLastBatch || !isLastPage }"
                        @click="() => { if(!isLastPage) currentSelectedTablePage++; else if(isLastPage && !isLastBatch) { currentSelectedTableBatch++; currentSelectedTablePage = 0;} else if(isLastBatch && !isEndOfList) getMoreTables() }"
                        ) right

</template>
<!-- script below -->
<script setup>
import { inject, ref, watch, computed, nextTick, onMounted, onBeforeUnmount } from 'vue';
import { state, skapi } from '@/main';
import { getSize, dateFormat, groupArray } from '@/helper/common.js';
import { useRoute, useRouter } from 'vue-router';
import { tableList, recordTables, refreshTables, getMoreRecords } from '@/helper/records.js';
import RecordSearch from '@/views/Service/Records/RecordSearch.vue';
import ViewRecord from '@/views/Service/Records/ViewRecord.vue';
import Icon from '@/components/Icon.vue';

let openRecord = ref(null);
let recordToOpen = inject('recordToOpen');
const viewRecord = ref(null);
let route = useRoute();
let router = useRouter();
let serviceId = route.params.service;
const service = inject('service');

// flag
let fetchingData = inject('fetchingData');

// for paginators
let fetchLimit = 50;
let numberOfTablePerPage = 10;
let numberOfPagePerBatch = fetchLimit / numberOfTablePerPage;

let currentSelectedTableBatch = ref(0);
let currentSelectedTablePage = ref(0);

const isLastBatch = computed(() => {
    return currentSelectedTableBatch.value === groupedTableList?.value?.length - 1;
});

const isLastPage = computed(() => {
    return currentSelectedTablePage.value === groupedTableList?.value?.[currentSelectedTableBatch.value]?.length - 1;
});

const isEndOfList = computed(() => {
    return recordTables.value?.endOfList;
});

let groupedTableList = computed(() => {
    if (!recordTables.value || !recordTables.value.list.length) {
        currentSelectedTableBatch.value = 0;
        return null;
    }

    return groupArray(recordTables.value.list, numberOfTablePerPage, numberOfPagePerBatch);
});

if (groupedTableList.value) {
    for (let t of groupedTableList.value[0][0]) {
        // close all opened
        t.opened = false;
    }
}

onMounted(() => {
    if (recordToOpen.value) {
        displayRecord(recordToOpen.value);
    }
});

function addRecord(mobile = false) {
    recordToOpen.value = {};
    if (mobile) {
        router.push({
            name: 'mobileRecordView',
            query: {
                id: 'Add Record'
            }
        });
    }
    else {
        nextTick(() => {
            viewRecord.value.editRecord();
            openRecord.value.open();
        });
    }
}

async function displayRecord(r) {
    if (typeof r === 'string') {
        let rec = await skapi.getRecords({
            service: serviceId,
            record_id: r
        });
        recordToOpen.value = rec.list[0];
        openRecord.value.open();
    }
    else {
        if (r.record_id) {
            recordToOpen.value = r;
            openRecord.value.open();
        }
    }
}

let getMoreTablesQueue = null;
async function getMoreTables() {
    if (recordTables.value.endOfList && groupedTableList.value.length - 1 === currentSelectedTableBatch.value) {
        return;
    }

    fetchingData.value = true;

    if (groupedTableList.value.length - 1 > currentSelectedTableBatch.value) {
        currentSelectedTableBatch.value += 1;
        currentSelectedTablePage.value = 0;
        fetchingData.value = false;
        return;
    }

    if (getMoreTablesQueue instanceof Promise) {
        return;
    }

    getMoreTablesQueue = skapi.getTables({ service: serviceId }, { fetchMore: true, limit: fetchLimit }).catch(err => {
        fetchingData.value = false;
        throw err;
    });

    let t = await getMoreTablesQueue;
    recordTables.value.endOfList = t.endOfList;

    t.list.map(m => {
        if (!tableList.includes(m.table)) {
            tableList.push(m.table);
        }

        m.opened = false;
        m.records = ref(null);

        skapi.getRecords({
            service: serviceId,
            table: { name: m.table }
        }, { limit: fetchLimit }).then(r => {
            m.records.value = r;
        });

        recordTables.value.list.push(m);
    });

    currentSelectedTableBatch.value += 1;
    currentSelectedTablePage.value = 0;

    fetchingData.value = false;
    getMoreTablesQueue = null;
}

function getTables() {
    // initial table fetch
    currentSelectedTablePage.value = 0;
    currentSelectedTableBatch.value = 0;

    fetchingData.value = true;
    refreshTables(serviceId).then(() => {
        fetchingData.value = false;
    });
}

// get tables on created (if not already fetched)
// console.log(recordTables.value)
if (!recordTables.value) {
    // console.log("Tables empty")
    getTables();
}

const close = async () => {
    await state.blockingPromise;
    viewRecord.value.close();
}

watch(currentSelectedTablePage, n => {
    // close opened table on page change
    for (let t of groupedTableList.value[currentSelectedTableBatch.value][n]) {
        t.opened = false;
    }
    nextTick(() => {
        window.document.getElementById('dataContainer').scrollIntoView({ behavior: 'smooth', block: 'center' });
    });
});

watch(currentSelectedTableBatch, n => {
    // close opened table on batch change
    for (let t of groupedTableList.value[n][currentSelectedTablePage.value]) {
        t.opened = false;
    }
    nextTick(() => {
        window.document.getElementById('dataContainer').scrollIntoView({ behavior: 'smooth', block: 'center' });
    });
});

document.body.classList.add('table');

onBeforeUnmount(() => {
    document.body.classList.remove('table');
});
</script>

<style lang="less" scoped>
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

#recordSearch {
    z-index: 1;
    display: inline-block;
    position: relative;
    max-width: 100%;
}

.tableContainer {
    background-color: #434343;
    position: relative;
    border: 1px solid rgba(255, 255, 255, 0.2);
    box-shadow: -1px -1px 1px rgba(0, 0, 0, 0.25),
        inset 1px 1px 1px rgba(0, 0, 0, 0.5);
    border-radius: 8px;
    margin: 24px 0 0 0;
    padding: 24px 20px;

    svg {
        color: white;
    }

    .header {
        padding: 0 16px 0 16px;
        color: rgba(255, 255, 255, 0.4);
        display: flex;
        align-items: center;
        flex-wrap: wrap;

        &+.tableWrapper {
            margin-top: 24px;
        }
    }

    .tableWrapper {
        background-color: #333333;
        border-radius: 8px;

        .records {
            display: flex;
            flex-wrap: wrap;
            flex-direction: column;
            justify-content: space-between;
            font-size: 14px;
            padding: 16px 20px;
            flex-direction: row;

            &:nth-child(odd) {
                background-color: rgba(255, 255, 255, 0.04);
            }

            &:last-child {
                border-radius: 0 0 8px 8px;
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

                    &.labelHead {
                        color: rgba(255, 255, 255, 0.6);
                    }
                }
            }

            &.deleting {
                opacity: 0.3;
            }
        }

        .loadMore {
            text-align: center;
            padding: 8px;
        }
    }


    .labelHead {
        &>span {
            display: inline-block;
            text-align: left;
            max-width: 50%;
            flex-grow: 1;
            padding-right: 1em;
            display: inline-block;
            text-overflow: ellipsis;
            overflow: hidden;
        }

        &>div {
            position: relative;
            min-width: calc(50% - 24px);

            &>span {
                display: inline-block;
                width: 50%;
                white-space: nowrap;
                padding-right: 1em;
            }
        }
    }

    .tableHead {
        height: 48px;
        background: #656565;
        border-radius: 4px;
        padding: 12px 16px;
        display: flex;
        align-items: center;
        flex-wrap: wrap;
        margin-top: 16px;
    }

    .noRecords {
        padding: 16px 0;
        text-align: center;
        display: flex;
        justify-content: center;
        align-items: center;
        opacity: .6;
        flex-wrap: wrap;

        p {
            font-size: 14px;
            margin: 0;
        }
    }

    .noRecords {
        color: rgba(255, 255, 255, 0.4);
        padding: 60px 0 60px 0;
        margin: 0 -20px -24px -20px;
        border-radius: 0 0 8px 8px;
        text-align: center;
        display: flex;
        justify-content: center;
        align-items: center;
        flex-wrap: wrap;

        .title {
            font-size: 28px;
        }

        p {
            margin: 20px 0 0 0;
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
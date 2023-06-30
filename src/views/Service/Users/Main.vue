<template lang="pug">
router-view
</template>

<script setup>
import { provide, ref, inject } from 'vue';
provide('searchResult', ref(null));
let pageTitle = inject('pageTitle');
let service = inject('service');
pageTitle.value = `Service "${service.value.name}"`;
</script>
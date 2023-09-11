<script setup>

defineProps(['steps'])

</script>

<template>
    <p v-for="step in steps"> {{ step }}</p>
</template>
<script setup lang="ts">
import Column from 'primevue/column';

const props = defineProps({
    fieldName: String,
    headerAbbv: String,
    headerTooltip: String,
    perGame: Boolean
});
</script>

<template>
    <Column :field="props.fieldName" sortable>
        <template #header>
            <div v-tooltip.top="props.headerTooltip">{{ props.headerAbbv }}</div></template
        >
        <template #body="{ data }"> {{ data[props.fieldName].toFixed(props.perGame ? 1 : 0) }} </template></Column
    >
</template>

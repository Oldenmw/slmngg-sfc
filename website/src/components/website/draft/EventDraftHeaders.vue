<template>
    <tr>
        <th>Name</th>
        <th class="text-center">SR</th>
        <th class="text-center">Role</th>
        <th class="text-center" v-if="hasDraftData && settings.heroes">Best heroes</th>
        <th v-if="settings.slmn_events">SLMN Events</th>
        <th v-if="hasDraftData && settings.info_for_captains">Info for captains</th>
        <th v-if="settings.custom_notes">Player notes</th>
        <th v-if="!hasDraftData">Player information</th>
        <th class="text-center">Controls</th>
    </tr>
</template>

<script>
export default {
    name: "EventDraftHeaders",
    props: ["hasDraftData", "settings"]
};
</script>

<style scoped>
    th {
        white-space: nowrap;
    }
</style>

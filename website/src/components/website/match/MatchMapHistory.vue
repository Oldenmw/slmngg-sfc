<template>
    <tr class="match-map-history" v-bind:class="{ 'scheduled': isScheduled }">
        <td class="map-name" style="width: 200px">
            {{ map.name }}
            <i class="fa fa-fw fa-check" v-if="isScheduled"></i>
        </td>
        <TeamMapStats class="map-team-stats" v-for="(data, ti) in data.stats"
                      v-bind:key="ti" :data="data"/>
    </tr>
</template>

<script>
import TeamMapStats from "@/components/website/match/TeamMapStats";

export default {
    name: "MatchMapHistory",
    components: { TeamMapStats },
    props: ["data", "map"],
    computed: {
        isScheduled() {
            return this.data?.meta?.scheduledForMatch;
        }
    }
};
</script>

<style scoped>
    .team-map-stats:last-of-type >>> div {
        flex-direction: row-reverse;
    }
    .match-map-history.scheduled .map-name {
        font-weight: bold;
        text-shadow: 0 0 4px rgba(255,255,255,0.34);
    }
</style>

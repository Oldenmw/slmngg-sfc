<template>
    <div class="standings-team d-flex">
        <div class="team-rank flex-shrink-0">{{ tieText && !teamStats["tie_show_number"] ? tieText : teamStats["rank"] }}</div>
        <ThemeLogo class="team-logo flex-shrink-0" :theme="team.theme" icon-padding="0.2em" border-width="0.15em" logo-size="30" />
        <router-link :to="url('team', team)" class="team-name ct-passive flex-grow-1 text-left d-none d-md-flex">{{ team.name }}</router-link>
        <router-link :to="url('team', team)" class="team-name team-code ct-passive flex-grow-1 text-left d-md-none">{{ team.code }}</router-link>
        <div class="team-stats d-flex">
            <div class="team-stat text-center" v-for="stat in stats" v-bind:key="stat" v-bind:class="{'d-none d-md-block': ['omw'].includes(stat) }">
                {{ teamStats[stat] }}
            </div>
        </div>
    </div>
</template>

<script>
import ThemeLogo from "@/components/website/ThemeLogo";
import { url } from "@/utils/content-utils";

function diffString(val) {
    if (val === 0) return "±0"; // TODO: update to special char
    if (val > 0) return `+${val}`;
    return val;
}

export default {
    name: "StandingsTeam",
    components: { ThemeLogo },
    props: ["team", "tieText"],
    data: () => ({
        // stats: ["diff", "map_diff"/*, "points" */]

    }),
    computed: {
        stats() {
            return [
                "matches",
                "maps",
                "map_diff",
                ...(this.team.standings?.omw !== undefined ? ["omw"] : [])
            /*, "winrate" */
            ];
        },
        teamStats() {
            return {
                matches: `${this.team.standings.wins}-${this.team.standings.losses}`,
                diff: diffString(this.team.standings.wins - this.team.standings.losses),
                maps: `${this.team.standings.map_wins}-${this.team.standings.map_losses}`,
                map_diff: diffString(this.team.standings.map_wins - this.team.standings.map_losses),
                // points: this.team.standings.points,
                rank: this.team.standings.rank,
                tie_show_number: this.team.standings.tie_show_number,
                winrate: this.team.standings.winrate,
                omw: this.team.standings?.omw !== undefined ? Math.floor(this.team.standings.omw * 100) + "%" : "-"
            };
        }
    },
    methods: {
        url
    }
};
</script>

<style scoped>
    .team-stat {
        width: 4em;
    }
    .standings-team {
        align-items: flex-start;
        padding: 8px 0px;
    }
    .team-name, .team-rank, .team-stat {
        line-height: 1;
        transform: translate(0, -0.0925em);
    }
    .team-logo {
        width: 1.25em;
        height: 1.1em;
        border-width: 2px;
        margin-right: .33em;
    }
    .team-logo>.icon {
        width: calc(100% - 4px);
        height: calc(100% - 4px);
    }
    .team-rank {
        width: 2em;
        text-align: center;
    }
</style>

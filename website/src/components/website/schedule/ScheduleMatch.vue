<template>
    <div class="match my-2" v-if="loaded" v-bind:class="{ 'bg-danger' : !loaded, 'special-event': match.special_event }">
        <div class="match-left match-details flex-center flex-column text-center">
            <div class="match-detail" v-for="detail in details" v-bind:key="detail">{{ detail }}</div>
        </div>

        <router-link :to="url('match', this.match)" v-if="match.special_event"
                     class="match-special-event-name flex-center text-center ct-passive link-text">
            {{ match.custom_name }}
        </router-link>

        <div v-for="(team, i) in teams"
             v-bind:key="team.id" :style="{order: i*2}"
             class="match-team flex-grow-1 d-flex align-items-center justify-content-end"
             v-bind:class="{'right': i === 1}">

            <div v-if="team.dummy" class="team-name team-name--spacer d-none d-lg-flex">{{ team.text }}</div>
            <router-link v-else-if="!team.dummy" :to="url('team', team)" class="team-name d-none d-lg-flex ct-passive">{{ team.name }}</router-link>


            <div v-if="team.dummy" class="team-code d-lg-none">{{ team.code || team.text || 'TBD' }}</div>
            <router-link v-else-if="!team.dummy" :to="url('team', team)" class="team-code d-lg-none ct-passive">{{ team.code }}</router-link>


            <ThemeLogo v-if="team && !team.dummy" :theme="team.theme" border-width="4" class="team-logo" icon-padding="4"/>
            <div class="team-logo team-logo--spacer" v-else></div>
        </div>

        <router-link :to="url('match', this.match)" class="match-center match-vs flex-center text-center ct-passive" v-if="!match.special_event">
            <div class="scores-wrap" v-if="scores.some(s => s)">
                <div class="scores">{{ match.score_1 }} - {{ match.score_2 }}</div>
                <div class="scores-forfeit" v-if="match.forfeit">Forfeit</div>
            </div>
            <div class="vs ct-passive" v-else>vs</div>
        </router-link>
        <div class="match-right match-time flex-center">
            <ScheduleTime :time="match.start" :custom-text="timeCustomText"/>
        </div>
    </div>
</template>

<script>
import ThemeLogo from "@/components/website/ThemeLogo";
import { url } from "@/utils/content-utils";
import ScheduleTime from "@/components/website/schedule/ScheduleTime";

export default {
    name: "ScheduleMatch",
    components: { ScheduleTime, ThemeLogo },
    methods: { url },
    props: ["match", "customText"],
    computed: {
        loaded() {
            if (this.match?.__loading) return false;
            return !!this.match && !!this.match.name;
        },
        scores() {
            if (!this.match) return [null, null];
            if (!this.match.teams || this.match.teams.length !== 2) return [null, null];
            return [this.match.score_1, this.match.score_2];
        },
        teams() {
            if (this.match?.special_event) return [];
            const dummy = { text: "TBD", dummy: true, id: null };
            if (!this.match) return [{ ...dummy, _empty: true }, { ...dummy, _empty: true }];

            let text = (this.match.placeholder_teams || "").trim().split("|").filter(t => t !== "");
            let extraText = null;

            if (text.length === 4) {
                extraText = [text[2], text[3]];
                text = [text[0], text[1]];
            }

            if (!this.match.teams || this.match.teams.length === 0) {
                if (text.length === 2) {
                    return text.map((t, ti) => ({ ...dummy, text: t, ...(extraText ? { code: extraText[ti] } : {}) }));
                } else if (text.length === 1) {
                    if (this.match.placeholder_right) return [dummy, { ...dummy, text: text[0], ...(extraText ? { code: extraText[0] } : {}) }];
                    return [{ ...dummy, text: text[0], ...(extraText ? { code: extraText[0] } : {}) }, dummy];
                } else if (text.length === 0) {
                    // no text, just use TBDs
                    return [dummy, dummy];
                }
            }
            if (this.match.teams.length === 1) {
                if (text.length === 2) {
                    if (this.match.placeholder_right) return [this.match.teams[0], { ...dummy, text: text[1], ...(extraText ? { code: extraText[1] } : {}) }];
                    return [{ ...dummy, text: text[0], ...(extraText ? { code: extraText[0] } : {}) }, this.match.teams[0]];
                } else if (text.length === 1) {
                    if (this.match.placeholder_right) return [this.match.teams[0], { ...dummy, text: text[0], ...(extraText ? { code: extraText[0] } : {}) }];
                    return [{ ...dummy, text: text[0], ...(extraText ? { code: extraText[0] } : {}) }, this.match.teams[0]];
                } else if (text.length === 0) {
                    // no text, just use TBDs
                    if (this.match.placeholder_right) return [this.match.teams[0], dummy];
                    return [dummy, this.match.teams[0]];
                }
            }

            if (this.match.teams.length === 2) return this.match.teams;
            return [];
        },
        details() {
            if (!this.match) return "";
            const details = [];

            if (this.match.match_number) details.push(`M${this.match.match_number}`);
            if (this.match.stream_code) details.push(`${this.match.stream_code} stream`);
            if (this.match.first_to) details.push(`FT${this.match.first_to}`);

            return details.slice(0, 2);
        },
        timeCustomText() {
            if (this.customText) return this.customText;
            if (this.match?.special_event) {
                return null;
            } else {
                return this.match?.custom_name;
            }
        }
    }
};
</script>

<style scoped>
    .team-logo {
        width:  64px;
        height: 48px;
    }

    .match-left { order: -1; }
    .match-center { order: 1; }
    .match-right { order: 3; }

    .match-team {
        text-align: right;
    }
    .match-team.right {
        flex-direction: row-reverse;
        text-align: left;
    }

    .match {
        display: grid;
        grid-template-columns: 0.75fr 2fr 0.25fr 2fr 0.75fr;
        grid-template-rows: 1fr;
        gap: 0px 0px;
        min-height: 3em;
    }
    .match.special-event {
        grid-template-columns: 0.75fr 4.25fr 0.75fr;
    }
    @media (max-width: 957px) {
        .match {
            grid-template-columns: 0.75fr 1fr 0.2fr 1fr 0.75fr;
        }
        .match.special-event {
            grid-template-columns: 0.75fr 2.2fr 0.75fr;
        }
    }
    @media (max-width: 767px) {
        .match {
            grid-template-columns: 0.5fr 1fr 0.2fr 1fr 0.75fr;
        }
        .match.special-event {
            grid-template-columns: 0.5fr 2.2fr 0.5fr;
        }
    }
    @media (max-width: 575px) {
        .match-left.match-details, .match-time {
            display: none;
        }
        .match {
            grid-template-columns: 1fr 0.2fr 1fr;
        }
        .match.special-event {
            grid-template-columns: 1fr;
        }
    }

    .match-special-event-name {
        font-size: 1.2em;
    }

    .match-vs {
        color: white;
        white-space: nowrap;
        margin: 0 12px;
        min-width: 40px;
    }

    .team-name, .team-code {
        padding: 0 .6em;
        font-size: 1.25em;
        color: white;
        line-height: 1;
    }
    .match-time {
        text-align: right;
        line-height: 1.2em;
        justify-content: flex-end;
    }

    .scores-forfeit {
        font-size: 0.7em;
        line-height: 1;
        margin-top: 2px;
    }

    .team-logo--spacer {
        display: none;
    }
</style>

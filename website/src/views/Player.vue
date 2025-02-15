<template>
    <div>
        <div class="container">
            <div class="top">
                <div class="player-top d-flex">
                    <div class="player-icon bg-center default-thing" v-if="player.overwatch_icon"
                         :style="{backgroundImage: `url(${player.overwatch_icon})`}"></div>
                    <div class="player-name">{{ player.name }} <i class="fas fa-badge-check" v-if="player.verified"></i>
                    </div>
                </div>
                <div class="player-socials" v-if="player.socials" :style="{ marginLeft: player.overwatch_icon ? '84px' : '16px'}">
                    <Social class="ct-active" :social="social" v-for="social in player.socials" v-bind:key="social.id"/>
                </div>
            </div>
        </div>
        <SubPageNav>
            <li class="nav-item"><router-link class="nav-link" :to="subLink('')">Overview</router-link></li>
            <li class="nav-item" v-if="player.casts"><router-link class="nav-link" :to="subLink('casts')">Casts</router-link></li>
            <li class="nav-item" v-if="player.news"><router-link class="nav-link" :to="subLink('news')">News</router-link></li>
            <li class="nav-item" v-if="hasMatchPlayerRelationships"><router-link class="nav-link" :to="subLink('matches')">Matches</router-link></li>
            <li class="nav-item" v-if="player.member_of && player.member_of.some(t => t.matches)"><router-link class="nav-link" :to="subLink('played-matches')">Played Matches</router-link></li>
        </SubPageNav>
        <router-view :player="player"></router-view>
    </div>
</template>

<script>
import { ReactiveArray, ReactiveRoot, ReactiveThing } from "@/utils/reactive";
import Social from "@/components/website/Social";
import SubPageNav from "@/components/website/SubPageNav";

export default {
    name: "Player",
    props: ["id"],
    components: {
        Social, SubPageNav
    },
    methods: {
        subLink(page) {
            return `/player/${this.player.id}/${page}`;
        }
    },
    computed: {
        player () {
            return ReactiveRoot(this.id, {
                member_of: ReactiveArray("member_of", {
                    theme: ReactiveThing("theme"),
                    accolades: ReactiveArray("accolades", {
                        event: ReactiveThing("event", {
                            theme: ReactiveThing("theme")
                        })
                    })
                }),
                owned_teams: ReactiveArray("owned_teams", {
                    theme: ReactiveThing("theme")
                }),
                captain_of: ReactiveArray("captain_of", {
                    theme: ReactiveThing("theme")
                }),
                event_staff: ReactiveArray("event_staff", {
                    theme: ReactiveThing("theme")
                }),
                casted_events: ReactiveArray("casted_events", {
                    theme: ReactiveThing("theme")
                }),
                team_staff: ReactiveArray("team_staff", {
                    theme: ReactiveThing("theme")
                }),
                accolades: ReactiveArray("accolades", {
                    event: ReactiveThing("event", {
                        theme: ReactiveThing("theme")
                    })
                }),
                brands_designed: ReactiveArray("brands_designed", {
                    theme: ReactiveThing("theme")
                }),
                socials: ReactiveArray("socials"),
                player_relationships: ReactiveArray("player_relationships", {
                    events: ReactiveArray("events", {
                        theme: ReactiveThing("theme")
                    }),
                    teams: ReactiveArray("teams", {
                        theme: ReactiveThing("theme")
                    })
                    // matches: ReactiveArray("matches")
                })
            });
        },
        hasMatchPlayerRelationships() {
            if (!this.player?.player_relationships) return false;
            return this.player.player_relationships.some(rel => !!rel.matches);
        }
    },
    metaInfo() {
        return {
            title: this.player.name
        };
    }
};

</script>

<style scoped>
.player-top {
    align-items: center;
}

.player-icon {
    width: 64px;
    height: 64px;
    margin-top: 6px;
    flex-shrink: 0;
}

.player-name {
    font-size: 64px;
    margin-left: 12px;
}

.player-socials {
    margin-top: -12px;
    margin-bottom: 12px;
}

@media (max-width: 575px) {
    .player-name {
        font-size: 48px;
    }
}
</style>

<template>
    <div class="detailed-match container">
        <div class="main-content row">
            <div class="center-holder col-9">
                <div class="maps-holder mt-1" v-if="match.maps && showMatchMaps">
                    <MapDisplay v-for="(map, i) in match.maps" :i="i" :map="map" :match="match" :theme="_theme" v-bind:key="map.id"/>
                </div>

                <div class="special-event-notice" v-if="match.special_event">
                    <h2>Special event</h2>
                    This is a special event and doesn't have team information.
                </div>

                <div class="team-holder f-row mb-2">
                    <div class="team f-col w-50 mt-2" v-for="team in match.teams" v-bind:key="team.id">
                        <div :style="theme(team)" class="team-header flex-center f-col default-thing">
                            <div class="team-code">{{ team.code }}</div>
                            <div class="team-overlay-text">{{ team.small_overlay_text }}</div>
                            <div class="team-logo bg-center" :style="icon(team)"></div>
                            <router-link :to="url('team', team, match.event)" class="team-name">{{ team.name }}</router-link>
                        </div>
                        <div class="team-players f-col p-1" v-if="showRosters">
                            <div class="team-player" v-for="player in team.players" v-bind:key="player.id">
                                <div class="player-info player-name flex-center">
                                    <div class="player-role-holder player-icon-holder flex-center" v-if="player.role">
                                        <div class="player-role" v-html="getRoleSVG(player.role)"></div>
                                    </div>
                                    <router-link class="ct-active" :to="url('player', player)">{{ player.name }} </router-link>
                                    <span v-if="showCastingInfo && player.pronouns" class="player-pronouns badge rounded-pill bg-light text-dark" :data-pronoun="player.pronouns">{{ player.pronouns }}</span></div>
                                <div class="player-info player-pronounce" v-if="showCastingInfo"><i class="fas fa-w fa-lips player-icon-holder"></i> {{ player.pronunciation }}</div>
                                <div class="player-info player-dtag" v-if="showPlayerInfo"><i class="fab fa-fw fa-discord player-icon-holder"></i> {{ player.discord_tag }}</div>
                                <div class="player-info player-btag" v-if="showPlayerInfo"><i class="fab fa-fw fa-battle-net player-icon-holder"></i> {{ player.battletag }}</div>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="team-holder f-row mb-2" v-if="showManagers">
                    <div class="team f-col w-50" v-for="team in match.teams" v-bind:key="team.id">
                        <div class="team-players team-managers f-col p-1">
                            <div class="team-player" v-for="player in team.staff" v-bind:key="player.id">
                                <div class="player-info player-name">
                                    <div class="player-role-holder player-icon-holder" v-if="player.staff_role">
                                        <div class="player-role" v-html="getRoleSVG(player.staff_role)"></div>
                                    </div>
                                    <router-link class="ct-active" :to="url('player', player)">{{ player.name }} <i class="fas fa-badge-check fa-fw" title="REAL" v-if="player.verified"></i></router-link>
                                </div>
                                <div class="player-info player-dtag" v-if="showPlayerInfo && player.discord_tag">
                                    <i class="fab fa-fw fa-discord player-icon-holder"></i> {{ player.discord_tag }}
                                </div>
                                <div class="player-info player-btag" v-if="showPlayerInfo && player.battletag">
                                    <i class="fab fa-fw fa-battle-net player-icon-holder"></i> {{ player.battletag }}
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="prev-matches-holder d-flex mt-2" v-if="showMatchHistory">
                    <div class="team-prev-wrapper w-50" v-for="team in match.teams" v-bind:key="team.id">
                        <PreviousMatch v-for="match in team.matches" :match="match" :team="team" v-bind:key="match.id" />
                    </div>
                </div>
                <div class="show-notes mt-3" v-if="showShowNotes && match.show_notes">
                    <h2>Show notes</h2>
                    <Markdown :markdown="match.show_notes"/>
                </div>
            </div>
            <div class="right-holder col-3">
                <ThemeLogo class="top-right-logo mb-3" :theme="_theme" border-width="8"/>
                <div class="info-block">

                    <router-link :to="`/match/${this.match.id}`" class="btn btn-block border-transparent btn-primary text-dark-low"
                    :style="theme(match.event)">
                        <i class="fa-fw fas fa-external-link"></i>
                        See match page
                    </router-link>

                    <div :class="`mt-3 btn btn-block btn-${showRosters ? 'light' : 'secondary'} mb-2`" v-on:click="showRosters = !showRosters">
                        <i class="fa-fw fas fa-users"></i> Rosters
                    </div>
                    <div :class="`mb-3 btn btn-block btn-${showManagers ? 'light' : 'secondary'} mb-2`"
                         v-on:click="showManagers = !showManagers">
                        <i class="fa-fw fas fa-user-tie"></i>
                        Team staff
                    </div>
                    <div :class="`btn btn-block btn-${showCastingInfo ? 'light' : 'secondary'} mb-2`" v-if="showRosters || showManagers"
                         v-on:click="showCastingInfo = !showCastingInfo">
                        <i class="fa-fw fas fa-headset"></i>
                        Casting info
                    </div>
                    <div :class="`mb-3 btn btn-block btn-${showPlayerInfo ? 'light' : 'secondary'} mb-2`" v-if="showRosters || showManagers"
                         v-on:click="showPlayerInfo = !showPlayerInfo">
                        <i class="fa-fw far fa-id-card"></i>
                        Contacts
                    </div>
                    <div v-if="match.maps" :class="`btn btn-block btn-${showMatchMaps ? 'light' : 'secondary'} mb-2`" v-on:click="showMatchMaps = !showMatchMaps">
                        <i class="fa-fw fas fa-map"></i> Match maps
                    </div>
                    <div :class="`btn btn-block btn-${showMatchHistory ? 'light' : 'secondary'} mb-2`"
                         v-on:click="showMatchHistory = !showMatchHistory">
                        <i class="fa-fw fas fa-history"></i>
                        Match history
                    </div>
                    <div v-if="match.show_notes" :class="`btn btn-block btn-${showShowNotes ? 'light' : 'secondary'} mb-2`"
                         v-on:click="showShowNotes = !showShowNotes">
                        <i class="fa-fw far fa-file-video"></i>
                        Show notes
                    </div>
<!--                    <div :class="`btn btn-block btn-${showVod ? 'light' : 'secondary'} mb-2`" v-if="match.vod"-->
<!--                         v-on:click="showVod = !showVod">-->
<!--                        <i class="fa-fw far fa-desktop-alt"></i> Toggle VOD-->
<!--                    </div>-->
                    <!--
                    <div :class="`btn btn-block btn-secondary`" v-if="!mapData"
                         v-on:click="loadMapData(match.id)">
                        <i class="fa-fw far fa-sort-numeric-down"></i>
                        Load map data
                    </div>
                    -->
                </div>
                <div class="info-block">
                    <stat :match="match" data="custom_name">Custom match name</stat>
                    <stat :match="match" data="match_number">Match number</stat>
                    <stat :match="match" data="casters" :players="true">Casters</stat>
                    <stat :match="match" data="sub_event">Sub Event</stat>
<!-- TODO: change this to spacetime -->
                    <stat :match="match" data="start" time="true">Scheduled start time</stat>
                    <stat :match="match" data="vod" :format="(e) => `<a class='ct-active' href='${e}' target='_blank'>${e.replace('https://', '')}</a>`" raw="true">VOD Link</stat>
                    <stat :match="match" data="clean_feed" :format="(e) => `<a href='${e}' target='_blank'>${getURL(e).hostname}</a>`" raw="true">Clean Feed</stat>
                    <stat :match="match" data="first_to">First to</stat>
                    <stat :match="match" v-for="relGroup in playerRelationshipGroups" v-bind:key="relGroup.meta.singular_name"
                          :override="relGroup.items" :players="true">
                        {{ relGroup.items.length === 1 ? relGroup.meta.singular_name : relGroup.meta.plural_name }}</stat>
                    <stat :match="match" data="replay_codes" :raw="true" :format="(t) => t[0].replace(/\n/g, '<br>')">Replay codes</stat>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import { ReactiveArray, ReactiveRoot, ReactiveThing } from "@/utils/reactive";
import MapDisplay from "@/components/website/match/MapDisplay";
import { cssImage, getRoleSVG, multiImage, url } from "@/utils/content-utils";
import { logoBackground1 } from "@/utils/theme-styles";
import ThemeLogo from "@/components/website/ThemeLogo";
import PreviousMatch from "@/components/website/match/PreviousMatch";
import DetailedMatchStat from "@/components/website/match/DetailedMatchStat";
import Markdown from "@/components/website/Markdown";

export default {
    name: "DetailedMatch",
    components: { Markdown, PreviousMatch, ThemeLogo, MapDisplay, stat: DetailedMatchStat },
    props: ["id"],
    data: () => ({
        showPlayerInfo: false,
        showCastingInfo: false,
        showMatchHistory: true,
        showShowNotes: true,
        showRosters: true,
        showMatchMaps: true,
        showVod: false,

        showManagers: false
    }),
    methods: {
        url,
        icon(team) {
            return cssImage("backgroundImage", team.theme, ["default_logo", "small_logo"], 60);
        },
        theme(team) {
            return logoBackground1(team);
        },
        getRoleSVG
    },
    computed: {
        match () {
            return ReactiveRoot(this.id, {
                teams: ReactiveArray("teams", {
                    theme: ReactiveThing("theme"),
                    players: ReactiveArray("players"),
                    matches: ReactiveArray("matches", {
                        teams: ReactiveArray("teams", {
                            theme: ReactiveThing("theme")
                        }),
                        maps: ReactiveArray("maps", {
                            winner: ReactiveThing("winner", {
                                theme: ReactiveThing("theme")
                            })
                        })
                    }),
                    staff: ReactiveArray("staff")
                }),
                event: ReactiveThing("event", {
                    theme: ReactiveThing("theme")
                }),
                maps: ReactiveArray("maps", {
                    winner: ReactiveThing("winner", {
                        theme: ReactiveThing("theme")
                    })
                }),
                casters: ReactiveArray("casters"),
                player_relationships: ReactiveArray("player_relationships", {
                    player: ReactiveThing("player")
                })
            });
        },
        _theme() {
            return this.match?.event?.theme;
        },
        playerRelationshipGroups() {
            if (!this.match?.player_relationships) return [];
            const groups = {};

            this.match?.player_relationships.forEach(rel => {
                if (!groups[rel.singular_name]) {
                    groups[rel.singular_name] = {
                        meta: {
                            player_text: rel.player_text,
                            plural_name: rel.plural_name,
                            singular_name: rel.singular_name
                        },
                        items: []
                    };
                }
                groups[rel.singular_name].items = groups[rel.singular_name].items.concat(rel.player);
            });

            if (groups[undefined]) return [];

            return Object.values(groups);
        }
    },
    metaInfo() {
        return {
            title: this.match ? (this.match.custom_name || this.match.name) + " | Detailed view" : "Detailed match view",
            link: [{ rel: "icon", href: multiImage(this._theme, ["small_logo", "default_logo"]) }]
        };
    }
};
</script>

<style scoped>
    .maps-holder {
        display: flex;
        justify-content: center;
    }


    .team-header {
        font-size: 20px;
        padding: 8px 4px;
        text-align: center;
        font-weight: bold;
        border-bottom-width: 4px;
        border-bottom-style: solid;
        position: relative;
    }
    .team-logo {
        width: 90%;
        height: 50px;
    }
    .team-code {
        position: absolute;
        right: 0;
        top: 0;
        padding: 2px 10px;
    }
    .team-overlay-text {
        position: absolute;
        left: 0;
        top: 0;
        padding: 2px 10px;
    }
    .team-player {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: flex-start;
        flex-wrap: wrap;
    }
    .team:nth-of-type(2) .team-code { left: 0; right: auto; }
    .team:nth-of-type(2) .team-overlay-text { left: auto; right: 0; }

    .player-info.player-name {
        font-weight: bold;
    }

    .player-pronouns[data-pronoun="they/them"] {
        background: #96ff94 !important;
    }
    .player-pronouns[data-pronoun="she/her"] {
        background: #ffbe80 !important;
    }
    .team-player {
        margin-bottom: 4px;
    }

    .info-block .btn {
        position: relative;
        padding-left: calc(.75rem + 14px);
    }
    .info-block i {
        position: absolute;
        left: 0;
        margin: 4px 7px;
    }
    .top-right-logo {
        width: 100%;
        height: 140px;
    }
    .team-name {
        color: inherit !important;
    }
    .maps-holder {
        align-items: flex-start;
    }
    .player-role {
        height: 1em;
        width: 1em;
        color: white;
        display: inline-flex;
    }

    .player-icon-holder {width: 1.5em;}
    .btn-primary.text-dark-low {
        color: #343a40
    }

    a.btn-primary.text-dark-low:focus, a.btn-primary.text-dark-low:hover {
        color: #121416
    }
</style>

<template>
    <div class="map d-flex position-relative" v-bind:class="{'next-map': map._next }">
        <div v-if="mapVideo" class="map-bg map-video w-100 h-100 bg-center" v-bind:class="{'grayscale': !!winnerBG || (map && map.draw)}" :style="mapBackground">
            <video :src="mapVideo" autoplay muted loop></video>
        </div>
        <div v-else class="map-bg w-100 h-100 bg-center" v-bind:class="{'grayscale': !!winnerBG || (map && map.draw)}" :style="mapBackground"></div>
        <div class="map-gel w-100 h-100 position-absolute" :style="winnerBG"></div>
        <div class="map-gel w-100 h-100 position-absolute draw-gel" v-if="map && map.draw"></div>
        <div class="map-main d-flex flex-column h-100 w-100 position-absolute">
            <div class="map-top flex-grow-1 h-100 w-100 flex-center">
                <div class="map-logo-holder w-100 h-50 flex-center" v-if="winnerBG">
                    <div class="map-logo bg-center" :style="winnerLogo"></div>
                </div>
                <div class="gel-text" v-if="map && map.draw">DRAW</div>
            </div>
            <div class="map-lower flex-center flex-column" :style="accent">
                <div class="map-lower-name"><span class="industry-align">{{ name }}</span></div>
                <div class="map-lower-type" v-if="type"><span class="industry-align">{{ type }}</span></div>
            </div>
        </div>
    </div>
</template>

<script>
import { logoBackground } from "@/utils/theme-styles";
import { cssImage } from "@/utils/content-utils";


export default {
    name: "BroadcastMapDisplay",
    props: ["broadcast", "map", "accentColor", "showMapVideo"],
    computed: {
        mapBackground() {
            if (!(this.map?.big_image || this.map?.image)) return {};

            try {
                return {
                    backgroundImage: `url(${(this.map.big_image || this.map.image)[0].url}`
                };
            } catch (e) {
                return {};
            }
        },
        name() {
            if (!this.map?.name) return null;
            return this.map.name[0];
        },
        type() {
            if (!this.broadcast?.map_settings?.includes("Show mode")) return null;
            // Do the map specific name first, then the map data first
            if (this.map?.mode) return this.map.mode; // Custom map instance text
            if (this.map?.type?.length) return this.map.type[0]; // Map data (.map.type is a rollup from Airtable)
            return this.map.mode;
        },
        accent() {
            if (!this.accentColor) return {};
            return {
                backgroundColor: this.accentColor.theme,
                color: this.accentColor.text_on_theme
            };
        },
        winnerBG() {
            if (!this.map?.winner?.theme) return null;
            return logoBackground(this.map.winner.theme);
        },
        winnerLogo() {
            if (!this.map?.winner?.theme) return {};
            return cssImage("backgroundImage", this.map.winner.theme, ["default_wordmark", "default_logo", "small_logo"], 400);
        },
        mapVideo() {
            if (!this.showMapVideo) return null;
            if (!this.map?.map?.map_video?.length) return null;
            return this.map.map.map_video[0].url;
        }
    }
};
</script>

<style scoped>
    .map {
        flex-direction: column;
        margin: 0 12px;
        overflow: hidden;

        width: 0 !important;
        flex-grow: 2;
        transition: all 800ms ease;
    }
    .map-bg {
        background-size: cover;
    }
    .map-bg.grayscale {
        filter: grayscale(1);
    }
    .map-lower {
        font-size: 36px;
        text-align: center;
        padding: 10px 5px;
        line-height: 1;
        min-height: 100px;

        /* default */
        background-color: #333333;
        color: #ffffff;
    }
    .map-lower-type {
        font-size: 0.6em;
        text-align: center;
        padding-top: 0.25em;
    }
    .map-main {
        z-index: 2;
    }
    .map-gel {
        opacity: 0.5;
    }
    .map-logo {
        width: 90%;
        height: 90%;
    }
    .map-gel.draw-gel {
        background-color: #555;
    }
    .gel-text {
        color: white;
        font-size: 60px;
    }

    .map-bg video {
        height: 100%;
        object-fit: cover;
        width: 100%;
    }

    .map-bg.map-video {
        display: flex;
        justify-content: center;
        align-items: center;
        width: 100%;
    }
</style>

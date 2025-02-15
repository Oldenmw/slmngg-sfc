<template>
    <router-link :to="overrideURL || url(type, linkTo || thing)" class="link content-thing default-thing" :style="bgStyle" v-bind:class="{ 'has-headshot' : showHeadshot && headshot }">
      <span class="link-headshot" v-if="showHeadshot">
        <span class="headshot" :style="headshot"></span>
      </span>
        <span class="link-text">
          <span
              class="icon-internal bg-center icon-internal-left"
              :style="logo"
              v-if="shouldShowLogo && !logoRight"></span>

        <span class="name">{{ text || thing.name }}</span>

        <i class="fas fa-badge-check fa-fw" title="REAL" v-if="thing.verified"></i>

        <LoadingIcon v-if="loading"/>

        <span
            class="icon-internal bg-center icon-internal-right"
            :style="logo"
            v-if="shouldShowLogo && logoRight"></span>
        </span></router-link>
</template>

<script>
import { resizedImage, url, cssImage } from "@/utils/content-utils";
import LoadingIcon from "@/components/website/LoadingIcon";

export default {
    name: "ContentThing",
    props: ["theme", "thing", "showLogo", "type", "text", "logoRight", "linkTo", "showHeadshot", "overrideURL"],
    components: { LoadingIcon },
    methods: {
        url
    },
    computed: {
        shouldShowLogo() {
            /*
            * Show logo when:
            * - requested and loading
            * - requested and found
            *
            * Hide logo when:
            * - not requested
            * - requested and not found
            * */

            if (this.showLogo) {
                // console.log(this.thing.name, this.theme);
                if (this.logo) return true;
                if ((this.text || this.thing.name) && (this.theme && !this.theme.id)) return true; // theme not started loading
                // there is team.has_theme but that's not universal
                if (this.theme?.__loading) return true; // theme loading
            }
            return false;
        },
        loading() {
            return this.thing.__loading || !this.thing || !this.thing.id;
        },
        logo () {
            if (!this.theme || !this.theme.default_logo) return null;
            return cssImage("backgroundImage", this.theme, ["small_logo", "default_logo"], 30);
        },
        headshot () {
            if (!this.thing) return null;
            return {
                backgroundImage: `url(${resizedImage(this.thing, "headshot", 100) || resizedImage(this.theme, "default_logo", 100)})`,
                backgroundColor: this.theme.color_accent || this.theme.color_alt
            };
        },
        bgStyle () {
            if (!this.theme) return {};
            return {
                backgroundColor: this.theme.color_logo_background || this.theme.color_theme,
                borderColor: this.theme.color_logo_accent || this.theme.color_accent,
                color: this.theme.color_text_on_logo_background || this.theme.color_text_on_theme
            };
        }
    }
};
</script>

<style scoped>
    .content-thing {
        padding: .25em .5em;
        margin: .25em .25em;
        border-bottom-width: .2em;
        border-bottom-style: solid;
        display: flex;
        justify-content: center;
        align-items: center;
    }
    .content-thing:hover {
        color: inherit;
    }

    .icon-internal {
        height: 1.5em;
        width: 1.5em;
    }
    .icon-internal.icon-internal-left { margin-right: .3em }
    .icon-internal.icon-internal-right { margin-left: .3em }
    .content-thing i {
      margin: 5px 0;
    }

    .content-thing.has-headshot {
      display: inline-flex;
      flex-direction: column;
    }

    .headshot {
      display: inline-flex;
      width: 180px;
      height: 180px;
      margin-top: 4px;
      background-size: contain;
      background-repeat: no-repeat;
      background-position: bottom;
    }
    .link-text {
      display: inline-flex;
    }
</style>

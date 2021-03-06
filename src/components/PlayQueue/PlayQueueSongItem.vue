<template>
  <v-hover>
    <v-layout
      row
      align-center
      @click.right.prevent="handleRightClick"
      :class="[
        'song-item__wrapper',
        {
          'song-item--playing': isActive,
          'primary lighten-2': hover && darkMode,
          'primary darken-2': hover && !darkMode,
          'py-1': $vuetify.breakpoint.xsOnly,
          'dark-mode': darkMode
        }
      ]"
      slot-scope="{ hover }"
    >
      <template v-if="song && song.attributes">
        <div :class="['ml-2', 'mr-3', $style['left-items']]">
          <v-progress-circular
            v-if="showLoading"
            indeterminate
            color="accent"
          ></v-progress-circular>
          <div v-else class="size-fit">
            <MediaArtwork
              :artwork="song.attributes.artwork"
              :width="50"
              :height="50"
            />

            <MediaArtworkOverlay
              v-if="!isSongBlocked && !isArtistBlocked"
              :is-playing="isPlaying"
              :is-active="isActive && isNowPlaying"
              @playing-control-clicked="onSongClicked"
            />
          </div>
        </div>

        <v-flex :class="$style['middle-items']">
          <v-layout row wrap>
            <v-flex xs12 class="pr-2">
              <v-layout>
                <div
                  class="long-text-truncated main-info-text"
                  :style="{ color: songNameColor }"
                  :title="song.attributes.name"
                >
                  {{ song.attributes.name }}
                </div>

                <v-icon
                  class="ml-1"
                  small
                  v-if="song.attributes.contentRating === 'explicit'"
                  >explicit</v-icon
                >
              </v-layout>
            </v-flex>

            <v-flex xs12 class="pr-2">
              <div :class="['long-text-truncated']">
                <span
                  :class="$style['artist-name']"
                  @click="() => goToArtistPage(song)"
                  >{{ song.attributes.artistName }}</span
                >
              </div>
            </v-flex>
          </v-layout>
        </v-flex>

        <v-btn
          v-if="isAuthenticated"
          slot="activator"
          class="song-actions"
          :style="{ opacity: hover ? 1 : 0 }"
          icon
          @click.stop="onActionsIconClicked"
        >
          <v-icon>more_horiz</v-icon>
        </v-btn>

        <div
          :class="[
            'sub-info-text pr-2',
            'hidden-xs-only',
            $style['right-items']
          ]"
        >
          <template v-if="!hover">
            {{ song.attributes.durationInMillis | formattedDuration }}
          </template>
          <template v-else>
            <v-icon
              v-if="!isNowPlaying"
              @click="$emit('remove-from-queue', index)"
              color="red"
              >remove_circle</v-icon
            >
          </template>
        </div>
      </template>

      <div v-else class="pa-2">Item not available</div>
    </v-layout>
  </v-hover>
</template>

<script lang="ts">
import { Component, Prop, Vue, Mixins } from 'vue-property-decorator';
import { State, Action, Getter } from 'vuex-class';

import { MusicPlayerState } from '@/store/types';
import { TOGGLE_CURRENT_TRACK } from '@/store/actions.type';
import { HandleSongClicked, Song } from '@/@types/model/model';
import MediaArtwork from '@/components/MediaArtwork.vue';
import MediaArtworkOverlay from '@/components/MediaArtworkOverlay.vue';
import SongItemMixin from '@/mixins/SongItemMixin';
import GoToArtistPageMixin from '@/mixins/GoToArtistPageMixin';

@Component({
  components: { MediaArtworkOverlay, MediaArtwork }
})
export default class PlayQueueSongItem extends Mixins(
  SongItemMixin,
  GoToArtistPageMixin
) {
  @Prop({ default: false }) isHistory!: boolean;
  @Prop({ default: false }) isNowPlaying!: boolean;
  @Prop() index!: number;

  get songNameColor() {
    if (this.isHistory) {
      return this.$vuetify.theme.secondaryText;
    }

    if (this.isNowPlaying) {
      return this.$vuetify.theme.accent;
    }

    return this.$vuetify.theme.primaryText;
  }

  onActionsIconClicked(event: MouseEvent) {
    event.preventDefault();

    // @ts-ignore
    this.$root.$mediaActionMenu.open(
      this.song,
      null,
      event.clientX,
      event.clientY,
      true
    );
  }

  handleRightClick(event: MouseEvent) {
    event.preventDefault();

    // @ts-ignore
    this.$root.$mediaActionMenu.open(
      this.song,
      null,
      event.clientX,
      event.clientY,
      true
    );
  }
}
</script>

<style lang="scss" module>
@import '@/styles/components/_song-list-item.scss';
</style>

<style lang="scss" scoped>
.song-item__wrapper {
  border-bottom: 0.1rem solid rgba(0, 0, 0, 0.08);
  height: 6rem;
  transition: background-color 0.25s ease-in-out;

  &.dark-mode {
    border-bottom: 0.1rem solid rgba(255, 255, 255, 0.08);
  }
}
</style>

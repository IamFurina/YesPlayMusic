<template>
  <transition name="slide-up">
    <div
      class="lyrics-page"
      :class="{ 'no-lyric': noLyric }"
      :data-theme="theme"
    >
      <div
        v-if="
          (settings.lyricsBackground === 'blur') |
            (settings.lyricsBackground === 'dynamic')
        "
        class="lyrics-background"
        :class="{
          'dynamic-background': settings.lyricsBackground === 'dynamic',
        }"
      >
        <div
          class="top-right"
          :style="{ backgroundImage: `url(${bgImageUrl})` }"
        />
        <div
          class="bottom-left"
          :style="{ backgroundImage: `url(${bgImageUrl})` }"
        />
      </div>
      <img
        v-if="settings.lyricsBackground === true"
        class="gradient-background slight-move"
        :style="{ background }"
        :src="imageUrl"
      />
      <Visualization :option="{}" ref="visualization"></Visualization>
      <div
        v-if="settings.lyricsBackground === true"
        class="gradient-background"
        :style="{ background }"
      ></div>
      <div class="main-container">
        <div class="left-side">
          <div>
            <div v-if="settings.showLyricsTime" class="date">
              {{ date }}
            </div>
            <div class="cover">
              <div class="cover-container">
                <img
                  :src="imageUrl"
                  @contextmenu="changeCover"
                  loading="lazy"
                  onerror="this.src='https://p2.music.126.net/UeTuwE7pvjBpypWLudqukA==/3132508627578625.jpg'; this.onerror=null;"
                />
                <img class="shadow" :src="imageUrl" />
              </div>
            </div>
            <div class="controls">
              <div class="top-part">
                <div class="track-info">
                  <div class="title" :title="currentTrack.name">
                    <router-link
                      v-if="hasList()"
                      :to="`${getListPath()}`"
                      @click.native="toggleLyrics"
                      >{{ currentTrack.name }}
                    </router-link>
                    <span v-else>
                      {{ currentTrack.name }}
                    </span>
                  </div>
                  <div class="subtitle">
                    <router-link
                      v-if="artist.id !== 0"
                      :to="`/artist/${artist.id}`"
                      @click.native="toggleLyrics"
                      >{{ artist.name }}
                    </router-link>
                    <span v-else>
                      {{ artist.name }}
                    </span>
                    <span v-if="album.id !== 0">
                      -
                      <router-link
                        :to="`/album/${album.id}`"
                        :title="album.name"
                        @click.native="toggleLyrics"
                        >{{ album.name }}
                      </router-link>
                    </span>
                  </div>
                </div>
                <div class="top-right">
                  <div class="volume-control">
                    <button-icon
                      :title="$t('player.mute')"
                      @click.native="mute"
                    >
                      <svg-icon v-show="volume > 0.5" icon-class="volume" />
                      <svg-icon
                        v-show="volume === 0"
                        icon-class="volume-mute"
                      />
                      <svg-icon
                        v-show="volume <= 0.5 && volume !== 0"
                        icon-class="volume-half"
                      />
                    </button-icon>
                    <div class="volume-bar">
                      <vue-slider
                        v-model="volume"
                        :min="0"
                        :max="1"
                        :interval="0.01"
                        :drag-on-click="true"
                        :duration="0"
                        tooltip="none"
                        :dot-size="12"
                      ></vue-slider>
                    </div>
                  </div>
                  <div class="buttons">
                    <button-icon
                      :title="$t('player.like')"
                      @click.native="likeATrack(player.currentTrack.id)"
                    >
                      <svg-icon
                        :icon-class="
                          player.isCurrentTrackLiked ? 'heart-solid' : 'heart'
                        "
                      />
                    </button-icon>
                    <button-icon
                      :title="$t('contextMenu.addToPlaylist')"
                      @click.native="addToPlaylist"
                    >
                      <svg-icon icon-class="plus" />
                    </button-icon>
                    <!-- <button-icon @click.native="openMenu" title="Menu"
                    ><svg-icon icon-class="more"
                  /></button-icon> -->
                  </div>
                </div>
              </div>
              <div class="progress-bar">
                <span>{{ formatTrackTime(player.progress) || '0:00' }}</span>
                <div class="slider">
                  <vue-slider
                    v-model="player.progress"
                    :min="0"
                    :max="player.currentTrackDuration"
                    :interval="1"
                    :drag-on-click="true"
                    :duration="0"
                    :dot-size="12"
                    :height="2"
                    :tooltip-formatter="formatTrackTime"
                    :lazy="true"
                    :silent="true"
                  ></vue-slider>
                </div>
                <span>{{ formatTrackTime(player.currentTrackDuration) }}</span>
              </div>
              <div class="media-controls">
                <button-icon
                  v-show="!player.isPersonalFM"
                  :title="
                    player.repeatMode === 'one'
                      ? $t('player.repeatTrack')
                      : $t('player.repeat')
                  "
                  :class="{ active: player.repeatMode !== 'off' }"
                  @click.native="switchRepeatMode"
                >
                  <svg-icon
                    v-show="player.repeatMode !== 'one'"
                    icon-class="repeat"
                  />
                  <svg-icon
                    v-show="player.repeatMode === 'one'"
                    icon-class="repeat-1"
                  />
                </button-icon>
                <div class="middle">
                  <button-icon
                    v-show="!player.isPersonalFM"
                    :title="$t('player.previous')"
                    @click.native="playPrevTrack"
                  >
                    <svg-icon icon-class="previous" />
                  </button-icon>
                  <button-icon
                    v-show="player.isPersonalFM"
                    title="不喜欢"
                    @click.native="moveToFMTrash"
                  >
                    <svg-icon icon-class="thumbs-down" />
                  </button-icon>
                  <button-icon
                    id="play"
                    :title="$t(player.playing ? 'player.pause' : 'player.play')"
                    @click.native="playOrPause"
                  >
                    <svg-icon :icon-class="player.playing ? 'pause' : 'play'" />
                  </button-icon>
                  <button-icon
                    :title="$t('player.next')"
                    @click.native="playNextTrack"
                  >
                    <svg-icon icon-class="next" />
                  </button-icon>
                </div>
                <button-icon
                  v-show="!player.isPersonalFM"
                  :title="$t('player.shuffle')"
                  :class="{ active: player.shuffle }"
                  @click.native="switchShuffle"
                >
                  <svg-icon icon-class="shuffle" />
                </button-icon>
              </div>
            </div>
          </div>
        </div>
        <div class="highlight-lyric">
          <div
            class="highlight-lyric-line"
            v-if="lyricWithTranslation[highlightLyricIndex]"
            :key="lyricWithTranslation[highlightLyricIndex].contents[0]"
          >
            <span
              class="highlight-lyric-line-char"
              v-for="(a, index) in lyricWithTranslation[highlightLyricIndex]
                .contents[0]"
              :key="index"
              :char-index="index"
              :style="{ 'animation-delay': index * 0.02 + 's' }"
              >{{ a }}</span
            >
          </div>
          <div
            class="highlight-lyric-line"
            v-if="lyricWithTranslation[highlightLyricIndex]"
            :key="lyricWithTranslation[highlightLyricIndex].contents[1]"
          >
            <span
              class="highlight-lyric-line-char"
              v-for="(a, index) in lyricWithTranslation[highlightLyricIndex]
                .contents[1]"
              :style="{ 'animation-delay': index * 0.02 + 's' }"
              :key="index"
              :char-index="index"
              >{{ a }}</span
            >
          </div>
        </div>
        <div
          class="right-side"
          :style="{
            transform: `perspective(${$store.state.visualSet.perspective}px) rotateY(${$store.state.visualSet.rotateY}deg)`,
          }"
        >
          <transition name="slide-fade">
            <div
              v-show="!noLyric"
              ref="lyricsContainer"
              class="lyrics-container"
              :style="lyricFontSize"
            >
              <div id="line-1" class="line"></div>
              <div
                v-for="(line, index) in lyricWithTranslation"
                :id="`line${index}`"
                :key="index"
                class="line"
                :class="{
                  highlight: highlightLyricIndex === index,
                }"
                @click="clickLyricLine(line.time)"
                @dblclick="clickLyricLine(line.time, true)"
              >
                <div class="content">
                  <span v-if="line.contents[0]">{{ line.contents[0] }}</span>
                  <br />
                  <span
                    v-if="
                      line.contents[1] &&
                      $store.state.settings.showLyricsTranslation
                    "
                    class="translation"
                    >{{ line.contents[1] }}</span
                  >
                </div>
              </div>
            </div>
          </transition>
        </div>
      </div>
      <div class="close-button" @click="toggleLyrics">
        <button>
          <svg-icon icon-class="arrow-down" />
        </button>
      </div>
    </div>
  </transition>
</template>

<script>
// The lyrics page of Apple Music is so gorgeous, so I copy the design.
// Some of the codes are from https://github.com/sl1673495/vue-netease-music
/* eslint-disable */

import { mapState, mapMutations, mapActions } from 'vuex';
import VueSlider from 'vue-slider-component';
import { formatTrackTime } from '@/utils/common';
import { getLyric } from '@/api/track';
import { lyricParser } from '@/utils/lyrics';
import ButtonIcon from '@/components/ButtonIcon.vue';
import Visualization from '@/components/Visualization';
import * as Vibrant from 'node-vibrant/dist/vibrant.worker.min.js';
import Color from 'color';
import { isAccountLoggedIn } from '@/utils/auth';
import { hasListSource, getListSourcePath } from '@/utils/playList';
import locale from '@/locale';
export default {
  name: 'Lyrics',
  components: {
    VueSlider,
    ButtonIcon,
    Visualization,
  },
  data() {
    return {
      lyricsInterval: null,
      lyric: [],
      tlyric: [],
      highlightLyricIndex: -1,
      minimize: true,
      resetImageUrl: false,
      background: '',
      date: this.formatTime(new Date()),
    };
  },
  computed: {
    ...mapState(['player', 'settings', 'showLyrics']),
    currentTrack() {
      return this.player.currentTrack;
    },
    volume: {
      get() {
        return this.player.volume;
      },
      set(value) {
        this.player.volume = value;
      },
    },
    imageUrl() {
      if (this.resetImageUrl) {
        return this.resetImageUrl;
      }
      return this.player.currentTrack?.al?.picUrl + '?param=1024y1024';
    },
    bgImageUrl() {
      return this.player.currentTrack?.al?.picUrl + '?param=512y512';
    },
    lyricWithTranslation() {
      let ret = [];
      // 空内容的去除
      const lyricFiltered = this.lyric.filter(({ content }) =>
        Boolean(content)
      );
      // content统一转换数组形式
      if (lyricFiltered.length) {
        lyricFiltered.forEach(l => {
          const { rawTime, time, content } = l;
          const lyricItem = { time, content, contents: [content] };
          const sameTimeTLyric = this.tlyric.find(
            ({ rawTime: tLyricRawTime }) => tLyricRawTime === rawTime
          );
          if (sameTimeTLyric) {
            const { content: tLyricContent } = sameTimeTLyric;
            if (content) {
              lyricItem.contents.push(tLyricContent);
            }
          }
          ret.push(lyricItem);
        });
      } else {
        ret = lyricFiltered.map(({ time, content }) => ({
          time,
          content,
          contents: [content],
        }));
      }
      return ret;
    },
    lyricFontSize() {
      return {
        fontSize: `${this.$store.state.settings.lyricFontSize || 28}px`,
      };
    },
    noLyric() {
      return this.lyric.length == 0;
    },
    artist() {
      return this.currentTrack?.ar
        ? this.currentTrack.ar[0]
        : { id: 0, name: 'unknown' };
    },
    album() {
      return this.currentTrack?.al || { id: 0, name: 'unknown' };
    },
    theme() {
      return this.settings.lyricsBackground === true ? 'dark' : 'auto';
    },
  },
  watch: {
    currentTrack() {
      if (this.currentTrack.source) {
        this.getLyric('tencent');
      } else {
        this.getLyric();
      }
      this.getCoverColor();
    },
    showLyrics(show) {
      if (show) {
        this.setLyricsInterval();
        this.$store.commit('enableScrolling', false);
      } else {
        clearInterval(this.lyricsInterval);
        this.$store.commit('enableScrolling', true);
      }
    },
  },
  created() {
    this.getLyric();
    this.getCoverColor();
    this.initDate();
  },
  beforeDestroy() {
    clearInterval(this.lyricsInterval);
  },
  beforeDestroy: function () {
    if (this.timer) {
      clearInterval(this.timer);
    }
  },
  destroyed() {
    clearInterval(this.lyricsInterval);
  },
  methods: {
    ...mapMutations(['toggleLyrics', 'updateModal']),
    ...mapActions(['likeATrack']),
    async changeCover() {
      const pastedText = await navigator.clipboard.readText();
      const isLink = text => {
        try {
          return !!new URL(text);
        } catch (error) {
          return false;
        }
      };
      if (!isLink(pastedText)) {
        return;
      }
      this.resetImageUrl = pastedText;
      this.getCoverColor();
    },
    initDate() {
      var _this = this;
      clearInterval(this.timer);
      this.timer = setInterval(function () {
        _this.date = _this.formatTime(new Date());
      }, 1000);
    },
    formatTime(value) {
      let hour = value.getHours().toString();
      let minute = value.getMinutes().toString();
      let second = value.getSeconds().toString();
      return (
        hour.padStart(2, '0') +
        ':' +
        minute.padStart(2, '0') +
        ':' +
        second.padStart(2, '0')
      );
    },
    addToPlaylist() {
      if (!isAccountLoggedIn()) {
        this.showToast(locale.t('toast.needToLogin'));
        return;
      }
      this.$store.dispatch('fetchLikedPlaylist');
      this.updateModal({
        modalName: 'addTrackToPlaylistModal',
        key: 'show',
        value: true,
      });
      this.updateModal({
        modalName: 'addTrackToPlaylistModal',
        key: 'selectedTrackID',
        value: this.currentTrack?.id,
      });
    },
    playPrevTrack() {
      this.player.playPrevTrack();
    },
    playOrPause() {
      this.player.playOrPause();
    },
    playNextTrack() {
      this.resetImageUrl = false;
      if (this.player.isPersonalFM) {
        this.player.playNextFMTrack();
      } else {
        this.player.playNextTrack();
      }
    },
    getLyric(server) {
      return getLyric(this.currentTrack.id, server).then(data => {
        if (!data?.lrc?.lyric) {
          this.lyric = [];
          this.tlyric = [];
          return false;
        } else {
          let { lyric, tlyric } = lyricParser(data);
          lyric = lyric.filter(
            l => !/^作(词|曲)\s*(:|：)\s*无$/.exec(l.content)
          );
          let includeAM =
            lyric.length <= 10 &&
            lyric.map(l => l.content).includes('纯音乐，请欣赏');
          if (includeAM) {
            let reg = /^作(词|曲)\s*(:|：)\s*/;
            let author = this.currentTrack?.ar[0]?.name;
            lyric = lyric.filter(l => {
              let regExpArr = l.content.match(reg);
              return (
                !regExpArr || l.content.replace(regExpArr[0], '') !== author
              );
            });
          }
          if (lyric.length === 1 && includeAM) {
            this.lyric = [];
            this.tlyric = [];
            return false;
          } else {
            this.lyric = lyric;
            this.tlyric = tlyric;
            return true;
          }
        }
      });
    },
    formatTrackTime(value) {
      return formatTrackTime(value);
    },
    clickLyricLine(value, startPlay = false) {
      // TODO: 双击选择还会选中文字，考虑搞个右键菜单复制歌词
      let jumpFlag = false;
      this.lyric.filter(function (item) {
        if (item.content == '纯音乐，请欣赏') {
          jumpFlag = true;
        }
      });
      if (window.getSelection().toString().length === 0 && !jumpFlag) {
        this.player.seek(value);
      }
      if (startPlay === true) {
        this.player.play();
      }
    },
    setLyricsInterval() {
      this.lyricsInterval = setInterval(() => {
        const progress = this.player.seek() ?? 0;
        let oldHighlightLyricIndex = this.highlightLyricIndex;
        this.highlightLyricIndex = this.lyric.findIndex((l, index) => {
          const nextLyric = this.lyric[index + 1];
          return (
            progress >= l.time && (nextLyric ? progress < nextLyric.time : true)
          );
        });
        if (oldHighlightLyricIndex !== this.highlightLyricIndex) {
          const el = document.getElementById(`line${this.highlightLyricIndex}`);
          if (el)
            el.scrollIntoView({
              behavior: 'smooth',
              block: 'center',
            });
        }
      }, 50);
    },
    moveToFMTrash() {
      this.player.moveToFMTrash();
    },
    switchRepeatMode() {
      this.player.switchRepeatMode();
    },
    switchShuffle() {
      this.player.switchShuffle();
    },
    getCoverColor() {
      if (this.settings.lyricsBackground !== true) return;
      const cover = this.imageUrl + '?param=256y256';
      Vibrant.from(cover, { colorCount: 1 })
        .getPalette()
        .then(palette => {
          const originColor = Color.rgb(palette.DarkMuted._rgb);
          const color = originColor.darken(0.1).rgb().fade(0.28).string();
          const color2 = originColor
            .lighten(0.28)
            .rotate(-30)
            .rgb()
            .fade(0.4)
            .string();
          this.background = `linear-gradient(to top left, ${color}, ${color2})`;
        });
    },
    hasList() {
      return hasListSource();
    },
    getListPath() {
      return getListSourcePath();
    },
    mute() {
      this.player.mute();
    },
  },
};
</script>

<style lang="scss" scoped>
.highlight-lyric-line-char {
  opacity: 0;
  animation-name: bounce-in;
  animation-duration: 0.5s;
  animation-fill-mode: forwards;
}

@keyframes bounce-in {
  0% {
    font-weight: 100;
    opacity: 0;
    // transform: translateX(100%) scale(0.2);
  }
  80% {
    opacity: 0.8;
    font-weight: 500;
    // transform: translateY(0) scale(1.5);
  }
  100% {
    font-weight: normal;
    opacity: 1;
    // transform: translateY(0) scale(1);
  }
}
.highlight-lyric {
  display: flex;
  flex-direction: column;
  position: fixed;
  color: #fff;
  font-size: 20px;
  bottom: 20px;
  user-select: none;
  white-space: pre;
  left: 0;
  width: 100vw;
}
.highlight-lyric .highlight-lyric-line {
  justify-content: center;
  width: 100%;
  line-height: 36px height：36px;
  display: flex;
  flex-direction: row;
}
.lyrics-page {
  position: fixed;
  top: 0;
  right: 0;
  left: 0;
  bottom: 0;
  z-index: 200;
  background: var(--color-body-bg);
  display: flex;
  clip: rect(auto, auto, auto, auto);
}

.lyrics-background {
  --contrast-lyrics-background: 75%;
  --brightness-lyrics-background: 150%;
}

[data-theme='dark'] .lyrics-background {
  --contrast-lyrics-background: 125%;
  --brightness-lyrics-background: 50%;
}
.main-container {
  display: flex;
  margin: 0 auto;
  flex-direction: row;
}
.lyrics-background {
  filter: blur(50px) contrast(var(--contrast-lyrics-background))
    brightness(var(--brightness-lyrics-background));
  position: absolute;
  height: 100vh;
  width: 100%;
  .top-right,
  .bottom-left {
    z-index: 0;
    width: 140vw;
    height: 140vw;
    opacity: 0.6;
    position: absolute;
    background-size: cover;
  }

  .top-right {
    right: 0;
    top: 0;
    mix-blend-mode: luminosity;
  }

  .bottom-left {
    left: 0;
    bottom: 0;
    animation-direction: reverse;
    animation-delay: 10s;
  }
}

.dynamic-background > div {
  animation: rotate 150s linear infinite;
}

@keyframes rotate {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
@keyframes slight-move {
  0% {
    transform: scale(1);
    transform-origin: center;
  }
  25% {
    transform: scale(1.2) translate(-8%, -8%);
    transform-origin: center;
  }
  50% {
    transform: scale(1.3) translate(0, 0);
    transform-origin: center;
  }
  75% {
    transform: scale(1.4) translate(8%, 8%);
    transform-origin: center;
  }
  100% {
    transform: scale(1);
    transform-origin: center;
  }
}
.slight-move {
  animation: slight-move 25s infinite;
}
.gradient-background {
  position: absolute;
  height: 100vh;
  width: 100vw;
  object-fit: cover;
  background-size: cover !important;
  background-position: center !important;
  filter: blur(6px);
  margin: 0;
}

.left-side {
  flex: 1;
  display: flex;
  justify-content: flex-end;
  margin: 80px 0;
  align-items: center;
  transition: all 0.5s;

  z-index: 1;

  .date {
    max-width: 54vh;
    margin: 24px 0;
    color: var(--color-text);
    text-align: center;
    font-size: 4rem;
    font-weight: 600;
    opacity: 0.88;
    display: -webkit-box;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 1;
    overflow: hidden;
  }

  .controls {
    max-width: 54vh;
    margin-top: 24px;
    color: var(--color-text);

    .title {
      margin-top: 8px;
      font-size: 1.4rem;
      font-weight: 600;
      opacity: 0.88;
      display: -webkit-box;
      -webkit-box-orient: vertical;
      -webkit-line-clamp: 1;
      overflow: hidden;
    }

    .subtitle {
      margin-top: 4px;
      font-size: 1rem;
      opacity: 0.58;
      display: -webkit-box;
      -webkit-box-orient: vertical;
      -webkit-line-clamp: 1;
      overflow: hidden;
    }

    .top-part {
      display: flex;
      justify-content: space-between;

      .top-right {
        display: flex;
        justify-content: space-between;

        .volume-control {
          margin: 0 10px;
          display: flex;
          align-items: center;
          .volume-bar {
            width: 84px;
          }
        }

        .buttons {
          display: flex;
          align-items: center;

          button {
            margin: 0 0 0 4px;
          }

          .svg-icon {
            height: 18px;
            width: 18px;
          }
        }
      }
    }

    .progress-bar {
      margin-top: 22px;
      display: flex;
      align-items: center;
      justify-content: space-between;

      .slider {
        width: 100%;
        flex-grow: grow;
        padding: 0 10px;
      }

      span {
        font-size: 15px;
        opacity: 0.58;
        min-width: 28px;
      }
    }

    .media-controls {
      display: flex;
      justify-content: center;
      margin-top: 18px;
      align-items: center;

      button {
        margin: 0;
      }

      .svg-icon {
        opacity: 0.38;
        height: 14px;
        width: 14px;
      }

      .active .svg-icon {
        opacity: 0.88;
      }

      .middle {
        padding: 0 16px;
        display: flex;
        align-items: center;

        button {
          margin: 0 8px;
        }

        button#play .svg-icon {
          height: 28px;
          width: 28px;
          padding: 2px;
        }

        .svg-icon {
          opacity: 0.88;
          height: 22px;
          width: 22px;
        }
      }
    }
  }
}

.cover {
  position: relative;

  .cover-container {
    position: relative;
  }

  img {
    border-radius: 0.75em;
    width: 54vh;
    height: 54vh;
    user-select: none;
    object-fit: cover;
  }

  .shadow {
    position: absolute;
    top: 0;
    height: 105%;
    left: 0;
    width: 105%;
    filter: blur(14px) opacity(0.8);
    transform: scale(0.92, 0.96);
    z-index: -1;
    background-size: cover;
    border-radius: 0.75em;
  }
}

.right-side {
  margin: 80px 0;
  flex: 1;
  font-weight: 600;
  color: var(--color-text);
  margin-right: 30px;
  z-index: 0;

  .lyrics-container {
    height: 100%;
    display: flex;
    flex-direction: column;
    padding-left: 78px;
    max-width: 460px;
    overflow-y: auto;
    transition: 0.5s;
    scrollbar-width: none; // firefox

    .line {
      margin: 2px 0;
      padding: 12px 18px;
      transition: 0.5s;
      border-radius: 12px;
      text-align: center;
      &:hover {
        background: var(--color-secondary-bg-for-transparent);
      }

      .content {
        transform-origin: center left;
        transform: scale(0.95);
        transition: all 0.35s cubic-bezier(0.25, 0.46, 0.45, 0.94);

        span {
          opacity: 0.28;
          cursor: default;
          font-size: 1em;
          transition: all 0.35s cubic-bezier(0.25, 0.46, 0.45, 0.94);
        }

        span.translation {
          opacity: 0.2;
          font-size: 0.925em;
        }
      }
    }

    .line#line-1:hover {
      background: unset;
    }

    .translation {
      margin-top: 0.1em;
    }

    .highlight div.content {
      transform: scale(1);
      span {
        opacity: 0.98;
        display: inline-block;
      }

      span.translation {
        opacity: 0.65;
      }
    }
  }

  ::-webkit-scrollbar {
    display: none;
  }

  .lyrics-container .line:first-child {
    margin-top: 50vh;
  }

  .lyrics-container .line:last-child {
    margin-bottom: calc(50vh - 128px);
  }
}

.close-button {
  position: fixed;
  top: 24px;
  right: 24px;
  z-index: 300;
  border-radius: 0.75rem;
  height: 44px;
  width: 44px;
  display: flex;
  justify-content: center;
  align-items: center;
  opacity: 0.28;
  transition: 0.2s;
  -webkit-app-region: no-drag;

  .svg-icon {
    color: var(--color-text);
    padding-top: 5px;
    height: 22px;
    width: 22px;
  }

  &:hover {
    background: var(--color-secondary-bg-for-transparent);
    opacity: 0.88;
  }
}

.lyrics-page.no-lyric {
  .left-side {
    transition: all 0.5s;
    margin-right: 0;
  }
}

@media screen and (min-width: 1200px) {
  .right-side .lyrics-container {
    max-width: 600px;
  }
}
@media screen and (max-width: 576px) {
  .gradient-background {
    filter: blur(5px);
  }
  .main-container {
    display: flex;
    margin: 30px auto;
    flex-direction: column;
    align-items: center;
  }
  .highlight-lyric {
    display: none;
  }
  .lyrics-container .line .content {
    transform-origin: center !important;
    font-size: 0.8em;
  }
  .left-side {
    flex: 4;
    margin: 0;
  }
  .right-side .lyrics-container {
    text-align: center;
    padding: 0;
    margin: 0;
    font-size: 0.8;
    width: 100%;
  }
  .cover {
    img {
      width: 80vw;
      height: 80vw;
    }
  }
  .left-side .controls {
    width: 80vw;
  }
  .right-side {
    height: 120px;
    margin: 0;
    .lyrics-container {
      .line {
        padding: 5px 18px;
      }
    }
  }
}
.slide-up-enter-active,
.slide-up-leave-active {
  transition: all 0.4s;
}

.slide-up-enter, .slide-up-leave-to /* .fade-leave-active below version 2.1.8 */ {
  transform: translateY(100%);
}

.slide-fade-enter-active {
  transition: all 0.5s ease;
}

.slide-fade-leave-active {
  transition: all 0.5s cubic-bezier(0.2, 0.2, 0, 1);
}

.slide-fade-enter,
.slide-fade-leave-to {
  opacity: 0;
}
</style>

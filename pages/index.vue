<template>
  <v-layout column justify-center align-center app>
    <!-- 播放控制的浮动按钮 -->
    <v-speed-dial
      v-model="fab"
      fixed
      bottom
      right
      direction="top"
      open-on-click
      transition="slide-y-reverse-transition"
    >
      <template v-slot:activator>
        <v-btn slot="activator" v-model="fab" :class="speed_dial_color" dark fab hover>
          <v-icon v-if="fab">
            {{ icons.close }}
          </v-icon>
          <v-icon v-else large>
            {{ icons.play }}
          </v-icon>
        </v-btn>
      </template>
      <v-btn fab small :class="fab_color" @click.stop="stop_all()">
        <span class="fab-tip">{{ $t('control.stop') }}</span>
        <v-icon :class="fab_icon">
          {{ icons.stop }}
        </v-icon>
      </v-btn>
      <v-btn fab small :class="fab_color" @click.stop="play_random_voice()">
        <span class="fab-tip">{{ $t('control.pick_one') }}</span>
        <v-icon :class="fab_icon">
          {{ icons.selection_ellipse_arrow_inside }}
        </v-icon>
      </v-btn>
      <!-- 倒放功能实现（试验性） -->
      <v-btn fab small :class="fab_color" @click.stop="reverse = !reverse">
        <span class="fab-tip">
          {{ reverse_text }}
        </span>
        <v-icon :class="fab_icon">
          {{ icons.undo_variant }}
        </v-icon>
      </v-btn>
      <v-btn fab small :class="fab_color" :disabled="random" @click.stop="overlap = !overlap">
        <span class="fab-tip">
          {{ overlap_text }}
        </span>
        <v-icon :class="fab_icon">
          {{ icons.view_parallel }}
        </v-icon>
      </v-btn>
      <v-btn fab small :class="fab_color" :disabled="random" @click.stop="repeat = !repeat">
        <span class="fab-tip">
          {{ repeat_text }}
        </span>
        <v-icon :class="fab_icon">
          {{ icons.repeat }}
        </v-icon>
      </v-btn>
      <v-btn fab small :class="fab_color" :disabled="overlap || repeat" @click.stop="random = !random">
        <span class="fab-tip">
          {{ random_text }}
        </span>
        <v-icon :class="fab_icon">
          {{ icons.shuffle }}
        </v-icon>
      </v-btn>
    </v-speed-dial>
    <v-flex xs12 sm8 md6 style="min-width: 85%;">
      <!-- 直播或者动态面板，不要取消注释，还没写功能 -->
      <!--
      <v-card :loading="lives_loading">
        <v-card-title>
          <v-icon class="primary--text" :class="dark_text" style="margin-right: 8px;">
            {{ icons.clock_outline }}
          </v-icon>
          {{ $t('live.activity') }}
        </v-card-title>
        <SkeletonLoading :loading="lives_loading">
          <v-card-text>
            <div v-for="live in lives" :key="live.id">
              <div v-if="live.title.length" :class="dark_text">
                <span class="warning--text">{{ $t('live.on_air') }}</span>
                <youtube-link :video-key="live.yt_video_key" :content="live.title" class="error--text" />
              </div>
            </div>
            <div v-for="live in upcoming_lives" :key="live.id">
              <div v-if="live.title.length" :class="dark_text">
                <span>{{ $t('live.schedule') + format_time(live.live_schedule) }}</span>
                <youtube-link :video-key="live.yt_video_key" :content="live.title" />
              </div>
            </div>
            <div v-if="lives.length === 0 && upcoming_lives.length === 0">
              <p>{{ lives_loading ? $t('live.loading') : $t('live.no_schedule') }}</p>
            </div>
            <div class="notification-board" v-html="$md.render($t('live.notification'))"></div>
          </v-card-text>
        </SkeletonLoading>
      </v-card>
      -->
      <!-- 对每个按钮组生成一个Card -->
      <v-card v-for="group in groups" :key="group.name">
        <v-card-title class="headline" :class="dark_text">
          {{ group.group_description[current_locale] }}
        </v-card-title>
        <v-card-text>
          <voice-btn
            v-for="item in group.voice_list"
            ref="voice_btn"
            :key="item.name"
            :class="voice_button_color"
            @click.native="play(item)"
          >
            {{ item.description[current_locale] }}
          </voice-btn>
        </v-card-text>
      </v-card>
    </v-flex>
  </v-layout>
</template>

<style lang="scss" scoped>
$nonlinear-transition: cubic-bezier(0.25, 0.8, 0.5, 1);
.v-card {
  margin: 8px auto;
}

.fab-tip {
  position: absolute;
  right: 52px;
  padding: 5px 16px;
  background: rgba(97, 97, 97, 0.9);
  border-radius: 4px;
  color: #fff;
  width: auto;
  font-size: 14px;
  font-weight: normal;
  line-height: 22px;
  opacity: 0.9;
  text-transform: none;
  letter-spacing: normal;
}

.notification-board {
  white-space: pre-line;
  height: auto;
  display: flex;
}
.notification-board ::v-deep p {
  margin-bottom: 0;
}
</style>

<script>
import voice_lists from '~/assets/voices.json';
import VoiceBtn from '../components/VoiceBtn';
//import SkeletonLoading from '../components/SkeletonLoading';
import {
  mdiClockOutline,
  mdiClose,
  mdiPlay,
  mdiRepeat,
  mdiSelectionEllipseArrowInside,
  mdiShuffle,
  mdiStop,
  mdiUndoVariant, //experimental
  mdiViewParallel
} from '@mdi/js';

export default {
  components: {
    VoiceBtn
    //SkeletonLoading
  },
  data() {
    return {
      icons: {
        close: mdiClose,
        play: mdiPlay,
        stop: mdiStop,
        selection_ellipse_arrow_inside: mdiSelectionEllipseArrowInside,
        view_parallel: mdiViewParallel,
        repeat: mdiRepeat,
        shuffle: mdiShuffle,
        undo_variant: mdiUndoVariant, // experimental
        clock_outline: mdiClockOutline
      },
      overlap: false,
      random: false,
      repeat: false,
      reverse: false, // experimental
      fab: false,
      groups: voice_lists.groups,
      now_playing: new Set(),
      upcoming_lives: [],
      lives: [],
      lives_loading: true,
      audioCtx: null
    };
  },
  computed: {
    voice_host() {
      return 'voices/';
      //可以选择生产环境用CDN以减少宽带使用量/缓解速度瓶颈，例如白嫖JsDeliver
      //if (process.env.NODE_ENV === 'production')
      //  return 'https://cdn.jsdelivr.net/gh/GitHub用户名/仓库名@master/static/voices/';
      //else return '/voices/';
    },
    dark_text() {
      return {
        'grey--text text--lighten-2': this.$vuetify.theme.dark
      };
    },
    voice_button_color() {
      return {
        'light-blue darken-4': this.$vuetify.theme.dark,
        'blue lighten-2 white--text': !this.$vuetify.theme.dark
      };
    },
    fab_icon() {
      return [this.$vuetify.theme.dark ? 'white--text' : 'light-blue--text'];
    },
    fab_color() {
      return [this.$vuetify.theme.dark ? 'indigo darken-1' : 'white'];
    },
    speed_dial_color: function () {
      return [this.$vuetify.theme.dark ? 'cyan darken-1' : 'cyan lighten-2'];
    },
    current_locale() {
      return this.$i18n.locale;
    },
    overlap_text() {
      return (
        this.$t('control.overlap') + ' ' + (this.overlap ? this.$t('control.enabled') : this.$t('control.disabled'))
      );
    },
    random_text() {
      return this.$t('control.random') + ' ' + (this.random ? this.$t('control.enabled') : this.$t('control.disabled'));
    },
    repeat_text() {
      return this.$t('control.repeat') + ' ' + (this.repeat ? this.$t('control.enabled') : this.$t('control.disabled'));
    },
    reverse_text() {
      // experimental
      return (
        this.$t('control.reverse') + ' ' + (this.reverse ? this.$t('control.enabled') : this.$t('control.disabled'))
      );
    }
  },
  async mounted() {
    this.$vuetify.theme.dark = this.$store.state.dark === 'true';
    await this.fetch_live_data();
    //Media Session Metadata
    if ('mediaSession' in navigator) {
      navigator.mediaSession.setActionHandler('nexttrack', () => {
        this.play_random_voice();
      });
      navigator.mediaSession.setActionHandler('previoustrack', () => {
        this.play_random_voice();
      });
      navigator.mediaSession.setActionHandler('pause', () => {
        this.stop_all();
        navigator.mediaSession.playbackState = 'paused';
      });
    }
  },
  methods: {
    async fetch_live_data() {
      //TODO: 获取B站动态或者直播信息
      //TODO：可选主要的平台，例如YouTube和Bilibili
      return null;
    },
    format_time(stamp) {
      return require('dayjs')(stamp).format('YYYY/M/DD HH:mm');
    },
    send_google_event(item) {
      if (process.client && process.env.NODE_ENV === 'production') {
        // eslint-disable-next-line no-undef
        ga('send', {
          hitType: 'event',
          eventCategory: 'Audios',
          eventAction: 'play',
          eventLabel: item.name + ' ' + item.description['zh']
        });
      }
    },
    async play(item) {
      //播放音频主逻辑部分
      let ref = null;
      let timer = null;
      this.$refs.voice_btn.forEach(i => {
        if (i.$vnode.data.key === item.name) {
          ref = i;
        }
      });
      if (!this.overlap) {
        this.now_playing.forEach(i => {
          if (i && typeof i.pause === 'function') {
            i.pause();
          }
          if (i && typeof i.stop === 'function') {
            i.stop();
          }
          this.now_playing.delete(i);
          console.log(item.name, 'paused before new playing');
        });
      }
      let setup_timer = () => {
        // 进度条前进
        if (timer !== null) clear_timer();
        // 初始化
        timer = setInterval(() => {
          let prog = Number(((audio.currentTime / audio.duration) * 100).toFixed(2));
          // 音频当前播放的时间除以总时间，乘 100 得到百分比，保留两位小数
          if (prog !== Infinity && !isNaN(prog)) {
            ref.progress = prog;
          } else {
            ref.progress = 0;
          }
        }, 20);
        // 每 20ms 更新
      };
      let smooth_end = () => {
        // 进度条返回
        let play_end_timer = setInterval(() => {
          ref.progress -= 5;
          // 进度每一次 -5，直到结束
          if (ref.progress <= 0) {
            clearInterval(play_end_timer);
            // 清除计时器
            play_end_timer = null;
          }
        }, 50);
        ref.playing = false;
      };
      let clear_timer = () => {
        clearInterval(timer);
        timer = null;
      };

      // 倒放功能（experimental）：使用 Web Audio API 将音频缓冲倒序后播放
      // 该功能为 AI 辅助实现，可能存在性能问题
      if (this.reverse) {
        // 仅在客户端环境执行倒放逻辑，SSR 环境回退为正常播放
        if (!process.client) {
          // 在服务端渲染阶段不执行倒放，继续走常规 HTMLAudio 播放分支
        } else {
          try {
            const ctx = this.audioCtx || new (window.AudioContext || window.webkitAudioContext)();
            this.audioCtx = ctx;
            const url = this.voice_host + item.path;
            const res = await fetch(url);
            const arrayBuffer = await res.arrayBuffer();
            const buffer = await ctx.decodeAudioData(arrayBuffer);

            for (let ch = 0; ch < buffer.numberOfChannels; ch++) {
              const data = buffer.getChannelData(ch);
              // TypedArray 原地倒序
              Array.prototype.reverse.call(data);
            }

            const source = ctx.createBufferSource();
            source.buffer = buffer;
            source.connect(ctx.destination);

            const startTime = ctx.currentTime;
            // 进度条前进（基于 AudioContext 时间轴）
            if (timer !== null) clear_timer();
            timer = setInterval(() => {
              const ratio = (ctx.currentTime - startTime) / buffer.duration;
              const prog = Number((ratio * 100).toFixed(2));
              ref.progress = prog !== Infinity && !isNaN(prog) ? prog : 0;
            }, 20);

            source.start(0);
            this.now_playing.add(source);
            this.send_google_event(item);
            ref.playing = true;

            source.onended = () => {
              if (this.repeat) {
                this.play(item);
              } else if (this.random) {
                this.play_random_voice();
              } else {
                smooth_end();
                clear_timer();
                this.now_playing.delete(source);
              }
            };

            this.$bus.$on('abort_play', () => {
              try {
                source.stop(0);
              } catch (e) {
                // 在已停止或未开始时调用 stop 可能抛错，此处仅记录并继续
                if (process.client) {
                  console.debug('Reverse source.stop error:', e);
                }
              }
              smooth_end();
              clear_timer();
              this.now_playing.delete(source);
            });

            if ('mediaSession' in navigator) {
              const metadata = {
                title: item.description[this.current_locale],
                artist: this.$t('control.full_name'),
                album: this.$t('site.title'),
                artwork: [{ src: 'img/media-cover.jpg', sizes: '128x128', type: 'image/jpeg' }]
              };
              navigator.mediaSession.metadata = new window.MediaMetadata(metadata);
              navigator.mediaSession.playbackState = 'playing';
            }
          } catch (err) {
            console.error('Reverse playback failed:', err);
          }
          return;
        }
      }

      let audio = new Audio(this.voice_host + item.path);
      // 文件路径
      audio.load(); //This could fix iOS playing bug
      if ('mediaSession' in navigator) {
        const metadata = {
          title: this.overlap ? this.$t('control.overlap_title') : item.description[this.current_locale],
          // 判断是否为重叠播放，是则在浏览器媒体控件显示 overlap_title，否则显示播放的文件名称
          artist: this.$t('control.full_name'),
          album: this.$t('site.title'),
          artwork: [{ src: 'img/media-cover.jpg', sizes: '128x128', type: 'image/jpeg' }]
          // 媒体控件显示的专辑图片
        };
        navigator.mediaSession.metadata = new window.MediaMetadata(metadata);
        navigator.mediaSession.playbackState = 'playing';
      }
      audio.addEventListener('canplay', () => {
        audio.volume = 1;
        audio.play();
        this.now_playing.add(audio);
        this.send_google_event(item);
        ref.playing = true;
        setup_timer();
      });
      audio.addEventListener('ended', () => {
        if (this.repeat) {
          audio.play();
          this.now_playing.add(audio);
          this.send_google_event(item);
          ref.playing = true;
          setup_timer();
        } else if (this.random) {
          this.play_random_voice();
        } else {
          smooth_end();
          clear_timer();
          this.now_playing.delete(audio);
        }
      });
      audio.addEventListener('pause', () => {
        // console.log(item.name, 'paused');
        smooth_end();
        //if (!this.repeat) {
        clear_timer();
        this.now_playing.delete(audio);
        //}
        if ('mediaSession' in navigator) {
          navigator.mediaSession.playbackState = 'paused';
        }
      });
      this.$bus.$on('abort_play', () => {
        audio.pause();
        smooth_end();
        clear_timer();
        this.now_playing.delete(audio);
        delete this.audio;
      });
    },
    get_random_int(max) {
      return Math.floor(Math.random() * Math.floor(max));
    },
    play_random_voice() {
      let random_list = this.groups[this.get_random_int(this.groups.length)];
      this.play(random_list.voice_list[this.get_random_int(random_list.voice_list.length)]);
    },
    stop_all() {
      console.log('stop-all');
      this.$bus.$emit('abort_play');
    }
  },
  head() {
    return {
      title: this.$t('site.title')
    };
  }
};
</script>

<template>
  <v-app>
    <!-- 侧边栏 -->
    <v-navigation-drawer v-model="drawer" :mobile-breakpoint="1024" class="elevation-3" fixed app>
      <!--
      <template v-slot:img>
        <v-img />
        <div
          class="nav-darwer-overlay"
          :class="$vuetify.theme.dark ? 'nav-darwer-overlay-dark' : 'nav-darwer-overlay-light'"
        />
      </template>
      -->
      <v-list style="padding-top: 0;">
        <v-list-item to="/" router exact dense>
          <v-list-item-action>
            <v-icon>{{ icons.home }}</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>{{ $t('site.index') }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-item
          v-for="(item, i) in $t('navigator.items')"
          :key="i"
          :href="item.to"
          target="_blank"
          rel="noreferrer"
          router
          exact
          dense
        >
          <v-list-item-action>
            <v-img :src="item.icon" style="width: 24px;" />
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title v-text="item.title" />
          </v-list-item-content>
        </v-list-item>
      </v-list>
      <v-divider />
      <v-list>
        <!-- 友情链接 -->
        <v-list-item dense to="/links" router exact>
          <v-list-item-action>
            <v-icon>{{ icons.link_variant }}</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>{{ $t('site.links') }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <!-- 关于 -->
        <v-list-item dense to="/about" router exact>
          <v-list-item-action>
            <v-icon>{{ icons.information_outline }}</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>{{ $t('site.about') }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
    <!-- 顶栏 -->
    <v-app-bar dense class="primary white--text" app>
      <!-- 打开/收起侧边栏 -->
      <v-app-bar-nav-icon class="white--text" @click.stop="drawer = !drawer" />
      <!-- 网站名称 -->
      <v-toolbar-title v-text="$t('site.title')" />
      <v-img src="icon.svg" style="max-width: 24px; margin-left: 6px;" />
      <v-spacer />
      <!-- 切换深色模式 -->
      <v-tooltip bottom>
        <template v-slot:activator="{ on }">
          <v-btn icon class="white--text" @click="switch_dark()" v-on="on">
            <v-icon>{{ icons.brightness }}</v-icon>
          </v-btn>
        </template>
        <span>{{ $t('site.switch_dark_mode') }}</span>
      </v-tooltip>
      <!-- 切换语言 -->
      <!-- <v-menu offset-y>
        <template v-slot:activator="{ on: menu }">
          <v-tooltip bottom>
            <template v-slot:activator="{ on: tooltip }">
              <v-btn icon class="white--text" v-on="{ ...tooltip, ...menu }">
                <v-icon>{{ icons.translate }}</v-icon>
              </v-btn>
            </template>
            <span>{{ $t('site.switch_language') }}</span>
          </v-tooltip>
        </template>
        <v-list>
          <v-list-item id="lang-switch-zh" @click="switch_lang('zh')">
            <v-list-item-title>简体中文</v-list-item-title>
          </v-list-item>
          <v-list-item id="lang-switch-ja" @click="switch_lang('ja')">
            <v-list-item-title>日本語</v-list-item-title>
          </v-list-item>
          <v-list-item id="lang-switch-en" @click="switch_lang('en')">
            <v-list-item-title>English</v-list-item-title>
          </v-list-item>
        </v-list>
      </v-menu> -->
    </v-app-bar>
    <v-main>
      <v-container class="page">
        <nuxt />
      </v-container>
      <!-- 页脚 -->
      <v-footer :fixed="false" class="footer">
        <div>
          <div style="vertical-align: middle;">
            <span>&copy; {{ new Date().getFullYear() }} </span>
            <span>
              <a :href="$t('site.footer.author_link_bili')" target="_blank" rel="noreferrer">
                {{ $t('site.footer.author') }}
              </a>
            </span>
            <v-btn
              icon
              :href="$t('site.footer.repo_link')"
              target="_blank"
              style="vertical-align: middle;"
              rel="noreferrer"
            >
              <v-icon>{{ icons.book }}</v-icon>
            </v-btn>
            <v-btn
              icon
              :href="$t('site.footer.author_link_github')"
              target="_blank"
              style="vertical-align: middle;"
              rel="noreferrer"
            >
              <v-icon>{{ icons.github }}</v-icon>
            </v-btn>
          </div>
          <div>
            <p>{{ $t('site.footer.content') }}</p>
          </div>
        </div>
      </v-footer>
    </v-main>
  </v-app>
</template>

<style lang="scss">
$blur-function: blur(3px);
.v-application {
  html[lang='en'] & {
    font-family: $en-body-fonts;
  }
  html[lang='ja'] & {
    font-family: $ja-body-fonts;
  }
  html[lang='zh'] & {
    font-family: $zh-body-fonts;
  }
}

.nav-drawer-img {
  width: auto;
  height: 100%;
  /*
  -webkit-filter: $blur-function;
  -moz-filter: $blur-function;
  -ms-filter: $blur-function;
  filter: $blur-function;
   */
}
.nav-darwer-overlay {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  height: 100%;
  width: 100%;
}
.nav-darwer-overlay-light {
  background-color: rgba(255, 255, 255, 0.8);
}
.nav-darwer-overlay-dark {
  background-color: rgba(0, 0, 0, 0.8);
}
a {
  text-decoration: none;
}
.page {
  box-sizing: border-box;
  min-height: 100%;
  padding-bottom: 150px;
}
.footer {
  margin-top: -88px;
}
</style>

<script>
import {
  mdiHome,
  mdiLinkVariant,
  mdiBrightness2,
  mdiTranslate,
  mdiGithub,
  mdiBook,
  mdiNewspaper,
  mdiAlphaBBox,
  mdiInformationOutline,
  mdiCommentAccountOutline
} from '@mdi/js';
//import themes from '../assets/themes.js';

export default {
  data() {
    return {
      icons: {
        home: mdiHome,
        link_variant: mdiLinkVariant,
        information_outline: mdiInformationOutline,
        brightness: mdiBrightness2,
        translate: mdiTranslate,
        github: mdiGithub,
        book: mdiBook,
        newspaper: mdiNewspaper,
        alpha_b_box: mdiAlphaBBox,
        account: mdiCommentAccountOutline
      },
      drawer: false,
      fixed: false
    };
  },
  computed: {
    current_locale() {
      return this.$i18n.locale;
    }
  },
  mounted() {
    this.$vuetify.theme.dark = this.$store.state.dark === 'true';
    if (window.innerWidth >= 1024) {
      this.drawer = true;
    }
  },
  methods: {
    switch_dark() {
      this.$vuetify.theme.dark = !this.$vuetify.theme.dark;
      this.$store.commit('SET_DARK', this.$vuetify.theme.dark);
    },
    switch_lang(lang) {
      console.log('switching to ' + lang);
      this.$store.commit('SET_LANG', lang);
      this.$i18n.locale = lang;
    }
  },
  head() {
    return {
      htmlAttrs: {
        lang: this.current_locale
      }
    };
  }
};
</script>

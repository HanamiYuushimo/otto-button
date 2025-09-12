<template>
  <v-layout column justify-center align-center app>
    <v-card class="pa-4">
      <v-card-title class="headline">
        {{ $t('site.links') || 'Links' }}
      </v-card-title>
      <v-card-text>Coming soon.</v-card-text>
    </v-card>
  </v-layout>
</template>
<script>
export default {
  head() {
    return {
      title: this.$t && this.$t('site.links') ? this.$t('site.links') + ' - ' + this.$t('site.title') : 'Links'
    };
  }
};
</script>
<style scoped></style>

<template>
  <div id="captive-preview" class="ui container">
    <div class="ui center aligned">
      <h2>{{extractValue('captive_2_title')}}</h2>
      <img :src="extractValue('captive_3_logo')" class="ui centered image tiny">
    </div>
    <div class="ui segments route-container">
      <div class="ui segment">
        <div>
          <div>
            <h3>{{extractValue('captive_4_subtitle')}}</h3>
            <img class="ui centered small image" :src="extractValue('captive_5_banner')">
            <p v-html="extractValue('captive_6_description')">
            </p>
            <div class="ui divider"></div>
            <router-link to='/login' class="big ui green button">
              {{ $t("hotspot.start_navigate") }}
            </router-link>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
  export default {
    name: 'AccountAction',
    props: ['obj'],
    data() {

      return {}
    },
    methods: {
      extractValue(key) {
        for (var p in this.obj) {
          var pref = this.obj[p]
          if (pref.key == key) {
            return pref.value
          }
        }
      }
    }
  }

</script>

<style scoped>
  #captive-preview {
    -webkit-font-smoothing: antialiased !important;
    -moz-osx-font-smoothing: grayscale !important;
    text-align: center !important;
    color: #eff7fc !important;
    margin-top: 30px !important;
    margin-bottom: 30px !important;
    padding: 25px;
  }

  h3 {
    color: #5a5a5a !important;
    font-size: 16px !important;
  }

  p {
    color: #5a5a5a !important;
    margin: 0 0 1em;
    line-height: 1.4285em;
    font-size: 12px;
  }

  img {
    margin-top: 20px !important;
    margin-bottom: 20px !important;
  }

  .ui.button {
    font-size: 15px;
  }

  .ui.small.image {
    width: 450px !important;
  }

</style>
<style scoped src="semantic-ui-offline/semantic.min.css"></style>

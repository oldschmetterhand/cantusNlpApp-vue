<template>
  <div id="vue_app" :style="maximizedStyle" :class="animClass">
    <app-the-main-grid></app-the-main-grid>
  </div>
</template>

<script>
import TheMainGrid from './components/TheMainGrid';
import {EventBus} from "./main";
export default {
  name: 'app',
  components: {
    appTheMainGrid: TheMainGrid
  },
  data () {
    return {
      searchResult: [],
      maximizedStyle: undefined,
      animClass:""
    }
  },
  created(){
    EventBus.$on('resultReceived',data=>{
      this.searchResult = data
    });
  }
}
</script>

<style lang="scss" scoped>
  #vue_app /deep/ {
    //old deep import of bootstrap 4 -> not used any longer
    /*@import "~bootstrap/scss/functions";*/
    /*@import "~bootstrap/scss/variables";*/
    /*@import "~bootstrap/scss/mixins";*/
    /*@import "~bootstrap/scss/card";*/
  }

</style>

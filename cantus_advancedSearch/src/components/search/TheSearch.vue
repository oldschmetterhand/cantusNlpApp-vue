<template>
  <div>
    <app-the-search-bar
      @routed_startIncipitSearch="searchIncipit()"
      @routed_incipitParamsUpdated="saveIncipitSearchData($event)"
      @demandFullTextSearch="doFullTextSearch($event)"
      :searchBarEnlarged="searchBarEnlarged"
    ></app-the-search-bar>
  </div>
</template>

<script>
  //TODO reduce param to only work with incipitSearch! -> I think i only need the IncipitArray as vueX store variable!
  // and of course search failed/succeeded states.

  import {incipitDummyDataMixin} from './../../mixins/incipitDummyData' //TODO remove when not longer needed
  //import {fullTextDummyDataMixin} from '../../mixins/fullTextDummyData'

  import TheSearchBar from './TheSearchBar'

  let incipitSearchTimer;

  export default {
    name: "TheSearch",
    mixins: [incipitDummyDataMixin],  //TODO remove when no longer needed
    components: {
      appTheSearchBar: TheSearchBar
    },
    props: {
      searchBarEnlarged: {
        type: Boolean
      }
    },
    data(){
      return {
        // settings data
        urlStart:'https://gams.uni-graz.at/archive/objects',
        useDummyData:false,

        // used class variables
        incipitSearchHistory:[],  //not used here, defined in vueX
        incipitSearch:{   //this object is pushed into the incipitSearchHistory-Array -> to vueX -> given to TheResult.vue
          response:'',
          query:'',
          searchParams:{
            chosenLO: {text:'',value:'o:cantus.passau.ur'},
            chosenGenre:{text:'RP - Responsorium prolixum',value:'RP'},
            chosenHora: {text:'',value:'M'},
            chosenTimeFrame:{
              text:'',
              value:''
            }
          }
        }
      }
    },
    computed: {
      incipitQuery(val){
        if(val===undefined)return;
        let queryStart = `${this.urlStart}/query:cantus.genres/methods/sdef:Query/getJSON?params=`
        let queryParams =
          `$1|<https://gams.uni-graz.at/${this.incipitSearch.searchParams.chosenLO.value}>;` +
          `$2|${this.incipitSearch.searchParams.chosenGenre.value};` +
          `$3|${this.incipitSearch.searchParams.chosenHora.value};` +
          `$4|${this.incipitSearch.searchParams.chosenTimeFrame.value}`

        return this.encodeUri(queryStart + queryParams)
      }
    },
    methods: {
      saveIncipitSearchData(data){
        console.debug('saveIncipitSearchData...')
        this.incipitSearch.searchParams = data
      },
      searchIncipit(){
        this.$store.dispatch('incipit_setSearchFailedAction',false)
        this.$store.dispatch('incipit_markOngoingSearchAction', true)

        //only accessed when no real ajax, just test data should be applied
        if(this.useDummyData){
          let response = {}
          response.body = this.testIncipitSearch()
          this.incipitSearch.response = response

          let copy = Object.assign({},this.incipitSearch)
          this.$store.dispatch('incipit_pushOntoSearchHistoryAction',copy)
          this.$store.dispatch('incipit_markOngoingSearchAction', false)
          return;
        }

        //if in 10 secs no response fail
        incipitSearchTimer = setTimeout(_=>{
          this.$store.dispatch('incipit_setSearchFailedAction',true)
          this.runningRequest.abort()
        },10000)

        //let vuexSearchQuery = this.$store.getters.search_getCurSearchQuery
        this.$http.get(this.incipitQuery, {
          //vue resource specific: using above to cancel current request
          before(request){
            this.runningRequest = request
          }
        }).then(response => {
          this.incipitSearch.response = response
          let copy = Object.assign({},this.incipitSearch)
          this.$store.dispatch('incipit_pushOntoSearchHistoryAction',copy)

          clearTimeout(incipitSearchTimer)
          console.info('Response received:')
          console.info(response)
        },err => {
          this.$store.dispatch('incipit_setSearchFailedAction',true)
          clearTimeout(incipitSearchTimer)
        }).finally(_=>{
          this.$store.dispatch('incipit_markOngoingSearchAction', false)
        });
      },
      encodeUri(uri){
        return encodeURI(uri)
      },
      doFullTextSearch(queryObject){
        console.info("FUllTextSearch with parameters: ")
        console.info(queryObject.query)
        console.info(queryObject.searchParams)

        let fullTextQuery = queryObject.query
        let searchParams = queryObject.searchParams

        this.$store.dispatch('fullText_setSearchParamsAction',searchParams)
        //this.$store.dispatch('fullText_setSearchResultAction',this.pageAsString) //testData from mixin

        /*this.$http.get('', {
        }).then(response => {
          this.incipitSearch.response = response
          let copy = Object.assign({},this.incipitSearch)
          this.$store.dispatch('incipit_pushOntoSearchHistoryAction',copy)

          clearTimeout(incipitSearchTimer)
          console.info('Response received:')
          console.info(response)
        },err => {
          this.$store.dispatch('incipit_setSearchFailedAction',true)
          clearTimeout(incipitSearchTimer)
        }).finally(_=>{
          this.$store.dispatch('incipit_markOngoingSearchAction', false)
        });*/


      }
    }
  }
</script>

<style scoped>

</style>

<template>
  <div>

    <div class="TheFullTextSearch_searchContstraintsContainer">
    <h4>Erweiterte Volltextsuche</h4>
      <br>
      <p>
        Bei der erweiterten Volltextsuche, bei der sowohl Incipits als auch Rubriken berücksichtigt werden, muss (im Gegensatz zur einfachen) der Suchbegriff exakt eingegeben werden. In den Auswahllisten kann optional auf einen vordefinierten Zeitraum oder eine bestimmte Hora (oder auf beides zugleich) eingeschränkt werden. Klicken Sie dazu auf die jeweiligen Buttons. Ein erneutes Anklicken hebt die jeweilige Suchbedingung auf. 
      </p>
      <p>
      </p>
      <p>
        Der Suchbutton am unteren Ende erscheint grün, sobald die Anforderungen für einen Suchvorgang ausreichend sind. Dafür muss zumindest ein Buchstabe im Suchfeld eingetragen worden sein. Die Suchergebnisse werden in einem neuen Browser-Tab in der Editionsansicht angezeigt. Die Suchanfrage bleibt im ursprünglichen Tab erhalten.
      </p>
      <br>



      <app-full-text-input
        v-model="searchText"
        class="TheFullTextSearch_inputs"
      ></app-full-text-input>
      <br>
      <br>

      <app-time-frame-or-feast-select
        v-model="chosenTimeFrame"
        :searchFieldsShown="false"
        @searchFieldToggled="timeFrameSearchActive = $event"
        class="TheFullTextSearch_inputs"
      ></app-time-frame-or-feast-select>

      <div>
        <br>
        <app-horae-select
          v-model="chosenHora"
          :search-toggable="true"
          :toggleOptions="{mode:'top'}"
          @searchFieldToggled="horaeSearchActive = $event"
          class="TheFullTextSearch_inputs"
        ></app-horae-select>
      </div>

      <br>
      <br>
      <!--<p>FullTextUrl:</p>
      <p>{{fullTextUrl}}</p>-->
      <app-v-search-button
        :buttonEnabled="allowSearch"
        :text="'Suche Starten'"
        @click="navigateToQuery"
      ></app-v-search-button>
      <br>
      <br>
      <hr>
    </div>
  </div>
</template>

<script>
  //TODO add incipit query etc.
  //TODO think about: better to let user add further fields???? --> might be much easier to implement!
  // -> because then -> as field is added the query will behave differently!

  import VTimeFrameSelect from './dropdowns/VTimeFrameSelect'
  import VHoraeSelect from './dropdowns/VHoraeSelect'
  import VTimeFrameOrFeastSelect from './dropdowns/VTimeFrameOrFeastSelect'
  import VFullTextInput from './dropdowns/VFullTextInput'
  import VSearchButton from './buttons/VSearchButton'

  export default {
    name: "TheFullTextSearch",
    components: {
      appVTimeFrameSelect:VTimeFrameSelect,
      appHoraeSelect:VHoraeSelect,
      appTimeFrameOrFeastSelect: VTimeFrameOrFeastSelect,
      appFullTextInput: VFullTextInput,
      appVSearchButton:VSearchButton
    },
    data(){
      return {
        urlStart:`https://gams.uni-graz.at/archive/objects/`,

        searchText:{text:'',value:''},
        chosenTimeFrame:{
          text:'',
          value:''
        },
        chosenHora:{text:'',value:''},

        horaeSearchActive:false,
        timeFrameSearchActive:false,

        queryObjects: {
          onlyTimeFrame:'query:cantus.fulladv-feast',
          onlyHorae:'query:cantus.fulladv-hora',
          timeFrameAndHorae:'query:cantus.fulladv-hf',
          standardFullText:'query:cantus.phrase'
        }


      }
    },
    computed: {
      fullTextUrl(){
        if(this.timeFrameSearchActive && !this.horaeSearchActive){
          let queryStart = `${this.urlStart + this.queryObjects.onlyTimeFrame}/methods/sdef:Query/get?params=`
          let params = `$4|${this.chosenTimeFrame.value};$5|${this.searchText.value.toLowerCase()};$6|${this.chosenTimeFrame.text}`
          return queryStart + encodeURIComponent(params)
        }

        if(this.timeFrameSearchActive && this.horaeSearchActive){
          let queryStart = `${this.urlStart + this.queryObjects.timeFrameAndHorae}/methods/sdef:Query/get?params=`
          let params = `$3|${this.chosenHora.value};$4|${this.chosenTimeFrame.value};$5|${this.searchText.value.toLowerCase()};$6|${this.chosenTimeFrame.text};$7|${this.chosenHora.text}`
          return queryStart + encodeURIComponent(params)
        }

        if(!this.timeFrameSearchActive && this.horaeSearchActive){
          let queryStart = `${this.urlStart + this.queryObjects.onlyHorae}/methods/sdef:Query/get?params=`
          let params = `$3|${this.chosenHora.value};$5|${this.searchText.value.toLowerCase()};$7|${this.chosenHora.text}`
          return queryStart + encodeURIComponent(params)

        }

        if(!this.timeFrameSearchActive && !this.horaeSearchActive){
          let queryStart = `${this.urlStart + this.queryObjects.standardFullText}/methods/sdef:Query/get?params=`
          let params = `$5|${this.searchText.value.toLowerCase()}`
          return queryStart + encodeURIComponent(params)
        }

      },
      allowSearch(){
        return !(this.searchText.value==='')
      }
    },
    methods:{
      navigateToQuery(){

        if(this.searchText.value==='')return window.alert('Bitte geben Sie einen Suchtext im Feld "Volltextsuche" an.')
        if(this.searchText.value.toLowerCase().replace(/ /g,'')==='chucknorris'){
          window.alert("Seriously? Chuck Norris in a Liber Ordinarius?")
          return window.location = "https://web.archive.org/web/20161125105323/http://www.chucknorrisfacts.com/"
        }
        //this.$emit('demandFullTextSearch',{query: this.fullTextUrl, searchParams:{chosenText:this.searchText, chosenFeasts:this.chosenTimeFrame, chosenHora:this.chosenHora}})

        this.$store.dispatch('incipit_markOngoingSearchAction', true)
        window.location = this.fullTextUrl




        //return window.open( this.fullTextUrl)
      }
    }
  }
</script>

<style scoped lang="scss">
  @import "../../../scss/globalVariables/globalVariables";

  .TheFullTextSearch_searchContstraintsContainer {
    /*padding:4em 2em 2em 2em;*/
    /*border: lightgrey solid .15em;*/
    margin:1em;
    border-radius: .25em;
    background-color: $primaryColor;
  }

  h4 {
    text-decoration: underline;
  }

  hr {
    border: none;
    padding:.25em;
    background-color: lightgrey;
    border-radius: 5em;
  }

  .TheFullTextSearch_inputs {
    width:65%;
  }

  p {
    text-align: justify;
  }

  button {
    color:black;
  }

</style>

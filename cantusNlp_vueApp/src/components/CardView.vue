<template>
  <div id="cardView_container">
    <div>
      <div
        class="cantusNlp_iconHolder"
        data-balloon="Intro Ansichtsleiste"
        data-balloon-pos="left">
        <i
          @click="notify('Intro Ansichtsleiste'), startVueTour()"
          class="fas fa-info-circle"
        ></i>
      </div>
    </div>
    <hr>
    <div>
      <div
        id="v-step-1000"
        class="cantusNlp_iconHolder"
        data-balloon="Ansichten vergrößern"
        data-balloon-pos="left">
        <i
          class="fas fa-th-large"
          @click="notify('Ansichten vergrößert');resizeAllCards('100%', '100vh');markActive($event,'top')">
        </i>
      </div>
    </div>
    <div>
      <div
        id="v-step-1001"
        class="cantusNlp_iconHolder"
        data-balloon="Ansichten verkleinern"
        data-balloon-pos="left">
        <i
          class="fas fa-th"
          @click="notify('Ansichten verkleinert'); resizeAllCards('400px', '600px');markActive($event,'top')">
        </i>
      </div>
    </div>
    <hr>
    <v-tour name="cardViewTour" :steps="steps">
      <template slot-scope="tour">
        <transition name="fade">
          <v-step
            v-if="tour.currentStep === index"
            v-for="(step, index) of tour.steps"
            :key="index"
            :step="step"
            :previous-step="tour.previousStep"
            :next-step="tour.nextStep"
            :stop="tour.stop"
            :is-first="tour.isFirst"
            :is-last="tour.isLast"
            :labels="tour.labels"
          >
            <template>
              <div slot="actions">
                <hr>
                <button @click="tour.previousStep" class="btn">Zurück</button>
                <button v-if="index<steps.length-1" @click="tour.nextStep" class="btn">Weiter</button>
                <button @click="tour.stop" class="btn">Beenden</button>
              </div>
            </template>
          </v-step>
        </transition>
      </template>
    </v-tour>
  </div>
</template>

<script>
  import {EventBus} from "../main";
  import {vueNotifyMixin} from "../mixins/vueNotifyMixin";
  import {iconMethodsMixin} from "../mixins/iconMethodsMixin";

  export default {
    name: "CardView",
    mixins: [vueNotifyMixin, iconMethodsMixin],
    data() {
      return {
        steps: [
          {
            target: '#v-step-1000',  // We're using document.querySelector() under the hood
            content: `Im <strong>Ansicht-Vergrößerungswerkzeug</strong>werden alle Karten auf eine größere Ansicht geschalten...`,
            offset: document.documentElement.scrollTop - 150,
            duration: 100,
            params: {
              placement: 'left'
            }
          },
          {
            target: '#v-step-1001',  // We're using document.querySelector() under the hood
            content: `... und via click hier wieder <strong>verkleinert</strong>.`,
            offset: document.documentElement.scrollTop - 150,
            duration: 100,
            params: {
              placement: 'left'
            }
          }
        ]
      }
    },
    methods: {
      resizeAllCards(minWidth, minHeight) {
        let cssSizeObj = {
          "min-width": minWidth,
          "height": minHeight
        }
        EventBus.$emit('resizeCards', cssSizeObj);
      },
      startVueTour() {
        this.$tours['cardViewTour'].start();
      },
      openInNewTab(url) {
        let win = window.open(url, '_blank');
        win.focus();
      }
    }
  }
</script>

<style scoped lang="scss">
  @import "../scss/globalVariables";

  #cardView_container {
    margin-top: 2em;
    @include lg {
      margin-top: 35vh
    }
    margin-bottom: 2em;
    @include lg {
      margin-bottom: 0
    }
    hr {
      color: $blankColor;
      border: .05em $blankColor solid;
      width: 3em;
    }
    .fa-adn {
      transform: rotate(180deg)
    }
    .cantusNlp_iconHolder {
      margin-top: 1em
    }
  }

  /*Styling for the vue tours*/
  #cardView_container .v-step {
    z-index: 9999;
    background-color: $secondaryColor; /*https://color.adobe.com/de/create/color-wheel/?base=2&rule=Compound&selected=3&name=Mein%20Color-Thema&mode=rgb&rgbvalues=0,0,0.5450980392156862,0.24,0.24000000000008187,0.6,0,0.526315789473756,1,1,0.6145833333331439,0.25,0.8,0.32499999999987267,0.07999999999999999&swatchOrder=0,1,2,3,4*/
    color: lightgrey;
    min-width: 20em;
    button {
      color: lightgrey;
      &:hover {
        background-color: $fifthColor;
      }
    }
  }

  /*Animations for vue tours*/
  .fade-enter-active, .fade-leave-active {
    transition: opacity .5s;
  }

  .fade-enter, .fade-leave-to {
    opacity: 0;
  }

</style>

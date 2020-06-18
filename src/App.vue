<template>
<div class="vue-world-map">
  <Map />
</div>
</template>

<script>
import chroma from 'chroma-js';
import Map from './Map';

export default {
  components: { Map },
  watch: {
    countryData() {
      this.renderMapCSS();
    },
  },
  props: {
    countryData: {
      type: Object,
      required: true,
    },
    defaultCountryFillColor: {
      type: String,
      default: '#dadada',
    },
    countryStrokeColor: {
      type: String,
      default: 'black',
    },
  },
  data() {
    return {
      node: document.createElement('style'),
    };
  },
  methods: {
    renderMapCSS() {
      const baseCss = `.vue-world-map .land{fill: \
                       ${this.$props.defaultCountryFillColor}; \
                       stroke:${this.$props.countryStrokeColor};}`

      var happyMin, happyMax, sadMin, sadMax, angryMin,
          angryMax, relaxedMin, relaxedMax;

      // Determine the upper- and lower-bounds of every mood.
      for (let [,[moodIntensity, mood]] of
           Object.entries(this.$props.countryData)) {
        switch (mood) {
          case 0: // Happy.
            if (moodIntensity < happyMin || happyMin === undefined)
                happyMin = moodIntensity;
            if (moodIntensity > happyMax || happyMax === undefined)
                happyMax = moodIntensity;
            break;
          case 1: // Sad.
            if (moodIntensity < sadMin || sadMin === undefined)
                sadMin = moodIntensity;
            if (moodIntensity > sadMax || sadMax === undefined)
                sadMax = moodIntensity;
            break;
          case 2: // Angry.
            if (moodIntensity < angryMin || angryMin === undefined)
                angryMin = moodIntensity;
            if (moodIntensity > angryMax || angryMax === undefined)
                angryMax = moodIntensity;
            break;
          case 3: // Relaxed.
            if (moodIntensity < relaxedMin || relaxedMin === undefined)
                relaxedMin = moodIntensity;
            if (moodIntensity > relaxedMax || relaxedMax === undefined)
                relaxedMax = moodIntensity;
            break;
        }
      }

      const dynamicMapCss = [];
      const happyChroma = chroma.scale(['#fffed4', '#f0ec00']),
            sadChroma = chroma.scale(['#e9e6ff', '#1e00ff']),
            angryChroma = chroma.scale(['#fde2e2', '#d83737']),
            relaxedChroma = chroma.scale(['#e2fae1', '#04e600']);

      // Set the css values of every country.
      for (let [country, [moodIntensity, mood]] of
           Object.entries(this.$props.countryData)) {
        var hex = '#dadada'
        switch (mood) {
          case 0: // Happy.
            var happyScaleValue = 1 / (happyMax - happyMin) *
                                    (moodIntensity - happyMin);
            hex = happyChroma(happyScaleValue).hex();
            break;
          case 1: // Sad.
            var sadScaleValue = 1 / (sadMax - sadMin) *
                                  (moodIntensity - sadMin);
            hex = sadChroma(sadScaleValue).hex();
            break;
          case 2: // Angry.
            var angryScaleValue = 1 / (angryMax - angryMin) *
                                    (moodIntensity - angryMin);
            hex = angryChroma(angryScaleValue).hex();
            break;
          case 3: // Relaxed.
            var relaxedScaleValue = 1 / (relaxedMax - relaxedMin) *
                                      (moodIntensity - relaxedMin);
            hex = relaxedChroma(relaxedScaleValue).hex();
            break;
        }
        dynamicMapCss.push(`.vue-world-map #${country} { fill: ${hex}; }`);
      }

      // Combine the country base colors with the dynamically assigned colors.
      function getCombinedCssString(baseCss, dynamicCss) {
        dynamicCss.push(baseCss);
        return dynamicCss.join(' ');
      }

      this.$data.node.innerHTML = getCombinedCssString(baseCss, dynamicMapCss);
    },
  },
  mounted() {
    document.body.appendChild(this.$data.node);
    this.renderMapCSS();
  },
};
</script>

<style>
.vue-world-map {
  height: 97vh;
  margin-left: auto;
  margin-right: auto;
}

#map-svg {
  height: 98vh;
  margin-left: auto;
  margin-right: auto;
}
</style>

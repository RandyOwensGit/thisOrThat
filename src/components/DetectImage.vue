/**
 * eslint-disable no-console
 */
<template>
  <div>
    <h1>{{ msg }}</h1>
    <div>
        <img class="image" ref="img" :src="require('../assets/images/' + getImgIndex + '.jpg')" />
    </div>
    <div>
        <button class="button" @click="next()" :disabled="disable">Next</button>
    </div>

    <div v-for="pred in predictions" :key="pred.index">{{ pred.label }}: {{ pred.probability.toFixed(0) + '%' }}</div>
    <div v-if="!predictions.length">hmm.....</div>

  </div>
</template>

<script>
import * as cvstfjs from "customvision-tfjs";
import labels from "raw-loader!../../public/models/labels.txt";

export default {
  name: 'app',

  props: {
    msg: String
  },

  data() {
    return {
      image: 1,
      numImages: 6,
      labels: labels,
      model: null,
      predictions: []
    };
  },

  computed: {
    getImgIndex() {
      return this.image.toString();
    },
    disable() {
      if (this.image == this.numImages) {
        return true;
      } else return false;
    },
  },

  async mounted() {
    this.image++;
    // load new model
    this.model = new cvstfjs.ClassificationModel();
    await this.model.loadModelAsync("models/model.json");
    // parse labels
    this.labels = labels.split("\n").map(e => {
      return e.trim();
    });
    // run prediction
    this.predict();
  },

  methods: {
    next() {
      this.image++;
      this.predictions = [];
      setTimeout(this.predict, 500);
    },
    async predict() {
      // execute inference
      let prediction = await this.model.executeAsync(this.$refs.img);
      let label = prediction[0];
      // generate prediction by parsing labels and probability
      this.predictions = label.map((p, i) => {
        return { index: i, label: this.labels[i], probability: p * 100 };
      });
    }
  },

}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
    margin: 40px 0 0;
}
.image {
    min-height: 300px;
    max-height: 300px;
    max-width: 100%;
}
.button {
    width: 200px;
    height: 50px;
    border-radius: 5px;
    background-color: blueviolet;
    color: white;
    font-size: 20pt;
    margin: 10px;
}
.button:disabled,
.button[disabled] {
    border: 1px solid #999999;
    background-color: #cccccc;
    color: #666666;
}
</style>

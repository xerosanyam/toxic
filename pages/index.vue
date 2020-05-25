<template>
  <div
    class="md:flex space-y-4 flex-col md:items-center md:justify-center text-center p-6"
  >
    <div class="w-full md:w-1/2">
      <Flask :loading="loading" />
      <h1 class="title">
        Is it toxic?
      </h1>
      <div
        v-if="!loading"
        class="flex mt-4 space-x-4 justify-center items-baseline"
      >
        <div class="w-full ">
          <input
            id=""
            v-model="usersThoughts"
            placeholder="Your thoughts?"
            class="inline-flex text-2xl bg-white focus:outline-none border border-gray-300 rounded-lg py-2 px-4 block w-full appearance-none leading-normal"
            type="text"
            name=""
            @input="results = []"
            @keydown.enter="classify"
          />
          <span class="text-sm text-gray-700">
            <span class="text-xs">Try:</span>
            <span class="space-x-2">
              <nuxt-link
                v-for="(example, index) in examples"
                :key="index"
                class="underline"
                :to="{ path: '/', query: { q: example } }"
                >{{ example }}</nuxt-link
              >
            </span>
          </span>
        </div>
        <div class="inline-flex">
          <button
            class="flex-shrink-0 text-lg font-medium bg-red-500 hover:bg-red-700 border-red-500 hover:border-red-700 text-sm border-4 text-white py-1 px-2 rounded"
            type="button"
            @click="classify"
          >
            Find out
          </button>
        </div>
      </div>
      <div v-else>
        <div>Loading, please wait. (takes around 2 minutes)</div>
      </div>
    </div>
    <div
      v-show="results.length > 0"
      class="w-full flex justify-center md:w-1/2"
    >
      <div>
        <table>
          <tr
            v-for="(result, index) in results"
            :key="index"
            :class="[
              result.results[0].match === true
                ? 'border-solid border-4 border-red-500'
                : 'border-solid border-4 border-green-500'
            ]"
          >
            <td class="px-4 py-2">{{ result.label | capitalize }}</td>
            <td class="px-4 py-2">{{ result.results[0].match || false }}</td>
          </tr>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import Flask from '~/components/Flask.vue'
const toxicity = require('@tensorflow-models/toxicity')
let model
export default {
  components: {
    Flask
  },
  filters: {
    capitalize(value) {
      if (!value) return ''
      value = value.toString()
      value = value.split('_').join(' ')
      return value.charAt(0).toUpperCase() + value.slice(1)
    }
  },
  data() {
    return {
      loading: true,
      results: [],
      examples: ['Trump is smart', 'You Suck!'],
      usersThoughts: '',
      sampleData: [
        {
          label: 'identity_attack',
          results: [
            {
              probabilities: {
                '0': 0.9659668803215027,
                '1': 0.03403308615088463
              },
              match: false
            }
          ]
        },
        {
          label: 'insult',
          results: [
            {
              probabilities: {
                '0': 0.08124478906393051,
                '1': 0.9187552332878113
              },
              match: true
            }
          ]
        },
        {
          label: 'obscene',
          results: [
            {
              probabilities: {
                '0': 0.39931702613830566,
                '1': 0.6006830334663391
              },
              match: null
            }
          ]
        },
        {
          label: 'severe_toxicity',
          results: [
            {
              probabilities: {
                '0': 0.9970396757125854,
                '1': 0.0029603182338178158
              },
              match: false
            }
          ]
        },
        {
          label: 'sexual_explicit',
          results: [
            {
              probabilities: {
                '0': 0.7053318619728088,
                '1': 0.29466813802719116
              },
              match: null
            }
          ]
        },
        {
          label: 'threat',
          results: [
            {
              probabilities: {
                '0': 0.9106759428977966,
                '1': 0.089323990046978
              },
              match: false
            }
          ]
        },
        {
          label: 'toxicity',
          results: [
            {
              probabilities: {
                '0': 0.03117617778480053,
                '1': 0.9688238501548767
              },
              match: true
            }
          ]
        }
      ]
    }
  },
  created() {
    const vm = this
    console.log(this.$route.query)
    if (this.$route.query && this.$route.query.q) {
      vm.usersThoughts = this.$route.query.q
    }
    // const threshold = 0.9
    // Load the model. Users optionally pass in a threshold and an array of
    // labels to include.
    console.time('load')
    console.log('started')
    toxicity.load().then((modelP) => {
      model = modelP
      vm.loading = false
      console.timeEnd('load')
      console.log('loaded')
      if (this.usersThoughts) {
        vm.classify()
      }
    })
    // this.results = {}
  },
  watchQuery(newQuery, oldQuery) {
    // console.log('watchQuery -> newQuery, oldQuery', newQuery, oldQuery)
    if (newQuery.q) {
      this.usersThoughts = newQuery.q
      if (!this.loading) {
        this.classify()
      }
    }
  },
  methods: {
    classify() {
      const sentences = [this.usersThoughts]
      console.time('clasify')
      model.classify(sentences).then((predictions) => {
        console.log('classify -> predictions', predictions)
        this.$set(this, 'results', predictions)
        this.results = predictions
        console.timeEnd('clasify')
      })
    }
  }
}
</script>

<style>
/* Sample `apply` at-rules with Tailwind CSS
.container {
  @apply min-h-screen flex justify-center items-center text-center mx-auto;
}
*/
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 60px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>

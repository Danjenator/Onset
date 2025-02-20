
<template>
    <div id="app">
        <div class="hero is-success">
          <div class="hero-body">
            <div class="container">
              <h1 class="title">
                Onset
              </h1>
              <h2 class="subtitle">
                A (mostly) linguistically-accurate language evolution simulator
              </h2>
            </div>
          </div>
        </div>
        <div class="main-body container">
        <div v-if="displayModal" v-bind:class="{ 'is-danger': modalType == 'is-danger', 'is-success': modalType == 'is-success'}" class="notification">
          <button @click="displayModal = false" class="delete"></button>
          {{modalMessage}}
        </div>
        <div class="columns">
            <div class="column">
                <div class="card is-fullwidth">
                    <header class="card-header">
                      <p class="card-header-title">Onset</p>
                    </header>
                    <div class="card-content">
                      <div class="content">
                        <p>Onset is a (mostly) linguistically-accurate language evolution simulator. </p>

                        <p><b>Evolve:</b> Generate new evolutions for the list of words to the right, according to the settings.</p>
                        <p><b>Save:</b> Save the current sound change rules under the given name (to browser storage).</p>
                        <p><b>Load:</b> Load sound change rules from the given name and apply them to the current list of words.</p>
                        <a
                          class="button is-fullwidth is-success"
                          v-bind:class="{ 'is-loading': isComputing }"
                          v-on:click="evolve">
                            Evolve
                        </a>
                      </div>
                    </div>
                    <footer class="card-footer">
                        <input v-model="filename" class="input" type="text" style="margin: 5px;" placeholder="filename"></input>

                        <a class="card-footer-item"
                           v-bind:class="{ disabled: !resultsPresent }"
                           v-on:click="saveLocal">
                           Save
                        </a>

                        <a class="card-footer-item"
                           v-on:click="loadLocal">
                           Load
                        </a>

                    </footer>
                </div>
            </div>
            <div class="column">
              <div class="card is-fullwidth">
                <header class="card-header">
                  <p class="card-header-title">Words</p>
                </header>
                  <div class="card-content">
                    <div class="content">
                      <p>Enter a list of words in <a href="https://en.wikipedia.org/wiki/International_Phonetic_Alphabet">IPA</a>, one per line, which will be evolved.</p>
                      <textarea class="textarea" ref="input" rows=20 v-model="wordString"></textarea>
                    </div>
                  </div>
              </div>
            </div>
            <div class="column">
              <div class="card is-fullwidth">
                <header class="card-header">
                  <p class="card-header-title">Settings</p>
                </header>
                <div class="card-content">
                <form>

                  <label class="label">Maximum Generations&nbsp;
                    <i class="fa fa-question-circle">
                      <div class="box content">The maximum number of sound
                      change rules to apply during evolution. If the rule
                      selector goal is achieved early, evolution will
                      end.</div>
                    </i>
                  </label>
                  <p class="control">
                    <input v-model="generations" type="number" class="input">
                  </p>

                  <label class="label">Transcriptions&nbsp;
                    <i class="fa fa-question-circle">
                      <div class="box content">A list of plain text to IPA
                        transcriptions, separated by colons. The transcriptions
                        will be made from top-to-bottom. For example, the pair
                        <em>sh:ʃ</em> will convert all <em>sh</em> occurences
                        in the input to <em>ʃ</em>, evolve the result, and then
                        convert back before displaying the evolved words.</div>
                    </i>
                  </label>
                  <p class="control">
                    <textarea v-model="transcriptionString" class="textarea" rows=5 placeholder="ng:ŋ"></textarea>
                  </p>

                  <label class="label">Direction&nbsp;
                    <i class="fa fa-question-circle">
                      <div class="box content">The direction to evolve the language in. If set to forward, the output will be the input with all changes applied. If set to reverse, the output will be the first state of the language, and applying the given changes will produce the input.</div>

                    </i>
                  </label>
                  <p class="control">
                    <span class="select">
                      <select v-model="direction">
                        <option>Forward</option>
                        <option>Reverse</option>
                      </select>
                    </span>
                  </p>

                  <label class="label">Rule selector&nbsp;
                    <i class="fa fa-question-circle">
                      <div class="box content">The algorithm to use when selecting rules.</div>
                    </i>
                  </label>
                  <p class="control has-addons">
                    <span class="select">
                      <select v-model="optimisationFunction">
                        <option>Minimise</option>
                        <option>Maximise</option>
                      </select>
                      </span>
                    <span class="select">
                      <select v-model="metricFunction" class="is-expanded">
                        <option>weighted phonetic product</option>
                        <option>phonetic product</option>
                        <option>Word Complexity Measure</option>
                        <option>number of syllables</option>
                        <option>number of consonant clusters</option>
                        <option>random value</option>
                      </select>
                    </span>
                  </p>
                </form>
                </div>
              </div>
            </div>
        </div>

    <div v-if="resultsPresent">
    <hr>
    <div class="columns">

      <div class="column">
        <div class="card is-fullwidth">
          <header class="card-header">
            <p class="card-header-title">Applied Rules</p>
          </header>
          <div class="card-content rule-container">
            <div v-for="rule in evolutionRules">
              <rule :rule=rule></rule>
            </div>
          </div>
        </div>
      </div>

      <div class="column">
        <div class="card is-fullwidth">
          <header class="card-header">
            <p class="card-header-title">Evolved Words</p>
          </header>
          <div class="card-content">
            <a class="panel-block" v-for="word in evolvedWords">{{word}}</a>
          </div>
        </div>
      </div>
    </div>
  </div>
  </div>

  <footer class="footer">
    <div class="container">
      <div class="content has-text-centered">
        <p>Onset is built by <a href="https://cadelwatson.com">Cadel Watson</a>. See the source at <a href="https://github.com/kdelwat"><i class="fa fa-github">kdelwat</a>/<a href="https://github.com/kdelwat/Onset">Onset</a>!</p>
      </div>
    </div>
  </footer>
</template>

<script>
import axios from 'axios';

import Rule from './components/Rule';

export default {
  name: 'app',
  components: { Rule },
  data() {
    return {
      wordString: '',
      generations: 5,
      transcriptionString: '',
      direction: 'Forward',
      optimisationFunction: 'Minimise',
      metricFunction: 'weighted phonetic product',
      isComputing: false,
      evolvedWords: [],
      evolutionRules: [],
      filename: '',
      modalMessage: '',
      modalType: 'is-danger',
      displayModal: false,
    };
  },
  computed: {
    resultsPresent() {
      return this.evolvedWords.length >= 1;
    },
  },
  methods: {
    // Call the backend to evolve the given words
    evolve() {
      const parameters = { words: this.wordString,
        generations: this.generations,
        transcriptions: this.transcriptionString,
        direction: this.direction,
        optimisationFunction: this.optimisationFunction,
        metric: this.metricFunction };

      // Signal to the user that something is happening.
      this.displayModal = false;
      this.isComputing = true;

      // Call the Flask API
      axios.get('http://localhost:5000/evolve',
                { params: parameters })
      // Handle a valid response
        .then((response) => {
          // Trigger API error display if necessary
          console.log('response for evolve', response);
          if (response.data.error !== 0) {
            this.showError(response.data.error);
          } else {
            // Otherwise, update data from request result
            this.isComputing = false;
            this.evolvedWords = response.data.words;
            this.evolutionRules = response.data.rules;
          }
        })
      // Handle an invalid response
        .catch(error => this.showError(error));
    },
    // Display an error
    showError(error) {
      console.log(error);

      if (Object.prototype.hasOwnProperty.call(error, 'message')) {
        this.modalMessage = error.message;
      } else {
        this.modalMessage = error;
      }

      this.isComputing = false;
      this.modalType = 'is-danger';
      this.displayModal = true;
    },
    // Display a success message
    showSuccess(error) {
      this.modalMessage = error;
      this.modalType = 'is-success';
      this.displayModal = true;
    },
    // Save the current rules to localStorage, identified by the given filename
    saveLocal() {
      localStorage.setItem(this.filename, JSON.stringify(this.evolutionRules));
      this.showSuccess('Rules saved successfully!');
    },
    // Load rules from localStorage identified by the given filename
    loadLocal() {
      // Load and store the rules
      const loadedRules = JSON.parse(localStorage.getItem(this.filename));

      if (loadedRules === null) {
        this.showError('No rules are saved under that name');
      } else {
        this.evolutionRules = loadedRules;

        const parameters = { words: this.wordString,
          rules: JSON.stringify(this.evolutionRules),
          transcriptions: this.transcriptionString,
          direction: this.direction };

        // Call the Flask API
        axios.get('http://localhost:5000/apply',
                  { params: parameters })
        // Handle a valid response
          .then((response) => {
            // Trigger API error display if necessary
            console.log('response for apply', response);
            if (response.data.error !== 0) {
              this.showError(response.data.error);
            } else {
              // Otherwise, update words from request result
              this.evolvedWords = response.data.words;
              this.showSuccess('Rules loaded and applied!');
            }
          })
        // Handle an invalid response
          .catch(error => this.showError(error));
      }
    },
  },
};
</script>

<style>
.disabled {
    opacity: 0.5;
}

.fa {
  vertical-align: baseline;
}

i {
    position: relative;
}

i > div {
    display: none;
    position: absolute;
    left: -140px;
    bottom: 110%;

    z-index: 2;
    width: 300px;
    text-align: left;
}

i:hover > div {
    display: block;
}

.box {
  display: none;
  padding: 10px;
}

.main-body {
  margin-top: 50px;
}

.rule-container {
    padding: 0;
}
</style>

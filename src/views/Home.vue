<template>
  <!-- Header -->
  <b-container class="home">
    <b-row class="text-left mt-3">
      <b-col>
        <h3>Pressupostos App</h3>
        <hr />
      </b-col>
    </b-row>

    <b-row>
      <!-- Services select form -->
      <b-col class="form" @submit="saveQuote">
        <b-form>
          <b-form-group v-slot="{ ariaDescribedby }">
            <b-row
              align-h="start"
              v-for="(option, index) in options"
              :key="option.id"
            >
              <b-col class="d-flex justify-content-start">
                <b-form-checkbox
                  v-model="selected"
                  @change="whichIs(index)"
                  :value="index"
                  :aria-describedby="ariaDescribedby"
                  name="flavour-3a"
                >
                  {{ option.text }}
                </b-form-checkbox>
              </b-col>
              <b-row align-h="start" v-if="option.panel">
                <b-col>
                  <transition name="movin">
                    <ThePanel @extrasPanel="dataPanel" />
                  </transition>
                </b-col>
              </b-row>
            </b-row>
          </b-form-group>
          <b-row my-2 class="total">
            <b-col>
              <div class="total_final">Preu: {{ total }}</div>
            </b-col>
          </b-row>
          <b-row class="my-2">
            <b-col class="d-flex justify-content-start">
              Vols guardar el teu pressupost?
            </b-col>
          </b-row>
          <b-row>
            <b-col>
              <b-row v-for="(formQuote, index) in formQuotes" :key="index">
                <label for="formQuote">{{ formQuote }}</label>
                <b-form-input
                  type="text"
                  v-model="quoteData[index]"
                ></b-form-input>
              </b-row>
            </b-col>
          </b-row>
          <b-row align-h="end">
            <b-button
              type="submit"
              class="my-3 btn btn-secondary"
              variant="secondary"
              size="sm"
              aria-required="required"
            >
              Guardar pressupost
            </b-button>
          </b-row>
        </b-form>
        <b-row class="text-left">
          <b-col>
            <GoBack />
          </b-col>
        </b-row>
      </b-col>
      <b-col class="quotes">
        <the-quotes :dataQuotes="quote" />
      </b-col>
    </b-row>
  </b-container>
</template>

<script>
// @ is an alias to /src
import ThePanel from "../components/ThePanel.vue";
import GoBack from "@/components/GoBack.vue";
import TheQuotes from "@/components/TheQuotes.vue";

export default {
  name: "Home",
  components: {
    ThePanel,
    GoBack,
    TheQuotes,
  },
  data() {
    return {
      selected: [], // Store the index of selected fields
      options: [
        // Data for select form
        { id: 1, text: "Una pàgina web (500 €)", value: 500, panel: false },
        {
          id: 2,
          text: "Una consultoria SEO (300 €)",
          value: 300,
          panel: false,
        },
        {
          id: 3,
          text: "Una campanya de Google Ads (200 €)",
          value: 200,
          panel: false,
        },
      ],
      pages: 0, // Data from The Panel component
      languages: 0, // Data from The Panel component
      formQuotes: ["Nom del pressupost", "Usuari"], // Data for input labels
      quoteData: [], // Data from input form
      quote: [], // Array that stores the saved quotes. Each object represents one quote
    };
  },
  mounted() {
    if (this.$route.query) {
      this.languages = this.$route.query.lan;
      this.pages = this.$route.query.pag;

      if (this.$route.query.opcions) {
        if (this.$route.query.opcions.length === 1) {
          this.selected.push(this.$route.query.opcions);
        } else {
          this.$route.query.opcions.forEach((element) =>
            this.selected.push(element)
          );
        }
        if (this.selected.includes("0")) {
          this.whichIs(0);
        }
      }
    }
  },
  computed: {
    total() {
      if (this.selected.length != 0) {
        let values = [];
        this.selected.forEach((index) =>
          values.push(this.options[index].value)
        );
        return (
          values.reduce((total, amount) => total + amount) +
          this.pages * this.languages * 30
        );
      } else {
        return 0;
      }
    },
    query() {
      return {
        pag: this.pages,
        lan: this.languages,
        opcions: this.selected,
      };
    },
  },
  watch: {
    query: function () {
      this.$router.replace({ query: this.query }).catch(() => {});
    },
  },
  methods: {
    dataPanel(pages, languages) {
      this.pages = pages;
      this.languages = languages;
    },
    whichIs: function (index) {
      if (index === 0 && this.options[0].panel === false) {
        this.options[0].panel = true;
        this.pages = 1;
        this.languages = 1;
        return;
      }
      if (index === 0 && this.options[0].panel === true) {
        this.options[0].panel = false;
        this.pages = 0;
        this.languages = 0;
      }
    },
    saveQuote(event) {
      event.preventDefault();
      if (this.isValidated()) {
        let services = [];
        this.selected.forEach((index) =>
          services.push(this.options[index].text)
        );
        let quote = {
          quote: this.quoteData[0],
          user: this.quoteData[1],
          choose: services,
          total: this.total,
          date: new Date(),
        };
        this.quote.push(quote);
        this.resetForm();
      }
    },
    isValidated() {
      if (
        !this.selected.length == 0 &&
        this.quoteData.length == 2 &&
        !this.quote.includes(this.quoteData[0])
      ) {
        return true;
      } else {
        return false;
      }
    },
    resetForm() {
      this.selected = [];
      this.quoteData = [];
      this.languages = 1;
      this.pages = 1;
      if (this.options[0].panel == true) {
        this.options[0].panel = false;
      }
    },
  },
};
</script>
<style lang="css"></style>

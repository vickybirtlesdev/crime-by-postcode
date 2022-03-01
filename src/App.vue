<template>
  <div class="header-bg">
    <div class="container">
      <div class="row">
        <div class="header-content-bg col col-md-6 offset-md-3">
          <HeaderMain title="Crime by Postcode" />
          <div class="postcode-search">
            <p class="text-center">Search for crimes by postcode below</p>
            <form @submit.prevent="searchSubmit">
              <label for="txtPostcode" class="form-label">Postcode</label>
              <input
                type="text"
                v-model.trim="v$.searchedPostcode.$model"
                class="form-control"
                :class="{ 'is-invalid': v$.searchedPostcode.$error }"
                placeholder="Postcode"
                aria-label="Postcode"
                aria-describedby="btnPostcode"
                id="txtPostcode"
                @mousedown="resetData"
              />
              <div id="postcodeHelp" class="form-text mb-3">
                for example BD17 6DN
              </div>
              <p
                v-for="error of v$.$errors"
                :key="error.$uid"
                class="invalid-field"
              >
                Please enter a valid Postcode.
              </p>
              <div class="text-center">
                <button
                  class="input-group-text btn btn-primary"
                  :disabled="v$.$invalid"
                  id="btnPostcode"
                  type="submit"
                  @click="searchSubmit"
                >
                  Search
                </button>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div id="main">
    <div class="inner">
      <div class="container">
        <div class="results-container py-5">
          <p v-if="this.formSubmitted && !this.errorAtEnd">
            Your search for crime in {{ this.searchedPostcode }} returned
            {{ this.crimes.length }} results
          </p>
          <p class="invalid-results text-center" v-if="this.errorAtEnd">
            Sorry, something went wrong. Please try again.
          </p>
          <table class="results table mb-5" v-if="this.crimes.length > 0">
            <thead>
              <tr>
                <th><strong>Nature of crime</strong></th>
                <th><strong>Outcome of result:</strong></th>
                <th><strong> Date of Outcome:</strong></th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="crime in crimes" :key="crime.id">
                <td>{{ crime.category.replace(/-/g, " ") }}</td>
                <td>{{ crime.outcome_status.category }}</td>
                <td>{{ crime.outcome_status.date }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
  <FooterMain />
</template>

<script>
import axios from "axios";
import HeaderMain from "./components/HeaderMain.vue";
import FooterMain from "./components/FooterMain.vue";
import useVuelidate from "@vuelidate/core";
import { required, helpers } from "@vuelidate/validators";

const postcode = helpers.regex(/^[a-z]{1,2}\d[a-z\d]?\s*\d[a-z]{2}$/i);

export default {
  name: "App",
  components: {
    HeaderMain,
    FooterMain,
  },
  setup() {
    return { v$: useVuelidate() };
  },
  data() {
    return {
      crimes: "",
      searchedPostcode: null,
      formSubmitted: null,
      errorAtEnd: null,
    };
  },
  validations() {
    return {
      searchedPostcode: { required, postcode },
    };
  },
  methods: {
    searchSubmit() {
      // Get Postcode from API based on search submit
      axios
        .get(`https://api.postcodes.io/postcodes/${this.searchedPostcode}`)
        .then((response) => {
          this.postcode = response.data;
          // Get Crimes based on given postcode lat lng position
          axios
            .get(
              `https://data.police.uk/api/crimes-at-location?date=2021-09&lat=${this.postcode.result.latitude}&lng=${this.postcode.result.longitude}`
            )
            .then((response) => (this.crimes = response.data))
            .catch((error) => {
              if (error.response) {
                //response status is an error code
                console.log(error.response.status);
              } else if (error.request) {
                //response not received though the request was sent
                console.log(error.request);
              } else {
                //an error occurred when setting up the request
                console.log(error.message);
              }
              this.errorAtEnd = true;
            });

          this.formSubmitted = true;
          this.errorAtEnd = false;
        })
        .catch((error) => {
          if (error.response) {
            //response status is an error code
            console.log(error.response.status);
          } else if (error.request) {
            //response not received though the request was sent
            console.log(error.request);
          } else {
            //an error occurred when setting up the request
            console.log(error.message);
          }
          this.errorAtEnd = true;
        });
    },
    resetData() {
      this.crimes = "";
      this.formSubmitted = null;
      this.errorAtEnd = false;
    },
  },
};
</script>

<style lang="scss">
body {
    margin: 0;
    padding: 0;
}
#app {
    font-family: "Raleway", sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
}
#main {
    color: #333333;
}
.header-bg {
    background: url("./assets/street.jpeg");
    background-repeat: no-repeat;
    background-size: cover;
    padding: 50px 0;
    @media only screen and (min-width: 600px) {
      padding: 100px 0;
    }
}
.header-content-bg {
    background: rgba(255, 255, 255, 0.8);
    padding-top: 20px;
    padding-bottom: 20px;
}
.postcode-search {
    padding: 20px;
    background: white;
    form {
        color: #111111;
        .form-text {
            color: #111111;
        }
    }
}
.invalid-field {
    color: red;
    font-size: 0.875em;
}
.invalid-results {
    color: red;
}
.btn.disabled,
.btn:disabled {
    opacity: 0.5;
}
.results {
    th{
        text-align: left;
    }
    tbody {
        tr td:first-child {
            text-transform: capitalize;
        }
    }
}
</style>

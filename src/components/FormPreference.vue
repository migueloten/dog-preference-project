<template>
  <div class="hello">
    <h1>{{ pageTitle }} <v-icon color="#62f9d5">mdi-dog</v-icon></h1>
    <v-form :model="valid">
      <v-container>
        <v-row>
          <v-col class="d-flex" cols="12" md="6">
            <v-text-field
              v-model="email"
              :rules="emailRules"
              label="E-mail"
              required
            ></v-text-field>
          </v-col>
          <v-col class="d-flex" cols="12" md="6">
            <v-select
              :items="breeds"
              label="What is your favorite dog breed?"
              solo
              :loading="isLoading"
              v-model="breed"
            ></v-select>
          </v-col>
          <v-col class="d-flex" cols="12" sm="12">
            <v-container fluid>
              <v-radio-group v-model="radio">
                <template v-slot:label>
                  <div>Have you ever had a {{ breed }}?</div>
                </template>
                <v-radio value="Yes" color="#62f9d5">
                  <template v-slot:label>
                    <div>Yes</div>
                  </template>
                </v-radio>
                <v-radio value="No" color="#62f9d5">
                  <template v-slot:label>
                    <div>No</div>
                  </template>
                </v-radio>
              </v-radio-group>
            </v-container>
          </v-col>
          <v-col class="d-flex" cols="12" sm="12">
            <v-switch
              v-model="switch1"
              background-color="#ccd6f6"
              color="#62f9d5"
              :label="selectLabel"
            ></v-switch>
          </v-col>
          <v-col class="d-flex" cols="12" sm="12">
            <RatingField
              name="rating"
              @input="
                (event) => {
                  value = event.target.value;
                }
              "
            />
          </v-col>
          <v-col class="d-flex" cols="12" sm="12">
            <v-btn class="mr-4" outlined @click="submitDog"> Submit </v-btn>
          </v-col>
          <img :src="randomImage" max-height="150" max-width="250" />
        </v-row>
      </v-container>
    </v-form>
  </div>
</template>

<script>
import RatingField from "./RatingField.vue";
import dogAPI from "@/services/axios";
export default {
  data() {
    return {
      email: "",
      breeds: [],
      breeds_paths: {},
      breed: null,
      items: [],
      isLoading: false,
      radio: null,
      switch1: true,
      value: null,
      randomImage: "",
    };
  },
  components: { RatingField },
  computed: {
    selectLabel: function () {
      return `Yes, I'd like to receive news about ${this.breed} puppies.`;
    },
  },
  name: "FormPreference",
  props: {
    pageTitle: String,
  },
  mounted() {
    this.isLoading = true;
    dogAPI.get("breeds/list/all").then((response) => {
      this.isLoading = false;
      let breeds = response.data.message;
      Object.entries(breeds).forEach(([breed, subbreeds]) => {
        if (subbreeds.length > 0) {
          subbreeds.forEach((subbreed) => {
            let option = `${this.capitalize(subbreed)} ${this.capitalize(
              breed
            )}`;
            this.breeds.push(option);
            this.breeds_paths[option] = `${breed}/${subbreed}`;
          });
        } else {
          let option = `${this.capitalize(breed)}`;
          this.breeds.push(option);
          this.breeds_paths[option] = `${breed}`;
        }
      });
      this.breed = "Shiba";
    });
  },
  methods: {
    capitalize(text) {
      return text.charAt(0).toUpperCase() + text.slice(1);
    },
    log(item) {
      console.log(item);
    },

    submitDog() {
      dogAPI
        .get(`breed/${this.breeds_paths[this.breed]}/images/random`)
        .then((response) => {
          let randomUrl = response.data.message;
          this.randomImage = randomUrl;
        });
    },
  },
};
</script>

<style lang="scss" scoped>
.hello {
  color: #ccd6f6;

  .v-col {
    align-items: center;
  }

  .v-btn,
  img {
    color: #62f9d5;
    background-color: #0a192f;
    border: #62f9d5 1px solid;
  }
}
</style>

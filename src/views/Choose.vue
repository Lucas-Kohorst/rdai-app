<template>
  <v-container fluid fill-height pt-2 pa-0>
    <v-sheet pa-3 pt-0>
      <v-flex
        xs12
        display-1
        text-center
        px-4
        mt-2
        py-1
        v-if="$route.name !== 'choose'"
      >
        Donate the interest, keep the capital
      </v-flex>
      <v-flex xs12 my-5 v-if="showLoader">
        <v-progress-linear indeterminate></v-progress-linear>
      </v-flex>
      <v-layout wrap grid-list-sm>
        <v-flex
          xs12
          sm6
          md4
          lg3
          v-for="(column, index) in columns"
          :key="index"
        >
          <v-flex v-for="i in column" :key="i.hatID" class="pa-0 ma-0">
            <v-card
              color="white"
              light
              elevation="3"
              :class="[
                { 'my-card': userHat && userHat.hatID === i.hatID },
                'text-sm-center ma-3 pa-4'
              ]"
              :style="{ borderColor: i.color || 'black' + '!important' }"
            >
              <v-icon
                color="#FFD700"
                large
                class="tilted"
                absolute
                v-if="userHat && userHat.hatID === i.hatID"
              >
                fas fa-crown
              </v-icon>
              <v-flex px-5 py-0>
                <v-img
                  class="round my-2 mx-auto my-round-image"
                  :max-width="140"
                  :max-height="140"
                  :src="i.image"
                  contain
                  :alt="i.title"
                />
              </v-flex>
              <template v-if="i.hasOwnProperty('title')">
                <h3>{{ i.title }}</h3>
                <p>{{ i.description }}</p>
              </template>
              <template v-else>
                <h3># {{ i.hatID }}</h3>
                <proportions :hat="i" />
              </template>
              <a
                v-if="i.shortTitle && i.shortTitle === 'YourCause'"
                href="https://twitter.com/rDAI_dao"
                target="_blank"
                alt="contact us on twitter"
                style="text-decoration: none;"
              >
                <v-btn color="secondary" class="mb-2"
                  ><v-icon>fab fa-twitter</v-icon>Contact Us</v-btn
                >
              </a>
              <v-btn
                v-else
                color="primary"
                class="mb-2"
                @click="hasWeb3 ? open(i.shortTitle || i.hatID) : activateWeb3()"
                :disabled="!i.hasOwnProperty('hatID')"
              >
                <span v-if="!hasWeb3">Connect your wallet</span>
                <span v-else-if="i.shortTitle && i.shortTitle === 'custom'"
                  >Build your own</span
                >
                <span v-else-if="userHat.hatID && i.hatID === userHat.hatID"
                  >Donate more!</span
                >
                <span v-else-if="userHat.hatID">Choose this pool</span>
                <span v-else>Donate interest!</span>
              </v-btn>
              <!--v-flex v-if="i.hasOwnProperty('loans') && i.hatID === rDAIdevs.hatID">
                              {{ i.loans[0] | formatNumber(2) + 'DAI'}}
                              <span class="caption"> deposited so far</span>
                          </v-flex>
                          <v-flex v-else-if="i.hasOwnProperty('loans') && i.totalLoan > i.loans[0]">
                              {{i.totalLoan - i.loans[0] | formatNumber(2) + ' DAI'}}
                              <span class="caption"> deposited so far</span>
                          </v-flex-->
            </v-card>
          </v-flex>
        </v-flex>
      </v-layout>
    </v-sheet>
  </v-container>
</template>
<style scoped>
.round {
  border-radius: 100%;
}

.my-card {
  box-sizing: border-box;
  border-top: 10px solid !important;
  border-bottom: 10px solid !important;
}

.tilted {
  position: absolute;
  right: 10px;
  transform: rotateZ(45deg);
}
</style>
<script>
import { mapState, mapGetters, mapActions } from "vuex";
import featured from "../featured.js";

export default {
  name: "choose",
  props: ["showLoader"],
  data: () => ({
    listOfHats: [],
    columns: [[], [], [], []],
    numberOfColumns: 1
  }),
  computed: {
    ...mapState(["allHats"]),
    ...mapGetters(["hasWeb3", "userHat", "rDAIdevs"]),
    allHatsLength() {
      return this.allHats.length + 2;
    },
    listOfHatsLength() {
      return this.listOfHats.length;
    }
  },
  methods: {
    ...mapActions(["activateWeb3"]),
    open(value) {
      if (typeof value === "string") this.openByShortTitle(value);
      else this.openById(value);
    },
    openCreate() {
      this.$router.push({
        path: "/create",
        params: {
          url: "create"
        }
      });
    },
    async openById(hatID) {
      await this.$store.dispatch("setInterfaceHat", {
        hatID
      });
      this.$router.push({
        path: `/deposit/${hatID}`,
        params: {
          url: "deposit"
        }
      });
    },
    async openByShortTitle(shortTitle) {
      if (shortTitle === "custom") return this.openCreate();
      await this.$store.dispatch("setInterfaceHat", {
        shortTitle
      });
      this.$router.push({
        path: `/donate/${shortTitle}`,
        name: "",
        params: {
          url: "donate",
          shortTitle
        }
      });
    },
    loadAll() {
      if (this.hasWeb3) {
        const list = [...this.allHats, featured[1], featured[2]];
        list.sort((a, b) => {
          return b.totalLoan - a.totalLoan;
        });
        this.listOfHats = list;
      } else this.listOfHats = featured;
      return this.listOfHats.length;
    }
  },
  watch: {
    allHatsLength(newV, oldV) {
      this.loadAll();
    },
    listOfHatsLength() {
      this.columns = [[], [], [], []];
      this.listOfHats.forEach((item, index) => {
        if (item.hatID === this.userHat.hatID)
          this.columns[index % this.numberOfColumns].unshift(item);
        else this.columns[index % this.numberOfColumns].push(item);
      });
    },
    hasWeb3(){
      this.loadAll();
    }
  },
  mounted() {
    if (this.$vuetify.breakpoint.xsOnly) this.numberOfColumns = 1;
    if (this.$vuetify.breakpoint.smOnly) this.numberOfColumns = 2;
    if (this.$vuetify.breakpoint.mdOnly) this.numberOfColumns = 3;
    if (this.$vuetify.breakpoint.lgOnly) this.numberOfColumns = 4;
    const length = this.loadAll();
    if(length < this.numberOfColumns) {
      console.log("length: ", length, " this.numberOfColumns: ", this.numberOfColumns);
      this.numberOfColumns = length;
      this.loadAll();
    }
  }
};
</script>

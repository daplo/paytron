<template>
  <div class="hello">
    <div class="row mt-4">
      <div class="col-12 col-lg-8">
        <div class="row">
          <div class="col-12">
            <!-- disabled due to lack of time - would have to switch currency pairs
             <h4>I would like to</h4>
            -->
            <!--div
              class="btn-group mt-2"
              role="group"
              aria-label="Basic radio toggle button group"
            >
              <input
                type="radio"
                class="btn-check"
                name="btnradio"
                id="btnradio1"
                autocomplete="off"
                checked
                v-model="buyOrSell"
                value="Buy"
              />
              <label class="btn btn-outline-primary" for="btnradio1">Buy</label>

              <input
                type="radio"
                class="btn-check"
                name="btnradio"
                id="btnradio2"
                autocomplete="off"
                v-model="buyOrSell"
                value="Sell"
              />
              <label class="btn btn-outline-primary" for="btnradio2"
                >Sell</label
              >
            </div -->
          </div>
        </div>

        <div class="row mt-4">
          <div class="col">
            <div class="row">
              <div class="col-12 col-lg-6">
                <label for="firstCurrency" class="form-label"
                  >{{ buyOrSell }} Currency
                </label>
                <select
                  v-model="firstCurrency"
                  class="form-select"
                  aria-label="Select Currency"
                  id="firstCurrency"
                  @change="onChange($event)"
                >
                  <option
                    v-for="(currency, index) in currencies"
                    :key="`cuyrrency-${index}`"
                    :disabled="currency == secondCurrency"
                  >
                    {{ currency }}
                  </option>
                </select>
              </div>
              <div class="col-12 col-lg-6">
                <label for="amount" class="form-label"
                  >Amount of {{ firstCurrency }} you want to {{ buyOrSell }}
                </label>
                <input
                  type="number"
                  class="form-control"
                  id="amount"
                  aria-describedby="currency amount"
                  v-model="currencyAmount"
                  @change="onChange($event)"
                />
              </div>
            </div>
          </div>
        </div>

        <div class="row mt-4">
          <div class="col-lg-6">
            <label for="secondCurrency" class="form-label"
              >{{ buyOrSell === "Buy" ? "Sell" : "Receive" }} Currency</label
            >
            <select
              v-model="secondCurrency"
              class="form-select"
              aria-label="Select Currency"
              id="firstCurrency"
              @change="onChange($event)"
            >
              <option
                v-for="(currency, index) in currencies"
                :key="`cuyrrency-${index}`"
                :disabled="currency == firstCurrency"
              >
                {{ currency }}
              </option>
            </select>
          </div>
          <div class="col-lg-6">
             <label for="secondCurrency" class="form-label"
              >{{ buyOrSell === "Buy" ? "Sell" : "Receive" }} Amount</label
            >
            <h5 class="mt-2">{{formatCurrency(sellAmount, secondCurrency)}}</h5>
          </div>
        </div>
        <div class="row mt-4">
          <div class="col-12">
            <button class="btn btn-primary" @click="fetchRates">
              Manually Refresh Rates
            </button>
          </div>
        </div>
      </div>
      <div class="col-12 col-lg-4 aside">
        <div class="row mt-1">
          <div class="col-lg-8">
            <h3>Breakdown</h3>
          </div>
          <div class="col-lg-4">
            <Loader v-if="isLoading" />
          </div>
        </div>
        <div class="row">
          <div class="col-12">
            <strong>Sell: </strong> {{formatCurrency(currencyAmount, firstCurrency)}}
          </div>
           <div class="col-12">
            <strong>Buy: </strong> {{formatCurrency(sellAmount, secondCurrency)}}
          </div>
           <div class="col-12">
            <strong>Paytrons Fee:  </strong> {{formatCurrency(sellAmount * paytronMarkup, secondCurrency)}} ({{paytronMarkup * 100}}%)
          </div>

          <div class="col-12 mt-1">
            <hr>
            <h4>
                 <strong>Total:  </strong> {{formatCurrency(sellAmount - sellAmount * paytronMarkup , secondCurrency)}} ({{paytronMarkup * 100}}%)
            </h4>
          </div>
        </div>
      </div>
    </div>

    <div class="row mt-5">
      <div class="col-12">
        <h5>Raw Results</h5>
        <hr />
        {{ info }}
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Loader from "./Loader.vue";

export default {
  name: "Table",
  components: { Loader },
  props: {
    msg: String,
  },
  data() {
    return {
      info: null,
      firstCurrency: "AUD",
      secondCurrency: "USD",
      buyOrSell: "Sell",
      currencyAmount: 1000,
      currencies: ["AUD", "EUR", "USD", "GBP"],
      paytronMarkup: 0.005,
      paytronMarkupAmount: 0,
      isLoading: false,
      sellAmount: null
    };
  },
  methods: {
    fetchRates() {
      const _self = this;
      const firstLabel = this.buyOrSell;
      const secondLabel = this.buyOrSell === "Buy" ? "Sell" : "Buy";
      this.isLoading = true;
      this.sellAmount = null

      axios
        .get(
          `https://wnvgqqihv6.execute-api.ap-southeast-2.amazonaws.com/Public/public/rates?${firstLabel}=${_self.firstCurrency}&Amount=${_self.currencyAmount}&${secondLabel}=${_self.secondCurrency}&Fixed=Sell`
        )
        .then((response) => {
          this.info = response.data;
          this.sellAmount = response.data.clientBuyAmount;
          }
        )
        .catch((error) => {
          //handle error
          console.log(error);

        })
        .then(() => {
          _self.isLoading = false;
        });
    },
    formatCurrency(val, curr) {
      const locales = {
        "AUD": 'en-AU',
        "EUR": 'sfb',
        "USD": 'en-US',
        "GBP" : 'en-GB'
      }

   let formatter = new Intl.NumberFormat( locales[curr], {
      style: 'currency',
      currency: curr,
      minimumFractionDigits: 2
    });
   return formatter.format(val)
    },
    onChange($event){
      console.log($event);
      this.fetchRates();
    }
  },
  computed: {

  },
  mounted() {
    this.fetchRates();
  },
};
</script>


<style scoped>
.aside {
  border-left: 1px solid #cdcdcd;
  padding-left: 2rem
}
</style>

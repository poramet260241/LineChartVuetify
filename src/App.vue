<template>
  <v-app>
    <v-app-bar app color="#2C3E50" dark>
      <div id="app-bar-title">
        <h2>Cryptocurrency Exchange Rates</h2>
      </div>
    </v-app-bar>

    <v-main id="content">
      <v-container fluid>
        <v-row style="margin:1rem">
          <v-col class="d-flex" cols="12" sm="2">
            <v-select
              dense
              :items="coinlist"
              item-value="coin"
              solo
              v-model="defaultSelectedOfCoin.coin"
              return-object
            ></v-select>
          </v-col>
          <v-col class="d-flex" cols="12" sm="2">
            <v-select
              dense
              :items="monthlist"
              item-value="month"
              solo
              v-model="defaultSelectedOfMonth.month"
              return-object
            ></v-select>
          </v-col>
          <v-col class="d-flex" cols="12" sm="2">
            <v-select
              dense
              :items="yearlist"
              label="Year"
              solo
              v-model="defaultSelectedOfYear.year"
              item-value="year"
              return-object
            ></v-select>
          </v-col>
          <v-col class="d-flex" cols="12" sm="1">
            <v-btn @click="fetchData" color="primary">OK</v-btn>
          </v-col>
        </v-row>
      </v-container>

      <v-container fluid>
        <v-row no-gutters class="justify-center">
          <v-col>
            <h2>{{ this.table_title }}</h2>
          </v-col>
          <v-col>
            <h2 style="text-align: center">
              Daily data of {{ this.table_label }}
            </h2>
          </v-col>
          <v-col></v-col>
        </v-row>
        <v-row no-gutters class="justify-center">
          <v-col>
            <line-chart
              v-if="loaded"
              :chartData="datacollection"
              :options="options"
            ></line-chart>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import LineChart from "./LineChart.js";
import axios from "axios";
import moment from "moment";

export default {
  name: "App",
  components: { LineChart },
  data() {
    return {
      loaded: false,
      datacollection: [],
      options: {
        responsive: true,
        maintainAspectRatio: false,
      },

      paramsData: {
        apikey: "852D82AA-DC86-47D8-BE94-8FFB4EA5108D",
        time_start: "2020-01-01T00:00:00",
        time_end: "2020-02-01T00:00:00",
        period_id: "1DAY",
      },
      table_title: "",
      table_label: "",

      coinlist: ["BTC", "ETH", "ADA", "BNB", "USDT"],

      monthlist: [
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December",
      ],
      yearlist: ["2018", "2019", "2020", "2021"],
      defaultSelectedOfCoin: {
        coin: "BTC",
      },
      defaultSelectedOfMonth: {
        month: "January",
      },
      defaultSelectedOfYear: {
        year: "2020",
      },
    };
  },
  created() {
    this.loaded = false;
    this.date_title = "";
    this.fetchData();
    this.loaded = true;
  },
  mounted() {},
  methods: {
    async fetchData() {
      const monthNo = this.getMonthNo(this.defaultSelectedOfMonth.month);
      const year = parseInt(this.defaultSelectedOfYear.year);

      if (monthNo < 9) {
        this.paramsData.time_start = `${year}-0${monthNo}-01T00:00:00`;
        this.paramsData.time_end = `${year}-0${monthNo + 1}-01T00:00:00`;
      }
      else if (monthNo === 9) {
        this.paramsData.time_start = `${year}-0${monthNo}-01T00:00:00`;
        this.paramsData.time_end = `${year}-${monthNo + 1}-01T00:00:00`;
      }
      else if (monthNo === 12) {
        this.paramsData.time_start = `${year}-${monthNo}-01T00:00:00`;
        this.paramsData.time_end = `${year + 1}-01-01T00:00:00`;
      } else {
        this.paramsData.time_start = `${year}-${monthNo}-01T00:00:00`;
        this.paramsData.time_end = `${year}-${monthNo + 1}-01T00:00:00`;
      }

      const { data } = await axios.get(
        `https://rest.coinapi.io/v1/ohlcv/${this.defaultSelectedOfCoin.coin}/USD/history`,
        {
          params: this.paramsData,
        }
      );
      console.log(data.length);

      if(data.length === 0){
        alert("DATA NOT FOUND.")
        return
      }

      let dateArr = [];
      let dataArr = [];

      data.forEach((element) => {
        const date = moment(element.time_period_start, "YYYY-MM-DD").format(
          "DD/MM/YYYY"
        );
        const price_close = element.price_close;

        dataArr.push(price_close);
        dateArr.push(date);
      });

      this.table_title = `${this.defaultSelectedOfCoin.coin}/USD`;
      this.table_label = `${this.defaultSelectedOfMonth.month} ${this.defaultSelectedOfYear.year}`;

      this.datacollection = {
        labels: dateArr,
        datasets: [
          {
            fill: false,
            borderColor: "#76D7C4",
            label: this.table_title,
            data: dataArr,
          },
        ],
      };
    },
    getMonthNo(m) {
      const monthlist = [
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December",
      ];

      return monthlist.indexOf(m) + 1;
    },
  },
};
</script>
<style scoped>
#app-bar-title {
  margin-left: 3rem;
}
.small {
  height: 50px;
  width: 100%;
  border: 1px solid #4caf50;
}
</style>

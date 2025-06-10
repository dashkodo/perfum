<template>
  <div id="app">
    <h1>Колекція парфумів <span v-if="rawData.length">({{ rawData.length }})</span></h1>
    <input v-model="search" placeholder="Search..." class="search-bar" />
    <table v-if="filteredData.length">
      <thead>
        <tr>
          <th v-for="(header, idx) in headers" :key="idx" @click="sortBy(header)">
            {{ header }}
            <span v-if="sortKey === header">{{ sortOrder === 1 ? "▲" : "▼" }}</span>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, idx) in filteredData" :key="idx">
          <td v-for="header in headers" :key="header">{{ row[header] }}</td>
        </tr>
      </tbody>
    </table>
    <div v-else>No data found.</div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "App",
  data() {
    return {
      rawData: [],
      headers: [],
      search: "",
      sortKey: "",
      sortOrder: 1,
    };
  },
  computed: {
    filteredData() {
      let data = this.rawData;
      if (this.search) {
        const s = this.search.toLowerCase();
        data = data.filter((row) =>
          this.headers.some((h) => String(row[h]).toLowerCase().includes(s))
        );
      }
      if (this.sortKey) {
        data = [...data].sort((a, b) => {
          if (a[this.sortKey] < b[this.sortKey]) return -1 * this.sortOrder;
          if (a[this.sortKey] > b[this.sortKey]) return 1 * this.sortOrder;
          return 0;
        });
      }
      return data;
    },
  },
  methods: {
    sortBy(header) {
      if(this.sortKey === "" && header === "№") {
        return; // Do not sort by "№"
      }
      if (this.sortKey === header) {
        this.sortOrder *= -1;
      } else {
        this.sortKey = header;
        this.sortOrder = 1;
      }
    },
    async fetchSheet() {
      const sheetId = process.env.VUE_APP_SHEET_ID;
      const range = process.env.VUE_APP_SHEET_RANGE;
      const api_key = process.env.VUE_APP_GOOGLE_API_KEY;
      const url = `https://sheets.googleapis.com/v4/spreadsheets/${sheetId}/values/${range}?alt=json&key=${api_key}`;
      try {
        const res = await axios.get(url);
        const [headerRow, ...rows] = res.data.values;
        this.headers = ["№", ...headerRow];
        this.rawData = rows
          .filter((row) => row[0] && row[0].trim() !== "") // skip rows where first column is empty
          .map((row, i) => {
            const obj = {};
            obj["№"] = i + 1;
            headerRow.forEach((h, idx) => {
              obj[h] = row[idx] || "";
            });
            return obj;
          });
      } catch (e) {
        this.rawData = [];
        this.headers = [];
      }
    },
  },
  mounted() {
    this.fetchSheet();
  },
};
</script>

<style>
body {
  font-family: "Montserrat Alternates", sans-serif;
  color: #ffffff;

  margin-top: 60px;
  background: linear-gradient(90deg, rgb(42, 29, 43) 0%, rgb(3, 27, 67) 100%);
}
h1 {
  color: #ffffff;
  font-size: 2.5em;
  margin-bottom: 20px;
}

#app {
  text-align: center;
  color: #2c3e50;
}

.search-bar {
  font-family: "Montserrat Alternates", sans-serif;
  margin-bottom: 20px;
  padding: 8px;
  width: 300px;
  border-radius: 120px;
  border: 2px solid white;
  background: rgba(255, 255, 255, 0);
  padding: 0 68px 0 68px;
  color: #ffffff;
  height: 60px;
  font-size: 1.32em;
  font-weight: 400;
  letter-spacing: -0.015em;
  outline: none;
}

.search-bar::-webkit-input-placeholder {
  color: #ffffff;
}
.search-bar::-moz-placeholder {
  color: #ffffff;
}
.search-bar:-ms-input-placeholder {
  color: #ffffff;
}
.search-bar:-moz-placeholder {
  color: #ffffff;
}

table {
  margin: 0 auto;
  border-collapse: collapse;
  width: 80%;
}

th,
td {
  border: 1px solid #ddd;
  padding: 8px;
}

th {
  cursor: pointer;
  background: #f4f4f4;
}

tr:nth-child(even){
  background: #f4f4f4;
}
td:first-child, th:first-child {
    font-weight: bold;
    text-align: right;
}
td:nth-child(2) td:last-child {
    text-align: left;
}
td:nth-child(2) {
    font-weight: bold;
}
tr {
  background: #f9f9f9;
}
x
@media (max-width: 600px) {
  body {
    margin-top: 20px;
    font-size: 1em;
  }
  h1 {
    font-size: 1.3em;
    margin-bottom: 10px;
  }
  .search-bar {
    width: 90vw;
    min-width: 0;
    max-width: 98vw;
    height: 44px;
    font-size: 1em;
    padding: 0 20px;
  }
  table {
    width: 98vw;
    font-size: 0.9em;
    overflow-x: auto;
    display: block;
  }
  th, td {
    padding: 4px;
    font-size: 0.95em;
    word-break: break-word;
  }
  td:first-child {
    font-weight: bold;
    text-align: left;
  }
  #app {
    padding: 0 2vw;
  }
}
</style>

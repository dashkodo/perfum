<template>
  <div id="app">
    <h1>Колекція парфумів</h1>
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
      if (this.sortKey === header) {
        this.sortOrder *= -1;
      } else {
        this.sortKey = header;
        this.sortOrder = 1;
      }
    },
    async fetchSheet() {
      const sheetId = process.env.SHEET_ID;
      const range = process.env.SHEET_RANGE;
      const api_key = process.env.GOOGLE_API_KEY;
      const url = `https://sheets.googleapis.com/v4/spreadsheets/${sheetId}/values/${range}?alt=json&key=${api_key}`;
      try {
        const res = await axios.get(url);
        const [headerRow, ...rows] = res.data.values;
        this.headers = headerRow;
        this.rawData = rows
          .filter((row) => row[0] && row[0].trim() !== "") // skip rows where first column is empty
          .map((row) => {
            const obj = {};
            headerRow.forEach((h, i) => {
              obj[h] = row[i] || "";
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
  background: linear-gradient(90deg, rgba(154, 57, 163, 1) 0%, rgb(9, 43, 102) 100%);
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

tr {
  background: #f9f9f9;
}
</style>

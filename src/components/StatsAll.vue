<template>
  <div class="container-fluid p-5">
    <div class="alert alert-danger" role="alert" v-if="error">
      Data not found!
    </div>
    <div class="container">
      <br />
      <form @submit.prevent class="w-400 mw-full">
        <div class="form-group">
          <label for="full-name">Filter by name</label>
          <input
            type="text"
            class="form-control"
            id="searchBox"
            v-model="queryName"
            autocomplete="off"
          />
        </div>
      </form>
    </div>
    <table class="table table-striped table-hover" v-if="!error">
      <thead>
        <tr>
          <th class="sticky-header" scope="col">
            Roll <i @click="sortByRoll" class="fa fa-fw fa-sort"></i>
          </th>
          <th scope="col">
            Name <i @click="sortByName" class="fa fa-fw fa-sort"></i>
          </th>
          <th scope="col">Compiled</th>
          <th scope="col">Launched</th>
          <th scope="col">
            Milliseconds
            <i @click="sortByMilliseconds" class="fa fa-fw fa-sort"></i>
          </th>
          <th scope="col">
            Cycles <i @click="sortByCycles" class="fa fa-fw fa-sort"></i>
          </th>
          <th scope="col">
            Pagefaults
            <i @click="sortByPagefaults" class="fa fa-fw fa-sort"></i>
          </th>
          <th scope="col">
            Page mapped issues
            <i @click="sortByPMI" class="fa fa-fw fa-sort"></i>
          </th>
          <th scope="col">
            Page file issues <i @click="sortByPFI" class="fa fa-fw fa-sort"></i>
          </th>
          <!-- <th scope="col">
            Runtime <i @click="sortByTime" class="fa fa-fw fa-sort"></i>
          </th> -->
        </tr>
      </thead>
      <tbody>
        <tr v-for="person in sorted_out" :key="person.roll">
          <th scope="row" v-if="person.roll != undefined">{{ person.roll }}</th>
          <td class="text-center">{{ person.name }}</td>
          <td class="text-center">
            {{ person.compiled == "true" ? "✔️" : "❌" }}
          </td>
          <td class="text-center">
            {{ person.launched == "true" ? "✔️" : "❌" }}
          </td>
          <td class="text-center">
            {{ person.milliseconds }}
          </td>
          <td class="text-center">
            {{ person.cycles }}
          </td>
          <td class="text-center">
            {{ person.pagefaults }}
          </td>
          <td class="text-center">
            {{ person.page_maps_issues }}
          </td>
          <td class="text-center">
            {{ person.page_file_issues }}
          </td>
          <!-- <td class="text-center">{{ person["Compilation Time"] }}</td> -->
        </tr>
      </tbody>
    </table>
    <br />
  </div>
</template>

<script>
export default {
  name: "StatsAll",
  data() {
    return {
      roll: 0,
      name: "Unknown",
      Compilation: "NA",
      Execution: "NA",
      Correctness: "NA",
      Output: "NA",
      time: "NA",
      day: "1",
      data_url_base: "https://aabhusanaryal.github.io/fake-json/MOCK_DATA",
      data_url: "",
      json_out: [],
      sorted_out: [],
      sortOrder: 0, //0 or 1, ascending or descending
      error: 0,
      errorMsg: "",
      queryName: "",
      dataLoaded: false,
    };
  },
  props: {
    systemTheme: String,
  },

  methods: {
    sortByTime() {
      this.sorted_out.sort((el1, el2) => {
        let d = el1["Compilation Time"] - el2["Compilation Time"];
        if (this.sortOrder) {
          return d;
        } else {
          return -d;
        }
      });
      this.sortOrder = !this.sortOrder;
    },
    sortByName() {
      this.sorted_out.sort((el1, el2) => {
        let d = el1.name > el2.name;
        if (this.sortOrder) {
          return d ? 1 : -1;
        } else {
          return d ? -1 : 1;
        }
      });
      this.sortOrder = !this.sortOrder;
    },
    sortByRoll() {
      this.sorted_out.sort((el1, el2) => {
        let d = el1.roll - el2.roll;
        if (this.sortOrder) {
          return d;
        } else {
          return -d;
        }
      });
      this.sortOrder = !this.sortOrder;
    },
    sortByMilliseconds() {
      this.sorted_out.sort((el1, el2) => {
        let d = el1.milliseconds - el2.milliseconds;
        if (this.sortOrder) {
          return d;
        } else {
          return -d;
        }
      });
      this.sortOrder = !this.sortOrder;
    },
    sortByCycles() {
      this.sorted_out.sort((el1, el2) => {
        let d = el1.cycles - el2.cycles;
        if (this.sortOrder) {
          return d;
        } else {
          return -d;
        }
      });
      this.sortOrder = !this.sortOrder;
    },
    sortByPagefaults() {
      this.sorted_out.sort((el1, el2) => {
        let d = el1.pagefaults - el2.pagefaults;
        if (this.sortOrder) {
          return d;
        } else {
          return -d;
        }
      });
      this.sortOrder = !this.sortOrder;
    },
    sortByPMI() {
      this.sorted_out.sort((el1, el2) => {
        let d = el1.page_maps_issues - el2.page_maps_issues;
        if (this.sortOrder) {
          return d;
        } else {
          return -d;
        }
      });
      this.sortOrder = !this.sortOrder;
    },
    sortByPFT() {
      this.sorted_out.sort((el1, el2) => {
        let d = el1.page_file_issues - el2.page_file_issues;
        if (this.sortOrder) {
          return d;
        } else {
          return -d;
        }
      });
      this.sortOrder = !this.sortOrder;
    },
    fetchData() {
      // Fetching CSV and converting to JSON
      fetch(this.data_url)
        .then((res) => {
          if (!res.ok) this.error = 1;
          else this.error = 0;
          return res.text();
        })
        .then((input) => {
          const lines = input.split("\n");
          const header = lines[0].split(",");
          let output = lines.slice(1).map((line) => {
            const fields = line.split(",");
            return Object.fromEntries(header.map((h, i) => [h, fields[i]]));
          });
          this.json_out = Object.values(output); // this is an array
          this.json_out.pop(); // Removes the last (useless) entry
          this.sorted_out = this.json_out;
        });
    },
    filterByName() {
      this.sorted_out = this.json_out.filter((el) => {
        return el.name.toLowerCase().startsWith(this.queryName.toLowerCase());
      });
      console.log(this.sorted_out);
    },
  },
  mounted() {
    if (this.$route.params.day != undefined) {
      this.day = this.$route.params.day;
      this.data_url = `${this.data_url_base}_DAY${this.day}.csv`;
      this.fetchData();
    } else {
      fetch("https://aabhusanaryal.github.io/fake-json/maxDays.json")
        .then((res) => res.json())
        .then((res) => {
          this.day = res[0];
          this.$router.replace({ path: `/${this.day}` });
        });
    }
  },
  watch: {
    $route(to, from) {
      console.log(to, from);
      this.data_url = `${this.data_url_base}_DAY${to.params.day}.csv`;
      this.fetchData();
      this.filterByName();
    },
    queryName() {
      this.filterByName();
    },
  },
};
</script>

<style>
thead {
  position: fixed;
  top: 0;
}

.container-fluid {
  min-height: 81vh;
}
#searchBox {
  width: 300px;
  height: 50px;
  padding-left: 5px;
}

i {
  cursor: pointer;
}
.container {
  overflow: auto !important;
}

.custom-container {
  padding-right: 0;
  padding-left: 0;
  width: 50%;
}

@media (min-width: 568px) {
  .container-fluid {
    width: 550px;
  }
}
@media (min-width: 1020px) {
  .container-fluid {
    width: 800px;
  }
}
@media (min-width: 1200px) {
  .container-fluid {
    width: 1200px;
  }
}
</style>

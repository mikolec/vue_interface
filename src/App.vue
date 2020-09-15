<template>
  <div id="main-app" class="container">
    <div class="row justify-content-center">
      <AddAppointment @add="addItem" />
      <SearchAppointments
        @searchRecords="searchItems"
        :filterKey="filterKey"
        :filterDir="filterDir"
        @changeFilterKey="changeFilterKey"
        @changeFilterDir="changeFilterDir"
      />
      <AppointmentList :appointments="filteredApts" @remove="removeItem" @edit="editItem" />
    </div>
  </div>
</template>

<script>
import axios from "axios";
import _ from "lodash";
import AppointmentList from "@/components/AppointmentList.vue";
import AddAppointment from "@/components/AddAppointment.vue";
import SearchAppointments from "@/components/SearchAppointments.vue";

export default {
  name: "MainApp",
  data() {
    return {
      appointments: [],
      aptIndex: 0,
      searchTerms: "",
      filterKey: "petName",
      filterDir: "asc",
    };
  },
  computed: {
    searchedApts() {
      return this.appointments.filter((item) => {
        return (
          item.petName.toLowerCase().match(this.searchTerms.toLowerCase()) ||
          item.petOwner.toLowerCase().match(this.searchTerms.toLowerCase()) ||
          item.aptNotes.toLowerCase().match(this.searchTerms.toLowerCase())
        );
      });
    },
    filteredApts() {
      return _.orderBy(
        this.searchedApts,
        (item) => {
          return item[this.filterKey].toLowerCase();
        },
        this.filterDir
      );
    },
  },
  methods: {
    addItem(apt) {
      apt.aptId = this.aptIndex;
      this.aptIndex++;
      this.appointments.push(apt);
    },
    removeItem(apt) {
      // this.appointments = _.without(this.appointments, apt);
      this.appointments = this.appointments.filter((item) => item != apt);
    },
    editItem(id, field, text) {
      //make sure we are getting an element in the appointments list...
      // const aptIndex = _.findIndex(this.appointments, { aptId: id });
      var aptIndex = this.appointments.findIndex((item) => (item.aptId = id));
      this.appointments[aptIndex][field] = text;
    },
    searchItems(terms) {
      this.searchTerms = terms;
    },
    changeFilterKey(key) {
      this.filterKey = key;
    },
    changeFilterDir(dir) {
      this.filterDir = dir;
    },
  },
  components: { AppointmentList, AddAppointment, SearchAppointments },
  mounted() {
    axios.get("./data/appointments.json").then(
      (response) =>
        (this.appointments = response.data.map((item) => {
          item.aptId = this.aptIndex;
          this.aptIndex++;
          return item;
        }))
    );
  },
};
</script>

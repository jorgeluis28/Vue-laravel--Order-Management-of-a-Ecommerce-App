<template>
  <section class="content-wrapper" style="min-height: 960px;">
    <section class="content-header">
      <h1>Printer Company Router</h1>
    </section>

    <section class="content">
      <div class="row">
        <div class="col-xs-12">
          <form @submit.prevent="submitForm">
            <div class="box">
              <div class="box-header with-border">
                <h3 class="box-title">Edit</h3>
              </div>

              <div class="box-body">
                <!-- <back-buttton></back-buttton> -->
                <router-link
                  v-if="$can('printer_company_access')"
                  :to="{ name: 'printer_routers.index'}"
                  class="btn btn-default btn-sm"
                >
                  <span class="glyphicon glyphicon-chevron-left"></span> Back to all Printer Companies
                </router-link>
                <button
                  type="button"
                  class="btn btn-default btn-sm"
                  @click="fetchData($route.params.id)"
                >
                  <i class="fa fa-refresh" :class="{'fa-spin': loading}"></i> Refresh
                </button>
              </div>
              <div class="row" v-if="loading">
                <div class="col-xs-4 col-xs-offset-4">
                  <div class="alert text-center">
                    <i class="fa fa-spin fa-refresh"></i> Loading
                  </div>
                </div>
              </div>

              <bootstrap-alert />

              <div class="box-body" v-if="!loading">
                <div class="form-group">
                  <label for="continent">Select Continent *</label>
                  <v-select
                    @change="updateContinent"
                    :options="continents"
                    :value="item.continent"
                    :disabled="continents.length == 0"
                  ></v-select>
                </div>
                <div class="form-group">
                  <label for="country">Select Country</label>
                  <v-select
                    @change="updateCountry"
                    :options="countries"
                    :value="item.country"
                    :disabled="countries.length == 0"
                  >
                    <template slot="option" slot-scope="country">
                      <span class="country-label">{{ country.label }}</span>
                    </template>
                  </v-select>
                </div>
                <div class="form-group">
                  <label for="region">Select Region</label>
                  <v-select
                    @change="updateRegion"
                    :options="regions"
                    :value="item.region"
                    :disabled="regions.length == 0"
                  ></v-select>
                </div>
                <div class="form-group">
                  <div>
                    <span style="font-weight:900">Select Printer Company *</span>
                    <i class="fas fa-plus-square add-btn" @click="addPrinterField"></i>
                  </div>
                  <br />
                  <div
                    class="row col-md-8 col-xs-12 col-sm-12 printer-field"
                    v-for="(printer,index) in printerRelationShip"
                    v-bind:key="index"
                  >
                    <div>
                      <img
                        class="my-image"
                        :src="printer.printer_company.logo_url"
                        v-if="printer.printer_company"
                      />
                      <div class="col-md-1 col-xs-1 col-sm-1">
                        <i
                          v-if="index != 0"
                          class="fas fa-minus-square remove-btn pull-right"
                          @click="removePrinterField(index)"
                        ></i>
                      </div>
                      <div class="col-md-3 col-xs-7 col-sm-7">
                        <v-select
                          :options="printer_companies"
                          :clearable="false"
                          label="name"
                          v-model="printer.printer_company"
                        >
                          <template slot="option" slot-scope="printer_company">
                            <div class="custome-disabled">
                              <img
                                class="my-image"
                                :src="printer_company.logo_url"
                                v-if="printer_company.logo_url"
                              />
                              <span class="country-label">{{ printer_company.name }}</span>
                            </div>
                          </template>
                        </v-select>
                      </div>
                      <div class="col-md-3 col-xs-3 col-sm-3">
                        <input
                          type="number"
                          min="1"
                          max="100"
                          class="form-control"
                          name="percent"
                          placeholder="Enter Percent % "
                          v-model="printer.percent"
                          required
                        />
                      </div>
                    </div>
                  </div>
                </div>
              </div>

              <div class="box-footer">
                <vue-button-spinner
                  class="btn btn-primary btn-sm"
                  :isLoading="loading"
                  :disabled="loading"
                >Save</vue-button-spinner>
              </div>
            </div>
          </form>
        </div>
      </div>
    </section>
  </section>
</template>


<script>
import { mapGetters, mapActions } from "vuex";

export default {
  data() {
    return {
      // Code...
      printerRelationShip: []
    };
  },
  computed: {
    ...mapGetters("PrinterRoutersSingle", [
      "item",
      "loading",
      "continents",
      "countries",
      "regions",
      "printer_companies",
      "country_set_flag",
      "region_set_flag"
    ])
  },
  created() {
    this.init();
  },
  destroyed() {
    this.resetState();
  },
  watch: {
    "$route.params.id": function() {
      this.init();
    }
  },
  methods: {
    ...mapActions("PrinterRoutersSingle", [
      "updateData",
      "resetState",
      "fetchData",
      "fetchContinent",
      "fetchCountry",
      "fetchRegion",
      "fetchPrinterCompany",
      "setContinent",
      "setCountry",
      "setRegion",
      "setPrinterCompanies"
    ]),
    init() {
      this.resetState();
      this.fetchContinent();
      this.fetchPrinterCompany();
      this.fetchData(this.$route.params.id)
        .then(() => {
          for (var i in this.item.printer_companies) {
            var item = this.item.printer_companies[i];
            this.printerRelationShip.push({
              printer_company: {
                id: item.pivot.printer_company_id,
                name: item.name,
                logo_url: item.logo_url
              },
              percent: item.percent
            });
          }
          if (this.printerRelationShip.length == 0) {
            this.printerRelationShip.push({
              printer_company: null,
              percent: null
            });
          }
        })
        .catch(error => {
          console.error(error);
        });
    },
    addPrinterField() {
      if (this.printerRelationShip.length >= this.printer_companies.length) {
        return;
      }
      this.printerRelationShip.push({
        printer_company: null,
        percent: null
      });
    },
    removePrinterField(index) {
      if (index != 0) {
        this.printerRelationShip.splice(index, 1);
      }
    },
    updateContinent(value) {
      console.log(value);
      this.setContinent(value);
      if (!this.country_set_flag) {
        this.setCountry(null);
      }

      this.fetchCountry(value);
    },
    updateCountry(value) {
      console.log(value);
      this.setCountry(value);
      if (!this.region_set_flag) {
        this.setRegion(null);
      }
      this.fetchRegion(value);
    },
    updateRegion(value) {
      console.log(value);
      this.setRegion(value);
    },
    updatePrinterCompany(value) {
      // console.log(value);
      // this.setprinterCompany(value);
    },
    updatePercent(e) {
      // this.setPercent(e.target.value);
    },
    makeRelation() {
      var printerMap = [];
      var totalPercent = 0;
      for (var i in this.printerRelationShip) {
        var printer = this.printerRelationShip[i].printer_company;
        if (printer == null) {
          continue;
        }
        if (
          printerMap
            .map(function(e) {
              return e.id;
            })
            .indexOf(printer.id) != -1
        ) {
          alert(printer.name + " is duplicated!");
          return false;
        }
        printerMap.push({
          id: printer.id,
          percent: Number(this.printerRelationShip[i].percent)
        });

        totalPercent += Number(this.printerRelationShip[i].percent);
      }
      console.log(totalPercent);
      if (totalPercent != 100) {
        alert("Total percent must be 100! But now it is " + totalPercent);
        return false;
      }
      console.log(printerMap);
      this.setPrinterCompanies(printerMap);
      return true;
    },
    submitForm() {
      if (!this.makeRelation()) {
        return;
      }
      this.updateData()
        .then(() => {
          this.$router.push({ name: "printer_routers.index" });
          this.$eventHub.$emit("update-success");
        })
        .catch(error => {
          console.error(error);
        });
    }
  }
};
</script>


<style scoped>
.my-image {
  width: 50px;
  /* height: 50px; */
  display: inline-block;
  border-radius: 3px;
}
.remove-btn {
  font-size: 30px;
  cursor: pointer;
}
.add-btn {
  /* font-size: 18px; */
  cursor: pointer;
}
.printer-field {
  padding-bottom: 2px;
}
.custome-disabled {
  pointer-events: none;
  cursor: not-allowed;
}
</style>


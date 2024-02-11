<template>
  <div class="show_all_content_wrapper">
    <!-- Start:: Main Section -->
    <main>
      <!--  =========== Start:: Filter Form =========== -->
      <div class="filter_content_wrapper" :class="{ active: filterFormIsActive }">
        <button class="filter_toggler" @click="filterFormIsActive = !filterFormIsActive">
          <i class="fal fa-times"></i>
        </button>
        <div class="filter_title_wrapper">
          <h5>{{ $t("TITLES.searchBy") }}</h5>
        </div>
        <div class="filter_form_wrapper">
          <form @submit.prevent="submitFilterForm">
            <div class="row justify-content-center align-items-center w-100">

              <!-- Start:: Name Input -->
              <base-input col="6" type="number" :placeholder="$t('PLACEHOLDERS.report_number')"
                v-model.trim="filterOptions.report_number" />
              <!-- End:: Name Input -->

              <!-- Start:: Status Input -->
              <base-input col="6" type="text" :placeholder="$t('PLACEHOLDERS.reporter_name')"
                v-model.trim="filterOptions.reporter_name" />
              <!-- End:: Status Input -->
            </div>

            <div class="btns_wrapper">

              <a-tooltip placement="bottom">
                <template slot="title">
                  <span>{{ $t("BUTTONS.search") }}</span>
                </template>
                <span class="submit_btn" @click="submitFilterForm" :disabled="isWaitingRequest">
                  <i class="fal fa-search"></i>
                </span>
              </a-tooltip>

              <a-tooltip placement="bottom">
                <template slot="title">
                  <span>{{ $t("BUTTONS.rseet_search") }}</span>
                </template>
                <span class="reset_btn" :disabled="isWaitingRequest" @click="resetFilter">
                  <i class="fal fa-redo"></i>
                </span>
              </a-tooltip>

            </div>
          </form>
        </div>
      </div>
      <!--  =========== End:: Filter Form =========== -->

      <!--  =========== Start:: Table Title =========== -->
      <div class="table_title_wrapper">
        <div class="title_text_wrapper">
          <h5>{{ $t("PLACEHOLDERS.report_data_list") }}</h5>
          <button v-if="!filterFormIsActive" class="filter_toggler"
            @click.stop="filterFormIsActive = !filterFormIsActive">
            <i class="fal fa-search"></i>
          </button>
        </div>
      </div>
      <!--  =========== End:: Table Title =========== -->

      <!--  =========== Start:: Data Table =========== -->
      <v-data-table class="thumb" :loading="loading" :loading-text="$t('TABLES.loadingData')" :search="searchValue"
        :headers="tableHeaders" :items="tableRows" item-class="ltr" :items-per-page="paginations.items_per_page"
        hide-default-footer>
        <!-- Start:: No Data State -->
        <template v-slot:no-data>
          {{ $t("TABLES.noData") }}
        </template>
        <!-- Start:: No Data State -->

        <template v-slot:[`item.serialNumber`]="{ item, index }">
          <div class="table_image_wrapper">
            <h6 class="text-danger" v-if="!item.serialNumber"> {{ $t("TABLES.noData") }} </h6>
            <p v-else>{{ (paginations.current_page - 1) * paginations.items_per_page + index + 1 }}</p>
          </div>
        </template>

        <!-- Start:: Name -->
        <template v-slot:[`item.name`]="{ item }">
          <h6 class="text-danger" v-if="!item.name"> {{ $t("TABLES.noData") }} </h6>
          <h6 v-else> {{ item.name }} </h6>
        </template>
        <!-- End:: Name -->


        <!-- ======================== Start:: Dialogs ======================== -->

        <!-- ======================== End:: Dialogs ======================== -->

      </v-data-table>
      <!--  =========== End:: Data Table =========== -->
    </main>
    <!-- End:: Main Section -->

    <!-- Start:: Pagination -->
    <template>
      <div class="pagination_container text-center mt-3 mb-0">
        <v-pagination class="py-0" square v-model="paginations.current_page" :length="paginations.last_page"
          :total-visible="6" @input="updateRouterQueryParam($event)" :prev-icon="getAppLocale == 'ar' ? 'fal fa-angle-right' : 'fal fa-angle-left'
            " :next-icon="getAppLocale == 'ar' ? 'fal fa-angle-left' : 'fal fa-angle-right'
    " />
      </div>
    </template>
    <!-- End:: Pagination -->
  </div>
</template>

<script>
import { mapGetters } from "vuex";

export default {
  name: "AllClients",

  computed: {
    ...mapGetters({
      getAppLocale: "AppLangModule/getAppLocale",
    }),

    activeStatuses() {
      return [
        {
          id: 1,
          name: this.$t("STATUS.active"),
          value: 1,
        },
        {
          id: 2,
          name: this.$t("STATUS.notActive"),
          value: 0,
        },
        {
          id: null,
          name: this.$t("STATUS.all"),
          value: null,
        },
      ];
    },

    activeStatus_modal() {
      return [
        {
          id: 1,
          name: this.$t("STATUS.published"),
          value: "published",
        },
        {
          id: 2,
          name: this.$t("STATUS.notPublished"),
          value: "blocked",
        }
      ];
    },
  },

  data() {
    return {
      // Start:: Loading Data
      loading: false,
      isWaitingRequest: false,
      // End:: Loading Data

      // Start:: Filter Data
      filterFormIsActive: false,
      filterOptions: {
        report_number: null,
        reporter_name: null,
      },
      // End:: Filter Data

      // Start:: Table Data
      searchValue: "",
      tableHeaders: [
        {
          text: this.$t("TABLES.Rates.serialNumber"),
          value: "serialNumber",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.report_number"),
          value: "id",
          align: "center",
          // width: "100",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.reporter_name"),
          value: "reporter.name",
          align: "center",
          // width: "150",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.reporter_mobile_number"),
          value: "reporter.mobile",
          align: "center",
          // width: "150",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.report_reason"),
          value: "reason",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.report_date"),
          value: "created_at",
          align: "center",
          // width: "150",
          sortable: false,
        },
      ],
      tableRows: [],
      // End:: Table Data

      // Start:: Pagination Data
      paginations: {
        current_page: 1,
        last_page: 1,
        items_per_page: 6,
      },
      // End:: Pagination Data

      dialogDeactivate: false,
      itemToChangeActivationStatus: null,
      deactivateReason: null,

      permissions: null,
      // Start:: Page Permissions

    };
  },

  watch: {
    // Start:: Page Query Param Watcher To Get Page Data Based On It's Change
    ["$route.query.page"]() {
      this.paginations.current_page = +this.$route.query.page;
      this.setTableRows();
    },
    // End:: Page Query Param Watcher To Get Page Data Based On It's Change
  },

  methods: {
    // Start:: Handel Filter
    submitFilterForm() {
      if (this.$route.query.page !== '1') {
        this.$router.push({ path: `/reports/show/+${this.$route.params.id}`, query: { page: 1 } });
      }
      this.setTableRows();
    },
    async resetFilter() {
      this.filterOptions.report_number = null;
      this.filterOptions.reporter_name = null;
      if (this.$route.query.page !== '1') {
        await this.$router.push({ path: `/reports/show/+${this.$route.params.id}`, query: { page: 1 } });
      }
      this.setTableRows();
    },
    // End:: Handel Filter
    // Start:: Set Table Rows
    updateRouterQueryParam(pagenationValue) {
      this.$router.push({
        query: {
          ...this.$route.query,
          page: pagenationValue,
        },
      });

      // Scroll To Screen's Top After Get Products
      document.body.scrollTop = 0; // For Safari
      document.documentElement.scrollTop = 0; // For Chrome, Firefox, IE and Opera
    },
    async setTableRows() {
      this.loading = true;
      try {


        let res = await this.$axios({
          method: "GET",
          url: `advertisements/${this.$route.params.id}/reports`,
          params: {
            page: this.paginations.current_page,
            id: this.filterOptions.report_number,
            user_name: this.filterOptions.reporter_name,
            //"hasReport": 1
          },
        });
        this.loading = false;
        // console.log("All Data ==>", res.data.data);
        res.data.data.reports.forEach((item, index) => {
          item.serialNumber = (this.paginations.current_page - 1) * this.paginations.items_per_page + index + 1;
        });
        this.tableRows = res.data.data.reports;
        this.paginations.last_page = res.data.meta.last_page;
        this.paginations.items_per_page = res.data.meta.per_page;;
      } catch (error) {
        this.loading = false;
        console.log(error.response.data.message);
      }
    },
    // End:: Set Table Rows

    // ==================== Start:: Crud ====================
    // ===== Start:: Show

    // ===== End:: Show

    // ==================== End:: Crud ====================
  },

  created() {
    // Start:: Fire Methods
    window.addEventListener("click", () => {
      this.filterFormIsActive = false;
    });
    if (this.$route.query.page) {
      this.paginations.current_page = +this.$route.query.page;
    }
    this.setTableRows();
    // End:: Fire Methods
  },
};
</script>
<style>
span.submit_btn {
  width: 45px;
  height: 45px;
  font-size: 16px;
  border-radius: 10px;
  color: var(--white_clr);
  transition: all 0.3s linear;
  background-color: #F6A513;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
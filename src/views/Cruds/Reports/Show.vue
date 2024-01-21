<template>
  <div class="crud_form_wrapper">
    <!-- Start:: Main Section -->
    <main>

      <div class="single_step_form_content_wrapper">
        <form @submit.prevent="submitFilterForm">
          <div class="row justify-content-center align-items-center w-100">
            <!-- Start:: report_number Input -->
            <base-input col="6" type="text" :placeholder="$t('PLACEHOLDERS.report_number')"
              v-model.trim="data.report_number" disabled />
            <!-- End:: report_number Input -->

            <!-- Start:: Phone Input -->
            <base-input col="6" type="number" :placeholder="$t('PLACEHOLDERS.report_submitter')"
              v-model.trim="data.report_submitter" disabled />
            <!-- End:: Phone Input -->

            <!-- Start:: email Input -->
            <base-input col="6" type="number" :placeholder="$t('PLACEHOLDERS.ad_number_in_report')"
              v-model.trim="data.ad_number_in_report" disabled />
            <!-- End:: email Input -->

            <base-input col="6" type="text" :placeholder="$t('PLACEHOLDERS.ad_name')" v-model.trim="data.ad_name"
              disabled />

            <base-input col="6" type="text" :placeholder="$t('PLACEHOLDERS.advertiser_name')"
              v-model.trim="data.advertiser_name" disabled />

          </div>

        </form>
      </div>

      <!--  =========== Start:: Table Title =========== -->
      <div class="table_title_wrapper mt-5">
        <div class="title_text_wrapper">
          <h5>{{ $t("PLACEHOLDERS.report_data_list") }}</h5>
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

        <!-- Start:: Item Image -->
        <template v-slot:[`item.id`]="{ item, index }">
          <div class="table_image_wrapper">
            <h6 class="text-danger" v-if="!item.id"> {{ $t("TABLES.noData") }} </h6>
            <p v-else>{{ (paginations.current_page - 1) * paginations.items_per_page + index + 1 }}</p>
          </div>
        </template>

        <template v-slot:[`item.price`]="{ item, index }">
          <div class="table_image_wrapper">
            <h6 class="text-danger" v-if="!item.price"> {{ $t("TABLES.noData") }} </h6>
            <p v-else>{{ item.price }}</p>
          </div>
        </template>

        <template v-slot:[`item.message`]="{ item }">
          <template>
            <h6 class="text-danger" v-if="item.message.length === 0"> {{ $t("TABLES.noData") }} </h6>
            <div class="actions" v-else>
              <button class="btn_show" @click="showReplayModal(item.message)">
                <i class="fal fa-file-alt"></i>
              </button>
            </div>
          </template>
        </template>

        <!-- ======================== Start:: Dialogs ======================== -->
        <template v-slot:top>
          <!-- Start:: Replay Modal -->
          <description-modal v-if="dialogReplay" :modalIsOpen="dialogReplay" :modalDesc="selectedReplayTextToShow"
            @toggleModal="dialogReplay = !dialogReplay" />
          <!-- End:: Replay Modal -->

        </template>
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
  name: "AllOrders",

  computed: {
    ...mapGetters({
      getAppLocale: "AppLangModule/getAppLocale",
    }),

  },

  data() {
    return {
      // Start:: Loading Data
      loading: false,
      isWaitingRequest: false,
      // End:: Loading Data

      data: {
        report_number: null,
        report_submitter: null,
        ad_number_in_report: null,
        ad_name: null,
        advertiser_name: null,
      },

      // Start:: Table Data
      searchValue: "",
      tableHeaders: [
        {
          text: this.$t("TABLES.StoresTypes.serialNumber"),
          value: "id",
          align: "center",
        },
        {
          text: this.$t("PLACEHOLDERS.report_number"),
          value: "type",
          align: "center",
        },
        {
          text: this.$t("PLACEHOLDERS.report_submitter"),
          value: "price",
          align: "center",
        },
        {
          text: this.$t("PLACEHOLDERS.reporter_mobile_number"),
          value: "tax",
          align: "center",
        },
        {
          text: this.$t("PLACEHOLDERS.report_reason"),
          value: "total",
          align: "center",
        },
        {
          text: this.$t("PLACEHOLDERS.report_date"),
          value: "start_at",
          align: "center",
        },

      ],
      tableRows: [],
      // End:: Table Data

      // Start:: Dialogs Control Data
      dialogDelete: false,
      itemToDelete: null,
      dialogReplay: false,
      selectedReplayTextToShow: "",
      dialogSendReplay: false,
      itemToSendReplay: null,
      messageReplay: null,
      // End:: Dialogs Control Data]

      // Start:: Pagination Data
      paginations: {
        current_page: 1,
        last_page: 1,
        items_per_page: 6,
      },
      // End:: Pagination Data

      // Start:: Page Permissions
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
        const { query } = this.$route;
        let res = await this.$axios({
          method: "GET",
          url: `clients/subscriptions/${this.$route.params.id}`,
        });
        this.loading = false;
        this.tableRows = res.data.data;
        this.paginations.last_page = res.data.meta.last_page;
        this.paginations.items_per_page = res.data.meta.per_page;
        this.permissions = res.data.permissions;
      } catch (error) {
        this.loading = false;
        console.log(error.response.data.message);
      }
    },
    // End:: Set Table Rows

    // ==================== Start:: Crud ====================
 // Start:: Control Modals
    showReplayModal(replay) {
      this.dialogReplay = true;
      this.selectedReplayTextToShow = replay;
    },
    // End:: Control Modals
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
    // this.setTableRows();
    // End:: Fire Methods
  },
};
</script>

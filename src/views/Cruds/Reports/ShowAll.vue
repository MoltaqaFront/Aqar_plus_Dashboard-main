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
                <base-input col="6" type="number" :placeholder="$t('PLACEHOLDERS.ad_number_in_report')"
                  v-model.trim="filterOptions.ad_number_in_report" />
                <!-- End:: Name Input -->

                <!-- Start:: Status Input -->
                <base-input col="6" type="text" :placeholder="$t('PLACEHOLDERS.advertiser_name')"
                  v-model.trim="filterOptions.advertiser_name" />
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
          <h5>{{ $t("PLACEHOLDERS.reports_management") }}</h5>
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

        <!-- Start:: status Type -->
          <template v-slot:[`item.ad_status`]="{ item }">
            <h6 class="text-danger" v-if="!item.ad_status"> {{ $t("TABLES.noData") }} </h6>
            <v-chip v-else color="blue-grey darken-3" text-color="white" small>
              {{ item.ad_status }}
            </v-chip>
          </template>
          <!-- End:: status Type -->

        <!-- Start:: Activation Status -->
        <template v-slot:[`item.user.is_active`]="{ item }">
          <span class="text-success text-h5" v-if="item.user.is_active">
            <i class="far fa-check"></i>
          </span>
          <span class="text-danger text-h5" v-else>
            <i class="far fa-times"></i>
          </span>
        </template>
        <!-- End:: Activation Status -->

        <!-- Start:: Actions -->
        <template v-slot:[`item.actions`]="{ item }">
          <div class="actions">
            <a-tooltip placement="bottom" v-if="$can('clients show', 'clients')">
              <template slot="title">
                <span>{{ $t("BUTTONS.show") }}</span>
              </template>
              <button class="btn_show" @click="showItem(item)">
                <i class="fal fa-eye"></i>
              </button>
            </a-tooltip>

            <a-tooltip placement="bottom">
                <template slot="title">
                  <span>{{ $t("PLACEHOLDERS.status_share_change") }}</span>
                </template>

                <button class="btn_edit" @click="selectUpdateItem(item)">
                  <i class="fas fa-exchange-alt"></i>
                </button>
            </a-tooltip>

            <template v-if="$can('clients activate', 'clients') && item.user.id !== 1">
              <a-tooltip placement="bottom" v-if="!item.user.is_active">
                <template slot="title">
                  <span>{{ $t("BUTTONS.active_cli") }}</span>
                </template>
                <button class="btn_activate" @click="HandlingItemActivationStatus(item)">
                  <i class="fad fa-check-circle"></i>
                </button>
              </a-tooltip>
              <a-tooltip placement="bottom" v-if="item.user.is_active">
                <template slot="title">
                  <span>{{ $t("BUTTONS.block_cli") }}</span>
                </template>
                <button class="btn_deactivate" @click="selectDeactivateItem(item)">
                  <i class="fad fa-times-circle"></i>
                </button>
              </a-tooltip>
            </template>

            <template v-else>
              <i class="fal fa-lock-alt fs-5 blue-grey--text text--darken-1"></i>
            </template>
          </div>
        </template>
        <!-- End:: Actions -->

        <!-- ======================== Start:: Dialogs ======================== -->
        <template v-slot:top>

          <!-- Start:: Deactivate Modal -->
          <v-dialog v-model="dialogDeactivate">
            <v-card>
              <v-card-title class="text-h5 justify-center" v-if="itemToChangeActivationStatus">
                {{ $t("TITLES.DeactivateConfirmingMessage", { name: itemToChangeActivationStatus.user.name }) }}
              </v-card-title>

              <form class="w-100">
                <base-input col="12" rows="3" type="textarea" :placeholder="$t('PLACEHOLDERS.deactivateReason')"
                  v-model.trim="deactivateReason" required />
              </form>

              <v-card-actions>
                <v-btn class="modal_confirm_btn" @click="HandlingItemActivationStatus" :disabled="!(!!deactivateReason)">
                  {{ $t("BUTTONS.ok") }}
                </v-btn>

                <v-btn class="modal_cancel_btn" @click="dialogDeactivate = false">{{ $t("BUTTONS.cancel") }}</v-btn>
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <!-- End:: Deactivate Modal -->

           <!-- Start:: Update Modal -->
            <v-dialog v-model="dialogUpdate">
              <v-card>
                <v-card-title class="text-h5 justify-center w-100" v-if="itemToUpdate">
                  {{ $t("MESSAGES.changeItem", { name: itemToUpdate.id }) }}

                  <div class="filter_form_wrapper w-100">
                    <form class="w-100">
                      <base-select-input col="12" :optionsList="activeStatus_modal" :placeholder="$t('PLACEHOLDERS.status')"
                        v-model="status_modal" />

                      <div class="form-group" v-if="(status_modal && status_modal.value === 'blocked')">
                        <base-input col="12" rows="3" type="textarea" :placeholder="$t('PLACEHOLDERS.reason')"
                          v-model="reason" required />
                      </div>

                    </form>
                  </div>

                </v-card-title>
                <v-card-actions>
                  <v-btn class="modal_confirm_btn" @click="confirmChangeStatus">{{
                    $t("BUTTONS.ok")
                  }}</v-btn>

                  <v-btn class="modal_cancel_btn" @click="dialogUpdate = false">{{ $t("BUTTONS.cancel") }}</v-btn>
                  <v-spacer></v-spacer>
                </v-card-actions>
              </v-card>
            </v-dialog>
            <!-- End:: Update Modal -->

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
        ad_number_in_report: null,
        advertiser_name: null
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
          text: this.$t("PLACEHOLDERS.ad_number_in_report"),
          value: "id",
          align: "center",
          sortable: false,
          width: "100"
        },
        {
          text: this.$t("PLACEHOLDERS.advertiser_name"),
          value: "user.name",
          align: "center",
          sortable: false,
          width:"120"
        },
        {
          text: this.$t("PLACEHOLDERS.ad_phone_advertiser"),
          value: "user.mobile",
          align: "center",
          sortable: false,
          width: "120"
        },
        {
          text: this.$t("PLACEHOLDERS.number_of_reports"),
          value: "report_count",
          align: "center",
          sortable: false,
          width: "100"
        },
        {
          text: this.$t("TABLES.Clients.active"),
          value: "user.is_active",
          align: "center",
          width: "120",
          sortable: false,
        },
         {
          text: this.$t("PLACEHOLDERS.status_share"),
          value: "ad_status",
          align: "center",
          width: "120",
          sortable: false,
        },
        {
          text: this.$t("TABLES.Clients.actions"),
          value: "actions",
          sortable: false,
          align: "center",
        }
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

      // Start:: Page Permissions
     dialogUpdate: false,
      itemToUpdate: null,
      status_modal: '',
      reason: '',

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
    async submitFilterForm() {
      if (this.$route.query.page !== '1') {
        await this.$router.push({ path: '/reports/all', query: { page: 1 } });
      }
      this.setTableRows();
    },
    async resetFilter() {
      this.filterOptions.ad_number_in_report = null;
      this.filterOptions.advertiser_name = null;
      if (this.$route.query.page !== '1') {
        await this.$router.push({ path: '/reports/all', query: { page: 1 } });
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
          url: "advertisements",
          params: {
            page: this.paginations.current_page,
            id: this.filterOptions.ad_number_in_report,
            user_name: this.filterOptions.advertiser_name,
            "hasReport": 1
          },
        });
        this.loading = false;
        // console.log("All Data ==>", res.data.data);
         res.data.data.forEach((item, index) => {
          item.serialNumber = (this.paginations.current_page - 1) * this.paginations.items_per_page + index + 1;
        });
        this.tableRows = res.data.data;
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
    showItem(item) {
      this.$router.push({ path: `/reports/show/${item.id}` });
    },
    // ===== End:: Show

    // ===== Start:: Handling Activation & Deactivation
    selectDeactivateItem(item) {
      this.dialogDeactivate = true;
      this.itemToChangeActivationStatus = item;
    },
    async HandlingItemActivationStatus(selectedItem) {
      this.dialogDeactivate = false;
      let targetItem = this.itemToChangeActivationStatus ? this.itemToChangeActivationStatus : selectedItem;
      // Start:: Append Request Data
      
        const REQUEST_DATA = {};
        REQUEST_DATA.message = this.deactivateReason;  

      // Start:: Append Request Data
      try {
        await this.$axios({
          method: "POST",
          url: `clients/active/${targetItem.user.id}`,
          data: targetItem.user.is_active ? REQUEST_DATA : null,
        });
        this.$message.success(this.$t("MESSAGES.changeActivation"));
        this.setTableRows();
        this.itemToChangeActivationStatus = null;
        this.deactivateReason = null;
      } catch (error) {
        this.$message.error(error.response.data.message);
      }
    },
    // ===== End:: Handling Activation & Deactivation

    selectUpdateItem(item) {
      this.dialogUpdate = true;
      this.itemToUpdate = item;
      // console.log(item);
    },

    async confirmChangeStatus() {
      try {

        const requestData = {
          status: this.status_modal.value
        };

       

        if (this.itemToUpdate.ad_status == 'published') {
          requestData.status = "blocked";
           if (this.reason.trim() !== '') {
          requestData.status_reason = this.reason.trim();
        }
        } else {
          requestData.status = "published";
        }

       await this.$axios({
          method: "POST",
          url: `advertisements/${this.itemToUpdate.id}/change-status`,
          data: requestData // Put the data in the 'data' property
        });
        this.$message.success(this.$t("MESSAGES.share_ad"));
        this.dialogUpdate = false;
        this.setTableRows();
        this.status_modal = null;
      } catch (error) {
        this.dialogUpdate = false;
        this.status_modal = null;
        this.$message.error(error.response.data.message);
      }
    },


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
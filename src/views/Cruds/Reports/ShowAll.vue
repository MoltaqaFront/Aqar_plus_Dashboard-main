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
             
              <!-- Start:: ad_number_in_report Input -->
              <base-input col="6" type="number" :placeholder="$t('PLACEHOLDERS.ad_number_in_report')"
                v-model.trim="filterOptions.ad_number_in_report" />
              <!-- End:: ad_number_in_report Input -->

              <!-- Start:: advertiser_name Input -->
              <base-input col="6" type="text" :placeholder="$t('PLACEHOLDERS.advertiser_name')"
                v-model.trim="filterOptions.advertiser_name" />
              <!-- End:: advertiser_name Input -->

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

        <template v-slot:[`item.serialNumber`]="{ item }">
          <p class="blue-grey--text text--darken-1 fs-3" v-if="!item.serialNumber">-</p>
          <p v-else>{{ item.serialNumber }}</p>
        </template>

        <template v-slot:[`item.id`]="{ item }">
          <div class="table_image_wrapper">
            <h6 class="text-danger" v-if="!item.id"> {{ $t("TABLES.noData") }} </h6>
            <p v-else>{{ item.id }}</p>
          </div>
        </template>

        <!-- Start:: Name -->
        <template v-slot:[`item.name`]="{ item }">
          <h6 class="text-danger" v-if="!item.name"> {{ $t("TABLES.noData") }} </h6>
          <h6 v-else> {{ item.name }} </h6>
        </template>
        <!-- End:: Name -->

        <!-- Start:: email -->
        <template v-slot:[`item.email`]="{ item }">
          <h6 class="text-danger" v-if="!item.email"> - </h6>
          <h6 v-else> {{ item.email }} </h6>
        </template>
        <!-- End:: email -->

        <template v-slot:[`item.is_verified`]="{ item }">
          <v-chip :color="item.is_verified ? 'green' : 'red'" text-color="white" small>
            <template v-if="item.is_verified">
              {{ $t("STATUS.active") }}
            </template>
            <template v-else>
              {{ $t("STATUS.notActive") }}
            </template>
          </v-chip>
        </template>

        
        <!-- Start:: Message Reply -->
        <template v-slot:[`item.reply_message`]="{ item }">
          <template v-if="$can('contactuses reply', 'contactuses')">
            <h6 class="text-danger" v-if="!item.reply_message"> {{ $t("TABLES.noData") }} </h6>
            <div class="actions" v-else>
              <button class="btn_show" @click="showReplayModal(item.reply_message)">
                <i class="fal fa-file-alt"></i>
              </button>
            </div>
          </template>

          <template v-else>
            <i class="fal fa-lock-alt fs-5 blue-grey--text text--darken-1"></i>
          </template>
          </template>
          <!-- End:: Message Reply -->

        <!-- Start:: Activation Status -->
        <template v-slot:[`item.is_active`]="{ item }">
          <span class="text-success text-h5" v-if="item.is_active">
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

            <template v-if="$can('contactuses reply', 'contactuses')">
              <span class="blue-grey--text text--darken-1" v-if="item.is_reply">
                <i class="far fa-horizontal-rule"></i>
              </span>
              <a-tooltip placement="bottom" v-else>
                <template slot="title">
                  <span>{{ $t("BUTTONS.share") }}</span>
                </template>
                <button class="btn_show" @click="selectItemToSendReplay(item)">
                  <i class="fal fa-reply"></i>
                </button>
              </a-tooltip>
            </template>
            
            <template v-else>
              <i class="fal fa-lock-alt fs-5 blue-grey--text text--darken-1"></i>
            </template>

            <template v-if="$can('users activate', 'users') && item.id !== 1">
              <a-tooltip placement="bottom" v-if="!item.is_active">
                <template slot="title">
                  <span>{{ $t("BUTTONS.activate") }}</span>
                </template>
                <button class="btn_activate" @click="HandlingItemActivationStatus(item)">
                  <i class="fad fa-check-circle"></i>
                </button>
              </a-tooltip>
              <a-tooltip placement="bottom" v-if="item.is_active">
                <template slot="title">
                  <span>{{ $t("BUTTONS.deactivate") }}</span>
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
                {{ $t("TITLES.DeactivateConfirmingMessage", { name: itemToChangeActivationStatus.name }) }}
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


           <!-- Start:: Replay Modal -->
           <description-modal v-if="dialogReplay" :modalIsOpen="dialogReplay" :modalDesc="selectedReplayTextToShow"
            @toggleModal="dialogReplay = !dialogReplay" />
          <!-- End:: Replay Modal -->

          <!-- Start:: Replay Modal -->
          <v-dialog v-model="dialogSendReplay">
            <v-card>
              <v-card-title class="text-h5 justify-center">
                {{ $t("TITLES.sendReplay") }}
              </v-card-title>

              <form class="w-100">
                <base-input col="12" rows="3" type="textarea" :placeholder="$t('PLACEHOLDERS.messageReplay')"
                  v-model.trim="messageReplay" required />
              </form>

              <v-card-actions>
                <v-btn class="modal_confirm_btn" @click="sendReplay" :disabled="!(!!messageReplay)">
                  {{ $t("BUTTONS.replay") }}
                </v-btn>

                <v-btn class="modal_cancel_btn" @click="dialogSendReplay = false">{{ $t("BUTTONS.cancel") }}</v-btn>
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <!-- End:: Replay Modal -->
          <!-- Start:: Balance Modal -->
          <!-- <v-dialog v-model="dialogBalance">
            <v-card>
              <v-card-title class="text-h5 justify-center" v-if="itemToBalance">
                <span>{{ $t('PLACEHOLDERS.current_balance') }} : </span>
                <span>{{ itemToBalance.balance }}</span>
              </v-card-title>

              <form class="w-100">

                <base-input col="12" type="text" :placeholder="$t('PLACEHOLDERS.current_balance')"
                  v-model.trim="balance_package" required />
              </form>

              <v-card-actions>
                <v-btn class="modal_confirm_btn" @click="confirmAcceptItem">{{
                  $t("BUTTONS.ok")
                }}</v-btn>

                <v-btn class="modal_cancel_btn" @click="dialogBalance = false">{{ $t("BUTTONS.cancel") }}</v-btn>
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog> -->
          <!-- End:: Balance Modal -->

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
        advertiser_name: null,
        isActive: null,
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
        },
        {
          text: this.$t("PLACEHOLDERS.advertiser_name"),
          value: "user.name",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.ad_phone_advertiser"),
          value: "user.mobile",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.number_of_reports"),
          value: "report_count",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("TABLES.Clients.active"),
          value: "is_active",
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

      dialogBalance: false,
      itemToBalance: null,
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
            "hasPeport" : 1
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
      const REQUEST_DATA = {};
      // Start:: Append Request Data
      REQUEST_DATA.message = this.deactivateReason;
      // Start:: Append Request Data

      try {
        await this.$axios({
          method: "POST",
          url: `bank-transfers/${targetItem.id}/change-status`,
          data: targetItem.is_active ? REQUEST_DATA : null,
        });
        this.$message.success(this.$t("MESSAGES.changeActivation"));
        let filteredElemet = this.tableRows.find(element => element.id === targetItem.id);
        filteredElemet.is_active = !filteredElemet.is_active;
        this.itemToChangeActivationStatus = null;
        this.deactivateReason = null;
      } catch (error) {
        this.$message.error(error.response.data.message);
      }
    },
    // ===== End:: Handling Activation & Deactivation

    selectAcceptItem(item) {
      this.dialogBalance = true;
      this.itemToBalance = item;
    },
    async confirmAcceptItem(item) {

      const REQUEST_DATA = new FormData();
      REQUEST_DATA.append("balance", this.balance_package);
      // REQUEST_DATA.append("_method", "PUT");

      try {
        await this.$axios({
          method: "POST",
          url: `change-client-balance/${this.itemToBalance.id}`,
          data: REQUEST_DATA,
        });
        this.dialogBalance = false;
        this.balance_package = null,
          this.setTableRows();
        this.$message.success(this.$t("MESSAGES.verifiedSuccessfully"));
      } catch (error) {
        this.dialogBalance = false;
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
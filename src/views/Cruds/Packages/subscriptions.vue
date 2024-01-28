<template>
  <div class="show_all_content_wrapper">
    <!-- Start:: Main Section -->
    <main>

      <!--  =========== Start:: Table Title =========== -->
      <div class="table_title_wrapper">
        <div class="title_text_wrapper">
          <h5>{{ $t("TITLES.subscriptions") }}</h5>
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

        <!-- Start:: Name -->
        <template v-slot:[`item.name`]="{ item }">
          <span class="text-danger" v-if="!item.name"> {{ $t("TABLES.noData") }} </span>
          <span v-else> {{ item.name }} </span>
        </template>
        <!-- End:: Name -->

        <!-- Start:: Phone -->
        <template v-slot:[`item.phone`]="{ item }">
          <span class="text-danger" v-if="!item.phone"> {{ $t("TABLES.noData") }} </span>
          <span v-else> {{ item.phone }} </span>
        </template>
        <!-- End:: Phone -->

        <!-- Start:: Actions -->
        <template v-slot:[`item.actions`]="{ item }">
          <div class="actions">
            <a-tooltip placement="bottom">
              <template slot="title">
                <span>{{ $t("PLACEHOLDERS.current_balance") }}</span>
              </template>

              <button class="btn_edit" @click="selectAcceptItem(item)">
                <i class="fal fa-wallet"></i>
              </button>
            </a-tooltip>
          </div>
        </template>
        <!-- End:: Actions -->

        <!-- ======================== Start:: Dialogs ======================== -->
        <template v-slot:top>
           <!-- Start:: Balance Modal -->
           <v-dialog v-model="dialogBalance">
            <v-card>
              <v-card-title class="text-h5 justify-center" v-if="itemToBalance">
                <span>{{ $t('PLACEHOLDERS.current_balance') }} : </span>
                <span>{{ itemToBalance.balance }}</span>
              </v-card-title>

              <form class="w-100">

                <base-input col="12" type="text" :placeholder="$t('PLACEHOLDERS.Remaining_advertising')"
                  v-model.trim="Remaining_advertising" disabled />
                <base-input col="12" type="text" :placeholder="$t('PLACEHOLDERS.Consumer_advertising')"
                  v-model.trim="Consumer_advertising" disabled />
                <base-input col="12" type="text" :placeholder="$t('PLACEHOLDERS.packega_balence')"
                  v-model.trim="balance_package" disabled />
              </form>

            </v-card>
          </v-dialog>
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
  name: "AllSubscriptons",

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
        name: null,
        phone: null,
        email: null,
        isActive: null,
      },
      // End:: Filter Data

      // Start:: Table Data
      searchValue: "",
      tableHeaders: [
        {
          text: this.$t("TABLES.subscriptions.serialNumber"),
          value: "id",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("TABLES.subscriptions.name"),
          value: "user_name",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("TABLES.subscriptions.phone"),
          value: "user_mobile",
          align: "center",
          sortable: false,
        },
        {
          text: this.$t("TABLES.subscriptions.packagePrice"),
          value: "balance",
          align: "center",
          width: "150",
          sortable: false,
        },
        {
          text: this.$t("TABLES.subscriptions.AddedTax"),
          value: "tax",
          align: "center",
          width: "150",
          sortable: false,
        },
        {
          text: this.$t("TABLES.subscriptions.totalSubscription"),
          value: "total_balance",
          align: "center",
          width: "200",
          sortable: false,
        },
        {
          text: this.$t("TABLES.subscriptions.starSubscription"),
          value: "start_date",
          align: "center",
          width: "150",
          sortable: false,
        },
        {
          text: this.$t("TABLES.subscriptions.endSubscription"),
          value: "end_date",
          align: "center",
          width: "150",
          sortable: false,
        },
        {
          text: this.$t("TABLES.subscriptions.UserBalance"),
          value: "actions",
          sortable: false,
          align: "center",
          width: "200"
        },
      ],
      tableRows: [],
      // End:: Table Data

      // Start:: Dialogs Control Data
      dialogImage: false,
      selectedItemImage: null,
      dialogDeactivate: false,
      itemToChangeActivationStatus: null,
      deactivateReason: null,
      dialogDelete: false,
      itemToDelete: null,
      dialogBalance: false,
      itemToBalance: null,
      // End:: Dialogs Control Data

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
      Remaining_advertising:'',
      Consumer_advertising:'',
      balance_package:'',
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
        await this.$router.push({ path: '/subscriptions/:id', query: { page: 1 } });
      }
      this.setTableRows();
    },
    async resetFilter() {
      this.filterOptions.user_name = null;
      this.filterOptions.user_mobile = null;
      this.filterOptions.balance = null;
      this.filterOptions.tax = null;
      this.filterOptions.total_balance = null;
      this.filterOptions.start_date = null;
      this.filterOptions.end_date = null;
      this.filterOptions.package_id = null
      if (this.$route.query.page !== '1') {
        await this.$router.push({ path: '/subscriptions/:id', query: { page: 1 } });
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
          url: `packages/subscriptions/${this.$route.params.id}`,
          params: {
            page: this.paginations.current_page,
            name: this.filterOptions.user_name,
            phone: this.filterOptions.user_mobile,
            packagePrice: this.filterOptions.balance,
            AddedTax: this.filterOptions.tax,
            totalSubscription :this.filterOptions.total_balance,
            starSubscription: this.filterOptions.start_date,
            endSubscription: this.filterOptions.end_date,
            UserBalance:  this.filterOptions.package_id,
            status: this.filterOptions.isActive?.value,
          },
        });
        this.loading = false;
        this.tableRows = res.data.data.subscriptions;
        this.paginations.last_page = res.data.meta.last_page;
        this.paginations.items_per_page = res.data.meta.per_page;

      } catch (error) {
        this.loading = false;
        console.log(error.response.data.message);
      }
    },
    // End:: Set Table Rows

    // Start:: Control Modals
    showImageModal(image) {
      this.dialogImage = true;
      this.selectedItemImage = image;
    },
    // End:: Control Modals

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
          url: `packages/active/${targetItem.id}`,
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

    // ==================== Start:: Crud ====================
    // ===== Start:: balance
    selectAcceptItem(item) {
      console.log("item",item);
      this.dialogBalance = true;
      this.itemToBalance = item;

      this.Remaining_advertising = item.user_balance.Remaining_advertising_balance;
      this.Consumer_advertising = item.user_balance.Consumer_advertising_balance;
      this.balance_package = item.user_balance.package;

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
    // ===== End:: balance
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

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
              <base-input col="4" type="text" :placeholder="$t('TABLES.BankTransferManagement.bank_name')" v-model.trim="filterOptions.bankname" />
              <!-- End:: Name Input -->

              <!-- Start:: Email Input -->
              <base-input col="4" type="text" :placeholder="$t('TABLES.BankTransferManagement.name')"
                v-model.trim="filterOptions.name" />
              <!-- End:: Email Input -->

              <!-- Start:: Phone Input -->
              <base-input col="4" type="tel" :placeholder="$t('TABLES.BankTransferManagement.phone')"
                v-model.trim="filterOptions.mobile" />
              <!-- End:: Phone Input -->

              <!-- Start:: Phone Input -->
              <base-select-input col="4" :optionsList="regions" :placeholder="$t('TABLES.Areas.name')"
                v-model="filterOptions.region_id" />
             <!--  <base-input col="4" type="tel" :placeholder="$t('TABLES.BankTransferManagement.bankTransfer')"
                v-model.trim="filterOptions.transformer_name" /> -->
              <!-- End:: Phone Input -->

              <!-- Start:: Status Input -->
              <base-select-input col="4" :optionsList="status" :placeholder="$t('PLACEHOLDERS.transfer_status')"
                v-model="filterOptions.status" />
              <!-- End:: Status Input -->

            </div>

            <div class="btns_wrapper">
              <button class="submit_btn" :disabled="isWaitingRequest">
                <i class="fal fa-search"></i>
              </button>
              <button class="reset_btn" type="button" :disabled="isWaitingRequest" @click="resetFilter">
                <i class="fal fa-redo"></i>
              </button>
            </div>
          </form>
        </div>
      </div>
      <!--  =========== End:: Filter Form =========== -->

      <!--  =========== Start:: Table Title =========== -->
      <div class="table_title_wrapper">
        <div class="title_text_wrapper">
          <h5>{{ $t("PLACEHOLDERS.BankTransferManagement") }}</h5>
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

        <!-- Start:: Item Image -->
        <template v-slot:[`item.receipt_image`]="{ item }">
          <div class="table_image_wrapper">
            <h6 class="text-danger" v-if="!item.receipt_image"> {{ $t("TABLES.noData") }} </h6>

            <button class="my-1" @click="showImageModal(item.receipt_image)" v-else>
              <img class="rounded" :src="item.receipt_image" :alt="item.name" width="60" height="60" />
            </button>
          </div>
        </template>
        <!-- End:: Item Image -->

        <!-- Start:: Item Image -->
        <template v-slot:[`item.id`]="{ item, index }">
          <div class="table_image_wrapper">
            <h6 class="text-danger" v-if="!item.id"> {{ $t("TABLES.noData") }} </h6>
            <p v-else>{{ (paginations.current_page - 1) * paginations.items_per_page + index + 1 }}</p>
          </div>
        </template>
        <!-- End:: Item Image -->
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
          <!-- Start:: Image Modal -->
          <image-modal v-if="dialogImage" :modalIsOpen="dialogImage" :modalImage="selectedItemImage"
            @toggleModal="dialogImage = !dialogImage" />
          <!-- End:: Image Modal -->

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

          <!-- Start:: Delete Modal -->
          <v-dialog v-model="dialogDelete">
            <v-card>
              <v-card-title class="text-h5 justify-center" v-if="itemToDelete">
                {{ $t("TITLES.DeleteConfirmingMessage", { name: itemToDelete.name }) }}
              </v-card-title>
              <v-card-actions>
                <v-btn class="modal_confirm_btn" @click="confirmDeleteItem">{{
                  $t("BUTTONS.ok")
                }}</v-btn>

                <v-btn class="modal_cancel_btn" @click="dialogDelete = false">{{ $t("BUTTONS.cancel") }}</v-btn>
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <!-- End:: Delete Modal -->
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
  name: "AllAdmins",

  computed: {
    ...mapGetters({
      getAppLocale: "AppLangModule/getAppLocale",
    }),

    status() {
      return [
        {
          id: 1,
          name: this.$t("PLACEHOLDERS.transfer_wait"),
          value: "1",
        },
        {
          id: 2,
          name: this.$t("PLACEHOLDERS.transfer_confirmed"),
          value: 0,
        },
        {
          id: 1,
          name: this.$t("PLACEHOLDERS.transfer_not_confirmed"),
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
        mobile: null,
        transformer_name: null,
        bankname: null,
        iban: null,
        transfer_amount: null,
       // receipt_image: null,
        is_active: null,
        created_at: null,
        region_id: null
      },
      // End:: Filter Data

      // Start:: Table Data
      searchValue: "",
      tableHeaders: [
        {
          text: this.$t("TABLES.BankTransferManagement.serialNumber"),
          value: "serialNumber",
          align: "center",
          width: "80",
          sortable: false
        },
        {
          text: this.$t("TABLES.BankTransferManagement.name"),
          value: "user.name",
          align: "center",
          sortable: false
        },
        {
          text: this.$t("TABLES.BankTransferManagement.phone"),
          value: "user.mobile",
          align: "center",
          sortable: false
        },
        {
          text: this.$t("TABLES.BankTransferManagement.bank_name"),
          value: "bank.name",
          align: "center",
          width: "100",
          sortable: false
        },
        {
          text: this.$t("TABLES.BankTransferManagement.bankTransfer"),
          value: "transformer_name",
          align: "center",
          width: "100",
          sortable: false
        },
        {
          text: this.$t("TABLES.BankTransferManagement.number"),
          value: "iban",
          width: "100",
          align: "center",
          sortable: false
        },
        {
          text: this.$t("TABLES.BankTransferManagement.money"),
          value: "transfer_amount",
          align: "center",
          width: "100",
          sortable: false
        },
        {
          text: this.$t("TABLES.BankTransferManagement.image"),
          value: "receipt_image",
          align: "center",
          sortable: false
        },
        {
          text: this.$t("TABLES.BankTransferManagement.data"),
          value: "created_at",
          align: "center",
          width: "150",
          sortable: false
        },
        {
          text: this.$t("TABLES.BankTransferManagement.status"),
          value: "is_active",
          align: "center",
          sortable: false
        },
        {
          text: this.$t("TABLES.BankTransferManagement.actions"),
          value: "actions",
          sortable: false,
          align: "center",
        }
      ],
      tableRows: [],
      regions: [],
      // End:: Table Data

       // Start:: Dialogs Control Data
      dialogImage: false,
      selectedItemImage: null,
      dialogDeactivate: false,
      itemToChangeActivationStatus: null,
      deactivateReason: null,
      dialogDelete: false,
      itemToDelete: null,
      // End:: Dialogs Control Data

      // Start:: Pagination Data
      paginations: {
        current_page: 1,
        last_page: 1,
        items_per_page: 6,
      },
      // End:: Pagination Data

      permissions: null,
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
        await this.$router.push({ path: '/bankTransfer/all', query: { page: 1 } });
      }
      this.setTableRows();
    },
    async resetFilter() {
      this.filterOptions.name = null;
      this.filterOptions.mobile = null;
      this.filterOptions.transformer_name = null;
      this.filterOptions.bankname = null;
      this.filterOptions.iban = null;
      this.filterOptions.transfer_amount = null;
      this.filterOptions.receipt_image = null;
      this.filterOptions.created_at = null;
      this.filterOptions.is_active = null;
      this.filterOptions.region_id = null;
      if (this.$route.query.page !== '1') {
        await this.$router.push({ path: '/bankTransfer/all', query: { page: 1 } });
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
          url: "bank-transfers",
          params: {
            page: this.paginations.current_page,
            name: this.filterOptions.name,
            phone: this.filterOptions.mobile,
            bank_name: this.filterOptions.bankname,
            area_id: this.filterOptions.region_id?.id,
            is_active: this.filterOptions.is_active?.value,
          },
        });
        this.loading = false;
        res.data.data.forEach((item, index) => {
          item.serialNumber = (this.paginations.current_page - 1) * this.paginations.items_per_page + index + 1;
        });
        this.tableRows = res.data.data;
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
          url: `users/active/${targetItem.id}`,
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
    // ===== Start:: End
   
    // ===== Start:: Delete
    selectDeleteItem(item) {
      this.dialogDelete = true;
      this.itemToDelete = item;
    },
    async confirmDeleteItem() {
      try {
        await this.$axios({
          method: "DELETE",
          url: `users/${this.itemToDelete.id}`,
        });
        this.dialogDelete = false;
        this.tableRows = this.tableRows.filter((item) => {
          return item.id != this.itemToDelete.id;
        });
        this.setTableRows();
        this.$message.success(this.$t("MESSAGES.deletedSuccessfully"));
      } catch (error) {
        this.dialogDelete = false;
        this.$message.error(error.response.data.message);
      }
    },
    // ===== End:: Delete

    async getRegions() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: `users`,
        });
        // console.log("Cities =>", res.data.data);
        this.regions = res.data.data;
      } catch (error) {
        console.log(error.response.data.message);
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
    this.getRegions()
    // End:: Fire Methods
  },
};
</script>

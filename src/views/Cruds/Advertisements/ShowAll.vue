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

              <base-input col="3" type="text" :placeholder="$t('PLACEHOLDERS.ad_number')"
                v-model.trim="filterOptions.ad_number" />

              <base-input col="3" type="text" :placeholder="$t('PLACEHOLDERS.user_name')"
                v-model.trim="filterOptions.user_name" />

              <base-select-input col="3" :optionsList="Types" :placeholder="$t('PLACEHOLDERS.ad_section')"
                v-model="filterOptions.type" />

              <base-select-input col="3" :optionsList="propertySections"
                :placeholder="$t('PLACEHOLDERS.property_section')" v-model="filterOptions.property_section" />

              <!-- Start:: Name Input -->
              <base-select-input col="3" :placeholder="$t('TABLES.Areas.name')" :optionsList="getAreasData"
                v-model.trim="filterOptions.area_id" @input="getCountries" />
              <!-- End:: Name Input -->

              <!-- {{ getCountriesData }} -->
              <base-select-input col="3" v-if="CountriesData" :optionsList="CountriesData"
                :placeholder="$t('SIDENAV.Cities.name')" v-model="filterOptions.country_id" @input="getDistricts" />

              <base-select-input col="3" v-if="Districts" :optionsList="Districts"
                :placeholder="$t('PLACEHOLDERS.neighborhood_name')" v-model.trim="filterOptions.district_id" />

              <base-input col="3" type="date" :placeholder="$t('PLACEHOLDERS.startDate')"
                v-model.trim="filterOptions.startDate" />

              <base-input col="3" type="date" :placeholder="$t('PLACEHOLDERS.endDate')"
                v-model.trim="filterOptions.endDate" />

              <!-- Start:: Status Input -->
              <base-select-input col="3" :optionsList="activeStatuses" :placeholder="$t('PLACEHOLDERS.status')"
                v-model="filterOptions.is_active" />
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
          <h5>{{ $t("PLACEHOLDERS.ads_management") }}</h5>
          <button v-if="!filterFormIsActive" class="filter_toggler"
            @click.stop="filterFormIsActive = !filterFormIsActive">
            <i class="fal fa-search"></i>
          </button>
        </div>
        <div class="title_route_wrapper">
          <base-button class="mt-0 pdf_btn" styleType="solid_btn" :btnText="$t('PLACEHOLDERS.download_pdf')"
            @fireClick="downloadPdf" :disabled="pdfDownloadBtnIsLoading">
            <template v-slot:btn_icon>
              <i class="fal fa-file-pdf"></i>
            </template>
          </base-button>
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

        <!-- Start:: Name -->
        <template v-slot:[`item.provider.name`]="{ item }">
          <h6 class="text-danger" v-if="!item.provider.name"> {{ $t("TABLES.noData") }} </h6>
          <h6 v-else> {{ item.provider.name }} </h6>
        </template>

        <!-- Start:: status Type -->
        <template v-slot:[`item.status`]="{ item }">
          <h6 class="text-danger" v-if="!item.status"> {{ $t("TABLES.noData") }} </h6>
          <v-chip v-else color="blue-grey darken-3" text-color="white" small>
            {{ item.status }}
          </v-chip>
        </template>
        <!-- End:: status Type -->

        <!-- Start:: Activation -->
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
        <!-- End:: Activation -->

        <!-- Start:: Actions -->
        <template v-slot:[`item.actions`]="{ item }">
          <div class="actions">

            <a-tooltip placement="bottom">
              <template slot="title">
                <span>{{ $t("PLACEHOLDERS.change_status") }}</span>
              </template>

              <button class="btn_edit" @click="selectUpdateItem(item)">
                <i class="fas fa-exchange-alt"></i>
              </button>
            </a-tooltip>

            <a-tooltip placement="bottom" v-if="$can('advertisements show', 'advertisements')">
              <template slot="title">
                <span>{{ $t("BUTTONS.show") }}</span>
              </template>
              <button class="btn_show" @click="showItem(item)">
                <i class="fal fa-eye"></i>
              </button>
            </a-tooltip>

            <!-- <a-tooltip placement="bottom" v-if="$can('advertisements edit', 'advertisements')">
              <template slot="title">
                <span>{{ $t("BUTTONS.edit") }}</span>
              </template>
              <button class="btn_edit" @click="editItem(item)">
                <i class="fal fa-edit"></i>
              </button>
            </a-tooltip> -->

            <template v-else>
              <i class="fal fa-lock-alt fs-5 blue-grey--text text--darken-1"></i>
            </template>
          </div>
        </template>
        <!-- End:: Actions -->

        <!-- ======================== Start:: Dialogs ======================== -->
        <template v-slot:top>

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

          <!-- Start:: Deactivate Modal -->
          <v-dialog v-model="dialogDeactivate">
            <v-card>
              <v-card-title class="text-h5 justify-center" v-if="itemToChangeActivationStatus">
                {{ $t("TITLES.rejectUpdateConfirmingMessage", { name: itemToChangeActivationStatus.provider.name }) }}
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

    <!-- Start:: Generate PDF Template Content -->
    <vue-html2pdf :show-layout="false" :float-layout="true" :enable-download="true" :preview-modal="true"
      :filename="$t('PLACEHOLDERS.ads_management')" :pdf-quality="2" pdf-format="a4" :manual-pagination="false"
      :paginate-elements-by-height="1400" pdf-content-width="100%" @progress="bdfDownloadBtnIsLoading = true"
      @hasGenerated="$message.success($t('MESSAGES.generatedSuccessfully'))" ref="html2Pdf">
      <section slot="pdf-content">
        <div class="pdf_file_content">
          <h1 class="file_title"> {{ $t('PLACEHOLDERS.ads_management') }} </h1>

          <v-simple-table>
            <template v-slot:default>
              <thead>
                <tr>
                  <th v-for="(header, index) in tableHeaders" :key="header.value">
                    <!-- {{ index < tableHeaders.length - 1 ? header.text : '' }}  -->
                    {{ index < tableHeaders.length - 1 ? header.text : '' }} </th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="row in tableRows" :key="row.id">
                  <td>{{ row.serialNumber ? row.serialNumber : '-' }}</td>
                  <td>{{ row.id ? row.id : '-' }}</td>
                  <td>{{ row.category ? row.category : '-' }}</td>
                  <td>{{ row.real_estate_department.name ? row.real_estate_department.name : '-' }}</td>
                  <td>{{ row.user.name ? row.user.name : '-' }}</td>
                  <td>{{ row.address ? row.address : '-' }}</td>
                  <td>{{ row.status ? row.status : '-' }}</td>
                  <td>{{ row.created_at ? row.created_at : '-' }}</td>
                </tr>
              </tbody>
            </template>
          </v-simple-table>
        </div>
      </section>
    </vue-html2pdf>
    <!-- End:: Generate PDF Template Content -->

  </div>
</template>

<script>
import VueHtml2pdf from 'vue-html2pdf';
import { mapGetters } from "vuex";

export default {
  name: "AllClients",
  components: {
    VueHtml2pdf
  },

  computed: {
    ...mapGetters({
      getAppLocale: "AppLangModule/getAppLocale",
    }),

    activeStatuses() {
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
        },
        {
          id: 3,
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

    Types() {
      return [
        {
          id: 1,
          name: this.$t("PLACEHOLDERS.for_sale"),
          value: "for_sale",
        },
        {
          id: 2,
          name: this.$t("PLACEHOLDERS.for_rent"),
          value: "for_rent",
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
        ad_number: null,
        user_name: null,
        type: null,
        property_section: null,
        area_id: null,
        country_id: null,
        district_id: null,
        startDate: null,
        endDate: null,
        is_active: null,
      },
      propertySections: [],
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
          text: this.$t("PLACEHOLDERS.ad_number"),
          value: "id",
          align: "center",
          width: "100",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.ad_section"),
          value: "category",
          align: "center",
          width: "100",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.property_section"),
          value: "real_estate_department.name",
          align: "center",
          width: "100",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.user_name"),
          value: "user.name",
          align: "center",
          width: "120",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.address"),
          value: "address",
          align: "center",
          width: "250",
          sortable: false,
        },
        {
          text: this.$t("PLACEHOLDERS.status"),
          value: "status",
          align: "center",
          sortable: false,
        },

        {
          text: this.$t("PLACEHOLDERS.joiningDate"),
          value: "created_at",
          align: "center",
          width: "120",
          sortable: false,
        },
        {
          text: this.$t("TABLES.Clients.actions"),
          value: "actions",
          sortable: false,
          align: "center",
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
      dialogDelete: false,
      itemToDelete: null,

      getAreasData: [],
      CountriesData: [],
      Districts: [],

      // Start:: Dialogs Control Data
      dialogUpdate: false,
      itemToUpdate: null,
      status_modal: '',
      reason: ''
      // End:: Dialogs Control Data

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
        await this.$router.push({ path: '/advertisements/all', query: { page: 1 } });
      }
      this.setTableRows();
    },
    async resetFilter() {

      this.filterOptions.ad_number = null;
      this.filterOptions.user_name = null;
      this.filterOptions.property_section = null;
      this.filterOptions.area_id = null;
      this.filterOptions.country_id = null;
      this.filterOptions.district_id = null;
      this.filterOptions.startDate = null;
      this.filterOptions.endDate = null;
      this.filterOptions.is_active = null;

      if (this.$route.query.page !== '1') {
        await this.$router.push({ path: '/advertisements/all', query: { page: 1 } });
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
            id: this.filterOptions.ad_number,
            user_name: this.filterOptions.user_name,
            real_estate_department_id: this.filterOptions.property_section?.id,
            area_id: this.filterOptions.area_id?.id,
            country_id: this.filterOptions.country_id?.id,
            district_id: this.filterOptions.district_id?.id,
            start_date: this.filterOptions.startDate,
            end_date: this.filterOptions.endDate,
            status: this.filterOptions.is_active?.value,
            category: this.filterOptions.type?.value
          },
        });
        this.loading = false;
        // console.log("All Data ==>", res.data.data);
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
          url: `providerUpdates/reject/${targetItem.id}`,
          data: REQUEST_DATA,
        });
        this.$message.error(this.$t("MESSAGES.reject_edit_request"));
        this.setTableRows();
        this.itemToChangeActivationStatus = null;
        this.deactivateReason = null;
      } catch (error) {
        this.$message.error(error.response.data.message);
      }
    },
    // ===== End:: Handling Activation & Deactivation

    // ==================== Start:: Crud ====================
    // ===== Start:: Show
    showItem(item) {
      this.$router.push({ path: `/advertisements/show/${item.id}` });
    },
    editItem(item) {
      this.$router.push({ path: `/advertisements/edit/${item.id}` });
    },
    // ===== End:: Show

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

        if (this.reason.trim() !== '') {
          requestData.status_reason = this.reason.trim();
        }
        let res = await this.$axios({
          method: "POST",
          url: `advertisements/${this.itemToUpdate.id}/change-status`,
          data: requestData // Put the data in the 'data' property
        });
        this.$message.success(res.data.message);
        this.dialogUpdate = false;
        this.setTableRows();
        this.status_modal = null;
      } catch (error) {
        this.dialogUpdate = false;
        this.status_modal = null;
        this.$message.error(error.response.data.message);
      }
    },


    async getPropertySections() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: `real-estate-departments`,
        });
        // console.log("Cities =>", res.data.data);
        this.propertySections = res.data.data;
      } catch (error) {
        console.log(error.response.data.message);
      }
    },

    async getAreas() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: `areas`,
        });
        // console.log("Cities =>", res.data.data);
        this.getAreasData = res.data.data;
      } catch (error) {
        console.log(error.response.data.message);
      }
    },

    async getCountries() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: `countries`,
          params: {
            area_id: `${this.filterOptions.area_id?.id}`
          }
        });
        // console.log("Cities =>", res.data.data);
        this.CountriesData = res.data.data;
      } catch (error) {
        console.log(error.response.data.message);
      }
    },

    async getDistricts() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: `districts`,
          params: {
            country_id: `${this.filterOptions.country_id?.id}`
          }
        });
        // console.log("Cities =>", res.data.data);
        this.Districts = res.data.data;
      } catch (error) {
        console.log(error.response.data.message);
      }
    },

    // Start:: Handling Download Files
    async downloadPdf() {
      await this.$refs.html2Pdf.generatePdf();
      this.pdfDownloadBtnIsLoading = false;
    },
    // End:: Handling Download Files

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
    this.getAreas();
    this.getPropertySections();
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
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

              <!-- Start:: Status Input -->
              <base-input col="3" type="text" :placeholder="$t('PLACEHOLDERS.user_name')" v-model="filterOptions.name" />
              <!-- End:: Status Input -->

              <!-- Start:: Start Date Input -->
              <base-picker-input col="3" type="date" :placeholder="$t('PLACEHOLDERS.send_date')"
                v-model.trim="filterOptions.startDate" />
              <!-- End:: Start Date Input -->

              <!-- Start:: End Date Input -->
              <!-- <base-picker-input col="3" type="date" :placeholder="$t('PLACEHOLDERS.endDate')"
                v-model.trim="filterOptions.endDate" /> -->
              <!-- End:: End Date Input -->

              <!-- Start:: Status Input -->
              <!-- <base-select-input col="3" :optionsList="activeStatuses" :placeholder="$t('PLACEHOLDERS.status')"
                v-model="filterOptions.is_active" /> -->
              <!-- End:: Status Input -->

              <!-- Start:: Rate Input -->
              <base-rate-input col="4" :placeholder="$t('PLACEHOLDERS.rating_stars')" v-model="filterOptions.rate"
                size="22" disabled />
              <!-- End:: Rate Input -->
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
          <h5>{{ $t("TABLES.Rates.rates") }}</h5>

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

        <!-- Start:: Client Name Route -->
        <template v-slot:[`item.client`]="{ item }">
          <p class="blue-grey--text text--darken-1 fs-3" v-if="!item.client">-</p>
          <p v-else>{{ item.client }}</p>
        </template>
        <!-- End:: Client Name Route -->

        <!-- Start:: Rate Comment Btns -->
        <template v-slot:[`item.rate_comment`]="{ item }">
          <h6 class="text-danger" v-if="!item.rate_comment"> {{ $t("TABLES.noData") }} </h6>

          <div class="actions" v-else>
            <button class="btn_show" @click="showCommentModal(item.rate_comment)">
              <i class="fal fa-file-alt"></i>
            </button>
          </div>
        </template>
        <!-- End:: Rate Comment Btns -->

        <!-- Start:: Actions -->
        <template v-slot:[`item.actions`]="{ item }">
          <div class="actions">
            <a-tooltip placement="bottom">
              <template slot="title">
                <span>{{ $t("TABLES.Rates.user_commet") }}</span>
              </template>

              <button class="btn_edit" @click="selectAcceptItem(item)">
                <i class="fal fa-wallet"></i>
              </button>
            </a-tooltip>
          </div>
        </template>
        <!-- End:: Actions -->

        <!-- Start:: Activation -->
        <template v-slot:[`item.is_active`]="{ item }">
          <div class="activation" dir="ltr" style="z-index: 1" v-if="$can('rates activate', 'rates')">
            <v-switch class="mt-2" color="success" v-model="item.is_active" hide-details
              @change="changeActivationStatus(item)"></v-switch>
          </div>

          <template v-else>
            <span class="text-success text-h5" v-if="item.is_active">
              <i class="far fa-check"></i>
            </span>
            <span class="text-danger text-h5" v-else>
              <i class="far fa-times"></i>
            </span>
          </template>
        </template>
        <!-- End:: Activation -->

        <!-- ======================== Start:: Dialogs ======================== -->
        <template v-slot:top>
          <!-- Start:: Desc Modal -->
          <description-modal v-if="dialogComment" :modalIsOpen="dialogComment" :modalDesc="selectedCommentTextToShow"
            @toggleModal="dialogComment = !dialogComment" />
          <!-- End:: Desc Modal -->
          <!-- Start:: Balance Modal -->
          <v-dialog v-model="dialogBalance">
            <v-card>
              <form class="w-100">
                <base-input col="12" type="text" :placeholder="$t('TABLES.Rates.user_commet')" v-model="balance_package"
                  disabled />
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
import RatingPreview from "@/components/ui/RatingPreview.vue";

export default {
  name: "AllRates",

  components: {
    RatingPreview,

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
          value: "unpublished",
        },
        // {
        //   id: null,
        //   name: this.$t("STATUS.new"),
        //   value: "new",
        // },
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
        service_requester_name: null,
        rate: 0,
        startDate: null,
        endDate: null,
        is_active: null,
        name: null
      },
      providers_list: [],
      // End:: Filter Data

      // Start:: Table Data
      searchValue: "",
      tableHeaders: [
        {
          text: this.$t("TABLES.Rates.serialNumber"),
          value: "serialNumber",
          align: "center",
          sortable: false
        },
        {
          text: this.$t("PLACEHOLDERS.user_name"),
          value: "user.name",
          align: "center",
          width: "120",
          sortable: false
        },
        {
          text: this.$t("TABLES.Users.phone"),
          value: "user.mobile",
          align: "center",
          sortable: false
        },
        {
          text: this.$t("TABLES.Rates.rate"),
          value: "rate",
          align: "center",
          width: "100",
          sortable: false
        },
        {
          text: this.$t("TABLES.Rates.sendDate"),
          value: "created_at",
          align: "center",
          sortable: false
        },
        {
          text: this.$t("TABLES.Rates.comment"),
          value: "actions",
          align: "center",
          sortable: false
        },
        {
          text: this.$t("TABLES.Rates.publishStatus"),
          value: "is_active",
          align: "center",
          sortable: false
        }
      ],
      tableRows: [],
      // End:: Table Data

      // Start:: Dialogs Control Data
      dialogComment: false,
      selectedCommentTextToShow: "",
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

      status_word: '',
      balance_package: '',

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
      // if (!this.filterOptions.startDate && this.filterOptions.endDate) {
      //   this.$message.error(this.$t("VALIDATION.startDate"));
      //   return;
      // } else if (this.filterOptions.startDate && !this.filterOptions.endDate) {
      //   this.$message.error(this.$t("VALIDATION.endDate"));
      //   return;
      // } else if (this.filterOptions.endDate && this.filterOptions.startDate) {
      //   const startDate = new Date(this.filterOptions.startDate);
      //   const endDate = new Date(this.filterOptions.endDate);
      //   if (endDate <= startDate) {
      //     this.$message.error(this.$t("VALIDATION.handle"));
      //     return;
      //   }
      // }
      // else {
      if (this.$route.query.page !== '1') {
        await this.$router.push({ path: '/rates/all', query: { page: 1 } });
      }
      this.setTableRows();
      // }
    },
    async resetFilter() {
      this.filterOptions.name = null;
      this.filterOptions.rate = 0;
      this.filterOptions.startDate = null;
      this.filterOptions.endDate = null;
      this.filterOptions.is_active = null;
      if (this.$route.query.page !== '1') {
        await this.$router.push({ path: '/rates/all', query: { page: 1 } });
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
          url: "rates",
          params: {
            page: this.paginations.current_page,
            clientName: this.filterOptions.name,
            rate: this.filterOptions.rate === 0 ? null : `${this.filterOptions.rate}`,
            status: this.filterOptions.is_active?.value,
            createdAt: this.filterOptions.startDate,
            end_date: this.filterOptions.endDate,
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
    showCommentModal(comment) {
      this.dialogComment = true;
      this.selectedCommentTextToShow = comment;
    },
    // End:: Control Modals

    // Start:: Change Activation Status
    async changeActivationStatus(item) {
      if (item.is_active == 1) {
        console.log("published");
        this.status_word = "published"
      } else {
        console.log("unpublished");

        this.status_word = "unpublished"
      }
      try {
        await this.$axios({
          method: "POST",
          url: `rates/${item.id}`,
          data: { status: this.status_word }
        });
        this.$message.success(this.$t("MESSAGES.changedSuccessfully"));
      } catch (error) {
        this.$message.error(error.response.data.message);
      }
    },
    // End:: Change Activation Status

    // ===== Start:: balance
    selectAcceptItem(item) {
      console.log("item", item);
      this.dialogBalance = true;
      this.itemToBalance = item;

      this.balance_package = item.comment;

    },
    async confirmAcceptItem(item) {

      const REQUEST_DATA = new FormData();
      REQUEST_DATA.append("comment", this.balance_package);
      // REQUEST_DATA.append("_method", "PUT");

      try {
        await this.$axios({
          method: "GET",
          url: "rates",
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


<style></style>
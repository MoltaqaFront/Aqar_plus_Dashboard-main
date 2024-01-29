<template>
  <div class="crud_form_wrapper">
    <!-- Start:: Single Step Form Content -->
    <div class="single_step_form_content_wrapper">
      <form @submit.prevent="validateFormInputs">
        <div class="row">

          <!-- Start:: Name Input -->
          <base-input col="6" type="number" :placeholder="$t('PLACEHOLDERS.VAT_percentage')"
            v-model.trim="data.VAT_percentage" />
          <!-- End:: Name Input -->

          <!-- Start:: Status Input -->
          <base-input col="6" type="number" :placeholder="$t('PLACEHOLDERS.number_of_reports_to_block_auto_ad')"
            v-model="data.number_of_reports_to_block_auto_ad" />
          <!-- End:: Status Input -->

          <base-input col="6" type="number" :placeholder="$t('PLACEHOLDERS.number_of_free_package_ads')"
            v-model="data.number_of_free_package_ads" />

          <base-input col="6" type="number" :placeholder="$t('PLACEHOLDERS.free_package_expiry_duration')"
            v-model="data.free_package_expiry_duration" />

          <base-input col="6" type="number" :placeholder="$t('PLACEHOLDERS.search_scope')" v-model="data.Search_scope" />

          <!-- Start:: Submit Button Wrapper -->
          <div class="btn_wrapper">
            <base-button class="mt-2" styleType="primary_btn" :btnText="$t('BUTTONS.save')" :isLoading="isWaitingRequest"
              :disabled="isWaitingRequest" />
          </div>
          <!-- End:: Submit Button Wrapper -->
        </div>
      </form>
    </div>
    <!-- End:: Single Step Form Content -->
  </div>
</template>

<script>

import { mapGetters, mapActions } from "vuex";
export default {
  name: "GeneralSettings",

  data() {
    return {
      // Start:: Loader Control Data
      isWaitingRequest: false,
      // End:: Loader Control Data

      // Start:: Data
      data: {
        VAT_percentage: null,
        number_of_reports_to_block_auto_ad: '',
        number_of_free_package_ads: '',
        free_package_expiry_duration: null,
        Search_scope: null,
      },
      // End:: Data

    };
  },

  computed: {
    // Start:: Vuex Getters
    ...mapGetters({
      getAuthenticatedUserData: "AuthenticationModule/getAuthenticatedUserData"
    }),
    // End:: Vuex Getters
  },


  methods: {

    // Start:: Get Data To Edit
    async getDataToEdit() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: `settings?key=dashboard-admin-setting`,
        });
        // console.log("DATA =>", res.data.data);
        this.data.VAT_percentage = res.data.data[0].value.Value_added_tax_rate;
        this.data.number_of_reports_to_block_auto_ad = res.data.data[0].value.Number_of_reports_to_block_an_automatic_ad;
        this.data.number_of_free_package_ads = res.data.data[0].value.Number_of_ads_for_the_free_package;
        this.data.free_package_expiry_duration = res.data.data[0].value.The_expiration_period_of_the_free_package;
        this.data.Search_scope = res.data.data[0].value.Search_scope;

      } catch (error) {
        console.log(error.response.data.message);
      }
    },
    // End:: Get Data To Edit

    // Start:: Submit Form
    async submitForm() {
      this.isWaitingRequest = !this.isWaitingRequest;

      const REQUEST_DATA = new FormData();
      // Start:: Append Request Data
      REQUEST_DATA.append("key", "dashboard-admin-setting");
      REQUEST_DATA.append("value[Value_added_tax_rate]", this.data.VAT_percentage);
      REQUEST_DATA.append("value[Number_of_reports_to_block_an_automatic_ad]", this.data.number_of_reports_to_block_auto_ad);
      REQUEST_DATA.append("value[Number_of_ads_for_the_free_package]", this.data.number_of_free_package_ads);
      REQUEST_DATA.append("value[The_expiration_period_of_the_free_package]", this.data.free_package_expiry_duration);
      REQUEST_DATA.append("value[Search_scope]", this.data.Search_scope);

      // Start:: Append Request Data

      try {
        await this.$axios({
          method: "POST",
          url: `settings`,
          data: REQUEST_DATA,
        });
        this.isWaitingRequest = false;
        this.$message.success(this.$t("MESSAGES.savedSuccessfully"));
        this.getDataToEdit();
      } catch (error) {
        this.isWaitingRequest = false;
        this.$message.error(error.response.data.errors);
      }
    },
    // End:: Submit Form

    // Start:: validate Form Inputs
    validateFormInputs() {
      this.isWaitingRequest = true;

      if (!this.data.number_of_free_package_ads || this.data.number_of_free_package_ads === 'null') {
        this.isWaitingRequest = false;
        this.$message.error(this.$t("VALIDATION.free_package_ads_field_required"));
        return;
      } else if (!this.data.number_of_free_package_ads || this.data.number_of_free_package_ads <= 0) {
        this.isWaitingRequest = false;
        this.$message.error(this.$t("VALIDATION.custom_positive_value_required"));
        return;
      } else if (this.data.VAT_percentage < 1 || this.data.VAT_percentage > 100) {
        this.isWaitingRequest = false;
        this.$message.error(this.$t("VALIDATION.custom_value_range_required"));
        } else if (!this.data.number_of_reports_to_block_auto_ad || this.data.number_of_reports_to_block_auto_ad === 'null') {
        this.isWaitingRequest = false;
        this.$message.error(this.$t("VALIDATION.reports_to_block_auto_ad_required"));
        return;
      } else if (!this.data.number_of_reports_to_block_auto_ad || this.data.number_of_reports_to_block_auto_ad <= 0) {
        this.isWaitingRequest = false;
        this.$message.error(this.$t("VALIDATION.custom_positive_value_required"));
        return;
      }else if (!this.data.free_package_expiry_duration || this.data.free_package_expiry_duration === 'null') {
        this.isWaitingRequest = false;
        this.$message.error(this.$t("VALIDATION.free_package_duration_field_required"));
        return;
      } else if (!this.data.free_package_expiry_duration || this.data.free_package_expiry_duration <= 0) {
        this.isWaitingRequest = false;
        this.$message.error(this.$t("VALIDATION.custom_positive_value_required"));
        return;
      } else {
        this.submitForm();
        return;
      }
    },
    // End:: validate Form Inputs
  },

  created() {
    // Start:: Fire Methods
    this.getDataToEdit();
    // End:: Fire Methods
  },
};
</script>


<style lang="scss" scoped>
.all_action {
  display: flex;
  gap: 15px
}

.add_another {
  border: none;
  padding: 8px;
  width: 40px;
  height: 40px;
  border: 1px solid var(--light_gray_clr);
  border-radius: 50%;
  font-size: 18px;
  color: var(--light_gray_clr);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  margin: auto;

  .fa-trash {
    color: #ff2159;
    cursor: pointer
  }
}
</style>
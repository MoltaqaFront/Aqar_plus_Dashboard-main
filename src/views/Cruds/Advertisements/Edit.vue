<template>
  <div class="crud_form_wrapper">
    <!-- Start:: Title -->
    <div class="form_title_wrapper">
      <h4>{{ $t("PLACEHOLDERS.update_ads") }}</h4>
    </div>
    <!-- End:: Title -->

    <!-- Start:: Single Step Form Content -->
    <div class="single_step_form_content_wrapper">
      <form @submit.prevent="validateFormInputs">
        <div class="row">

          <!-- <base-image-upload-input col="12" identifier="admin_image" :preSelectedImage="data.image.path"
            :placeholder="$t('PLACEHOLDERS.quick_response_code')" @selectImage="selectImage" required /> -->

          <!-- <base-input col="4" type="text" :placeholder="$t('PLACEHOLDERS.ad_number')" v-model.trim="data.ad_number" /> -->

          <base-select-input col="4" :optionsList="propertySections" :placeholder="$t('PLACEHOLDERS.property_section')"
            v-model="data.property_section" />

          <base-input col="4" type="text" :placeholder="$t('PLACEHOLDERS.price')" v-model.trim="data.price" />

          <!-- Start:: Name Input -->
          <base-select-input col="4" :placeholder="$t('TABLES.Areas.name')" :optionsList="getAreasData"
            v-model.trim="data.area_id" @input="getCountries" />
          <!-- End:: Name Input -->

          <base-select-input col="4" v-if="CountriesData" :optionsList="CountriesData"
            :placeholder="$t('SIDENAV.Cities.name')" v-model="data.country_id" @input="getDistricts" />

          <base-select-input col="4" v-if="Districts" :optionsList="Districts"
            :placeholder="$t('PLACEHOLDERS.neighborhood_name')" v-model.trim="data.district_id" />

          <base-input col="4" type="text" :placeholder="$t('PLACEHOLDERS.address')" v-model.trim="data.address" />

          <base-input col="4" type="text" :placeholder="$t('PLACEHOLDERS.area_wide')" v-model.trim="data.area_wide" />

          <base-input col="4" type="text" :placeholder="$t('PLACEHOLDERS.street_view')" v-model.trim="data.street_view" />

          <base-input col="4" type="text" :placeholder="$t('PLACEHOLDERS.facade')" v-model.trim="data.facade" />

          <base-input col="4" type="text" :placeholder="$t('PLACEHOLDERS.property_purpose')"
            v-model.trim="data.property_purpose" />

          <base-select-input col="3" :optionsList="Types" :placeholder="$t('PLACEHOLDERS.rent_type')"
            v-model="data.rent_type" />

          <!-- <base-select-input col="6" :optionsList="allVehicleTypes" :placeholder="$t('PLACEHOLDERS.vechiles_type')"
            v-model="data.type_vechile" @input="getAllAdditionalFields" /> -->

          <!-- <base-select-input col="6" v-if="data.AdditionalFields && data.AdditionalFields.length > 0"
            :optionsList="data.AdditionalFields" :placeholder="$t('PLACEHOLDERS.additional_fields')"
            v-model.trim="data.additional_field" multiple required /> -->

          <!-- Additional Fields -->

          <div class="row" v-if="data.additional_field">
            <div class="col-lg-6 col-12" v-for="(item_additional, index) in filteredAdditionalFieldsInputs"
              :key="'l' + index">

              <div v-if="item_additional.type == 'textbox'">
                <base-input type="text" :placeholder="item_additional.name" v-model.trim="item_additional.value" />
              </div>

              <div v-if="item_additional.type == 'textarea'">
                <base-input type="text" :placeholder="item_additional.name" v-model.trim="item_additional.value" />
              </div>

              <div v-if="item_additional.type == 'number'">
                <base-input type="text" :placeholder="item_additional.name" v-model.trim="item_additional.value" />
              </div>

            </div>

            <div class="col-lg-6 col-12" v-for="(item_additional, index) in filteredAdditionalFieldsSelect"
              :key="'lp' + index">

              <div v-if="item_additional.type == 'checkbox'">
                <base-select-input :optionsList="item_additional.values" :placeholder="item_additional.name"
                  v-model.trim="item_additional.value" multiple />
              </div>

              <div v-if="item_additional.type == 'dropdown'">
                <base-select-input :optionsList="item_additional.values" :placeholder="item_additional.name"
                  v-model.trim="item_additional.value" multiple />
              </div>

              <div v-if="item_additional.type == 'radio'">
                <base-select-input :optionsList="item_additional.values" :placeholder="item_additional.name"
                  v-model.trim="item_additional.value" />
              </div>

            </div>

          </div>

          <!-- Start:: Submit Button Wrapper -->
          <div class="btn_wrapper">
            <base-button class="mt-2" styleType="primary_btn" :btnText="$t('BUTTONS.save')" :isLoading="isWaitingRequest"
              :disabled="isWaitingRequest" />
          </div>
          <!-- End:: Submit Button Wrapper -->

        </div>

      </form>
    </div>
    <!-- END:: Single Step Form Content -->
  </div>
</template>

<script>

export default {
  name: "showCity",

  data() {
    return {
      // Start:: Loader Control Data
      isWaitingRequest: false,
      // End:: Loader Control Data

      allVehicleTypes: [],
      // Start:: Data Collection To Send
      data: {
        image: {
          path: null,
          file: null,
        },
        advertiser_image: {
          path: null,
          file: null,
        },
        type_vechile: null,
        additional_field: [],
        advertiser_name: null,
        advertiser_mobile_number: null,
        advertiser_whatsapp_number: null,
        created_at: null,
        price: null,
        ad_number: null,
        rent_type: null,
        area_id: null,
        country_id: null,
        district_id: null,

        new_country_id: null,
        new_district_id: null,

        address: null,
        area_wide: null,
        street_view: null,
        facade: null,
        property_purpose: null,
        property_section: null,
        last_update_before_duration: null,
        field_type: null,
        type: null
      },
      field_values: [
        {
          value_ar: "",
          value_en: ""
        }
      ],
      getAreasData: [],
      CountriesData: [],
      Districts: [],
      propertySections: [],
      // End:: Data Collection To Send
    };
  },

  computed: {
    Fields_type() {
      return [
        {
          id: 1,
          name: this.$t("PLACEHOLDERS.textbox"),
          value: "textbox",
        },
        {
          id: 2,
          name: this.$t("PLACEHOLDERS.numberbox"),
          value: "number",
        },
        {
          id: 3,
          name: this.$t("PLACEHOLDERS.textarea"),
          value: "textarea",
        },
        {
          id: 4,
          name: this.$t("PLACEHOLDERS.checkbox"),
          value: "checkbox",
        },
        {
          id: 5,
          name: this.$t("PLACEHOLDERS.Radio_Button"),
          value: "radio",
        },
        {
          id: 6,
          name: this.$t("PLACEHOLDERS.drop_down"),
          value: "dropdown",
        }
      ];
    },

    Types() {
      return [
        {
          id: 1,
          name: this.$t("PLACEHOLDERS.daily"),
          value: "daily",
        },
        {
          id: 2,
          name: this.$t("PLACEHOLDERS.monthly"),
          value: "monthly",
        },
        {
          id: 2,
          name: this.$t("PLACEHOLDERS.yearly"),
          value: "yearly",
        }
      ];
    },

    filteredAdditionalFieldsInputs() {
      return this.data.additional_field.filter(item => ['textbox', 'number', 'textarea'].includes(item.type));
    },
    filteredAdditionalFieldsSelect() {
      return this.data.additional_field.filter(item => !['textbox', 'number', 'textarea'].includes(item.type));
    },
  },

  methods: {

    addRow() {
      this.field_values.push(
        {
          value_ar: "",
          value_en: ""
        }
      )
    },

    removeRow(index) {
      this.field_values.splice(index, 1)
    },

    // Start:: Select Upload Image
    selectImage(selectedImage) {
      this.data.image = selectedImage;
    },
    // End:: Select Upload Image

    // start show data
    async showData() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: `advertisements/${this.$route.params.id}`,
        });

        this.data.image.path = res.data.data.Advertisement.qr_code_url;
        this.data.ad_number = res.data.data.Advertisement.id;
        this.data.price = res.data.data.Advertisement.price;
        this.data.created_at = res.data.data.Advertisement.created_at;
        this.data.address = res.data.data.Advertisement.address;
        this.data.area_id = res.data.data.Advertisement.area;
        this.data.country_id = res.data.data.Advertisement.country;
        this.data.district_id = res.data.data.Advertisement.district;

        this.data.new_country_id = res.data.data.Advertisement.country;
        this.data.new_district_id = res.data.data.Advertisement.district;

        this.data.area_wide = res.data.data.Advertisement.space;
        this.data.rent_type = { "id": 4, "name": res.data.data.Advertisement.rent_type, "value": res.data.data.Advertisement.rent_type };

        this.data.street_view = res.data.data.Advertisement.street_width;
        this.data.facade = res.data.data.Advertisement.interface;
        this.data.property_section = res.data.data.Advertisement.real_estate_department;
        this.data.property_purpose = res.data.data.Advertisement.real_estate_purpose;
        this.data.last_update_before_duration = res.data.data.Advertisement.updated_at;

        this.data.advertiser_image.path = res.data.data.Advertisement.user.avatar;
        this.data.advertiser_name = res.data.data.Advertisement.user.name;
        this.data.advertiser_mobile_number = res.data.data.Advertisement.user.mobile;
        this.data.advertiser_whatsapp_number = res.data.data.Advertisement.user.watsapp;

        this.data.additional_field = res.data.data.Advertisement.additional_fields;

      } catch (error) {
        this.loading = false;
        console.log(error.response.data.message);
      }
    },
    // end show data

    validateFormInputs() {
      this.submitForm();
      // this.isWaitingRequest = true;

      // if (!this.data.nameAr) {
      //   this.isWaitingRequest = false;
      //   this.$message.error(this.$t("VALIDATION.nameAr"));
      //   return;
      // } else if (!this.data.nameEn) {
      //   this.isWaitingRequest = false;
      //   this.$message.error(this.$t("VALIDATION.nameEn"));
      //   return;
      // } else {
      //   return;
      // }
    },

    // Start:: Submit Form
    async submitForm() {
      const REQUEST_DATA = new FormData();
      // Start:: Append Request Data
      REQUEST_DATA.append("rent_type", this.data.rent_type?.value);
      REQUEST_DATA.append("price", this.data.price);
      REQUEST_DATA.append("space", this.data.area_wide);
      REQUEST_DATA.append("street_width", this.data.street_view);
      REQUEST_DATA.append("interface", this.data.facade);
      REQUEST_DATA.append("real_estate_department_id", this.data.property_section?.id);
      REQUEST_DATA.append("area_id", this.data.area_id?.id);
      REQUEST_DATA.append("country_id", this.data.country_id?.id);
      REQUEST_DATA.append("district_id", this.data.district_id?.id);
      REQUEST_DATA.append("address", this.data.address);
      REQUEST_DATA.append("real_estate_description", this.data.property_purpose);

      REQUEST_DATA.append("type", this.data.field_type?.value);

      if (this.data.additional_field) {
        // Create arrays based on type
        const inputTypes = this.data.additional_field.filter(item => ['textbox', 'number', 'textarea'].includes(item.type));
        const selectionTypes = this.data.additional_field.filter(item => !['textbox', 'number', 'textarea'].includes(item.type));

        // First loop for input types (textarea, textbox, number)
        inputTypes.forEach((item, index_input) => {
          REQUEST_DATA.append(`additional_field_input_type_ids[${index_input}]`, item.id);

          if (item.value && item.type === 'textbox') {
            REQUEST_DATA.append(`additional_field_input_type_values[${index_input}]`, item.value);
          } else if (item.value && item.type === 'number') {
            REQUEST_DATA.append(`additional_field_input_type_values[${index_input}]`, item.value);
          } else if (item.value && item.type === 'textarea') {
            REQUEST_DATA.append(`additional_field_input_type_values[${index_input}]`, item.value);
          }
        });

        // Second loop for selection types (dropdown, checkbox, radio)
        selectionTypes.forEach((item, index_selection) => {
          console.log("index_selection", item);

          if (item.value.length && item.type === 'dropdown') {
            item.value.forEach((inner_item, index_inner) => {
              REQUEST_DATA.append(`additional_field_selected_type_ids[${index_selection}]`, item.id);
              REQUEST_DATA.append(`additional_field_selected_type_value_ids[${index_selection}][${index_inner}]`, inner_item.id);
            });
          }

          if (item.value.length && item.type === 'checkbox') {
            item.value.forEach((inner_item, index_inner) => {
              REQUEST_DATA.append(`additional_field_selected_type_ids[${index_selection}]`, item.id);
              REQUEST_DATA.append(`additional_field_selected_type_value_ids[${index_selection}][${index_inner}]`, inner_item.id);
            });
          }

          if (item.value && item.type === 'radio') {
            // console.log("radioIndex", radioIndex)
            REQUEST_DATA.append(`additional_field_selected_type_ids[${index_selection}]`, item.id);
            REQUEST_DATA.append(`additional_field_selected_type_value_ids[${index_selection}][0]`, item.value[0].id);
          }
        });

      }
      REQUEST_DATA.append("_method", "PUT");
      // Start:: Append Request Data

      try {
        await this.$axios({
          method: "POST",
          url: `advertisements/${this.$route.params.id}`,
          data: REQUEST_DATA,
        });
        this.isWaitingRequest = false;
        this.$message.success(this.$t("MESSAGES.addedSuccessfully"));
        this.$router.push({ path: "/advertisements/all" });
      } catch (error) {
        this.isWaitingRequest = false;
        this.$message.error(error.response.data.message);
      }
    },

    async getAreas() {

      try {
        this.data.country_id = null;
        this.data.district_id = null;
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
            area_id: `${this.data.area_id?.id}`
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
            country_id: `${this.data.country_id?.id}`
          }
        });
        // console.log("Cities =>", res.data.data);
        this.Districts = res.data.data;
      } catch (error) {
        console.log(error.response.data.message);
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

    // async getAllAdditionalFields() {
    //   try {
    //     this.data.additional_field = null;
    //     this.additional_field_number = [];
    //     this.additional_field_checkbox = [];
    //     this.additional_field_dropdown = [];
    //     this.additional_field_radio = [];
    //     this.additional_field_textbox = [];
    //     this.additional_field_textarea = [];
    //     let res = await this.$axios({
    //       method: "POST",
    //       url: `additionalFields`
    //     });
    //     this.data.additional_field = res.data.data.Vehicle_Additional_Fields;
    //     console.log(res.data.data.Vehicle_Additional_Fields)
    //   } catch (error) {
    //     this.loading = false;
    //     console.log(error.response.data.message);
    //   }
    // },


  },

  async created() {
    // Start:: Fire Methods
    await this.showData();
    this.getAreas();
    this.getCountries();
    this.getDistricts();
    this.data.country_id = this.data.new_country_id;
    this.data.district_id = this.data.new_district_id;
    this.getPropertySections();
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

  .fa-trash {
    color: #ff2159;
    cursor: pointer
  }
}
</style>
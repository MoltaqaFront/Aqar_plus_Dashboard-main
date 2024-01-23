<template>
  <div class="crud_form_wrapper">
    <!-- Start:: Title -->
    <div class="form_title_wrapper">
      <h4>{{ $t("PLACEHOLDERS.show_ads") }}</h4>
    </div>
    <!-- End:: Title -->

    <!-- Start:: Single Step Form Content -->
    <div class="single_step_form_content_wrapper">
      <form @submit.prevent="validateFormInputs">
        <div class="row">

          <base-image-upload-input col="12" identifier="admin_image" :preSelectedImage="data.image.path"
            :placeholder="$t('PLACEHOLDERS.quick_response_code')" @selectImage="selectImage" disabled />

          <base-input col="4" type="text" :placeholder="$t('PLACEHOLDERS.ad_number')" v-model.trim="data.ad_number"
            disabled />

          <base-input col="4" type="text" :placeholder="$t('PLACEHOLDERS.joiningDate')" v-model.trim="data.created_at"
            disabled />

          <base-select-input col="4" :optionsList="propertySections" :placeholder="$t('PLACEHOLDERS.property_section')"
            v-model="data.property_section" disabled />

          <base-input col="4" type="text" :placeholder="$t('PLACEHOLDERS.price')" v-model.trim="data.price" disabled />

          <base-input col="4" type="text" :placeholder="$t('PLACEHOLDERS.address')" v-model.trim="data.address"
            disabled />

          <base-input col="4" type="text" :placeholder="$t('PLACEHOLDERS.area_wide')" v-model.trim="data.area_wide"
            disabled />

          <base-input col="4" type="text" :placeholder="$t('PLACEHOLDERS.rent_type')" v-model.trim="data.rent_type"
            disabled />

          <base-input col="4" type="text" :placeholder="$t('PLACEHOLDERS.street_view')" v-model.trim="data.street_view"
            disabled />

          <base-input col="4" type="text" :placeholder="$t('PLACEHOLDERS.facade')" v-model.trim="data.facade" disabled />

          <base-input col="4" type="text" :placeholder="$t('PLACEHOLDERS.property_purpose')"
            v-model.trim="data.property_purpose" disabled />

          <base-input col="4" type="text" :placeholder="$t('PLACEHOLDERS.last_update_before_duration')"
            v-model.trim="data.last_update_before_duration" disabled />

          <!-- Start:: Status Input -->
          <base-select-input col="6" :optionsList="Fields_type" :placeholder="$t('PLACEHOLDERS.additional_field_type')"
            v-model="data.field_type" disabled />
          <!-- End:: Status Input -->


          <div v-if="(data.field_type && data.field_type.value == 'checkbox')
            || (data.field_type && data.field_type.value == 'radio')
            || (data.field_type && data.field_type.value == 'dropdown')">


            <div class="row align-items-center" v-for="(item, index) in field_values" :key="'o' + index">

              <div class="col-lg-5 col-12">
                <base-input col="12" type="text" :placeholder="$t('PLACEHOLDERS.nameAr')" v-model.trim="item.value_ar"
                  disabled />
              </div>
              <div class="col-lg-5 col-12">
                <base-input col="12" type="text" :placeholder="$t('PLACEHOLDERS.nameEn')" v-model.trim="item.value_en"
                  disabled />
              </div>


            </div>

          </div>

        </div>

        <div class="row">
          <h3>{{ $t('PLACEHOLDERS.advertiser_details') }}</h3>
          <base-image-upload-input col="12" identifier="admin_image" :preSelectedImage="data.advertiser_image.path"
            :placeholder="$t('PLACEHOLDERS.advertiser_image')" disabled />

          <base-input col="4" type="text" :placeholder="$t('PLACEHOLDERS.advertiser_name')"
            v-model.trim="data.advertiser_name" disabled />

          <base-input col="4" type="text" :placeholder="$t('PLACEHOLDERS.advertiser_mobile_number')"
            v-model.trim="data.advertiser_mobile_number" disabled />

          <base-input col="4" type="text" :placeholder="$t('PLACEHOLDERS.advertiser_whatsapp_number')"
            v-model.trim="data.advertiser_whatsapp_number" disabled />
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
        advertiser_name: null,
        advertiser_mobile_number: null,
        advertiser_whatsapp_number: null,
        created_at: null,
        price: null,
        ad_number: null,
        rent_type: null,
        address: null,
        area_wide: null,
        street_view: null,
        facade: null,
        property_purpose: null,
        property_section: null,
        last_update_before_duration: null,
        field_type: null,
      },
      field_values: [
        {
          value_ar: "",
          value_en: ""
        }
      ],
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
        this.data.area_wide = res.data.data.Advertisement.space;
        this.data.rent_type = res.data.data.Advertisement.rent_type;
        this.data.street_view = res.data.data.Advertisement.street_width;
        this.data.facade = res.data.data.Advertisement.interface;
        this.data.property_section = res.data.data.Advertisement.real_estate_department;
        this.data.property_purpose = res.data.data.Advertisement.real_estate_purpose;
        this.data.last_update_before_duration = res.data.data.Advertisement.updated_at;

        this.data.advertiser_image.path = res.data.data.Advertisement.user.avatar;
        this.data.advertiser_name = res.data.data.Advertisement.user.name;
        this.data.advertiser_mobile_number = res.data.data.Advertisement.user.mobile;
        this.data.advertiser_whatsapp_number = res.data.data.Advertisement.user.watsapp;

        this.field_values = res.data.data.Advertisement.additional_fields[0].values;
        this.data.field_type = res.data.data.Advertisement.additional_fields[0].type;

        if (this.data.field_type) {
          this.data.field_type =
          {
            id: 0,
            name: this.data.field_type,
            value: this.data.field_type,
          }
        }

      } catch (error) {
        this.loading = false;
        console.log(error.response.data.message);
      }
    },
    // end show data


  },

  created() {
    // Start:: Fire Methods
    this.showData();
    // End:: Fire Methods
  },
};
</script>

<template>
  <div class="crud_form_wrapper">
    <!-- Start:: Title -->
    <div class="form_title_wrapper">
      <h4>{{ $t("SIDENAV.Districts.show") }}</h4>
    </div>
    <!-- End:: Title -->

    <!-- Start:: Single Step Form Content -->
    <div class="single_step_form_content_wrapper">
      <form @submit.prevent="validateFormInputs">
        <div class="row">

          <!-- Start:: Name Input -->
          <base-input col="6" type="text" :placeholder="$t('PLACEHOLDERS.nameAr')" v-model.trim="data.nameAr" disabled />
          <!-- End:: Name Input -->

          <!-- Start:: Name Input -->
          <base-input col="6" type="text" :placeholder="$t('PLACEHOLDERS.nameEn')" v-model.trim="data.nameEn" disabled />
          <!-- End:: Name Input -->

          <!-- Start:: Name Input -->
          <base-select-input col="6" :placeholder="$t('TABLES.Areas.name')" :optionsList="getAreasData"
            v-model.trim="data.area_id" @input="getCountries" disabled />
          <!-- End:: Name Input -->

          <!-- {{ getCountriesData }} -->
          <base-select-input col="6" v-if="CountriesData" :optionsList="CountriesData"
            :placeholder="$t('SIDENAV.Cities.name')" v-model="data.country_id" disabled />

          <!-- Start:: Deactivate Switch Input -->
          <div class="input_wrapper switch_wrapper my-5">
            <v-switch color="green" :label="data.active ? $t('PLACEHOLDERS.active') : $t('PLACEHOLDERS.notActive')"
              v-model="data.active" hide-details disabled></v-switch>
          </div>
          <!-- End:: Deactivate Switch Input -->

        </div>
      </form>
    </div>
    <!-- END:: Single Step Form Content -->
  </div>
</template>

<script>
import { mapGetters, mapActions } from "vuex";
export default {
  name: "CreateCity",

  data() {
    return {
      // Start:: Loader Control Data
      isWaitingRequest: false,
      // End:: Loader Control Data

      // Start:: Data Collection To Send
      data: {
        nameAr: null,
        nameEn: null,
        area_id: null,
        country_id: null,
        active: true,
      },

      getAreasData: [],
      CountriesData: [],
      // End:: Data Collection To Send
    };
  },

  computed: {
    ...mapGetters({
      getAppLocale: "AppLangModule/getAppLocale",
    }),
  },

  methods: {

    async getData() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: `districts/${this.$route.params.id}`,
        });
        // console.log("Cities =>", res.data.data);
        this.data.nameAr = res.data.data.District.name_ar;
        this.data.nameEn = res.data.data.District.name_en;
        this.data.area_id = res.data.data.District.area;
        this.data.country_id = res.data.data.District.country;
        this.data.active = res.data.data.District.is_active;
      } catch (error) {
        console.log(error.response.data.message);
      }
    },


  },

  created() {
    this.getData();
    this.getAreas();
  },
};
</script>

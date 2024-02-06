<template>
  <div class="crud_form_wrapper">
    <div class="table_title_wrapper">
          <div class="title_text_wrapper">
            <h5 style="color: #49a956;">{{ $t("PLACEHOLDERS.bank_accounts_settings") }}</h5>
          </div>
    </div>
    <!-- Start:: Single Step Form Content -->
    <div class="single_step_form_content_wrapper">
      <form @submit.prevent="validateFormInputs">
        <div class="row">            
          <div class="row justify-content-center">

              <!-- Start:: Tax Percentage Input -->
              <base-select-input col="6" :optionsList="banks" :placeholder="$t('PLACEHOLDERS.bank_name')"
                v-model.trim="data.bank_id" />
              <!-- End:: Tax Percentage Input -->
            <base-input col="6" type="text" :placeholder="$t('PLACEHOLDERS.iban')" v-model.trim="data.num" />
            <div class="col-l2">
              <div class="add_another" @click="addRow()">
                <i class="fas fa-plus"></i>
              </div>
            </div>

            <div class="row align-items-center ml-8" v-for="(item, index) in ibans" :key="'l' + index">

              <div class="col-lg-5 col-12">
                <base-select-input col="12" :optionsList="banks" :placeholder="$t('PLACEHOLDERS.bank_name')"
                    v-model.trim="item.name" />
              </div>
               <div class="col-lg-5 col-12">
                  <base-input col="12" type="text" :placeholder="$t('PLACEHOLDERS.iban')" v-model.trim="item.iban_number" />
                </div>
              <div class="col-2">
                  <div class="all_actions">
                    <span class="add_another" @click="removeRow(index)">
                      <i class="fas fa-minus"></i>
                    </span>

                  </div>
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
    <!-- End:: Single Step Form Content -->
  </div>
</template>

<script>
export default {
  name: "GeneralSettings",

  data() {
    return {
      // Start:: Loader Control Data
      isWaitingRequest: false,
      // End:: Loader Control Data

      // Start:: Data Collection To Send
      data: {
        bank_id: null,
        num: null
      },
      banks: [
        {
          name: ''
        }
      ],
      // End:: Data Collection To Send

      ibans: [
        {
          iban_number: '',
          name:''
        }
      ]
    };
  },

  methods: {

    addRow() {
      this.ibans.push({ phone: "" })
      // this.banks.push({name: ""})
    },

    removeRow(index) {
      this.ibans.splice(index, 1)
    },

    // Start:: Get Data To Edit
    async getDataToEdit() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: `bank-accounts`,
        });
        // Start:: Set Data

        // Transform the API response

        console.log("res.data.data.bank", res.data.data)
        this.ibans = res.data.data[0].iban.map(item => ({ iban: item }));
        this.data.num = res.data.data[0].iban;
        this.data.bank_id = res.data.data[0].bank;
        // End:: Set Data
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
      if (this.ibans) {
        this.ibans.forEach((element, index) => {
          if (element.iban_number) {
            REQUEST_DATA.append(`iban[${index}]`, element.iban_number);
          }
        });
      }
      REQUEST_DATA.append("bank_id", this.data.bank_id?.id);

      // Start:: Append Request Data

      try {
        await this.$axios({
          method: "POST",
          url: `bank-accounts`,
          data: REQUEST_DATA,
        });
        this.isWaitingRequest = false;
        this.$message.success(this.$t("MESSAGES.savedSuccessfully"));
      } catch (error) {
        this.isWaitingRequest = false;
        this.$message.error(error.response.data.errors);
      }
    },
    // End:: Submit Form

    async getBanks() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: `banks`
        });
        // console.log("Cities =>", res.data.data);
        this.banks = res.data.data;
      } catch (error) {
        console.log(error.response.data.message);
      }
    },

    // Start:: validate Form Inputs
    validateFormInputs() {
      this.submitForm();
    },
    // End:: validate Form Inputs
  },

  created() {
    // Start:: Fire Methods
    this.getDataToEdit();
    this.getBanks();
    // End:: Fire Methods
  },
};
</script>


<style lang="scss" scoped>
.item {
  gap: 10px
}

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
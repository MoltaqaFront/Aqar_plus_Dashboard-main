<template>
  <div class="show_all_content_wrapper">
    <!-- Start:: Main Section -->
    <main>
      <!--  =========== Start:: Table Title =========== -->
      <div class="table_title_wrapper">
        <div class="title_text_wrapper">
          <h5>{{ $t("SIDENAV.Users.control") }}</h5>
        </div>
      </div>
      <!--  =========== End:: Table Title =========== -->

      <!--  =========== Start:: Data Table =========== -->
       <!-- <ComingSoon /> -->
      <div class="statics mt-4">
      <div class="row">
        <div class="col-lg-6 col-md-6 col-12" v-for="(value, key) in statics" :key="'i' + key">
          <div class="box">
            <div class="icon">
              <i v-if="key === 'users_count'" class="fas fa-users"></i>
              <i v-else-if="key === 'advertisements_count'" class="fas fa-ad"></i>
              <i v-else-if="key === 'blocked_advertisements_count'" class="fas fa-ban"></i>
              <i v-else-if="key === 'packages_count'" class="fas fa-toolbox"></i>
              <i v-else-if="key === 'total_subscribe_packages_price'" class="fas fa-dollar-sign"></i>
              <i v-else-if="key === 'published_advertisements_count'" class="fas fa-plus"></i>
            </div>
            <div class="info_box">
              <span v-if="key == 'users_count'">{{ $t('PLACEHOLDERS.new_requests_count') }}</span>
              <span v-else-if="key === 'advertisements_count'">{{ $t('PLACEHOLDERS.incomplete_requests_count')
              }}</span>
             <span v-else-if="key === 'blocked_advertisements_count'">{{ $t('PLACEHOLDERS.canceled_requests_count') }}</span>
             <span v-else-if="key === 'packages_count'">{{ $t('PLACEHOLDERS.ongoing_requests_count') }}</span>
             <span v-else-if="key === 'total_subscribe_packages_price'">{{ $t('PLACEHOLDERS.rejected_requests_count') }}</span>
             <span v-else-if="key === 'published_advertisements_count'">{{ $t('PLACEHOLDERS.completed_requests_count') }}</span>
              <p class="number_box">{{ value }}</p>
            </div>
          </div>
        </div>
        </div>
    </div>
      <!--  =========== End:: Data Table =========== -->
    </main>
    <!-- End:: Main Section -->
  </div>
</template>

<script>
import ComingSoon from "@/components/ui/ComingSoon.vue";

export default {
  name: "HomePage",

  components: {
    ComingSoon
  },

  data() {
    return {
      statics: []
    };
  },

  methods: {

    async getStatics() {
      try {
        let res = await this.$axios({
          method: "GET",
          url: "home-statistics"
        });

        // console.log("All Data ==>", res.data.data);
        this.statics = res.data.data;
      } catch (error) {
        this.loading = false;
        console.log(error.response.data.message);
      }
    },
    // End:: Set Table Rows

  },

  mounted() {

    this.getStatics()

  },
};
</script>

<style lang="scss" scoped>
.show_all_content_wrapper {
  .statics {
    .box {
      background: #49a956;
      padding: 25px;
      margin-bottom: 10px;
      border-radius: 5px;
      overflow: hidden;
      display: flex;
      justify-content: space-between;

      cursor: pointer;
      transition: all 0.5s;

      &:hover {
        background: #fff;
        box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);

        .icon {
          i{
            color: #49a956;
          }
        }
        .info_box {

          span,
          p.number_box {
            color: #000
          }
        }
      }

      .icon {
        height: 60px;
        min-width: 60px;
        width: 60px;
        border-radius: 60px;
        background: #FFF;
        display: flex;
        align-items: center;
        justify-content: center;

        i {
          font-size: 25px ;
          color: #000;
        }
      }

      .info_box {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: flex-end;

        span {
          font-size: 19px;
          color: #FFF;
        }

        p.number_box {
          font-size: 30px;
          color: #FFF;
          font-weight: bold;
        }
      }
    }
  }
 /*  .statics {
    .box {
      background: aliceblue;
      padding: 25px;
      margin-bottom: 10px;
      border-radius: 5px;
      overflow: hidden;
      display: flex;
      justify-content: space-between;

      cursor: pointer;
      transition: all 0.5s;

      &:hover {
        background: #fff;
        box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
      }

      .icon {
        height: 60px;
        min-width: 60px;
        width: 60px;
        border-radius: 60px;
        background: rgba(0, 123, 255, 0.2);
        display: flex;
        align-items: center;
        justify-content: center;

        i {
          font-size: 25px
        }
      }

      .info_box {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: flex-end;

        span {
          font-size: 19px;
          color: #6c757d;
        }

        p.number_box {
          font-size: 30px;
          color: #000;
          font-weight: bold;
        }
      }
    }
  } */
}
</style>
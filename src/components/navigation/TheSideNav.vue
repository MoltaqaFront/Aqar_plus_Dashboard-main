<template>
  <div class="side_nav_wrapper" :class="{ active: navIsActive }">
    <!-- Start:: Toggle Btn -->
    <button class="close_btn" @click="$emit('fireToggleNavDrawerEmit')">
      <i class="fal fa-times"></i>
    </button>
    <!-- End:: Toggle Btn -->

    <!-- Start:: Logo -->
    <div class="logo_wrapper">
      <router-link class="logo" to="/home">
        <img src="@/assets/media/logo/logo.png" alt="Logo" width="170" height="125" />
      </router-link>
    </div>
    <!-- End:: Logo -->

    <!-- Start:: Side Nav Links -->
    <div class="side_nav_links_wrapper">
      <!-- Start:: Home Route -->
      <div class="home_route" @click="$emit('fireToggleNavDrawerEmit')">
        <router-link to="/home">
          <span class="route_icon">
            <img src="@/assets/media/icons/ui_icons/home.svg" alt="icon" width="40" height="40" />
          </span>
          <span class="route_text">
            {{ $t("SIDENAV.dashboard") }}
          </span>
        </router-link>
      </div>
      <!-- End:: Home Route -->

      <!-- Start:: notifications Route -->
      <div class="home_route" @click="$emit('fireToggleNavDrawerEmit')"
        v-if="$can('notifications index', 'notifications')">
        <router-link to="/all-notifications/all">
          <span class="route_icon">
            <img src="@/assets/media/icons/ui_icons/notifications.svg" alt="icon" width="40" height="40" />
          </span>
          <span class="route_text">
            {{ $t("SIDENAV.notifications.title") }}
          </span>
        </router-link>
      </div>
      <!-- End:: notifications Route -->

      <!-- Start:: estate-departments Route -->
      <div class="home_route" @click="$emit('fireToggleNavDrawerEmit')"
        v-if="$can('realEstateDepartments index', 'realEstateDepartments')">
        <router-link to="/estate-departments/all">
          <span class="route_icon">
            <img src="@/assets/media/icons/ui_icons/category.svg" alt="icon" width="40" height="40" />
          </span>
          <span class="route_text">
            {{ $t("PLACEHOLDERS.property_management") }}
          </span>
        </router-link>
      </div>
      <!-- End:: estate-departments Route -->

      <!-- Start:: banks Route -->
      <div class="home_route" @click="$emit('fireToggleNavDrawerEmit')" v-if="$can('banks index', 'banks')">
        <router-link to="/banks/all">
          <span class="route_icon">
            <img src="@/assets/media/icons/ui_icons/store.svg" alt="icon" width="40" height="40" />
          </span>
          <span class="route_text">
            {{ $t("PLACEHOLDERS.bank_management") }}
          </span>
        </router-link>
      </div>
      <!-- End:: banks Route -->

      <!-- Start:: clients Route -->
      <div class="home_route" @click="$emit('fireToggleNavDrawerEmit')" v-if="$can('clients index', 'clients')">
        <router-link to="/clients/all">
          <span class="route_icon">
            <img src="@/assets/media/icons/ui_icons/clients.svg" alt="icon" width="40" height="40" />
          </span>
          <span class="route_text">
            {{ $t("SIDENAV.Clients.title") }}
          </span>
        </router-link>
      </div>
      <!-- End:: clients Route -->

      <!-- Start:: offers Route -->
      <div class="home_route" @click="$emit('fireToggleNavDrawerEmit')" v-if="$can('offers index', 'offers')">
        <router-link to="/offers/all">
          <span class="route_icon">
            <img src="@/assets/media/icons/ui_icons/offers.svg" alt="icon" width="40" height="40" />
          </span>
          <span class="route_text">
            {{ $t("BUTTONS.offers") }}
          </span>
        </router-link>
      </div>
      <!-- End:: Home Route -->

      <!-- Start:: packages Route -->
      <div class="home_route" @click="$emit('fireToggleNavDrawerEmit')" v-if="$can('packages index', 'packages')">
        <router-link to="/packages/all">
          <span class="route_icon">
            <img src="@/assets/media/icons/ui_icons/service.svg" alt="icon" width="40" height="40" />
          </span>
          <span class="route_text">
            {{ $t("PLACEHOLDERS.packages") }}
          </span>
        </router-link>
      </div>
      <!-- End:: packages Route -->

      <!-- Start:: reports_management Route -->
      <div class="home_route" @click="$emit('fireToggleNavDrawerEmit')" v-if="$can('reports index', 'reports')">
        <router-link to="/reports/all">
          <span class="route_icon">
            <img src="@/assets/media/icons/ui_icons/reports.png" alt="icon" width="40" height="40" />
          </span>
          <span class="route_text">
            {{ $t("PLACEHOLDERS.reports_management") }}
          </span>
        </router-link>
      </div>
      <!-- End:: reports_management Route -->

      <!-- Start:: advertisements Route -->
      <div class="home_route" @click="$emit('fireToggleNavDrawerEmit')"
        v-if="$can('advertisements index', 'advertisements')">
        <router-link to="/advertisements/all">
          <span class="route_icon">
            <img src="@/assets/media/icons/ui_icons/image.svg" alt="icon" width="40" height="40" />
          </span>
          <span class="route_text">
            {{ $t("PLACEHOLDERS.ads_management") }}
          </span>
        </router-link>
      </div>
      <!-- End:: advertisements Route -->

      <!-- Start:: rates Route -->
      <div class="home_route" @click="$emit('fireToggleNavDrawerEmit')" v-if="$can('rates index', 'rates')">
        <router-link to="/rates/all">
          <span class="route_icon">
            <img src="@/assets/media/icons/ui_icons/star.svg" alt="icon" width="40" height="40" />
          </span>
          <span class="route_text">
            {{ $t("TABLES.Rates.rates") }}
          </span>
        </router-link>
      </div>
      <!-- End:: rates Route -->

      <!-- Start:: areas Route -->
      <div class="home_route" @click="$emit('fireToggleNavDrawerEmit')" v-if="$can('areas index', 'areas')">
        <router-link to="/areas/all">
          <span class="route_icon">
            <img src="@/assets/media/icons/ui_icons/areas.svg" alt="icon" width="40" height="40" />
          </span>
          <span class="route_text">
            {{ $t("SIDENAV.Areas.title") }}
          </span>
        </router-link>
      </div>
      <!-- End:: areas Route -->

      <!-- Start:: cities Route -->
      <div class="home_route" @click="$emit('fireToggleNavDrawerEmit')" v-if="$can('countries index', 'countries')">
        <router-link to="/cities/all">
          <span class="route_icon">
            <img src="@/assets/media/icons/ui_icons/city.svg" alt="icon" width="40" height="40" />
          </span>
          <span class="route_text">
            {{ $t("SIDENAV.Cities.title") }}
          </span>
        </router-link>
      </div>
      <!-- End:: cities Route -->

      <!-- Start:: districts Route -->
      <div class="home_route" @click="$emit('fireToggleNavDrawerEmit')" v-if="$can('districts index', 'districts')">
        <router-link to="/districts/all">
          <span class="route_icon">
            <img src="@/assets/media/icons/ui_icons/addresses.svg" alt="icon" width="40" height="40" />
          </span>
          <span class="route_text">
            {{ $t("SIDENAV.Districts.title") }}
          </span>
        </router-link>
      </div>
      <!-- End:: districts Route -->

      <!-- Start:: additionalFields Route -->
      <div class="home_route" @click="$emit('fireToggleNavDrawerEmit')"
        v-if="$can('additionalFields index', 'additionalFields')">
        <router-link to="/additionalFields/all">
          <span class="route_icon">
            <img src="@/assets/media/icons/ui_icons/folder.svg" alt="icon" width="40" height="40" />
          </span>
          <span class="route_text">
            {{ $t("PLACEHOLDERS.additional_fields") }}
          </span>
        </router-link>
      </div>
      <!-- End:: additionalServices Route -->

      <!-- Start:: Bank transfer management -->
      <div class="home_route" @click="$emit('fireToggleNavDrawerEmit')"
        v-if="$can('bankTransfers index', 'bankTransfers')">
        <router-link to="/bankTransfer/all">
          <span class="route_icon">
            <img src="@/assets/media/icons/ui_icons/exchange.png" alt="icon" width="40" height="40" />
          </span>
          <span class="route_text">
            {{ $t("PLACEHOLDERS.BankTransferManagement") }}
          </span>
        </router-link>
      </div>
      <!-- End:: Bank transfer management -->
      <!-- Start:: characteristics Route -->
      <div class="home_route" @click="$emit('fireToggleNavDrawerEmit')"
        v-if="$can('characteristics index', 'characteristics')">
        <router-link to="/characteristics/all">
          <span class="route_icon">
            <img src="@/assets/media/icons/ui_icons/image.svg" alt="icon" width="40" height="40" />
          </span>
          <span class="route_text">
            {{ $t("PLACEHOLDERS.advertiser_traits_management") }}
          </span>
        </router-link>
      </div>
      <!-- End:: characteristics Route -->

      <!-- Start:: contacts Route -->
      <div class="home_route" @click="$emit('fireToggleNavDrawerEmit')" v-if="$can('contactuses index', 'contactuses')">
        <router-link to="/contact-messages/all">
          <span class="route_icon">
            <img src="@/assets/media/icons/ui_icons/messages.svg" alt="icon" width="40" height="40" />
          </span>
          <span class="route_text">
            {{ $t("SIDENAV.ContactMessages.title") }}
          </span>
        </router-link>
      </div>
      <!-- End:: contacts Route -->

      <!-- Start:: Side Nav Routes -->

      <div class="side_routes_wrapper" v-if="$can('staticPages index', 'staticPages')">
        <a-menu style="width: 100%" mode="vertical" :open-keys="openKeys" @openChange="onOpenChange">
          <!-- Start:: Side Nav Tab -->
          <a-sub-menu v-for="item in sideNavbarList" :key="item.key" :data-type="!item.children ? 'single_route' : ''">
            <template v-if="item.children">
              <!-- ========= Start:: Main Tab -->
              <span slot="title">
                <img :src="item.icon" alt="icon" width="35" height="35" />
                <span> {{ item.title }} </span>
              </span>
              <!-- ========= End:: Main Tab -->

              <!-- ========= Start:: Nested Tabs -->
              <!-- v-if="childItem.hasPermission" -->
              <a-menu-item v-for="childItem in item.children" :key="childItem.hey">
                <button class="text-start w-100" @click="$emit('fireToggleNavDrawerEmit')">
                  <router-link :to="childItem.route">
                    <i class="fad fa-circle"></i>
                    {{ childItem.title }}
                  </router-link>
                </button>
              </a-menu-item>
              <!-- ========= End:: Nested Tabs -->
            </template>

          </a-sub-menu>
          <!-- End:: Side Nav Tab -->
        </a-menu>
      </div>
      <!-- End:: Side Nav Routes -->

      <!-- Start:: settings Route -->
      <div class="home_route" @click="$emit('fireToggleNavDrawerEmit')" v-if="$can('settings create', 'settings')">
        <router-link to="/settings">
          <span class="route_icon">
            <img src="@/assets/media/icons/ui_icons/settings.svg" alt="icon" width="40" height="40" />
          </span>
          <span class="route_text">
            {{ $t("SIDENAV.settings.title") }}
          </span>
        </router-link>
      </div>
      <!-- End:: settings Route -->

      <!-- Start:: Bank Account Route -->
      <div class="home_route" @click="$emit('fireToggleNavDrawerEmit')"
        v-if="$can('bankAccounts create', 'bankAccounts')">
        <router-link to="/banks-account/all">
          <span class="route_icon">
            <img src="@/assets/media/icons/ui_icons/accounting.png" alt="icon" width="40" height="40" />
          </span>
          <span class="route_text">
            {{ $t("PLACEHOLDERS.bank_accounts_settings") }}
          </span>
        </router-link>
      </div>
      <!-- End:: Bank Account Route -->

      <!-- Start:: users Route -->
      <div class="home_route" @click="$emit('fireToggleNavDrawerEmit')" v-if="$can('users index', 'users')">
        <router-link to="/admins/all">
          <span class="route_icon">
            <img src="@/assets/media/icons/ui_icons/admins.svg" alt="icon" width="40" height="40" />
          </span>
          <span class="route_text">
            {{ $t("SIDENAV.Admins.title") }}
          </span>
        </router-link>
      </div>
      <!-- End:: Users Route -->

      <!-- Start:: roles Route -->
      <div class="home_route" @click="$emit('fireToggleNavDrawerEmit')" v-if="$can('roles index', 'roles')">
        <router-link to="/roles/all">
          <span class="route_icon">
            <img src="@/assets/media/icons/ui_icons/roles.svg" alt="icon" width="40" height="40" />
          </span>
          <span class="route_text">
            {{ $t("SIDENAV.Roles.title") }}
          </span>
        </router-link>
      </div>
      <!-- End:: roles Route -->

      <div class="side_routes_wrapper">
        <a-menu mode="inline">
          <!-- Start:: Logout Tab -->
          <a-sub-menu key="logout" @titleClick="logoutConfirmationModalIsOpen = true">
            <!-- ========= Start:: Main Tab -->
            <span slot="title" class="logout_btn">
              <img src="../../assets/media/icons/ui_icons/logout.svg" alt="icon" width="35" height="35" />
              <span>
                {{ $t("SIDENAV.Logout.title") }}
              </span>
            </span>
            <!-- ========= End:: Main Tab -->
          </a-sub-menu>
          <!-- End:: Logout Tab -->
        </a-menu>
      </div>
      <!-- End:: Side Nav Routes -->

      <!-- Start:: Logout Confirmation Modal -->
      <v-dialog v-model="logoutConfirmationModalIsOpen">
        <v-card>
          <v-card-title class="text-h5 justify-center">
            {{ $t("TITLES.logoutConfirmingMessage") }}
          </v-card-title>
          <v-card-actions>
            <v-btn class="modal_confirm_btn" @click="logout">{{
              $t("BUTTONS.logout")
            }}</v-btn>

            <v-btn class="modal_cancel_btn" @click="logoutConfirmationModalIsOpen = false">{{ $t("BUTTONS.cancel")
            }}</v-btn>
            <v-spacer></v-spacer>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <!-- End:: Logout Confirmation Modal -->
    </div>
    <!-- End:: Side Nav Links -->
  </div>
</template>

<script>
import { mapGetters, mapActions } from "vuex";

export default {
  name: "SideNav",

  emits: ["fireToggleNavDrawerEmit"],

  props: {
    navIsActive: {
      type: Boolean,
      required: true,
    },
  },

  computed: {
    // Start:: Vuex Get Authenticated User Data
    ...mapGetters({
      getAuthenticatedUserData: "AuthenticationModule/getAuthenticatedUserData"
    }),
    // End:: Vuex Get Authenticated User Data

    // Start:: Extract Side Nav Keys
    extractSideNavItemsKeys() {
      let extractedKeys = [];
      this.sideNavbarList.forEach((element) => {
        extractedKeys.push(element["key"]);
      });
      return extractedKeys;
    },
    // End:: Extract Side Nav Keys

  },

  data() {
    return {
      // Start:: Controle Open Tabs Data
      openKeys: [],
      // End:: Controle Open Tabs Data

      // Start:: Side Navbar List
      sideNavbarList: [
        {
          key: "appContent",
          title: this.$t("SIDENAV.AppContent.title"),
          icon: require("@/assets/media/icons/ui_icons/book_mark.svg"),
          children: [

            {
              key: "aboutUs",
              title: this.$t("SIDENAV.AppContent.aboutUs"),
              route: "/app-content/about-us",
              hasPermission: this.$can('settings create', 'settings'),
            },
            {
              key: "termsAndConditions",
              title: this.$t("SIDENAV.AppContent.termsAndConditions"),
              route: "/app-content/terms",
              hasPermission: this.$can('settings create', 'settings'),
            },
            {
              key: "privacyPolicy",
              title: this.$t("SIDENAV.AppContent.privacyPolicy"),
              route: "/app-content/policy",
              hasPermission: this.$can('settings create', 'settings'),
            },
            {
              key: "aqar-uses",
              title: this.$t("PLACEHOLDERS.terms_of_use_in_akarplus_app"),
              route: "/app-content/aqar-uses",
              hasPermission: this.$can('settings create', 'settings'),
            },
            {
              key: "aqar-conditions",
              title: this.$t("PLACEHOLDERS.regulations_of_general_authority_for_real_estate"),
              route: "/app-content/aqar-conditions",
              hasPermission: this.$can('settings create', 'settings'),
            },
            {
              key: "contact",
              title: this.$t("PLACEHOLDERS.contact_admins"),
              route: "/contact_settings",
              hasPermission: this.$can('settings create', 'settings'),
            },
          ],
        },

      ],

      // Start:: Modal Controlling Data
      logoutConfirmationModalIsOpen: false,
      // End:: Modal Controlling Data
    };
  },

  methods: {
    // Start:: Vuex Auth Actions
    ...mapActions({
      logout: "AuthenticationModule/logout"
    }),
    // End:: Vuex Auth Actions

    // Start:: Controle Open Tabs
    onOpenChange(openKeys) {
      const latestOpenKey = openKeys.find(
        (key) => this.openKeys.indexOf(key) === -1
      );
      if (this.extractSideNavItemsKeys.indexOf(latestOpenKey) === -1) {
        this.openKeys = openKeys;
      } else {
        this.openKeys = latestOpenKey ? [latestOpenKey] : [];
      }
    },
    // End:: Controle Open Tabs
  },

  created() {
  },
};
</script>

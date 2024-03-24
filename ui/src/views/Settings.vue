<!--
  Copyright (C) 2022 Nethesis S.r.l.
  SPDX-License-Identifier: GPL-3.0-or-later
-->
<template>
  <cv-grid fullWidth>
    <cv-row>
      <cv-column class="page-title">
        <h2>{{ $t("settings.title") }}</h2>
      </cv-column>
    </cv-row>
    <cv-row v-if="error.getConfiguration">
      <cv-column>
        <NsInlineNotification
          kind="error"
          :title="$t('action.get-configuration')"
          :description="error.getConfiguration"
          :showCloseButton="false"
        />
      </cv-column>
    </cv-row>
    <cv-row>
      <cv-column>
        <cv-tile light>
          <cv-form @submit.prevent="configureModule">
            <cv-text-input
              :label="$t('settings.dolibarr_fqdn')"
              placeholder="dolibarr.example.org"
              v-model.trim="host"
              class="mg-bottom"
              :invalid-message="$t(error.host)"
              :disabled="loading.getConfiguration || loading.configureModule"
              ref="host"
            >
            </cv-text-input>
            <cv-text-input
              :label="$t('settings.dolibarr_admin')"
              placeholder="Dolibar Admin Username"
              v-model="DOLI_ADMIN_LOGIN"
              class="mg-bottom"
              :invalid-message="$t(error.DOLI_ADMIN_LOGIN)"
              :disabled="loading.getConfiguration || loading.configureModule"
              ref="DOLI_ADMIN_LOGIN"
            >
            </cv-text-input>
            <cv-text-input
              :label="$t('settings.dolibarr_admin_password')"
              placeholder="password"
              v-model="DOLI_ADMIN_PASSWORD"
              class="mg-bottom"
              :invalid-message="$t(error.DOLI_ADMIN_PASSWORD)"
              :disabled="loading.getConfiguration || loading.configureModule"
              type="password"
              ref="DOLI_ADMIN_PASSWORD"
            >
            </cv-text-input>
            <NsComboBox
                v-model.trim="ldap_domain"
                :autoFilter="true"
                :autoHighlight="true"
                :title="$t('settings.ldap_domain')"
                :label="$t('settings.choose_ldap_domain')"
                :options="user_domains_list"
                :userInputLabel="core.$t('settings.choose_ldap_domain')"
                :acceptUserInput="false"
                :showItemType="true"
                :invalid-message="$t(error.ldap_domain)"
                :disabled="loading.getConfiguration || loading.configureModule"
                tooltipAlignment="start"
                tooltipDirection="top"
                ref="ldap_domain"
            >
              <template slot="tooltip">
                {{
                  $t("settings.choose_the_ldap_domain_to_authenticate_users")
                }}
              </template>
            </NsComboBox>
            <cv-accordion ref="accordion" class="maxwidth mg-bottom">
              <cv-accordion-item :open="toggleAccordion[0]">
                <template slot="title">{{ $t("settings.advanced") }}</template>
                <template slot="content">
                  <cv-text-area
                      :label="$t('settings.adminList')"
                      v-model.trim="admin_users"
                      :invalid-message="error.admin_users"
                      :helper-text="$t('settings.Write_administrator_list')"
                      :value="admin_users"
                      class="maxwidth textarea mg-left"
                      ref="admin_users"
                      :placeholder="$t('settings.Write_administrator_list')"
                      :disabled="
                      loading.getConfiguration || loading.configureModule
                    "
                  >
                  </cv-text-area>
                </template>
              </cv-accordion-item>
            </cv-accordion>
            <cv-toggle
              value="letsEncrypt"
              :label="$t('settings.lets_encrypt')"
              v-model="isLetsEncryptEnabled"
              :disabled="loading.getConfiguration || loading.configureModule"
              class="mg-bottom"
            >
              <template slot="text-left">{{
                $t("settings.disabled")
              }}</template>
              <template slot="text-right">{{
                $t("settings.enabled")
              }}</template>
            </cv-toggle>
            <cv-toggle
              value="httpToHttps"
              :label="$t('settings.http_to_https')"
              v-model="isHttpToHttpsEnabled"
              :disabled="loading.getConfiguration || loading.configureModule"
              class="mg-bottom"
            >
              <template slot="text-left">{{
                $t("settings.disabled")
              }}</template>
              <template slot="text-right">{{
                $t("settings.enabled")
              }}</template>
            </cv-toggle>
              <!-- advanced options -->
            <cv-accordion ref="accordion" class="maxwidth mg-bottom">
              <cv-accordion-item :open="toggleAccordion[0]">
                <template slot="title">{{ $t("settings.advanced") }}</template>
                <template slot="content">
                </template>
              </cv-accordion-item>
            </cv-accordion>
            <cv-row v-if="error.configureModule">
              <cv-column>
                <NsInlineNotification
                  kind="error"
                  :title="$t('action.configure-module')"
                  :description="error.configureModule"
                  :showCloseButton="false"
                />
              </cv-column>
            </cv-row>
            <NsButton
              kind="primary"
              :icon="Save20"
              :loading="loading.configureModule"
              :disabled="loading.getConfiguration || loading.configureModule"
              >{{ $t("settings.save") }}</NsButton
            >
          </cv-form>
        </cv-tile>
      </cv-column>
    </cv-row>
  </cv-grid>
</template>

<script>
import to from "await-to-js";
import { mapState } from "vuex";
import {
  QueryParamService,
  UtilService,
  TaskService,
  IconService,
  PageTitleService,
} from "@nethserver/ns8-ui-lib";

export default {
  name: "Settings",
  mixins: [
    TaskService,
    IconService,
    UtilService,
    QueryParamService,
    PageTitleService,
  ],
  pageTitle() {
    return this.$t("settings.title") + " - " + this.appName;
  },
  data() {
    return {
      q: {
        page: "settings",
      },
      urlCheckInterval: null,
      host: "",
      isLetsEncryptEnabled: false,
      isHttpToHttpsEnabled: true,
      DOLI_ADMIN_LOGIN: "",
      DOLI_ADMIN_PASSWORD: "",
      ldap_domain: "",
      admin_users: "",
      user_domains_list: [],
      loading: {
        getConfiguration: false,
        configureModule: false,
      },
      error: {
        getConfiguration: "",
        configureModule: "",
        host: "",
        lets_encrypt: "",
        http2https: "",
        DOLI_ADMIN_LOGIN: "",
        DOLI_ADMIN_PASSWORD: "",
        ldap_domain: "",
        admin_users: "",
      },
    };
  },
  computed: {
    ...mapState(["instanceName", "core", "appName"]),
  },
  created() {
    this.getConfiguration();
  },
  beforeRouteEnter(to, from, next) {
    next((vm) => {
      vm.watchQueryData(vm);
      vm.urlCheckInterval = vm.initUrlBindingForApp(vm, vm.q.page);
    });
  },
  beforeRouteLeave(to, from, next) {
    clearInterval(this.urlCheckInterval);
    next();
  },
  methods: {
    async getConfiguration() {
      this.loading.getConfiguration = true;
      this.error.getConfiguration = "";
      const taskAction = "get-configuration";
      const eventId = this.getUuid();

      // register to task error
      this.core.$root.$once(
        `${taskAction}-aborted-${eventId}`,
        this.getConfigurationAborted
      );

      // register to task completion
      this.core.$root.$once(
        `${taskAction}-completed-${eventId}`,
        this.getConfigurationCompleted
      );

      const res = await to(
        this.createModuleTaskForApp(this.instanceName, {
          action: taskAction,
          extra: {
            title: this.$t("action." + taskAction),
            isNotificationHidden: true,
            eventId,
          },
        })
      );
      const err = res[0];

      if (err) {
        console.error(`error creating task ${taskAction}`, err);
        this.error.getConfiguration = this.getErrorMessage(err);
        this.loading.getConfiguration = false;
        return;
      }
    },
    getConfigurationAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.getConfiguration = this.$t("error.generic_error");
      this.loading.getConfiguration = false;
    },
    getConfigurationCompleted(taskContext, taskResult) {
      const config = taskResult.output;
      this.host = config.host;
      this.DOLI_ADMIN_LOGIN = config.DOLI_ADMIN_LOGIN
      this.DOLI_ADMIN_PASSWORD = config.DOLI_ADMIN_PASSWORD
      this.isLetsEncryptEnabled = config.lets_encrypt;
      this.isHttpToHttpsEnabled = config.http2https;
      this.admin_users = config.admin_users.split(",").join("\n");
      // force to reload mail_server value after dom update
      this.$nextTick(() => {
        this.ldap_domain = config.ldap_domain;
      });
      this.user_domains_list = config.user_domains_list;
      this.loading.getConfiguration = false;
      this.focusElement("host");
    },
    isValidUser(user) {
      // test if user is valid login
      const re = /^[a-zA-Z0-9._-]+$/;
      return re.test(user);
    },
    validateConfigureModule() {
      this.clearErrors(this);

      let isValidationOk = true;
      if (!this.host) {
        this.error.host = "common.required";

        if (isValidationOk) {
          this.focusElement("host");
        }
        isValidationOk = false;
      }
      if (!this.ldap_domain) {
        this.error.ldap_domain = "common.required";

        if (isValidationOk) {
          this.focusElement("ldap_domain");
        }
        isValidationOk = false;
      }
      if (this.admin_users) {
        // test if the admin_users is valid
        const admin_users = this.admin_users.split("\n");
        for (const user of admin_users) {
          if (!this.isValidUser(user)) {
            this.toggleAccordion[0] = true;
            // set i18n error message and return user in object
            this.error.admin_users = this.$t("settings.invalid_user", {
              user: user,
            });
            isValidationOk = false;
            if (isValidationOk) {
              this.focusElement("admin_users");
            }
            break;
          }
        }
      }
      return isValidationOk;
    },
    configureModuleValidationFailed(validationErrors) {
      this.loading.configureModule = false;
      let focusAlreadySet = false;

      for (const validationError of validationErrors) {
        const param = validationError.parameter;
        // set i18n error message
        this.error[param] = this.$t("settings." + validationError.error);

        if (!focusAlreadySet) {
          this.focusElement(param);
          focusAlreadySet = true;
        }
      }
    },
    async configureModule() {
      this.error.test_imap = false;
      this.error.test_smtp = false;
      const isValidationOk = this.validateConfigureModule();
      if (!isValidationOk) {
        return;
      }

      this.loading.configureModule = true;
      const taskAction = "configure-module";
      const eventId = this.getUuid();

      // register to task error
      this.core.$root.$once(
        `${taskAction}-aborted-${eventId}`,
        this.configureModuleAborted
      );

      // register to task validation
      this.core.$root.$once(
        `${taskAction}-validation-failed-${eventId}`,
        this.configureModuleValidationFailed
      );

      // register to task completion
      this.core.$root.$once(
        `${taskAction}-completed-${eventId}`,
        this.configureModuleCompleted
      );
      const res = await to(
        this.createModuleTaskForApp(this.instanceName, {
          action: taskAction,
          data: {
            host: this.host,
            DOLI_ADMIN_PASSWORD: this.DOLI_ADMIN_PASSWORD,
            DOLI_ADMIN_LOGIN: this.DOLI_ADMIN_LOGIN,
            lets_encrypt: this.isLetsEncryptEnabled,
            http2https: this.isHttpToHttpsEnabled,
            ldap_domain: this.ldap_domain,
            admin_users: this.admin_users.split("\n").join(",").toLowerCase().trim(),
          },
          extra: {
            title: this.$t("settings.instance_configuration", {
              instance: this.instanceName,
            }),
            description: this.$t("settings.configuring"),
            eventId,
          },
        })
      );
      const err = res[0];

      if (err) {
        console.error(`error creating task ${taskAction}`, err);
        this.error.configureModule = this.getErrorMessage(err);
        this.loading.configureModule = false;
        return;
      }
    },
    configureModuleAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.configureModule = this.$t("error.generic_error");
      this.loading.configureModule = false;
    },
    configureModuleCompleted() {
      this.loading.configureModule = false;

      // reload configuration
      this.getConfiguration();
    },
  },
};
</script>

<style scoped lang="scss">
@import "../styles/carbon-utils";
.mg-bottom {
  margin-bottom: $spacing-06;
}

.maxwidth {
  max-width: 38rem;
}
</style>
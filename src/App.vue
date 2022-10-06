<template>
  <v-app>
    <v-app-bar app color="grey darken-4 white--text">
      <v-toolbar-title>
        <v-icon left color="white">{{ mdiKeyboard }}</v-icon>
        TinyPICO BLE Keypad by DriftKingTW
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn
        icon
        color="white"
        @click="$vuetify.theme.dark = !$vuetify.theme.dark"
        class="mr-2"
      >
        <v-icon>{{ mdiThemeLightDark }}</v-icon>
      </v-btn>
    </v-app-bar>

    <v-main>
      <v-container fluid>
        <v-row>
          <v-col cols="12" md="6">
            <v-card flat :loading="isInitializing">
              <v-card-title>
                <v-icon left>{{ mdiHammerScrewdriver }}</v-icon>
                Keymap Configuration Update
              </v-card-title>
              <v-card-text>
                <v-textarea
                  v-model="keyconfig"
                  :disabled="isInitializing"
                  outlined
                  hide-details
                  :prepend-icon="mdiCodeJson"
                  label="Content of keyconfig.json"
                ></v-textarea>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                  text
                  color="primary"
                  :loading="isUpdatingConfig"
                  :disabled="isInitializing"
                  @click="beautifyConfig('keyconfig')"
                >
                  <v-icon left small>{{ mdiCodeJson }}</v-icon>
                  Beautify JSON
                </v-btn>
                <v-btn
                  text
                  color="primary"
                  :loading="isUpdatingConfig"
                  :disabled="isInitializing"
                  @click="updateConfig('keyconfig')"
                >
                  <v-icon left small>{{ mdiCloudUpload }}</v-icon>
                  Update
                </v-btn>
              </v-card-actions>
            </v-card>
            <v-card flat :loading="isInitializing">
              <v-card-title>
                <v-icon left>{{ mdiHammerScrewdriver }}</v-icon>
                Macros Configuration Update
              </v-card-title>
              <v-card-text>
                <v-textarea
                  v-model="macros"
                  :disabled="isInitializing"
                  outlined
                  hide-details
                  :prepend-icon="mdiCodeJson"
                  label="Content of macros.json"
                ></v-textarea>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                  text
                  color="primary"
                  :loading="isUpdatingConfig"
                  :disabled="isInitializing"
                  @click="beautifyConfig('macros')"
                >
                  <v-icon left small>{{ mdiCodeJson }}</v-icon>
                  Beautify JSON
                </v-btn>
                <v-btn
                  text
                  color="primary"
                  :loading="isUpdatingConfig"
                  :disabled="isInitializing"
                  @click="updateConfig('macros')"
                >
                  <v-icon left small>{{ mdiCloudUpload }}</v-icon>
                  Update
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-col>
          <v-col cols="12" md="6">
            <v-card>
              <v-card-title>
                <v-icon left>{{ mdiSd }}</v-icon>
                SPIFFS Information
              </v-card-title>
              <v-card-text>
                <v-data-iterator :items="spiffs.files" hide-default-footer>
                  <template v-slot:default="props">
                    <v-row>
                      <v-col cols="6">
                        <v-list dense>
                          <v-list-item>
                            <v-list-item-content class="align-end">
                              <span>
                                <v-icon small left>{{ mdiDatabase }}</v-icon>
                                Total
                              </span>
                            </v-list-item-content>
                            <v-list-item-content class="align-end">
                              {{ spiffs.total }}
                            </v-list-item-content>
                          </v-list-item>
                          <v-list-item>
                            <v-list-item-content class="align-end">
                              <span>
                                <v-icon small left>
                                  {{ mdiFileDocumentMultiple }}
                                </v-icon>
                                Used
                              </span>
                            </v-list-item-content>
                            <v-list-item-content class="align-end">
                              {{ spiffs.used }}
                            </v-list-item-content>
                          </v-list-item>
                          <v-list-item>
                            <v-list-item-content class="align-end">
                              <span>
                                <v-icon small left>{{ mdiCheck }}</v-icon>
                                Free
                              </span>
                            </v-list-item-content>
                            <v-list-item-content class="align-end">
                              {{ spiffs.free }}
                            </v-list-item-content>
                          </v-list-item>
                        </v-list>
                      </v-col>
                      <v-col cols="6">
                        <v-list dense>
                          <v-list-item
                            v-for="data in props.items"
                            :key="data.name"
                          >
                            <v-list-item-content class="align-end">
                              <span>
                                <v-icon small left>{{ mdiCodeJson }}</v-icon>
                                {{ data.name }}
                              </span>
                            </v-list-item-content>
                            <v-list-item-content class="align-end">
                              {{ data.size }}
                            </v-list-item-content>
                          </v-list-item>
                        </v-list>
                      </v-col>
                    </v-row>
                  </template>
                </v-data-iterator>
              </v-card-text>
              <v-card-title>
                <v-icon left>{{ mdiWifi }}</v-icon>
                Network Information
              </v-card-title>
              <v-card-text>
                <v-row>
                  <v-col cols="12" md="5">
                    <v-text-field
                      label="SSID"
                      v-model="network.ssid"
                      outlined
                      dense
                    >
                    </v-text-field>
                  </v-col>
                  <v-col cols="12" md="5">
                    <v-text-field
                      label="Password"
                      v-model="network.password"
                      outlined
                      dense
                      :append-icon="showPassword ? mdiEye : mdiEyeOff"
                      :type="showPassword ? 'text' : 'password'"
                      @click:append="showPassword = !showPassword"
                      :rules="[rules.required, rules.min]"
                    >
                    </v-text-field>
                  </v-col>
                  <v-col cols="12" md="2">
                    <div class="d-flex">
                      <v-spacer></v-spacer>
                      <v-btn
                        text
                        color="primary"
                        @click="updateNetworkConfig"
                        :loading="isUpdatingConfig"
                        :disabled="isInitializing"
                      >
                        <v-icon left>{{ mdiCheck }}</v-icon>
                        Update
                      </v-btn>
                    </div>
                  </v-col>
                  <v-col cols="6">
                    <v-list dense>
                      <v-list-item>
                        <v-list-item-content class="align-end">
                          <span>
                            <v-icon small left>{{ mdiIpNetwork }}</v-icon>
                            IP
                          </span>
                        </v-list-item-content>
                        <v-list-item-content class="align-end">
                          {{ network.ip }}
                        </v-list-item-content>
                      </v-list-item>

                      <v-list-item>
                        <v-list-item-content class="align-end">
                          <span>
                            <v-icon small left>{{ mdiNetwork }}</v-icon>
                            Subnet Mask
                          </span>
                        </v-list-item-content>
                        <v-list-item-content class="align-end">
                          {{ network.subnetMask }}
                        </v-list-item-content>
                      </v-list-item>

                      <v-list-item>
                        <v-list-item-content class="align-end">
                          <span>
                            <v-icon small left>{{
                              mdiNetworkStrength4
                            }}</v-icon>
                            RSSI
                          </span>
                        </v-list-item-content>
                        <v-list-item-content class="align-end">
                          {{ network.rssi }}
                        </v-list-item-content>
                      </v-list-item>
                    </v-list>
                  </v-col>
                  <v-col cols="6">
                    <v-list dense>
                      <v-list-item>
                        <v-list-item-content class="align-end">
                          <span>
                            <v-icon small left>
                              {{ mdiAccessPointNetwork }}
                            </v-icon>
                            Soft AP IP
                          </span>
                        </v-list-item-content>
                        <v-list-item-content class="align-end">
                          {{ network.apIp }}
                        </v-list-item-content>
                      </v-list-item>

                      <v-list-item>
                        <v-list-item-content class="align-end">
                          <span>
                            <v-icon small left>{{ mdiRouterNetwork }}</v-icon>
                            Gateway IP
                          </span>
                        </v-list-item-content>
                        <v-list-item-content class="align-end">
                          {{ network.gatewayIP }}
                        </v-list-item-content>
                      </v-list-item>

                      <v-list-item>
                        <v-list-item-content class="align-end">
                          <span>
                            <v-icon small left>
                              {{ mdiExpansionCardVariant }}
                            </v-icon>
                            MAC Address
                          </span>
                        </v-list-item-content>
                        <v-list-item-content class="align-end">
                          {{ network.mac }}
                        </v-list-item-content>
                      </v-list-item>
                    </v-list>
                  </v-col>
                </v-row>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
      <v-snackbar
        v-model="snackbar"
        :timeout="snackbarTimeout"
        :color="snackbarColor"
      >
        <v-icon v-if="snackbarColor === 'info'" small left>
          {{ mdiInformationOutline }}
        </v-icon>
        <v-icon v-else-if="snackbarColor === 'success'" small left>
          {{ mdiCheck }}
        </v-icon>
        <v-icon v-else-if="snackbarColor === 'error'" small left>
          {{ mdiAlert }}
        </v-icon>

        {{ snackbarText }}

        <template v-slot:action="{ attrs }">
          <v-btn text v-bind="attrs" @click="closeSnackbar"> Close </v-btn>
        </template>
      </v-snackbar>
    </v-main>
  </v-app>
</template>

<script>
import {
  mdiKeyboard,
  mdiThemeLightDark,
  mdiHammerScrewdriver,
  mdiCodeJson,
  mdiCloudUpload,
  mdiSd,
  mdiDatabase,
  mdiFileDocumentMultiple,
  mdiCheck,
  mdiWifi,
  mdiEye,
  mdiEyeOff,
  mdiIpNetwork,
  mdiNetwork,
  mdiNetworkStrength4,
  mdiAccessPointNetwork,
  mdiRouterNetwork,
  mdiExpansionCardVariant,
  mdiInformationOutline,
  mdiAlert,
} from "@mdi/js";

export default {
  name: "App",

  components: {},

  data() {
    return {
      tinypicoUrl: "",
      keyconfig: "",
      macros: "",
      serverResponse: "",
      spiffs: [],
      network: {
        ssid: "",
        password: "",
      },
      networkInfo: {},
      snackbar: false,
      snackbarTimeout: 3000,
      snackbarColor: "",
      snackbarText: "",
      isUpdatingConfig: false,
      isInitializing: false,
      showPassword: false,
      rules: {
        required: (value) => !!value || "Required.",
        min: (v) => v.length >= 8 || "Min 8 characters",
      },
      // Icons
      mdiKeyboard,
      mdiThemeLightDark,
      mdiHammerScrewdriver,
      mdiCodeJson,
      mdiCloudUpload,
      mdiSd,
      mdiDatabase,
      mdiFileDocumentMultiple,
      mdiCheck,
      mdiWifi,
      mdiEye,
      mdiEyeOff,
      mdiIpNetwork,
      mdiNetwork,
      mdiNetworkStrength4,
      mdiAccessPointNetwork,
      mdiRouterNetwork,
      mdiExpansionCardVariant,
      mdiInformationOutline,
      mdiAlert,
    };
  },

  mounted() {
    let dark = window.matchMedia("(prefers-color-scheme: dark)").matches;
    if (dark) this.$vuetify.theme.dark = true;
    else this.$vuetify.theme.dark = false;

    let protocol = location.protocol;
    let host = location.host;
    this.tinypicoUrl = `${protocol}//${host}`;
    // this.tinypicoUrl = `http://tp-keypad.local`;
    console.log(this.tinypicoUrl);

    this.initialize();
  },

  methods: {
    async initialize() {
      this.isInitializing = true;
      let res = await fetch(`${this.tinypicoUrl}/api/config?type=keyconfig`);
      let data = await res.json();
      this.keyconfig = JSON.stringify(data.config);
      res = await fetch(`${this.tinypicoUrl}/api/config?type=macros`);
      data = await res.json();
      this.macros = JSON.stringify(data.config);
      res = await fetch(`${this.tinypicoUrl}/api/spiffs`);
      data = await res.json();
      this.spiffs = JSON.parse(JSON.stringify(data));
      res = await fetch(`${this.tinypicoUrl}/api/network`);
      data = await res.json();
      this.network = JSON.parse(JSON.stringify(data.wifi));
      this.isInitializing = false;
    },

    beautifyConfig(type) {
      if (type === "keyconfig") {
        this.keyconfig = JSON.stringify(JSON.parse(this.keyconfig), null, 4);
      } else if (type === "macros") {
        this.macros = JSON.stringify(JSON.parse(this.macros), null, 4);
      }
    },

    async updateConfig(type) {
      this.isUpdatingConfig = true;
      try {
        const options = {
          method: "PUT",
          headers: {
            Accept: "application/json",
            "Content-Type": "application/json;charset=UTF-8",
          },
          body: this[type],
        };
        const res = await fetch(
          `${this.tinypicoUrl}/api/config?type=${type}`,
          options
        );
        const data = await res.json();
        if (data.message === "success") {
          this.serverResponse = "Configurations updated";
          this.triggerSnackbar({
            status: "success",
            text: this.serverResponse,
          });
        } else {
          this.serverResponse = data.message;
          this.triggerSnackbar({ status: "error", text: this.serverResponse });
        }
      } catch (e) {
        console.log(e);
        this.serverResponse = e;
      }
      this.isUpdatingConfig = false;
    },

    async updateNetworkConfig() {
      this.isUpdatingConfig = true;
      try {
        const options = {
          method: "PUT",
          headers: {
            Accept: "application/json",
            "Content-Type": "application/json;charset=UTF-8",
          },
          body: JSON.stringify({
            ssid: this.network.ssid,
            password: this.network.password,
          }),
        };
        const res = await fetch(`${this.tinypicoUrl}/api/network`, options);
        const data = await res.json();
        if (data.message === "success") {
          this.serverResponse = "Network configurations updated";
          this.triggerSnackbar({
            status: "success",
            text: this.serverResponse,
          });
        } else {
          this.serverResponse = data.message;
          this.triggerSnackbar({ status: "error", text: this.serverResponse });
        }
      } catch (e) {
        console.log(e);
        this.serverResponse = e;
      }
      this.isUpdatingConfig = false;
    },

    triggerSnackbar({ status, text }) {
      if (status) this.snackbarColor = status;
      if (text) this.snackbarText = text;
      this.snackbar = true;
    },

    closeSnackbar() {
      this.snackbar = false;
    },
  },
};
</script>

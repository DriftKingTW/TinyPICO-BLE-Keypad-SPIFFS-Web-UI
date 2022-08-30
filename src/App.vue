<template>
  <v-app>
    <v-app-bar app color="grey darken-4 white--text">
      <v-toolbar-title>
        <v-icon left color="white">mdi-keyboard</v-icon>
        TinyPICO BLE Keypad by DriftKingTW
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn
        icon
        color="white"
        @click="$vuetify.theme.dark = !$vuetify.theme.dark"
        class="mr-2"
      >
        <v-icon>mdi-theme-light-dark</v-icon>
      </v-btn>
    </v-app-bar>

    <v-main>
      <v-container fluid>
        <v-row>
          <v-col cols="12" md="6">
            <v-card flat :loading="isInitializing">
              <v-card-title>
                <v-icon left>mdi-hammer-screwdriver</v-icon>
                Keymap Configuration Update
              </v-card-title>
              <v-card-text>
                <v-textarea
                  v-model="keyconfig"
                  :disabled="isInitializing"
                  outlined
                  hide-details
                  prepend-icon="mdi-code-json"
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
                  @click="updateConfig"
                >
                  <v-icon left small>mdi-cloud-upload</v-icon>
                  Update
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-col>
          <v-col cols="12" md="6">
            <v-card>
              <v-card-title>
                <v-icon left>mdi-sd</v-icon>
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
                                <v-icon small left>mdi-database</v-icon>
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
                                <v-icon small left
                                  >mdi-file-document-multiple
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
                                <v-icon small left>mdi-check</v-icon>
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
                                <v-icon small left>mdi-code-json</v-icon>
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
                <v-icon left>mdi-wifi</v-icon>
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
                      :append-icon="showPassword ? 'mdi-eye' : 'mdi-eye-off'"
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
                        <v-icon left>mdi-check</v-icon>
                        Update
                      </v-btn>
                    </div>
                  </v-col>
                  <v-col cols="6">
                    <v-list dense>
                      <v-list-item>
                        <v-list-item-content class="align-end">
                          <span>
                            <v-icon small left>mdi-ip-network</v-icon>
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
                            <v-icon small left>mdi-network</v-icon>
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
                            <v-icon small left>mdi-network-strength-4</v-icon>
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
                            <v-icon small left
                              >mdi-access-point-network
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
                            <v-icon small left>mdi-router-network</v-icon>
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
                            <v-icon small left
                              >mdi-expansion-card-variant</v-icon
                            >
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
          mdi-information-outline
        </v-icon>
        <v-icon v-else-if="snackbarColor === 'success'" small left>
          mdi-check
        </v-icon>
        <v-icon v-else-if="snackbarColor === 'error'" small left>
          mdi-alert
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
export default {
  name: "App",

  components: {},

  data() {
    return {
      tinypicoUrl: "",
      keyconfig: "",
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
      let res = await fetch(`${this.tinypicoUrl}/api/keyconfig`);
      let data = await res.json();
      this.keyconfig = JSON.stringify(data.keyconfig);
      res = await fetch(`${this.tinypicoUrl}/api/spiffs`);
      data = await res.json();
      this.spiffs = JSON.parse(JSON.stringify(data));
      res = await fetch(`${this.tinypicoUrl}/api/network`);
      data = await res.json();
      this.network = JSON.parse(JSON.stringify(data.wifi));
      this.isInitializing = false;
    },

    async updateConfig() {
      this.isUpdatingConfig = true;
      try {
        const options = {
          method: "PUT",
          headers: {
            Accept: "application/json",
            "Content-Type": "application/json;charset=UTF-8",
          },
          body: this.keyconfig,
        };
        const res = await fetch(`${this.tinypicoUrl}/api/keyconfig`, options);
        const data = await res.json();
        if (data.message === "success") {
          this.serverResponse = "Key configurations updated";
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

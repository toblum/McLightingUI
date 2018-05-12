<template>
  <div id="app">

    <v-app v-bind:dark="dark_theme">
      <v-toolbar color="primary" dark>
        <!-- <v-toolbar-side-icon></v-toolbar-side-icon> -->
        <v-toolbar-title>Mc Lighting UI</v-toolbar-title>
        <v-spacer></v-spacer>
        <v-spacer></v-spacer>
        <span class="group pa-2" v-if="is_connected"><v-icon>wifi</v-icon>&nbsp;<span class="hidden-xs-only">&nbsp;McLighting </span>connected</span>
        <span class="group pa-2" v-else><v-icon>signal_wifi_off</v-icon>&nbsp;&nbsp;McLighting disconnected</span>
      </v-toolbar>

      <v-content id="content" class="mx-auto">
        <v-card>
          <v-layout row wrap>
            <v-flex xs12 sm5>
              <ColorPicker v-on:selected="onColorSelected" :prop_color="color" :prop_type="picker_type"/>
            </v-flex>
            <v-flex xs12 sm7>
              <v-card-text>
                <v-slider v-model="color.r" hint="Red" label="R" :max="255" v-on:input="set_color" color="red"></v-slider>
                <v-slider v-model="color.g" hint="Green" label="G" :max="255" v-on:input="set_color" color="green"></v-slider>
                <v-slider v-model="color.b" hint="Blue" label="B" :max="255" v-on:input="set_color" color="blue"></v-slider>
                <v-slider v-model="brightness" prepend-icon="wb_incandescent" hint="Brightness" :max="255" step="12" ticks v-on:input="set_brightness"></v-slider>
                <v-slider v-model="speed" prepend-icon="slow_motion_video" hint="Speed" :max="255" step="12" ticks v-on:input="set_speed"></v-slider>
              </v-card-text>
            </v-flex>
          </v-layout>

          <v-container pb-5>
            <v-layout row wrap>
              <v-flex xs12 sm6 lg4 xl3 v-for="(mode, index) in modes" :key="mode.id" v-if="edit_mode || !mode.hidden">
                <span v-if="edit_mode">
                  <v-icon
                    color="grey lighten-1"
                    @click="toggle(mode, index)"
                    v-if="mode.hidden === true"
                    >star_border</v-icon>
                  <v-icon
                    color="yellow darken-2"
                    @click="toggle(mode, index)"
                    v-else>star</v-icon>
                </span>

                <v-btn :id="mode.id" v-on:click="set_mode(mode.id)" class="elevation-6 white--text" v-bind:class="[modeIsActive(mode) ? 'red' : 'blue']">{{ mode.title }} ({{ mode.id }})</v-btn>
              </v-flex>
            </v-layout>
          </v-container>

          <v-fab-transition>
            <v-btn
              v-show="!edit_mode"
              transition="scale-transition"
              @click.native.stop="edit_mode = !edit_mode"
              color="pink"
              dark
              absolute
              bottom
              right
              fab
              hover
            >
              <v-icon>settings</v-icon>
            </v-btn>
          </v-fab-transition>
        </v-card>

        <v-card row wrap v-if="edit_mode">
          <v-layout row wrap class="pa-3 my-2">
            <v-flex xs12 sm6>
              <p>Dark mode:</p>
              <v-switch :label="`${(dark_theme) ? 'ON' : 'OFF'} `" v-model="dark_theme" color="primary"></v-switch>
            </v-flex>
            <v-flex xs12 sm6>
              <p>Select color picker:</p>
              <v-btn-toggle v-model="picker_type" mandatory>
                <v-btn flat value="circle">
                  Circle
                </v-btn>
                <v-btn flat value="wheel">
                  Wheel
                </v-btn>
              </v-btn-toggle>
            </v-flex>
            <v-flex xs12>
              <v-btn block color="green" dark class="elevation-6 mb-2" @click="saveSettings()"><v-icon>save</v-icon> Save settings</v-btn>
            </v-flex>
          </v-layout>
        </v-card>

        <v-snackbar
          :timeout="snackbar_timeout"
          top
          :color="snackbar_color"
          v-model="snackbar"
        >
          {{ snackbar_text }}
        </v-snackbar>

        <v-footer color="primary white--text" class="pa-3">
          <div><a href="https://github.com/toblum/McLighting/" target="_blank" class="white--text">Project home</a> - &copy; {{ new Date().getFullYear() }}</div>
          <v-spacer></v-spacer>
        </v-footer>
      </v-content>
    </v-app>

  </div>
</template>

<script>
/* eslint-disable */
import ColorPicker from "./components/ColorPicker";
import * as RWS from 'reconnecting-websocket';

var host = "192.168.0.49";

export default {
  name: "McLightingUI",
  
  components: {
    ColorPicker
  },

  data: () => ({
    color: {r:0, g:0, b:0, hex:"000000"},
    brightness: 192,
    speed: 192,
    modes: [{ title: "TV", id: "tv" }],
    connection: null,
    is_connected: false,
    dark_theme: false,
    ws2812fx_mode: null,
    settings: {},
    edit_mode: false,
    picker_type: "circle",
    snackbar: false,
    snackbar_text: "",
    snackbar_color: "info",
    snackbar_timeout: 2000
  }),

  methods: {
    getModes() {
      var that = this;
      this.$http.get("//" + host + "/get_modes").then(data => {
        // console.log("Getting modes list via REST:", data);
        data.body.forEach(item => {
          if (item.name && item.name.length > 0) {
            that.modes.push({ title: item.name, id: item.mode });
          }
        });
        this.readSettings();
      });
    },
    modeIsActive(mode) {
      return mode.id === this.ws2812fx_mode;
    },

    showSnackbar(message, color, timeout) {
      this.snackbar_text = message;
      this.snackbar_color = color;
      this.snackbar_timeout = timeout;
      this.snackbar = true;
    },

    readSettings() {
      this.$http.get("//" + host + "/uistate.json").then(data => {
        console.log("readSettings()", data.body);
        this.settings = data.body || {};
        this.applySettings();
      });
    },
    applySettings() {
      this.modes.forEach((mode, index) => {
        if (this.settings.visibility.includes(mode.id)) {
          mode.hidden = true;
          this.$set(this.modes, index, mode);
        }
      });
      this.dark_theme = this.settings.dark_theme || false;
      this.picker_type = this.settings.picker_type || "circle";
    },
    saveSettings() {
      var visibility = this.modes.map(mode => {
        if (mode.hidden) {
          return mode.id;
        }
      }).filter((x) => {return x >= 0});
      this.settings.visibility = visibility;
      this.settings.dark_theme = this.dark_theme;
      this.settings.picker_type = this.picker_type;

      var formData = new FormData();
      var blob = new Blob([JSON.stringify(this.settings)], {type: 'application/json'});
      formData.append('settings', blob, '/uistate.json');

      this.$http.post("//" + host + "/edit", formData).then(data => {
        console.log("SUCCESS saveSettings()", data, this.settings);
        this.showSnackbar("Settings saved", "success", 1500);
        this.edit_mode = false;
      }, err => {
        console.error("ERROR saveSettings()", err);
        this.showSnackbar("Error saving settings, please try again...", "error", 5000);
      });
    },

    toggle(mode, index) {
      mode.hidden = !mode.hidden;
      this.$set(this.modes, index, mode);
    },

    ws_connect() {
      var that = this;
      this.connection = new RWS("ws://" + host + ":81");
      this.connection.debug = true;
      // connection.timeoutInterval = 5400;

      // When the connection is open, send some data to the server
      this.connection.onopen = function() {
        console.log("WebSocket Open");
        that.is_connected = true;
        that.ws_send("$");
      };

      // When the connection is open, send some data to the server
      this.connection.onclose = function() {
        console.log("WebSocket Closed");
        that.is_connected = false;
      };

      // Log errors
      this.connection.onerror = function(error) {
        that.is_connected = false;
        console.error("WebSocket Error", error);
      };

      // Log messages from the server
      this.connection.onmessage = function(e) {
        console.log("WebSocket from server:", e.data);
        try {
          var res = JSON.parse(e.data);
          // console.log("res", res);
          if (res && res.ws2812fx_mode) {
            that.ws2812fx_mode = res.ws2812fx_mode;
          }
          if (res && res.speed) {
            that.speed = res.speed;
          }
          if (res && res.brightness) {
            that.brightness = res.brightness;
          }
          if (res && res.color) {
            that.color.r = res.color[0];
            that.color.g = res.color[1];
            that.color.b = res.color[2];
            that.color.hex = that.rgbToHex([that.color.r, that.color.g, that.color.b]);
          }
        } catch (e) {}
      };
    },

    ws_send(message) {
      console.log("WS send: ", message);
      this.connection.send(message);
    },

    set_mode(mode_id) {
      this.ws2812fx_mode = mode_id;
      this.ws_send("/" + mode_id);
    },
    set_speed(speed) {
      this.ws_send("?" + speed);
    },
    set_brightness(brightness) {
      this.ws_send("%" + brightness);
    },
    set_color() {
      this.ws_send(
        "#" + this.rgbToHex([this.color.r, this.color.g, this.color.b])
      );
    },

    componentToHex(c) {
      return ("0" + Number(c).toString(16)).slice(-2).toUpperCase();
    },
    rgbToHex(rgb) {
      return (
        this.componentToHex(rgb[0]) +
        this.componentToHex(rgb[1]) +
        this.componentToHex(rgb[2])
      );
    },

    onColorSelected(color) {
      this.color = color;
      this.set_color();
    }
  },

  mounted() {
    this.getModes();
    this.ws_connect();
  }
};
</script>

<style>
#content {
  max-width: 1380px;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
</style>

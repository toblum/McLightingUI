<template>
    <div class="mt-3">
        <!-- <v-text-field v-model="message" label=""></v-text-field> -->
        <v-jumbotron color="primary elevation-6" dark>
            <v-container fill-height>
                <v-layout align-center>
                    <v-flex text-xs-center>
                        <div class="display-2">{{ message1 }}</div>
                        <div class="display-2"><v-progress-circular indeterminate color="white"></v-progress-circular>&nbsp;{{ message2 }}</div>
                        <v-progress-linear v-model="progress" color="white"></v-progress-linear>
                    </v-flex>
                </v-layout>
            </v-container>
        </v-jumbotron>
    </div>
</template>

<script>
export default {
  name: "Loader",

  data() {
    return {
    };
  },

  props: ["state", "max_states"],
  computed: {
    progress: function() {
      return this.state / this.max_states * 100;
    },
    message1: function() {
      let msg = "";
      if (this.state === 1) {
        msg = "Loading modes ...";
      }
      if (this.state === 2) {
        msg = "Loading settings ...";
      }
      if (this.state === 3) {
        msg = "Connecting websockets ...";
      }
      if (this.state === 4) {
        msg = "Ready ...";
      }
      return msg;
    },
    message2: function() {
      let msg = "";
      msg = "step " + this.state + " / " + this.max_states + "";
      return msg;
    }
  },
  watch: {},

  methods: {},
  mounted() {}
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>

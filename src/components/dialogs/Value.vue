<template>
  <v-dialog v-model="value" max-width="500px" persistent>
    <v-card>
      <v-card-title>
        <span class="headline">{{ title }}</span>
      </v-card-title>

      <v-card-text>
        <v-container grid-list-md>
          <v-form v-model="valid" ref="form" lazy-validation>
            <v-layout wrap>
              <v-flex xs12>
                <v-autocomplete
                  v-model="editedValue.client_id"
                  label="Client"
                  hint="The destination MQTT Client"
                  :rules="[required]"
                  persistent-hint
                  required
                  item-text="name"
                  item-value="_id"
                  :items="clients"
                ></v-autocomplete>
              </v-flex>

               <v-flex xs12 sm7>
                <v-select
                  v-model="editedValue.mode"
                  label="Mode"
                  hint="Specify if the broker need to subscribe (SET) or publish (GET) on this topic"
                  persistent-hint
                  required
                  :items="optionsMode"
                ></v-select>
              </v-flex>

              <v-flex xs12 sm5>
                <v-switch
                  v-model="editedValue.customTopic"
                  persistent-hint
                  hint="Enable this to use a custom topic"
                  label="Custom topic"
                ></v-switch>
              </v-flex>
              <v-flex xs12 v-bind="{[`sm${editedValue.customTopic ? 6 : 12}`]: true}">
                <v-text-field
                  v-model.trim="editedValue.from"
                  label="Topic"
                  :rules="[validTopic]"
                  :append-outer-icon="editedValue.customTopic ? 'arrow_right_alt' : ''"
                  required
                ></v-text-field>
              </v-flex>
              <v-flex v-if="editedValue.customTopic" xs12 sm6>
                <v-text-field
                  v-model.trim="editedValue.to"
                  append-outer-icon="clear"
                  :rules="[validTopic]"
                  @click:append-outer="clearTopic"
                  label="Custom topic"
                  required
                ></v-text-field>
              </v-flex>
              <v-flex xs12 sm6>
                <v-switch
                  v-model="editedValue.retain"
                  label="Retain"
                  hint="The retain flag"
                  persistent-hint
                  required
                ></v-switch>
              </v-flex>
              <v-flex xs12 sm6>
                <v-select
                  v-model="editedValue.qos"
                  label="QoS"
                  hint="Quality of service"
                  :rules="[required]"
                  persistent-hint
                  required
                  :items="[0,1,2]"
                ></v-select>
              </v-flex>
              <v-flex xs12>
                <v-autocomplete
                  v-model="editedValue.map_id"
                  label="Map"
                  hint="Select the map function to use for the payload. If the map uses a function and Custom Topic is disable the function will be used also to map the topic. All other map values like retain qos and wildecard are ignored"
                  persistent-hint
                  required
                  item-text="name"
                  item-value="_id"
                  :items="maps"
                ></v-autocomplete>
              </v-flex>
            </v-layout>
          </v-form>
        </v-container>
      </v-card-text>

      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="blue darken-1" flat @click="$emit('close')">Cancel</v-btn>
        <v-btn color="blue darken-1" flat @click="$refs.form.validate() && $emit('save')">Save</v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
import { mapGetters } from "vuex";

export default {
  props: {
    value: Boolean,
    title: String,
    editedValue: Object
  },
  watch: {
    value(val) {
      this.$refs.form.resetValidation();
    }
  },
  computed: {
    ...mapGetters(["clients", "maps"])
  },
  methods: {
    clearTopic(){
      this.$set(this.editedValue, 'to', "");
    }
  },
  data() {
    return {
      valid: true,
      defaultOptions: { qos: -1, retain: 0, payload: 0, from: "", to: ""},
      optionsMode: ["GET", "SET"],
      required: v => !!v || v === 0 || "This field is required",
      validTopic: v => !/[#+\s]+/.test(v) || "Whitespace and wildecard (#, +) charaters are not allowed on topics"
    };
  }
};
</script>

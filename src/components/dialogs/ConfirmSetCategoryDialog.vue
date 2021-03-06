<template>
  <v-dialog :value="true" @input="closeDialog" :fullscreen="phoneLayout" width="40em">
    <v-card>
      <v-card-title
        class="headline grey lighten-4"
      >
        <v-icon class="mr-2">mdi-folder</v-icon>
        <span>Set category</span>
      </v-card-title>
      <v-card-text class="pb-0">
        <template v-if="category">
         Are you sure to move selected torrents to category {{ category }}?
        </template>
        <template v-else>
         Are you sure to reset category of selected torrents?
        </template>
        <ol class="torrents pt-6">
          <li v-for="(row, i) in torrents" :key="i">
            {{ row.name }}
          </li>
        </ol>

        <v-checkbox
          v-if="sameNamedTorrents.length > 0"
          v-model="moveSameNamed"
          prepend-icon="mdi-file-multiple"
          class="mt-0"
          :label="`Also move ${sameNamedTorrents.length} same named torrents`"
        />
      </v-card-text>
      <v-card-actions>
        <v-spacer />
        <v-btn text @click="closeDialog">Cancel</v-btn>
        <v-btn
          @click="submit"
          color="warning"
          :disabled="submitting"
          :loading="submitting"
        >
          Submit
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script lang="ts">
import _ from 'lodash';
import Vue from 'vue';
import { mapGetters } from 'vuex';
import api from '@/Api';
import { findSameNamedTorrents } from '@/utils';

export default Vue.extend({
  props: {
    value: Array,
    category: String,
  },
  data() {
    return {
      moveSameNamed: false,
      submitting: false,
      torrents: [],
      sameNamedTorrents: [],
    };
  },
  created() {
    this.torrents = this.value;
    this.sameNamedTorrents = findSameNamedTorrents(this.allTorrents, this.torrents);
  },
  computed: {
    ...mapGetters(['allTorrents']),
    phoneLayout() {
      return this.$vuetify.breakpoint.xsOnly;
    },
  },
  methods: {
    closeDialog() {
      this.$emit('input', []);
    },
    async submit() {
      if (this.submitting) {
        return;
      }

      this.submitting = true;

      let torrentsToMove;
      if (this.moveSameNamed) {
        torrentsToMove = this.torrents.concat(this.sameNamedTorrents);
      } else {
        torrentsToMove = this.torrents;
      }
      const hashes = torrentsToMove.map((t: any) => t.hash);
      await api.setTorrentsCategory(hashes, this.category);

      this.closeDialog();
    },
  },
});
</script>

<style lang="scss" scoped>
.torrents {
  overflow: auto;
}
.v-dialog--fullscreen {
  .v-card__text {
    padding-bottom: 52px;
  }

  .v-card__actions {
    position: absolute;
    bottom: 0;
    right: 0;
  }
}
</style>

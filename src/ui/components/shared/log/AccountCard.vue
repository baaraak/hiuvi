<template>
  <div class="card-container">
    <v-card elevation="4">
      <v-list>
        <v-list-item>
          <v-list-item-avatar>
            <v-img
              v-if="accountMetadata.logo"
              :src="accountMetadata.logo"
            />
            <v-card-text
              v-else
              v-text="accountMetadata.companyName"
            />
          </v-list-item-avatar>
          <v-list-item-content>
            <v-list-item-title v-text="accountMetadata.companyName" />
            <v-list-item-subtitle v-text="latestEventMessage" />
          </v-list-item-content>
          <v-list-item-icon
            v-if="displayExtrnalLink"
            class="status-indicator-wrapper"
          >
            <v-icon
              @click="onOpenExtrnalLink"
            >
              mdi-open-in-new
            </v-icon>
          </v-list-item-icon>
          <v-dialog width="500px">
            <template #activator="{ on, attrs }">
              <v-list-item-icon>
                <v-icon
                  v-if="displayTheShowLogsIcon"
                  v-bind="attrs"
                  v-on="on"
                >
                  mdi-information-outline
                </v-icon>
              </v-list-item-icon>
            </template>
            <v-card>
              <v-card-text class="log-details-text">
                {{ fullLog }}
              </v-card-text>
            </v-card>
          </v-dialog>

          <v-list-item-icon class="status-indicator-wrapper">
            <account-card-status-indicator
              :account-status="accountState.status"
              :error-message="errorMessage"
            />
          </v-list-item-icon>
        </v-list-item>
      </v-list>
    </v-card>
  </div>
</template>

<script lang="ts">
import { shell } from 'electron';
import { computed, defineComponent } from '@vue/composition-api';
import { AccountState } from '@/ui/components/app/accountsState';
import { AccountStatus } from '@/backend/eventEmitters/EventEmitter';
import { CsvConfig, JsonConfig, OutputVendorName } from '@/backend/configManager/configManager';
import AccountCardStatusIndicator from '@/ui/components/shared/log/AccountCardStatusIndicator.vue';
import ACCOUNT_METADATA, { AccountMetadata } from '@/accountsMetadata';
import store from '@/ui/store';

export default defineComponent({
  components: {
    AccountCardStatusIndicator
  },
  props: {
    accountState: {
      type: Object,
      required: true
    }
  },
  setup(props: { accountState: AccountState }) {
    const latestEventMessage = computed<string>(() => {
      const { events } = props.accountState;
      const latestEvent = events.length ? events[events.length - 1] : null;
      return latestEvent ? latestEvent.message : '';
    });

    const displayTheShowLogsIcon = computed<boolean>(() => [AccountStatus.DONE, AccountStatus.ERROR].includes(props.accountState.status));
    const fullLog = computed<string>(() => props.accountState.events.map((event) => event.message.trim()).join('\n'));
    const errorMessage = computed(() => props.accountState.events.find((event) => event.error)?.error?.message);

    const accountMetadata: AccountMetadata = ACCOUNT_METADATA.importers[props.accountState.id] || ACCOUNT_METADATA.exporters[props.accountState.id];

    const displayExtrnalLink = computed<boolean>(() => Object.values(OutputVendorName).includes(accountMetadata.companyKey as OutputVendorName)
    && props.accountState.status === AccountStatus.DONE);

    return {
      latestEventMessage, accountMetadata, displayTheShowLogsIcon, fullLog, errorMessage, displayExtrnalLink
    };
  },

  methods: {
    onOpenExtrnalLink() {
      const exporter = store.getters.Config.getExporter(this.accountState.id as OutputVendorName);
      switch (this.accountState.id) {
        case OutputVendorName.CSV:
          shell.openPath((exporter as CsvConfig).options.filePath);
          break;
        case OutputVendorName.JSON:
          shell.openPath((exporter as JsonConfig).options.filePath);
          break;
        case OutputVendorName.YNAB:
          break;
        case OutputVendorName.GOOGLE_SHEETS:
          break;

        default:
          break;
      }
    }
  }
});
</script>

<style scoped>
.card-container {
  margin: 20px;
  max-width: 600px;
}
.status-indicator-wrapper {
  margin-right: 20px;
}
.log-details-text {
  white-space: pre;
}
</style>

<template>
  <div class="transactions-container">



    <DxDataGrid
      :data-source="transactions"
      :show-borders="true"
      key-expr="id"
      class="transactions-table"
      :allow-column-reordering="true"
      :allow-column-resizing="true"
      :row-alternation-enabled="true"
    >
      <DxColumn data-field="id" caption="Transaction ID" width="95" alignment="center" />
      <DxColumn data-field="date" caption="Date" data-type="date" width="100" alignment="center" />
      <DxColumn
        data-field="amount"
        caption="Amount"
        width="85"
        alignment="right"
        cell-template="priceTemplate"
      />
      <DxColumn data-field="currency" caption="Currency" width="85" alignment="center" />
      <DxColumn data-field="type" caption="Type" width="110" alignment="center" />
      <DxColumn data-field="counterAccount" caption="Counter Account" width="150" alignment="center" />
      <DxColumn data-field="bankName" caption="Bank" width="120" alignment="center" />
      <DxColumn data-field="executedBy" caption="Executed by" width="120" alignment="center" />
      <DxColumn data-field="instructionId" caption="Instruction ID" width="100" alignment="center" />
      
      <DxColumn
        data-field="status"
        caption="Status"
        width="50"
        alignment="center"
        cell-template="statusTemplate"
      />
      <DxColumn
        data-field="log"
        caption="Log"
        width="250"
        alignment="left"
        cell-template="logTemplate"
      />

      <DxFilterRow :visible="true" />
      <DxHeaderFilter :visible="true" />
      <DxSearchPanel :visible="true" :highlight-case-sensitive="false" />
      <DxColumnChooser :enabled="true" />
      <DxSelection mode="multiple" />

      <DxPaging :page-size="100" />
      <DxPager :show-page-size-selector="true" :allowed-page-sizes="[5, 10, 20]" />

      <template #priceTemplate="{ data }">
        <span>{{ data.value }} CZK</span>
      </template>

      <template #statusTemplate="{ data }">
        <div :class="['status-indicator', data.value]"></div>
      </template>

      <template #logTemplate="{ data }">
        <div class="import-log">{{ data.value }}</div>
      </template>
    </DxDataGrid>
  </div>
</template>

<script>
import {
  DxDataGrid,
  DxColumn,
  DxPaging,
  DxPager,
  DxFilterRow,
  DxHeaderFilter,
  DxSearchPanel,
  DxColumnChooser,
  DxSelection
} from "devextreme-vue/data-grid";
import { DxButton } from "devextreme-vue/button";

export default {
  components: {
    DxDataGrid,
    DxColumn,
    DxPaging,
    DxPager,
    DxFilterRow,
    DxHeaderFilter,
    DxSearchPanel,
    DxColumnChooser,
    DxSelection,
    DxButton
  },
  data() {
    return {
      transactions: [],
      lastUpdated: "",
      refreshInterval: 15,
      refreshTimer: null
    };
  },
  methods: {
    async fetchTransactions() {
      try {
        const response = await fetch("http://localhost:1880/get-fio", {
          headers: { Accept: "application/xml" }
        });
        if (!response.ok) throw new Error("Error fetching transactions");
        const xmlText = await response.text();
        const parser = new DOMParser();
        const xmlDoc = parser.parseFromString(xmlText, "application/xml");

        const transactionNodes = xmlDoc.querySelectorAll("TransactionList > Transaction");
        this.transactions = Array.from(transactionNodes).map((tx) => ({
          id: tx.querySelector("column_22")?.textContent,
          date: tx.querySelector("column_0")?.textContent,
          amount: parseFloat(tx.querySelector("column_1")?.textContent || 0).toFixed(2),
          currency: tx.querySelector("column_14")?.textContent,
          counterAccount:
            tx.querySelector("column_2")?.textContent || tx.querySelector("column_5")?.textContent,
          bankName: tx.querySelector("column_12")?.textContent,
          type: tx.querySelector("column_8")?.textContent,
          executedBy: tx.querySelector("column_9")?.textContent,
          instructionId: tx.querySelector("column_17")?.textContent,
          status: "success",
          log: "Imported successfully"
        }));
        this.updateLastUpdatedTime();
      } catch (error) {
        console.error("Error fetching transactions:", error);
      }
    },
    updateLastUpdatedTime() {
      const now = new Date();
      this.lastUpdated = now.toLocaleTimeString("es-ES", {
        hour: "2-digit",
        minute: "2-digit",
        second: "2-digit"
      });
    },
    async startAutoRefresh() {
      const interval = this.refreshInterval;
      if (interval < 1) {
        alert("El intervalo debe ser al menos 1 minuto.");
        return;
      }
      try {
        const response = await fetch("http://localhost:1880/start-auto-refresh", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ minutes: interval })
        });
        const data = await response.json();
        console.log(data.message);
        alert(`Auto Refresh activado cada ${interval} minutos`);
      } catch (error) {
        console.error("Error al iniciar Auto Refresh:", error);
        alert("Error al configurar Auto Refresh");
      }
    }
  },
  mounted() {
    this.fetchTransactions();
  },
  beforeUnmount() {
    if (this.refreshTimer) {
      clearInterval(this.refreshTimer);
    }
  }
};
</script>

<style scoped>
.transactions-container {
  position: absolute;
  left: 0;
  width: 100vw;
  height: 100vh;
  padding: 0;
  margin: 0;
  overflow: hidden;
  background: white;
}

.transactions-table {
  width: 98%;
  height: 90%;
}

.status-indicator {
  width: 15px;
  height: 15px;
  border-radius: 50%;
  display: inline-block;
}

.status-indicator.success {
  background-color: #2ecc71;
}

.status-indicator.warning {
  background-color: #f1c40f;
}

.status-indicator.error {
  background-color: #e74c3c;
}

.import-log {
  white-space: pre-wrap;
  font-size: 12px;
  color: #444;
}

.controls {
  display: flex;
  gap: 10px;
  margin: 10px;
  align-items: flex-end;
  width: 97%;
  justify-content: space-between;
}

.controls p {
  font-size: 14px;
  color: #555;
}

.controls input {
  width: 50px;
  padding: 5px;
  text-align: center;
}

.minutes {
  margin-top: 1em;
  display: flex;
  align-items: center;
}

.button-rec {
  margin-left: 2em;
}
</style>

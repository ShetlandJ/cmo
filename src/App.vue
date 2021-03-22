<template>
    <div id="app">
        <div>
            <textarea v-model="pastedContent" />
        </div>
        <div style="margin-bottom: 10px">
            <button @click="parsePastedContent">Parse</button>
        </div>

        <input
            v-if="tables.length > 0"
            type="text"
            v-model="searchString"
            placeholder="Search line items by name"
        />

        <div v-for="dataTable in tables" :key="dataTable.id">
            <h3
                style="margin-bottom: 0"
                v-if="searchedItems(dataTable.rows).length > 0"
            >
                {{ dataTable.name }}
            </h3>
            <table
                class="output"
                v-if="searchedItems(dataTable.rows).length > 0"
            >
                <tr
                    v-for="(data, index) in searchedItems(dataTable.rows)"
                    :key="`${dataTable.id}-${index}`"
                    :id="`row-${dataTable.id}-${index}`"
                    @click="toggleRowComplete(index, dataTable.id)"
                    :class="{
                        complete: data.complete,
                    }"
                >
                    <td class="lineItemName">
                        {{ data.name }}
                    </td>

                    <td><strong>Total</strong>: {{ data.total }}</td>

                    <td
                        class="cell"
                        v-for="(outcome, index) in data.outcomes"
                        :key="Math.random() + index"
                    >
                        <strong>{{ outcome.name }}</strong
                        >: {{ outcome.count }}
                        <span v-if="outcome.name !== 'N/A'"
                            >({{ outcome.percentage }}%)
                        </span>
                    </td>

                    <template v-if="data.outcomes.length < 3">
                        <td
                            v-for="(num, index) in 3 - data.outcomes.length"
                            :key="index"
                        ></td>
                    </template>
                </tr>
            </table>

            <p v-else-if="output.length > 0 && searchedItems.length === 0">
                No line items found
            </p>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            pastedContent: "",
            tables: [],
            output: [],
            searchString: "",
        };
    },
    computed: {
        parsedContent() {
            return JSON.parse(this.pastedContent);
        },
        searchedItems() {
            return (inputData) => {
                if (!this.searchString) return inputData;

                return inputData.filter((data) =>
                    data.name
                        .toLowerCase()
                        .includes(this.searchString.toLowerCase())
                );
            };
        },
    },
    methods: {
        parsePastedContent() {
            this.parsedContent.data.metrics.forEach((metrics) => {
                this.output = [];
                const { lineItems, level } = metrics;

                lineItems.forEach((lineItem) => {
                    this.output.push(this.formatRowForOutput(lineItem));
                });

                this.tables.push({
                    id: this.createUUID(),
                    rows: this.output,
                    name: level.name,
                });
            });
        },
        formatRowForOutput(lineItem) {
            const payload = {
                name: "",
                outcomes: [],
            };

            payload.name = lineItem.lineItemName;

            let totalWithoutNa = 0;

            lineItem.templateLineOutcomes.forEach((outcome) => {
                if (outcome.scoreName !== "N/A") {
                    totalWithoutNa += outcome.applicable;
                }
            });

            payload.total = totalWithoutNa;

            lineItem.templateLineOutcomes.forEach((outcome) => {
                payload.outcomes.push({
                    count: outcome.applicable,
                    name: outcome.scoreName,
                    percentage: this.percentage(
                        outcome.applicable,
                        totalWithoutNa
                    ),
                });
            });

            return payload;
        },
        toggleRowComplete(index, tableId) {
            const id = `row-${tableId}-${index}`;

            let classList = document.getElementById(id).classList;

            if (Array.from(classList).includes("complete"))
                document.getElementById(id).classList = "";
            else document.getElementById(id).classList = "complete";
        },
        percentage(percent, total) {
            return ((percent / total) * 100).toFixed(2);
        },
        createUUID() {
            let dt = new Date().getTime();

            const uuid = "xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx".replace(
                /[xy]/g,
                function (c) {
                    const r = (dt + Math.random() * 16) % 16 | 0;
                    dt = Math.floor(dt / 16);
                    return (c == "x" ? r : (r & 0x3) | 0x8).toString(16);
                }
            );

            return uuid;
        },
    },
};
</script>

<style>
#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
}

textarea {
    width: 600px;
    height: 120px;
    border: 3px solid #cccccc;
    padding: 5px;
    font-family: Tahoma, sans-serif;
    background-position: bottom right;
    background-repeat: no-repeat;
}

input {
    width: 600px;
    height: 20px;
    border: 3px solid #cccccc;
    padding: 5px;
    font-family: Tahoma, sans-serif;
    background-position: bottom right;
    background-repeat: no-repeat;
    margin-bottom: 10px;
}

button {
    background-color: #00109f; /* Green */
    border: none;
    color: white;
    padding: 15px 32px;
    text-align: center;
    text-decoration: none;
    display: inline-block;
    font-size: 16px;
}

.complete {
    background-color: #4caf50;
}

.output {
    text-align: left;
    font-family: Arial, Helvetica, sans-serif;
    border-collapse: collapse;
    width: 95%;
}

.output td,
.output th {
    border: 1px solid #ddd;
    padding: 8px;
}

/* .output tr:nth-child(even) {
    background-color: #f2f2f2;
}

.output tr:hover {
    background-color: #ddd;
} */

.output th {
    padding-top: 12px;
    padding-bottom: 12px;
    text-align: left;
    background-color: #4caf50;
    color: white;
}
</style>

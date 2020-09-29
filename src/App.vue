<template>
    <div id="app">
        <h1>Invoice CSV Importer</h1>
        <h3>Please select a file to import</h3>
        <input type="file" @change="onFileChange">
        <br><br>
        <ul v-if="showFields">
            <li v-for="(field, label) in fields">
                <h4>{{ label }}</h4>
                <select>
                    <option :value="csvLabel" v-for="(csvField, csvLabel) in csvFields" :selected="csvLabel == field.map">{{ csvLabel }}</option>
                </select>
            </li>
            <li v-for="(field, label) in itemsFields">
                <h4>{{ label }}</h4>
                <select>
                    <option :value="csvLabel" v-for="(csvField, csvLabel) in csvFields" :selected="csvLabel == field.map">{{ csvLabel }}</option>
                </select>
            </li>
        </ul>
        <div class="controls" v-if="showFields">
            <button @click="uploadCSV">Upload</button>
        </div>
        <template v-if="showInvoice">
            <ul>
                <li>
                    Date: {{ data["Date"] }}
                </li>
                <li>
                    Invoice Number: {{ data["Invoice Number"] }}
                </li>
                <li>
                    PO Number: {{ data["PO Number"] }}
                </li>
            </ul>
            <ul class="items">
                <li class="items-labels" v-for="(item, label) in itemsFields">{{ label }}</li>
                <template v-for="(item, label) in items">
                    <li v-for="(data) in item">{{ data }}</li>
                </template>
            </ul>
            <ul class="totals">
                <li>
                    Subtotal: {{ data["Subtotal"] }}
                </li>
                <li>Calculated Subtotal: {{ data.calculatedSubTotal }}</li>
                <li>
                    Shipping: {{ data["Shipping"] }}
                </li>
                <li>
                    Tax: {{ data["Tax"] }}
                </li>
                <li>
                    Total: {{ data["Total"] }}
                </li>
                <li>Calculated Total: {{ data.calculatedTotal }}</li>
            </ul>
        </template>
        <!-- <div class="results" v-if="parsedCSV">
            <h3>Results:</h3>
            <pre>{{ JSON.stringify(parsedCSV, null, 2) }}</pre>
        </div> -->
    </div>
</div>
</template>

<script>
import Papa from 'papaparse';
export default {
    name: 'App',
    data() {
        return {
            parsedCSV: null,
            config: {
                complete: this.processCSV,
                error: this.processError
            },
            showFields: false,
            showInvoice: false,
            fields: {
                'Date': {
                    search: 'date',
                    map: null,
                    position: {
                        row: null,
                        column: null
                    }
                },
                'Invoice Number': {
                    search: 'invoice',
                    map: null,
                    position: {
                        row: null,
                        column: null
                    }
                },
                'PO Number': {
                    search: 'po',
                    map: null,
                    position: {
                        row: null,
                        column: null
                    }
                },
                'Subtotal': {
                    search: 'subtotal',
                    map: null,
                    position: {
                        row: null,
                        column: null
                    }
                },
                'Tax': {
                    search: 'tax',
                    map: null,
                    position: {
                        row: null,
                        column: null
                    }
                },
                'Shipping': {
                    search: 'ship',
                    map: null,
                    position: {
                        row: null,
                        column: null
                    }
                },
                'Total': {
                    search: 'total',
                    map: null,
                    position: {
                        row: null,
                        column: null
                    }
                }
            },
            itemsFields: {
                'Description': {
                    search: 'description',
                    map: null,
                    position: {
                        row: null,
                        column: null
                    }
                },
                'SKU': {
                    search: 'sku',
                    map: null,
                    position: {
                        row: null,
                        column: null
                    }
                },
                'Qty': {
                    search: 'qty',
                    map: null,
                    position: {
                        row: null,
                        column: null
                    }
                },
                'Price': {
                    search: 'price',
                    map: null,
                    position: {
                        row: null,
                        column: null
                    }
                },
                'Rebate eligible': {
                    search: 'rebate',
                    map: null,
                    position: {
                        row: null,
                        column: null
                    }
                }
            },
            csvFields: {},
            data: {},
            items: []
        }
    },
    methods: {
        onFileChange(e) {
            var files = e.target.files || e.dataTransfer.files;
            if (!files.length) return;

            Papa.parse(files[0], this.config);
        },
        processCSV(results, file) {
            this.parsedCSV = results.data
            let rowIndex = 0;
            while (rowIndex < this.parsedCSV.length) {
                let columnIndex = 0;
                let row = this.parsedCSV[rowIndex]
                while (columnIndex < row.length) {
                    // console.log(row[columnIndex]);
                    for (let key in this.fields) {
                        var isHeaderRow = false
                        if (key == "Total" && row[columnIndex].toString().toLowerCase() == "total") {
                            isHeaderRow = true
                            this.fields[key] = {
                                map: row[columnIndex].toString(),
                                position: {
                                    row: rowIndex,
                                    column: columnIndex
                                }
                            }
                        } else {
                            if (row[columnIndex].toString().toLowerCase().indexOf(this.fields[key].search) > -1) {
                                isHeaderRow = true
                                this.fields[key] = {
                                    map: row[columnIndex].toString(),
                                    position: {
                                        row: rowIndex,
                                        column: columnIndex
                                    }
                                }
                            }
                        }
                        if (isHeaderRow) {
                            this.csvFields[row[columnIndex].toString()] = {
                                row: rowIndex,
                                column: columnIndex
                            }
                        }
                    }
                    for (let key in this.itemsFields) {
                        var isHeaderRow = false
                        if (row[columnIndex].toString().toLowerCase().indexOf(this.itemsFields[key].search) > -1) {
                            isHeaderRow = true
                            this.itemsFields[key] = {
                                map: row[columnIndex].toString(),
                                position: {
                                    row: rowIndex,
                                    column: columnIndex
                                }
                            }
                        }
                        if (isHeaderRow) {
                            this.csvFields[row[columnIndex].toString()] = {
                                row: rowIndex,
                                column: columnIndex
                            }
                        }
                    }
                    columnIndex++;
                }
                rowIndex++;
            }
            this.showFields = true
        },
        uploadCSV() {
            var startingRow = this.itemsFields["Description"].position.row + 1;
            var endingRow = null;
            for (let key in this.fields) {
                if (key == "Date") {
                    this.data[key] = this.parsedCSV[this.fields[key].position.row + 1][this.fields[key].position.column]
                } else {
                    this.data[key] = (isNaN(parseFloat(this.parsedCSV[this.fields[key].position.row + 1][this.fields[key].position.column])))
                        ? this.parsedCSV[this.fields[key].position.row + 1][this.fields[key].position.column]
                        : parseFloat(this.parsedCSV[this.fields[key].position.row + 1][this.fields[key].position.column])
                }
                if (key == 'Subtotal') {
                    endingRow = this.fields[key].position.row
                }
            }
            this.processItems(startingRow, endingRow)
        },
        processItems(startingRow, endingRow) {
            let items = this.parsedCSV.slice(startingRow, endingRow)
            let processedItems = []
            this.data.calculatedSubTotal = 0;
            items.forEach((item, i) => {
                let processedItem = {}
                let quantity = 0;
                let price = 0;
                for (let key in this.itemsFields) {
                    if (key == 'Qty') {
                        quantity = parseInt(item[this.itemsFields[key].position.column])
                    }
                    if (key == 'Price') {
                        price = parseFloat(item[this.itemsFields[key].position.column])
                    }
                    processedItem[key] = item[this.itemsFields[key].position.column]
                }
                this.items.push(processedItem)
                let subTotal = (quantity * price)
                this.data.calculatedSubTotal += subTotal
            });
            this.data.calculatedTotal = this.data.calculatedSubTotal + parseFloat(this.data["Tax"]) + parseFloat(this.data["Shipping"])
            this.showFields = false
            this.showInvoice = true
        },
        processError(error, file) {

        }
    }
}
</script>

<style lang="scss">
#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    width: 60%;
    max-width: 800px;
    margin: 60px auto;
    color: #2c3e50;
    margin-top: 60px;
}

ul {
    list-style: none;
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-gap: 2em;

    &.items {
        grid-template-columns: repeat(5, 1fr);
        grid-column-gap: 2em;
        grid-row-gap: 1em;
    }
}
ul.totals {
    display: block;
    text-align: right;

    li {
        padding-bottom: 1em;
    }
}
</style>

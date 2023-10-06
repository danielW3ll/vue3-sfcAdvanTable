<script setup>
// @ts-check
import { inject, onBeforeMount, onMounted, reactive, ref, computed, defineAsyncComponent } from 'vue'

const parentProps = defineProps({
    tableData: {
        /**
            tableData = {
                tblHeadDisplay: ['ID', 'Der Name', 'Die Email'],
                tblHeadDisplayShow: [false, true, true],
                tblData:[
                    { id: 1, name: 'John Doe', email: 'john@doe'},
                    { id: 2, name: 'Jane Doe', email: 'jane@doe'},
                    { id: 3, name: 'John Smith', email: 'john@smith'},
                    { id: 4, name: 'Jane Smith', email: 'jane@smith'},
                ]
            }
         */
        type: Object,
        required: true
    }
})

const ReloadTable = defineEmits(['updateTable'])

/**
 *  Table Functions and Variables
 */
const tableHeadDisplay = ref([])
const tableHead = ref([])
const tableBody = ref([])
const tableSearchQuery = ref('')
const tableSort = ref({
    column: '',
    order: 'asc',
    searchColumn: ''
})
const setTableSearchColumn = (s) => {
    tableSearchQuery.value = ''
    tableSort.value.searchColumn = tableHead.value[s]
}
const sortTable = (s) => {
    // console.log('sortTable')
    s = tableHead.value[s]
    if (s === tableSort.value.column) {
        tableSort.value.order = tableSort.value.order === 'asc' ? 'desc' : 'asc'
    } else {
        tableSort.value.order = 'asc'
    }
    tableSort.value.column = s
}
const sortTableBody = computed(() => {
    // console.log('sortTableBody')
    // Search Query
    let sortedList = tableBody.value.filter((item) => {
        let column = tableSort.value.searchColumn
        // check item[column] is only string
        if (typeof item[column] !== 'string') {
            // @ts-ignore
            item[column] = item[column].toString()
        }
        // @ts-ignore
        return item[column].toLowerCase().indexOf(tableSearchQuery.value.toLowerCase()) > -1
    })
    // Sort
    if (tableSort.value.column !== '') {
        sortedList = sortedList.sort((a, b) => {
            if (a[tableSort.value.column] < b[tableSort.value.column]) {
                return tableSort.value.order === 'asc' ? -1 : 1
            }
            if (a[tableSort.value.column] > b[tableSort.value.column]) {
                return tableSort.value.order === 'asc' ? 1 : -1
            }
            return 0
        })
    }
    return sortedList
})

/**
 *  Validate tableData
 *  Check tableHeadDisplay and tableHeadDisplayShow length with tableData 
 */
const validateTableData = (tblObj) => {
    console.log('validateTableData' + JSON.stringify(tblObj, null, 2))
    if (tblObj.tblHeadDisplay.length !== tblObj.tblHeadDisplayShow.length) {
        console.log('tableHeadDisplay and tableHeadDisplayShow length not equal')
    }
    if (tblObj.tblData.length === 0) {
        console.log('tableData is empty')
    } else if (tblObj.tblHeadDisplay.length !== Object.keys(tblObj.tblData[0]).length) {
        console.log('tableHeadDisplay and tblData length not equal')
    }
    return tblObj
}

/**
 * set ref variables for table from tableData
 */

const setTable = (tbl) => {
    const tblObj = validateTableData(tbl)
    console.log('setTable' + JSON.stringify(tblObj, null, 2))

    // Set only tablehead when tableheadDisplayShow index is true
    tableHeadDisplay.value = tblObj.tblHeadDisplay.filter((item, index) => {
        return tblObj.tblHeadDisplayShow[index]
    })
    tableHead.value = Object.keys(tblObj.tblData[0]).filter((item, index) => {
        return tblObj.tblHeadDisplayShow[index]
    })
    tableBody.value = tblObj.tblData
    tableSort.value.column = tableHead.value[0]
    tableSort.value.searchColumn = tableHead.value[0]

}

// const tableDataBuild = async () => {
//     let tableBodyData = []
//     for (const node of tree.value) {
//         if (node['object_type'].includes(Cfg.typeDomain)) {
//             tableBodyData.push({
//                 name: node['object_name'],
//                 systemCount: node['nodes'].length.toString(),
//                 shortid: node['shortid'],
//             })
//         }
//     }
//     if (tableBodyData.length === 0) {
//         tableBodyData.push({
//             name: 'Keine Domänen vorhanden',
//             systemCount: '',
//             shortid: '',
//         })
//         // tableHead.value = ['', '']
//     }

//     // @ts-ignore
//     tableBody.value = tableBodyData
//     tableHead.value = [
//         // @ts-ignore
//         Object.keys(tableBodyData[0])[0],
//         // @ts-ignore
//         Object.keys(tableBodyData[0])[1]
//     ]

//     // @ts-ignore
//     tableHeadDisplay.value = ['Name', 'Anzahl Systeme']


//     tableSort.value.column = tableHead.value[0]
//     tableSort.value.searchColumn = tableHead.value[0]
//     // console.log('tableDataBuild')
// }

const tableDemoData = {
    tblHeadDisplay: ['ID', 'Der Name', 'Die Email'],
    tblHeadDisplayShow: [true, false, true],
    tblData: [
        { id: 1, name: 'John2 Doe', email: 'john@doe' },
        { id: 2, name: 'Jane Doe', email: 'jane@doe' },
        { id: 3, name: 'John Smith', email: 'john@smith' },
        { id: 4, name: 'Jane Smith', email: 'jane@smith' },
    ]
}
onMounted(async () => {
    console.log('onMounted')
    // validateTableData(tableDemoData)
    setTable(parentProps.tableData)
})
</script>
<template>
    <div>
        <button @click="$emit('updateTable')" class="btn btn-sm btn-outline-secondary" type="button">
            <i class="bi bi-arrow-repeat"></i>
        </button>
        <div class="table-responsive">
            <table class="table table-striped table-sm table-bordered">
                <thead>
                    <tr>
                        <th v-for="(item, index) in tableHead" :key="index">
                            <div class="d-flex align-items-center">
                                <div @click="sortTable(index)">
                                    <span> {{ tableHeadDisplay[index] }}</span>
                                    <span v-if="tableSort.column === tableHead[index]">
                                        <span v-if="tableSort.order === 'asc'">
                                            <i class="bi bi-sort-up"></i>
                                        </span>
                                        <span v-else>
                                            <i class="bi bi-sort-down"></i>
                                        </span>
                                    </span>
                                </div>
                                <div class="flex-grow-1"></div>
                                <input v-if="tableSort.searchColumn === tableHead[index]" v-model="tableSearchQuery"
                                    type="text" class="border-0 border-bottom" placeholder="Suche">
                                <button @click="setTableSearchColumn(index)"
                                    class="btn btn-sm border-0 btn-outline-secondary" type="button">
                                    <i class="bi bi-search"></i>
                                </button>
                            </div>
                        </th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(item, index) in sortTableBody" :key="index">
                        <td v-for="i in tableHead" :key="i">
                            <span>
                                {{ item[i] }}
                            </span>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>
<style scoped></style>
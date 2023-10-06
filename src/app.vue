<script setup>
import { inject, onBeforeMount, onMounted, reactive, ref, computed, defineAsyncComponent } from 'vue'

const TableComponent = defineAsyncComponent(() => import('./table.vue'))

const tableData = {
    tblHeadDisplay: ['ID', 'Der Name', 'Die Email'],
    tblHeadDisplayShow: [false, true, true],
    tblData: [
        { id: 1, name: 'John2 Doe', email: 'john@doe' },
        { id: 2, name: 'Jane Doe', email: 'jane@doe' },
        { id: 3, name: 'John Smith', email: 'john@smith' },
        { id: 4, name: 'Jane Smith', email: 'jane@smith' },
    ]
}
const tableCompRenderKey = ref(0)
const updateTableData = () => {
    console.log('updateTableData')
    tableData.tblData[0].name = 'John Doe'
    tableCompRenderKey.value++
}

</script>

<template>
    <div>
        <div class="container py-4 px-3 mx-auto">
            <h1>Hello</h1>
            <p>Welcome to vue3-sfcAdvanTable</p>
            <TableComponent :tableData="tableData" :key="tableCompRenderKey" @updateTable="updateTableData" />
        </div>
    </div>
</template>

<style scoped></style>
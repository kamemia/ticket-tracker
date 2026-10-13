<template>
    <v-container>
        <div>
            <div>Mini Ticket Tracker</div>

        </div>
        <v-row class="mt-5">
            <v-col v-for="(count, status) in stats" :key="status" cols="12" sm="4">
                <TicketCard :label="labels[status]" :count="count" />
            </v-col>
        </v-row>
        <div class="d-flex align-center justify-space-between mt-5">
            <v-select  v-model="statusFilter" :items="filterItems" label="Status" density="compact" variant="outlined"
                clearable hide-details style="max-width: 200px" />
            <v-btn color="primary" @click="openDialog(null)">New ticket</v-btn>
        </div>
        <v-data-table-server :headers="headers" :items="tickets" :items-per-page="itemsPerPage" :items-length="count"
            :loading="loading" @update:options="fetchTickets">
            <template #item.status="{ item }">
                <v-menu>
                    <template #activator="{ props }">
                        <v-chip v-bind="props" :color="statusColors[item.status]" size="small"
                            append-icon="mdi-chevron-down" style="cursor: pointer">
                            {{ labels[item.status] }}
                        </v-chip>
                    </template>
                    <v-list density="compact">
                        <v-list-item v-for="(label, value) in labels" :key="value" :active="item.status === value"
                            @click="updateStatus(item, value)">
                            <v-list-item-title>{{ label }}</v-list-item-title>
                        </v-list-item>
                    </v-list>
                </v-menu>
            </template>

            <template #item.priority="{ item }">
                <v-chip :color="priorityColors[item.priority]" size="small" class="text-capitalize">
                    {{ item.priority }}
                </v-chip>
            </template>

            <template #item.actions="{ item }">
                <v-btn icon="mdi-pencil" size="small" variant="text" @click="openDialog(item)" />
            </template>
        </v-data-table-server>

        <TicketDialog v-model="dialog" :ticket="selected" @saved="refresh" />
    </v-container>
</template>
<script setup>
import { ref, onMounted, watch } from 'vue'
import axiosInst from '../services/api'
import TicketCard from './TicketCard.vue'
import TicketDialog from './TicketDialog.vue'

const dialog = ref(false)
const selected = ref(null)
const itemsPerPage = ref(25)
const statusFilter = ref(null)
const stats = ref({})
const tickets = ref([])
const count = ref(0)
const loading = ref(false)
const options = ref({ page: 1, itemsPerPage: itemsPerPage })

const labels = {
    open: 'Open',
    in_progress: 'In Progress',
    closed: 'Closed',
}

const statusColors = {
    open: 'blue',
    in_progress: 'orange',
    closed: 'green'
}
const priorityColors = {
    low: 'grey',
    medium: 'amber',
    high: 'red'
}

const headers = [
    { title: 'Title', key: 'title', align: 'start' },
    { title: 'Description', key: 'description' },
    { title: 'Status', key: 'status' },
    { title: 'Priority', key: 'priority' },
    { title: '', key: 'actions', sortable: false },
]

const filterItems = Object.entries(labels).map(([value, title]) => ({ title, value }))
function openDialog(ticket) {
    selected.value = ticket
    dialog.value = true
}

function refresh() {
    fetchStats()
    fetchTickets(options.value)
}


async function fetchStats() {
    try {
        const { data } = await axiosInst.get('/tickets/stats')
        stats.value = data
    } catch (err) {
        console.log(err)
    }
}


async function updateStatus(ticket, status) {
    if (ticket.status === status) return
    try {
        await axiosInst.patch(`/tickets/${ticket.id}/`, { status })
        refresh()
    } catch (err) {
        console.log(err)
    }
}

async function fetchTickets({ page = 1, itemsPerPage }) {
    loading.value = true
    try {
        const { data } = await axiosInst.get('/tickets/', {
            params: { page, page_size: itemsPerPage, status: statusFilter.value || undefined },
        })
        tickets.value = data.results
        count.value = data.count
    } catch (err) {
        console.log(err)
    } finally {
        loading.value = false
    }
}

watch(statusFilter, () => fetchTickets({ page: 1, itemsPerPage: options.value.itemsPerPage }))
onMounted(fetchStats)
</script>
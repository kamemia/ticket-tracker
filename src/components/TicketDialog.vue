<template>
    <v-dialog :model-value="modelValue" max-width="500" @update:model-value="emit('update:modelValue', $event)">
        <v-card :title="isEdit ? 'Edit ticket' : 'New ticket'">
            <v-card-text>
                <v-text-field
                variant="outlined"
                density="compact"
                    v-model="form.title"
                    label="Title"
                    :rules="[v => !!v || 'Title is required']"
                />
                <v-textarea variant="outlined" v-model="form.description" label="Description" rows="3" />
                <v-select variant="outlined" density="compact" v-model="form.status" label="Status" :items="statuses" />
                <v-select variant="outlined" density="compact" v-model="form.priority" label="Priority" :items="priorities" />
            </v-card-text>
            <v-card-actions>
                <v-spacer />
                <v-btn @click="close">Cancel</v-btn>
                <v-btn color="primary" :loading="saving" :disabled="!form.title" @click="save">
                    {{ isEdit ? 'Save' : 'Create' }}
                </v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import axiosInst from '../services/api'

const props = defineProps({
    modelValue: { type: Boolean, default: false },
    ticket: { type: Object, default: null },
})
const emit = defineEmits(['update:modelValue', 'saved'])

const statuses = [
    { title: 'Open', value: 'open' },
    { title: 'In Progress', value: 'in_progress' },
    { title: 'Closed', value: 'closed' },
]
const priorities = [
    { title: 'Low', value: 'low' },
    { title: 'Medium', value: 'medium' },
    { title: 'High', value: 'high' },
]

const defaults = { title: '', description: '', status: 'open', priority: 'low' }
const form = ref({ ...defaults })
const saving = ref(false)
const isEdit = computed(() => !!props.ticket)

watch(() => props.modelValue, (open) => {
    if (open) {
        form.value = props.ticket
            ? { ...defaults, ...props.ticket }
            : { ...defaults }
    }
})

function close() {
    emit('update:modelValue', false)
}

async function save() {
    saving.value = true
    try {
        if (isEdit.value) {
            await axiosInst.patch(`/tickets/${props.ticket.id}/`, form.value)
        } else {
            await axiosInst.post('/tickets/', form.value)
        }
        emit('saved')
        close()
    } catch (err) {
        console.log(err)
    } finally {
        saving.value = false
    }
}
</script>
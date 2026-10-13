<template>
    <v-card>
        <v-card-text class="text-center">
            <div class="text-h4">{{ count }}</div>
            <div class="text-caption text-medium-emphasis">{{ label }}</div>
        </v-card-text>
    </v-card>
</template>

<script setup>
defineProps({
    label: { type: String, required: true },
    count: { type: Number, required: true },
})
</script>
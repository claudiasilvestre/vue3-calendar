<script setup>
    defineProps({
        show: Boolean
    });
</script>

<template>
    <div v-if="show" class="modal-mask">
        <div class="modal-container">
            <div class="modal-close">
                <span @click="$emit('close')" class="material-symbols-outlined">
                    close
                </span>
            </div>
            <slot></slot>
        </div>
    </div>
</template>

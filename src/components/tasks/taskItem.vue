<template>
    <div class="task-item">
        <div class="task">
            <slot name="task" />
        </div>
        <div class="remove-item" @click="handleRemove">-</div>
    </div>
</template>

<script setup lang="ts">
const emit = defineEmits(['removeItem'])
const handleRemove = () => {
    emit('removeItem')
}
</script>

<style lang="less" scoped>
.task-item {
    height: auto;
    border: 1px solid var(--common-background-color);
    background-color: var(--kanban-item-background-color);
    position: relative;
    cursor: move;
    border-radius: 10px;

    .task {
        padding: 10px;
    }
    .remove-item {
        position: absolute;
        bottom: 12px;
        right: 10px;
        font-size: 2rem;
        font-weight: 300;
        line-height: 0.2em;
        transition: all 0.05s linear;
        cursor: pointer;

        &:hover {
            font-size: 3rem;
            line-height: 0.1em;
        }
    }
}
</style>

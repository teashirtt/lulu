<template>
    <div class="side-bar">
        <div class="bar">
            <svg-icon name="folder" class="icon" @click="handlePick(markRaw(folderPanel))" />
            <svg-icon name="kanban" class="icon" @click="handlePick(markRaw(kanbanPanel))" />
            <svg-icon name="settings" class="icon" @click="handlePickSettings" />
        </div>
        <div class="panel" v-show="expandComponent">
            <keep-alive>
                <component :is="expandComponent" />
            </keep-alive>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref, markRaw } from 'vue'
import type { Component } from 'vue'
import folderPanel from '../panels/folder/folderPanel.vue'
import kanbanPanel from '../panels/kanban/kanbanPanel.vue'
import { useRouter } from 'vue-router'

const expandComponent = ref<Component | null>(null)

const handlePick = (component: Component) => {
    expandComponent.value = expandComponent.value === component ? null : component
}

const router = useRouter()
const handlePickSettings = () => {
    router.push({ name: 'settings' })
}
</script>

<style lang="less" scoped>
.side-bar {
    display: flex;
    background-color: var(--bar-background-color);
    height: 100%;

    .bar {
        display: flex;
        flex-direction: column;
        width: 50px;
        gap: 30px;
        border-right: 1px solid var(--common-border-color);
        margin-top: 10px;
    }

    .panel {
        width: 320px;
        height: 100%;
        overflow: auto;
    }
}

.icon {
    margin: 0 auto;
    width: 30px;
    height: 30px;
    cursor: pointer;
}

.icon:nth-child(3) {
    margin: auto auto 15px;
}
</style>

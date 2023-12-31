<template>
    <div class="lulu-md-editor">
        <textarea
            v-if="mode === 'md'"
            @blur="handleBlur"
            @focus="handleFocus"
            @input="handleInput"
            v-model="content"
            ref="textareaRef"
            @keydown.enter.prevent="handleEnter($event)"
            @keydown.tab.prevent="handleTab($event)" />
        <div class="markdown" v-else v-html="html" @dblclick="handleFocus" ref="htmlRef" />
    </div>
</template>

<script setup lang="ts">
import { ref, nextTick, PropType, onMounted, watch } from 'vue'
import { render } from '../../../utils/mdRender.ts'
import { getLineScope, getLineContext, getTabContext } from '../../../utils/mdContext'
import type { LuluInfo } from '../../../types/LuluInfo'
import '../../../styles/markdown.less'
import { LuluStore } from '../../../stores/LuluStore'
import { openUrl } from '../../../utils/openUrl'

const props = defineProps({
    luluInfo: {
        type: Object as PropType<LuluInfo>,
        required: true
    }
})

const content = ref<string>(props.luluInfo.content)
const html = ref<string>(render(content.value).html)
const mode = ref<'md' | 'html'>(props.luluInfo.content === '' ? 'md' : 'html')
const luluStore = LuluStore()
const handleBlur = () => {
    setTimeout(() => {
        luluStore.changeFocus(false, props.luluInfo.id)
    }, 200)
    if (content.value.trim()) {
        html.value = render(content.value).html
        mode.value = 'html'
    }
}

const textareaRef = ref<HTMLTextAreaElement | null>(null)
const handleFocus = () => {
    luluStore.changeFocus(true, props.luluInfo.id)
    mode.value = 'md'
    nextTick(() => {
        textareaRef.value!.focus()
        handleInput()
    })
}

let lastLine: string | null = null
const handleEnter = (event: KeyboardEvent) => {
    const target = event.target as HTMLInputElement
    const pos = target.selectionStart!
    let { l, r } = getLineScope(content.value, pos)
    const curLine = content.value.slice(l, r)
    const context = getLineContext(curLine, pos - l)
    content.value = content.value.slice(0, l) + context + content.value.slice(r)
    const bias = context.split('\n')[1].length
    lastLine = content.value.slice(l, pos)
    nextTick(() => {
        target.selectionEnd = pos + bias + 1
        handleInput()
    })
}

const handleTab = (event: KeyboardEvent) => {
    const target = event.target as HTMLInputElement
    const pos = target.selectionStart!
    let { l, r } = getLineScope(content.value, pos)
    const curLine = content.value.slice(l, r)
    const context = lastLine === null ? null : getTabContext(lastLine, curLine)
    if (context === null || context === curLine) {
        content.value = content.value.slice(0, pos) + ' '.repeat(4) + content.value.slice(pos)
        nextTick(() => {
            target.selectionEnd = pos + 4
        })
    } else {
        content.value = content.value.slice(0, l) + context + content.value.slice(r)
        nextTick(() => {
            target.selectionEnd = pos + 2
        })
    }
}

const handleInput = () => {
    textareaRef.value!.style.height = 'auto'
    textareaRef.value!.style.height = textareaRef.value!.scrollHeight + 'px'
}

const handleClickLink = (anchors: NodeListOf<HTMLAnchorElement>) => {
    for (let anchor of anchors) {
        anchor.addEventListener('click', event => {
            event.preventDefault()
            openUrl(anchor.getAttribute('href'))
        })
    }
}

const htmlRef = ref<HTMLDivElement | null>(null)
onMounted(() => {
    if (mode.value === 'md') {
        handleInput()
    }

    watch(
        mode,
        newV => {
            if (newV === 'html') {
                nextTick(() => {
                    const anchors = htmlRef.value!.querySelectorAll('a')
                    handleClickLink(anchors)
                })
            }
        },
        {
            immediate: true
        }
    )
})

const getContent = () => {
    return content.value
}

defineExpose({
    getContent
})
</script>

<style lang="less" scoped>
.lulu-md-editor {
    margin-left: 37px;
    display: flex;

    textarea {
        padding: 15px 28px 0 28px;
        word-wrap: break-word;
        width: 100%;
        font-size: 20px;
        background-color: var(--block-background-color);
        &:focus {
            outline: none;
            box-shadow: none;
        }
    }

    div {
        cursor: pointer;
        width: 100%;
        word-wrap: break-word;
        &:hover {
            box-shadow: 2px 2px 3px 3px var(--box-shadow-color);
        }
    }
}
</style>

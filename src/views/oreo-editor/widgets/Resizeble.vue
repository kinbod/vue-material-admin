<template>
    <template v-if="props.data.visible">
        <div v-if="isDiv" :style="styles" :class="classNames" :uid="props.data.id"></div>
        <DragResizeBle
            v-else
            :active="_active"
            :w="_width"
            :h="_height"
            :x="_left"
            :y="_top"
            @activated="onActivated"
            @deactivated="onDeactivated"
            @resizestop="resize"
            @dragstop="resize"
            @dragging="onDragging"
            @refLineParams="getRefLineParams"
            :style="styles"
            :class="classNames"
            :uid="props.data.id"
            v-on:contextmenu.prevent="onMouser"
        >
            <div v-if="props.data.label" class="text">{{ props.data.label }}</div>
            <slot></slot>
        </DragResizeBle>
    </template>
</template>
<script setup lang="ts">
import { computed } from 'vue';
import type { VirtualDom } from '../hooks/useOreoApp';
// @ts-ignore
import DragResizeBle from '@/components/DragResizeble/index.vue';

const props = withDefaults(
    defineProps<{
        data: VirtualDom;
        active: boolean;
        width: number;
        height: number;
        top: number;
        left: number;
    }>(),
    {}
);
const emit = defineEmits([
    'update:active',
    'update:width',
    'update:height',
    'update:top',
    'update:left',
    'change',
    'snapLine',
    'mouser',
    'dragging',
    'activated',
]);

const _active = computed(() => props.active);
const _width = computed(() => props.width);
const _height = computed(() => props.height);
const _top = computed(() => props.top);
const _left = computed(() => props.left);

const isDiv = computed(() => {
    return !!props.data.groupId && !!props.data.type;
});

const resize = (e: any) => {
    emit('update:width', e.width);
    emit('update:height', e.height);
    emit('update:top', e.top);
    emit('update:left', e.left);
    emit('change', e);
};

const getRefLineParams = (params: any) => {
    const { vLine, hLine } = params;
    let id = 0;
    const vs = vLine.map((item: any) => {
        item['id'] = ++id;
        return item;
    });
    const hl = hLine.map((item: any) => {
        item['id'] = ++id;
        return item;
    });
    emit('snapLine', [vs || [], hl || []]);
};
const onActivated = () => {
    emit('update:active', true);
    emit('activated', props.data);
};
const onDeactivated = () => {
    emit('update:active', false);
};
const onDragging = (left_: number, top_: number, f: object) => {
    // console.log(f, 'f');
    emit('dragging', f, props.data);
};

const styles = computed(() => {
    let background = 'none';
    if (props.data.styles.fill) {
        background = props.data.styles.background;
    }
    let border = 'none';
    if (props.data.styles.border) {
        border =
            props.data.styles.borderWidth +
            'px ' +
            props.data.styles.borderStyle +
            ' ' +
            props.data.styles.borderColor;
    }
    let boxShadow = 'none';
    if (props.data.styles.shadow) {
        boxShadow = `${props.data.styles.shadowX} ${props.data.styles.shadowY}px ${props.data.styles.shadowBlur}px ${props.data.styles.shadowSpread}px ${props.data.styles.shadowColor}`;
    }
    let zIndex: any = '';
    if (props.data.active) {
        zIndex = 1;
    }

    // 字体样式
    let fontStyle: any = {};
    if (props.data.fontStyle) {
        fontStyle = {
            ...props.data.fontStyle,
        };
        fontStyle.fontSize = props.data.fontStyle.fontSize + 'px';
        if (props.data.fontStyle.shadow) {
            fontStyle.textShadow = `${props.data.fontStyle.shadowX}px ${props.data.fontStyle.shadowY}px ${props.data.fontStyle.shadowBlur}px ${props.data.fontStyle.shadowColor}`;
        }
        // console.log(fontStyle, 'fontStyle');
    }
    const div: any = {};
    if (isDiv.value) {
        div.width = props.data.styles.width + 'px';
        div.height = props.data.styles.height + 'px';
        div.transform = `translate(${props.data.styles.left}px, ${props.data.styles.top}px)`;
    }

    return {
        borderRadius: `${props.data.styles.radius}px`,
        background,
        border,
        opacity: props.data.styles.opacity,
        boxShadow,
        zIndex,
        ...div,
        ...fontStyle,
    };
});
const classNames = computed(() => {
    return [
        'vdr',
        props.data.selected ? 'selected' : '',
        props.data.virtualGroup ? 'virtualGroup' : '',
        props.data.type === 0 ? 'group' : '',
    ];
});

const onMouser = (e: PointerEvent) => {
    emit('mouser', e);
};
</script>

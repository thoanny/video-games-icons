<script setup>
import { ref } from 'vue';
import { useToast } from 'primevue/usetoast';

import '@/fonts/genshin/vgi-genshin.css';
import '@/fonts/enshrouded/vgi-enshrouded.css';
import '@/fonts/palia/vgi-palia.css';
import GenshinIcons from '@/fonts/genshin/info.json';
import EnshroudedIcons from '@/fonts/enshrouded/info.json';
import PaliaIcons from '@/fonts/palia/info.json';

const data = [
    { id: 'enshrouded', name: 'Enshrouded', icons: EnshroudedIcons },
    { id: 'genshin', name: 'Genshin Impact', icons: GenshinIcons },
    { id: 'palia', name: 'Palia', icons: PaliaIcons },
];

const visible = ref(false);
const currentIcon = ref({ category: null, key: null, icon: null, content: null });
const fontSize = ref(32);

const toast = useToast();

const getSvgContent = async (id, file) => {
    return await fetch(`/svg/${id}/${file}.svg`).then((data) => {
        return data.text();
    });
};

const openIconDetails = async (category, uid, icon) => {
    visible.value = true;
    currentIcon.value = {
        category: category,
        key: uid,
        icon: icon,
        content: await getSvgContent(category, uid),
    };
};

const copyToClipboard = (content, type) => {
    navigator.clipboard.writeText(content);
    toast.add({
        severity: 'success',
        summary: currentIcon.value.icon?.className,
        detail: `${type} copied!`,
        life: 3000,
    });
};

const items = ref([
    {
        label: 'Games',
        icon: 'pi pi-folder',
        items: data.map((d) => {
            return {
                label: d.name,
                href: `#${d.id}`,
            };
        }),
    },
    {
        label: 'Documentation',
        href: '#documentation',
    },
]);
</script>

<template>
    <ScrollTop />
    <Toast />

    <Dialog
        v-model:visible="visible"
        modal
        :header="currentIcon.icon?.className"
        :dismissableMask="true"
        :draggable="false"
        :style="{ width: '25rem' }"
    >
        <div
            class="bg-surface-50 aspect-video w-full h-full flex items-center justify-center rounded-lg"
        >
            <span :class="[currentIcon.icon?.className, '!text-8xl']"></span>
        </div>
        <div class="grid grid-cols-2 gap-2 mt-5">
            <Button
                label="HTML code"
                icon="pi pi-clipboard"
                outlined
                @click="
                    copyToClipboard(
                        `<span class=&quot;${currentIcon.icon?.className}&quot;></span>`,
                        'HTML code',
                    )
                "
            />
            <Button
                label="CSS Class"
                icon="pi pi-clipboard"
                outlined
                @click="copyToClipboard(currentIcon.icon?.className, 'CSS class')"
            />
            <Button
                :label="currentIcon.icon?.encodedCode"
                icon="pi pi-clipboard"
                outlined
                @click="copyToClipboard(currentIcon.icon?.encodedCode, 'Unicode')"
            />
            <Button
                :label="currentIcon.icon?.unicode"
                icon="pi pi-clipboard"
                outlined
                @click="copyToClipboard(currentIcon.icon?.unicode, 'HTML character')"
            />
            <Button
                label="SVG code"
                icon="pi pi-clipboard"
                outlined
                @click="copyToClipboard(currentIcon.content, 'SVG code')"
            />
            <a
                :href="`/svg/${currentIcon.category}/${currentIcon.key}.svg`"
                :download="`${currentIcon.icon?.className}.svg`"
                class="flex flex-col"
            >
                <Button label="SVG file" icon="pi pi-download" outlined />
            </a>
        </div>
    </Dialog>

    <div class="fixed w-full z-10 top-6">
        <div class="container mx-auto px-4">
            <Menubar :model="items" class="">
                <template #start>
                    <a href="#top" class="text-[#93C045]">
                        <strong class="text-lg hidden sm:block">Video Games Icons</strong>
                        <strong class="text-lg block sm:hidden">VGI</strong>
                    </a>
                </template>
                <template #item="{ item, props, hasSubmenu, root }">
                    <a class="flex items-center gap-1" v-bind="props.action" :href="item.href">
                        <span :class="item.icon" />
                        <span class="">{{ item.label }}</span>
                        <i
                            v-if="hasSubmenu"
                            :class="[
                                'pi pi-angle-down',
                                { 'pi-angle-down': root, 'pi-angle-right': !root },
                            ]"
                        ></i>
                    </a>
                </template>
                <template #end>
                    <div class="flex items-center gap-6 mr-4">
                        <span class="hidden sm:block"
                            ><strong>Font size:</strong> {{ fontSize }}px</span
                        >
                        <Slider
                            v-model="fontSize"
                            :step="2"
                            :min="12"
                            :max="52"
                            class="w-[15rem]"
                        />
                    </div>
                </template>
            </Menubar>
        </div>
    </div>

    <div
        class="bg-surface-50 h-[75dvh] pt-28 pb-12 flex items-center justify-center text-center"
        id="top"
    >
        <div class="container px-4">HERO</div>
    </div>

    <div class="container mx-auto relative px-4">
        <div v-for="{ name, icons, id } in data" :key="id" class="pt-28" :id="id">
            <h2 class="text-4xl font-bold text-[#93C045] mb-6">
                {{ name }}
                <small class="text-2xl text-surface-400 font-semibold"
                    >({{ Object.keys(icons).length }} icons)</small
                >
            </h2>
            <div
                class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-5 xl:grid-cols-6 2xl:grid-cols-8 gap-6"
            >
                <a
                    class="flex flex-col gap-2 items-center rounded-lg hover:text-[#93C045]"
                    href="#!"
                    v-for="(icon, key) in icons"
                    :key="key"
                    @click.prevent="openIconDetails(id, key, icon)"
                >
                    <span
                        class="bg-surface-50 p-4 rounded-lg w-full flex items-center justify-center min-h-[10rem]"
                    >
                        <span
                            :class="[icon.className, '']"
                            :style="`font-size: ${fontSize}px`"
                            class=""
                        ></span>
                    </span>
                    <span class="font-bold text-sm">{{ key }}</span>
                </a>
            </div>
        </div>
    </div>

    <div class="container mx-auto py-28 px-4" id="documentation">
        <h2 class="text-4xl font-bold mb-6 text-[#93C045]">Documentation</h2>
    </div>

    <footer class="mt-28 mb-6">
        <div class="container mx-auto px-4">FOOTER</div>
    </footer>
</template>

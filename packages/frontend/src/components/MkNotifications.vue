<!--
SPDX-FileCopyrightText: syuilo and other misskey contributors
SPDX-License-Identifier: AGPL-3.0-only
-->

<template>
<MkPagination ref="pagingComponent" :pagination="pagination">
	<template #empty>
		<div class="_fullinfo">
			<img :src="infoImageUrl" class="_ghost"/>
			<div>{{ i18n.ts.noNotifications }}</div>
		</div>
	</template>

	<template #default="{ items: notifications }">
		<MkDateSeparatedList v-slot="{ item: notification }" :class="$style.list" :items="notifications" :noGap="true">
			<MkNote v-if="['reply', 'quote', 'mention'].includes(notification.type)" :key="notification.id" :note="notification.note"/>
			<XNotification v-else :key="notification.id" :notification="notification" :withTime="true" :full="true" class="_panel notification"/>
		</MkDateSeparatedList>
	</template>
</MkPagination>
</template>

<script lang="ts" setup>
import { onUnmounted, onMounted, computed, shallowRef } from 'vue';
import MkPagination, { Paging } from '@/components/MkPagination.vue';
import XNotification from '@/components/MkNotification.vue';
import MkDateSeparatedList from '@/components/MkDateSeparatedList.vue';
import MkNote from '@/components/MkNote.vue';
import { useStream } from '@/stream.js';
import { $i } from '@/account.js';
import { i18n } from '@/i18n.js';
import { notificationTypes } from '@/const.js';
import { infoImageUrl } from '@/instance.js';

const props = defineProps<{
	excludeTypes?: typeof notificationTypes[number][];
}>();

const pagingComponent = shallowRef<InstanceType<typeof MkPagination>>();

const pagination: Paging = {
	endpoint: 'i/notifications' as const,
	limit: 20,
	params: computed(() => ({
		excludeTypes: props.excludeTypes ?? undefined,
	})),
};

const onNotification = (notification) => {
	const isMuted = props.excludeTypes ? props.excludeTypes.includes(notification.type) : false;
	if (isMuted || document.visibilityState === 'visible') {
		useStream().send('readNotification');
	}

	if (!isMuted) {
		pagingComponent.value.prepend(notification);
	}
};

let connection;

onMounted(() => {
	connection = useStream().useChannel('main');
	connection.on('notification', onNotification);
});

onUnmounted(() => {
	if (connection) connection.dispose();
});
</script>

<style lang="scss" module>
.list {
	background: var(--panel);
}
</style>

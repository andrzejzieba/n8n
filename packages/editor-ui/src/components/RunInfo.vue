<script setup lang="ts">
import type { ITaskData } from 'n8n-workflow';
import { convertToDisplayDateComponents } from '@/utils/formatters/dateFormatter';
import { computed } from 'vue';
import { useI18n } from '@/composables/useI18n';

const i18n = useI18n();

const props = defineProps<{
	taskData: ITaskData | null;
	hasStaleData?: boolean;
	hasPinData?: boolean;
}>();

const runTaskData = computed(() => {
	return props.taskData;
});

const theme = computed(() => {
	return props.taskData?.error ? 'danger' : 'success';
});

const runMetadata = computed(() => {
	if (!runTaskData.value) {
		return null;
	}
	const { date, time } = convertToDisplayDateComponents(runTaskData.value.startTime);
	return {
		executionTime: runTaskData.value.executionTime,
		startTime: `${date} at ${time}`,
	};
});
</script>

<template>
	<n8n-info-tip
		v-if="hasStaleData"
		theme="warning"
		type="tooltip"
		tooltip-placement="right"
		data-test-id="node-run-info-stale"
	>
		<span
			v-n8n-html="
				i18n.baseText(
					hasPinData
						? 'ndv.output.staleDataWarning.pinData'
						: 'ndv.output.staleDataWarning.regular',
				)
			"
		></span>
	</n8n-info-tip>
	<div v-else-if="runMetadata" :class="$style.tooltipRow">
		<n8n-info-tip type="note" :theme="theme" :data-test-id="`node-run-status-${theme}`" />
		<n8n-info-tip
			type="tooltip"
			theme="info"
			:data-test-id="`node-run-info`"
			tooltip-placement="right"
		>
			<div>
				<n8n-text :bold="true" size="small"
					>{{
						runTaskData?.error
							? i18n.baseText('runData.executionStatus.failed')
							: i18n.baseText('runData.executionStatus.success')
					}} </n8n-text
				><br />
				<n8n-text :bold="true" size="small">{{
					i18n.baseText('runData.startTime') + ':'
				}}</n8n-text>
				{{ runMetadata.startTime }}<br />
				<n8n-text :bold="true" size="small">{{
					i18n.baseText('runData.executionTime') + ':'
				}}</n8n-text>
				{{ runMetadata.executionTime }} {{ i18n.baseText('runData.ms') }}
			</div>
		</n8n-info-tip>
	</div>
</template>

<style lang="scss" module>
.tooltipRow {
	display: flex;
	column-gap: var(--spacing-4xs);
}
</style>

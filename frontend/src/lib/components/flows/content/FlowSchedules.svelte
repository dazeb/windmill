<script lang="ts">
	import CronInput from '$lib/components/CronInput.svelte'
	import SchemaForm from '$lib/components/SchemaForm.svelte'
	import Toggle from '$lib/components/Toggle.svelte'
	import { emptyString } from '$lib/utils'
	import { getContext } from 'svelte'
	import type { FlowEditorContext } from '../types'
	import { Alert, Button, Skeleton } from '$lib/components/common'
	import ScheduleEditor from '$lib/components/ScheduleEditor.svelte'
	import { ScheduleService, type Schedule } from '$lib/gen'
	import { workspaceStore } from '$lib/stores'
	import { Calendar } from 'lucide-svelte'
	import Label from '$lib/components/Label.svelte'

	const { schedule, flowStore, initialPath } = getContext<FlowEditorContext>('FlowEditorContext')
	let schedules: Schedule[] | undefined = undefined

	async function loadSchedules() {
		try {
			schedules = (
				await ScheduleService.listSchedules({
					workspace: $workspaceStore ?? '',
					path: initialPath,
					isFlow: true
				})
			).filter((s) => s.path != initialPath)
		} catch (e) {
			console.error('impossible to load schedules')
		}
	}

	let scheduleEditor: ScheduleEditor

	$: initialPath && loadSchedules()
</script>

<div class="w-full flex flex-col gap-4 mb-4">
	<!-- svelte-ignore a11y-autofocus -->
	<Label label="Summary">
		<input
			autofocus
			type="text"
			placeholder="Short summary to be displayed when listed"
			class="text-sm w-full"
			bind:value={$schedule.summary}
		/>
	</Label>
</div>

<CronInput bind:schedule={$schedule.cron} bind:timezone={$schedule.timezone} />
<div class="mt-10" />
<SchemaForm schema={$flowStore.schema} bind:args={$schedule.args} />
{#if emptyString($schedule.cron)}
	<p class="text-xs text-tertiary mt-10">Define a schedule frequency first</p>
{/if}
<div class="mt-10" />
<Toggle
	disabled={emptyString($schedule.cron)}
	bind:checked={$schedule.enabled}
	options={{
		right: 'Schedule enabled'
	}}
/>
<Alert bgClass="my-4" type="warning" title="Changes only applied upon deploy">
	Changes to the primary schedule are only applied upon deploy. Other schedules' changes are applied
	immediately.
</Alert>

{#if initialPath != ''}
	<ScheduleEditor
		on:update={() => {
			loadSchedules()
		}}
		bind:this={scheduleEditor}
	/>
	<h2 class="pt-7">Other schedules</h2>
	<div class="py-4 flex">
		<Button
			on:click={() => scheduleEditor?.openNew(true, initialPath)}
			variant="border"
			color="light"
			size="xs"
			startIcon={{ icon: Calendar }}
		>
			New Schedule
		</Button>
	</div>

	{#if schedules}
		{#if schedules.length == 0}
			<div class="text-xs text-secondary px-2"> No other schedules </div>
		{:else}
			<div class="flex flex-col divide-y px-2 pt-2 max-w-lg">
				{#each schedules as schedule (schedule.path)}
					<div class="grid grid-cols-6 text-2xs items-center py-2"
						><div class="col-span-3 truncate">{schedule.path}</div><div>{schedule.schedule}</div>
						<div>{schedule.enabled ? 'on' : 'off'}</div>
						<button on:click={() => scheduleEditor?.openEdit(schedule.path, true)}>Edit</button>
					</div>
				{/each}
			</div>
		{/if}
	{:else}
		<Skeleton layout={[[8]]} />
	{/if}
{/if}

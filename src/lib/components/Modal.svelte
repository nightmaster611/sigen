<script lang="ts">
	import clickOutsideElement from '$lib/utils/svelte-actions/clickOutsideElement';
	import { createEventDispatcher, onDestroy, onMount } from 'svelte';
	import Backdrop from './Backdrop.svelte';

	interface $$Slots {
		header: any;
		default: any;
	}

	export let visible = false;
	export let backdropVisible = true;
	export let header: string | null | undefined = 'Modal Header';
	export let outClickToClose = true;
	export let containerClassName = '';
	export let contentClassName = '';
	export let destroyOnClose = false;

	const dispatch = createEventDispatcher();
	$: onCloseModal = visible
		? () => {
				visible = false;
				dispatch('close');
		  }
		: undefined;

	$: onKeyEscPressHandler = (e: KeyboardEvent) => {
		if (e.key === 'Escape' && visible) {
			onCloseModal?.();
		}
	};
	onMount(async () => {
		document.addEventListener('keydown', onKeyEscPressHandler);
	});
	onDestroy(() => {
		document.removeEventListener('keydown', onKeyEscPressHandler);
	});
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<Backdrop {backdropVisible} {destroyOnClose} {visible}>
	<div
		class="absolute top-0 left-0 flex justify-center items-center h-screen w-screen bg-opacity-50"
		class:bg-gray-500={backdropVisible && visible}
	>
		<div
			class="modal {visible ? 'scale-100' : 'scale-0'}  {containerClassName}"
			use:clickOutsideElement
			on:outclick={() => {
				if (outClickToClose) {
					onCloseModal?.();
				}
			}}
		>
			{#if header !== null}
				<div class="modal-header border-b border-input" class:invisible={!visible}>
					{#if $$slots.header}
						<slot name="header" />
					{:else}
						<div class="flex justify-between items-center">
							<div class="p-4 leading-[0.9]">{header}</div>
							<!-- <div class="p-5 cursor-pointer" on:click={onCloseModal}>
								<Icon class="text-placeholder text-sm" name="close" />
							</div> -->
						</div>
					{/if}
				</div>
			{/if}
			<div class="modal-content {contentClassName}" class:invisible={!visible}>
				<slot />
			</div>
		</div>
	</div>
</Backdrop>

<style lang="scss">
	.modal {
		@apply flex flex-col max-h-[80%] overflow-hidden bg-white transition-transform duration-100;
		@apply rounded-default;
	}
	.modal-content {
		@apply overflow-auto;
	}
</style>

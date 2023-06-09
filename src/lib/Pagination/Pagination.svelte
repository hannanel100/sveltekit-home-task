<script lang="ts">
	import { goto } from '$app/navigation';
	import { totalPages, currentPage, pageSize, sort } from '$lib/stores/pages';
	import { page } from '$app/stores';
	import Fa from 'svelte-fa/src/fa.svelte';
	import {
		faArrowLeft,
		faArrowLeftLong,
		faArrowRight,
		faArrowRightLong
	} from '@fortawesome/free-solid-svg-icons';
	import Select from '$lib/select/Select.svelte';
	// constants reused throughout the component
	const BREAKPOINT_WIDTH = 768;
	const MIN_PAGE_LINKS = 2;
	const MAX_PAGE_LINKS = 5;
	const SMALL_PAGE_SKIP = 10;
	const LARGE_PAGE_SKIP = 50;
	// store values
	$totalPages;
	$currentPage;
	$pageSize;
	$sort;
	// props
	export let totalItems: number;
	// setting store values
	$: totalPages.set(Math.ceil(totalItems / selected) - 1);
	$: currentPage.set(Number($page.url.searchParams.get('page')) || 1);
	$: pageSize.set(Number($page.url.searchParams.get('limit')) || 10);
	// setting local state values
	let numOfPagesLinks = 5;
	let innerWidth = 0;

	$: {
		if (innerWidth < BREAKPOINT_WIDTH) {
			numOfPagesLinks = MIN_PAGE_LINKS;
		} else {
			numOfPagesLinks = MAX_PAGE_LINKS;
		}
	}
	function reloadPage() {
		goto(`/users?page=${$currentPage}&limit=${$pageSize}&sort=${$sort}`);
	}
	let options = [10, 20, 50, 100];
	let selected = options.filter((option) => option === 10)[0];
	$: {
		if (selected) {
			pageSize.set(Number(selected));
		}
	}
</script>

<svelte:window bind:innerWidth on:change={reloadPage} />
<div class="pagination-select-container">
	<Select bind:value={selected} onChange={() => pageSize.set($pageSize)} bind:options />
	<div class="pagination-container">
		{#if $currentPage - LARGE_PAGE_SKIP > 0}
			<a
				href={`/users?page=${$currentPage - LARGE_PAGE_SKIP}&limit=${$pageSize}&sort=${$sort}`}
				class="tooltip"
			>
				<Fa icon={faArrowLeftLong} />

				<span class="tooltiptext top-right">Back {LARGE_PAGE_SKIP} pages</span></a
			>
		{/if}
		{#if $currentPage - SMALL_PAGE_SKIP > 0}
			<a
				href={`/users?page=${$currentPage - SMALL_PAGE_SKIP}&limit=${$pageSize}&sort=${$sort}`}
				class="tooltip"
			>
				<Fa icon={faArrowLeft} />
				<span class="tooltiptext top-right">Back {SMALL_PAGE_SKIP} pages</span></a
			>
		{/if}
		{#if $currentPage > SMALL_PAGE_SKIP}
			<a href={`/users?page=1&limit=${$pageSize}&sort=${$sort}`} class="tooltip"
				>1
				<span class="tooltiptext">First page</span></a
			>
			<span>...</span>
		{/if}
		{#each Array.from({ length: $totalPages }) as _, i}
			<!-- only show the previous 5 if they exist and the next five if they exist -->
			{#if i + 1 >= $currentPage - numOfPagesLinks && i + 1 <= $currentPage + numOfPagesLinks}
				<a
					href={`/users?page=${i + 1}&limit=${$pageSize}&sort=${$sort}`}
					data-sveltekit-preload-data="hover"
					class={`${$currentPage === i + 1 ? 'active' : ''}`}>{i + 1}</a
				>
			{/if}
		{/each}
		<!-- last page, if not currently on it or within range to show it -->
		{#if $currentPage < $totalPages - SMALL_PAGE_SKIP}
			<span>...</span>
			<a href={`/users?page=${$totalPages}&limit=${$pageSize}&sort=${$sort}`} class="tooltip"
				>{$totalPages}
				<span class="tooltiptext">Last page</span></a
			>
		{/if}
		<!-- add an arrow if not in the end of the list, arrow will skip 10 pages -->
		{#if $currentPage + SMALL_PAGE_SKIP < $totalPages}
			<a
				href={`/users?page=${$currentPage + SMALL_PAGE_SKIP}&limit=${$pageSize}&sort=${$sort}`}
				class="tooltip"
			>
				<Fa icon={faArrowRight} />
				<span class="tooltiptext">Skip {SMALL_PAGE_SKIP} pages</span></a
			>
		{/if}
		{#if $currentPage + LARGE_PAGE_SKIP < $totalPages}
			<a
				href={`/users?page=${$currentPage + LARGE_PAGE_SKIP}&limit=${$pageSize}&sort=${$sort}`}
				class="tooltip"
			>
				<Fa icon={faArrowRightLong} />
				<span class="tooltiptext">Skip {LARGE_PAGE_SKIP} pages</span></a
			>
		{/if}
	</div>
</div>

<style>
	.pagination-select-container {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		flex-wrap: wrap;
		gap: 1rem;
		padding-top: 4rem;
		padding-bottom: 4rem;
	}

	.pagination-container {
		display: flex;
		justify-content: center;
		align-items: center;
		flex-wrap: wrap;
		gap: 1rem;
	}
	.pagination-container a {
		text-decoration: none;
		color: #000;
		display: grid;
		place-items: center;
		width: 2rem;
		height: 2rem;

		border-radius: 5px;
		transition: all 0.2s ease-in-out;
	}
	.pagination-container a:hover {
		background-color: #000;
		color: #fff;
	}
	/* class to highlight current page*/
	.pagination-container a.active {
		background-color: #000;
		color: #fff;
	}
	.tooltip {
		position: relative;
		display: inline-block;
	}

	.tooltip .tooltiptext {
		visibility: hidden;
		width: 120px;
		background-color: black;
		color: #fff;
		text-align: center;
		border-radius: 6px;
		padding: 5px 0;

		/* Position the tooltip */
		position: absolute;
		top: -40px;
		left: -40px;
		z-index: 1;
	}

	.tooltip:hover .tooltiptext {
		visibility: visible;
	}

	@media (max-width: 768px) {
		.pagination-container {
			gap: 0.25rem;
		}
		.pagination-container a {
			width: 1.15rem;
			height: 1.15rem;
			padding: 0.15rem;
		}
	}
	@media (max-width: 480px) {
		.pagination-container {
			gap: 0.15rem;
		}
		.tooltip .tooltiptext {
			width: 80px;
			left: -50px;
		}

		.pagination-container a {
			width: 1rem;
			height: 1rem;
			padding: 0.15rem;
		}
	}
</style>

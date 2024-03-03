<template>
	<div class="flex justify-between">
		<div>
			Showing  {{ pageSize }} to {{ currentPage }} of {{ paginationTotal }} results
		</div>
		<div>
			<ul :class="containerClass ?? 'pagination-wrapper'">
				<li
					v-if="firstLastButton"
					:class="[
					pageClass ?? 'page-item',
					firstPageSelected ? disabledClass : '',
					]"
				>
					<button
					@click="selectFirstPage()"
					@keyup.enter="selectFirstPage()"
					:class="pageLinkClass"
					:tabindex="firstPageSelected ? -1 : 0"
					>
					<span :html="firstButtonText" v-if="firstButtonText" />
					<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" class="">
						<path
						d="M18.41 16.59L13.82 12l4.59-4.59L17 6l-6 6 6 6 1.41-1.41M6 6h2v12H6V6z"
						></path>
					</svg>
					</button>
				</li>
				<li
					v-if="!(firstPageSelected && hidePrevNext)"
					:class="[prevClass, firstPageSelected ? disabledClass : '']"
				>
					<button
					@click="prevPage()"
					@keyup.enter="prevPage()"
					:class="prevLinkClass"
					:tabindex="firstPageSelected ? -1 : 0"
					>
					<span :html="prevText" v-if="prevText" />
					<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" class="">
						<path
						d="M15.41 16.58L10.83 12l4.58-4.59L14 6l-6 6 6 6 1.41-1.42z"
						></path>
					</svg>
					</button>
				</li>
				<li
					v-for="page in pages"
					:class="[
					pageClass ?? 'page-item',
					page.selected ? activeClass : '',
					page.disabled ? disabledClass : '',
					page.breakView ? breakViewClass : '',
					]"
				>
					<button
					v-if="page.breakView"
					:class="[pageLinkClass, breakViewLinkClass]"
					tabindex="0"
					>
					<slot name="breakViewContent">
						<span>{{ breakViewText }}</span>
					</slot>
					</button>
					<button v-else-if="page.disabled" :class="pageLinkClass" tabindex="0">
					{{ page.content }}
					</button>
					<button
					v-else
					@click="handlePageSelected(page.index + 1)"
					@keyup.enter="handlePageSelected(page.index + 1)"
					:class="pageLinkClass"
					tabindex="0"
					>
					{{ page.content }}
					</button>
				</li>
				<li
					v-if="!(lastPageSelected && hidePrevNext)"
					:class="[nextClass, lastPageSelected ? disabledClass : '']"
				>
					<button
					@click="nextPage()"
					@keyup.enter="nextPage()"
					:class="nextLinkClass"
					:tabindex="lastPageSelected ? -1 : 0"
					>
					<span :html="nextText" v-if="nextText" />
					<svg
						v-else
						xmlns="http://www.w3.org/2000/svg"
						viewBox="0 0 24 24"
						class=""
					>
						<path
						d="M8.59 16.58L13.17 12 8.59 7.41 10 6l6 6-6 6-1.41-1.42z"
						></path>
					</svg>
					</button>
				</li>
				<li
					v-if="firstLastButton"
					:class="[pageClass ?? 'page-item', lastPageSelected ? disabledClass : '']"
				>
					<button
					@click="selectLastPage()"
					@keyup.enter="selectLastPage()"
					:class="pageLinkClass"
					:tabindex="lastPageSelected ? -1 : 0"
					>
					<span :html="lastButtonText" v-if="lastButtonText" />
					<svg
						v-else
						xmlns="http://www.w3.org/2000/svg"
						viewBox="0 0 24 24"
						class=""
					>
						<path
						d="M5.59 7.41L10.18 12l-4.59 4.59L7 18l6-6-6-6-1.41 1.41M16 6h2v12h-2V6z"
						></path>
					</svg>
					</button>
				</li>
			</ul>
		</div>
	</div>
  </template>
  <script setup>
  import { reactive, watch, ref, defineEmits,computed } from 'vue'


  const props = defineProps({
	value: {
	  type: Number,
	},
	pageSize: {
	  type: Number,
	},
	paginationTotal: {
	  type: Number,
	},
	currentPage: {
		type: Number,
	},
	pageCount: {
	  type: Number,
	  required: true,
	},
	forcePage: {
	  type: Number,
	},
	pageRange: {
	  type: Number,
	  default: 3,
	},
	marginPages: {
	  type: Number,
	  default: 1,
	},
	prevText: {
	  type: String,
	},
	nextText: {
	  type: String,
	},
	breakViewText: {
	  type: String,
	  default: '…',
	},
	containerClass: {
	  type: String,
	},
	pageClass: {
	  type: String,
	},
	pageLinkClass: {
	  type: String,
	},
	prevClass: {
	  type: String,
	},
	prevLinkClass: {
	  type: String,
	},
	nextClass: {
	  type: String,
	},
	nextLinkClass: {
	  type: String,
	},
	breakViewClass: {
	  type: String,
	},
	breakViewLinkClass: {
	  type: String,
	},
	activeClass: {
	  type: String,
	  default: 'active',
	},
	disabledClass: {
	  type: String,
	  default: 'disabled',
	},
	firstLastButton: {
	  type: Boolean,
	  default: false,
	},
	firstButtonText: {
	  type: String,
	},
	lastButtonText: {
	  type: String,
	},
	hidePrevNext: {
	  type: Boolean,
	  default: false,
	},
  });
  const emit = defineEmits(['update:modelValue', 'input']);
  const innerValue = ref(1);
  const handlePageSelected = (selected) => {
	if (selected === innerValue.value) return;
	innerValue.value = selected;
	emit('input', selected);
	emit('update:modelValue', selected);
  };
  const prevPage = () => {
	if (innerValue.value <= 1) return;
	handlePageSelected(innerValue.value - 1);
  };
  const nextPage = () => {
	if (innerValue.value >= props.pageCount) return;
	handlePageSelected(innerValue.value + 1);
  };
  const firstPageSelected = computed(() => innerValue.value === 1);
  const lastPageSelected = computed(
	() => innerValue.value === props.pageCount || props.pageCount === 0,
  );
  const selectFirstPage = () => {
	if (innerValue.value <= 1) return;
	handlePageSelected(1);
  };
  const selectLastPage = () => {
	if (innerValue.value >= props.pageCount) return;
	handlePageSelected(props.pageCount);
  };
  const selected = computed({
	get: () => props.value || innerValue.value,
	set: (newValue) => {
	  innerValue.value = newValue;
	},
  });
  const pages = computed(() => {
	let items = {};
	if (props.pageCount <= props.pageRange) {
	  for (let index = 0; index < props.pageCount; index++) {
		let page = {
		  index: index,
		  content: index + 1,
		  selected: index === innerValue.value - 1,
		};
		items[index] = page;
	  }
	} else {
	  const halfPageRange = Math.floor(props.pageRange / 2);
	  let setPageItem = (index) => {
		let page = {
		  index: index,
		  content: index + 1,
		  selected: index === innerValue.value - 1,
		};
		items[index] = page;
	  };
	  let setBreakView = (index) => {
		let breakView = {
		  disabled: true,
		  breakView: true,
		};
		items[index] = breakView;
	  };
	  // 1st - loop thru low end of margin pages
	  for (let i = 0; i < props.marginPages; i++) {
		setPageItem(i);
	  }
	  // 2nd - loop thru selected range
	  let selectedRangeLow = 0;
	  if (innerValue.value - halfPageRange > 0) {
		selectedRangeLow = innerValue.value - 1 - halfPageRange;
	  }
	  let selectedRangeHigh = selectedRangeLow + props.pageRange - 1;
	  if (selectedRangeHigh >= props.pageCount) {
		selectedRangeHigh = props.pageCount - 1;
		selectedRangeLow = selectedRangeHigh - props.pageRange + 1;
	  }
	  for (
		let i = selectedRangeLow;
		i <= selectedRangeHigh && i <= props.pageCount - 1;
		i++
	  ) {
		setPageItem(i);
	  }
	  // Check if there is breakView in the left of selected range
	  if (selectedRangeLow > props.marginPages) {
		setBreakView(selectedRangeLow - 1);
	  }
	  // Check if there is breakView in the right of selected range
	  if (selectedRangeHigh + 1 < props.pageCount - props.marginPages) {
		setBreakView(selectedRangeHigh + 1);
	  }
	  // 3rd - loop thru high end of margin pages
	  for (
		let i = props.pageCount - 1;
		i >= props.pageCount - props.marginPages;
		i--
	  ) {
		setPageItem(i);
	  }
	}
	return items;
  });
  watch(
	() => props.forcePage,
	(newValue) => {
	  if (newValue === undefined) return;
	  if (newValue !== selected.value) {
		innerValue.value = newValue;
	  }
	},
  );
  </script>
  <style scoped>
	.pagination-wrapper {
		display: flex;
		flex-flow: row;
		flex-wrap: wrap;
		align-items: center;
		gap: 1px;
		margin: 0;
		padding: 0;
		list-style-type: none;
		border:#D1D5DA solid 1px;
		border-radius: 3px;
		}

		.pagination-wrapper li,
		.pagination-wrapper .page-item {
		height: 32px !important;
		width: 32px !important;
		display: flex;
		align-items: center;
		justify-content: center;
		border-radius: var(--ct-border-radius);
		overflow: hidden;
		background-color: white !important;
		/* border: var(--ct-border-width) solid transparent; */
		border-right: #D1D5DA solid 1px;
		transition: all var(--ct-transition-duration) ease;
		/* border-radius: 4px !important; */
		}

		@media (min-width: 768px) {
		.pagination-wrapper li,
		.pagination-wrapper .page-item {
			height: 32px !important;
			width: 32px !important;
			/* border-radius: 4px !important; */
		}
		}

		.pagination-wrapper li:hover,
		.pagination-wrapper .page-item:hover {
		border: var(--ct-border-width) solid var(--ct-secondary);
		}

		.pagination-wrapper li.disabled,
		.pagination-wrapper .page-item.disabled {
		position: relative;
		display: block;
		fill: #bbbbbb;
		}

		.pagination-wrapper li.disabled button,
		.pagination-wrapper .page-item.disabled button {
		cursor: not-allowed;
		}

		.pagination-wrapper li.disabled:hover,
		.pagination-wrapper .page-item.disabled:hover {
		border: var(--ct-border-width) solid transparent !important;
		}

		.pagination-wrapper li.active,
		.pagination-wrapper .page-item.active {
		border-radius: 4px !important;
		background-color: #7C3AED !important;
		}

		.pagination-wrapper li.active button,
		.pagination-wrapper .page-item.active button {
		height: 100%;
		width: 100%;
		color: #f3f3f3 !important;
		}

		.pagination-wrapper li.active:hover,
		.pagination-wrapper .page-item.active:hover {
		border: var(--ct-border-width) solid transparent !important;
		}

		.pagination-wrapper li.active button:hover,
		.pagination-wrapper .page-item.active button:hover {
		background-color: #7C3AED !important;
		border: var(--ct-border-width) solid transparent !important;
		}

		.pagination-wrapper button {
		display: block;
		font-size: var(--ct-body-font-size);
		font-weight: 500;
		color: #333333;
		transition: all var(--ct-transition-duration) ease;
		outline: 0 !important;
		height: 100%;
		width: 100%;
		display: flex;
		align-items: center;
		justify-content: center;
		border: var(--ct-border-width) solid transparent;
		}

		.pagination-wrapper button * {
		vertical-align: text-bottom;
		}

		a {
		cursor: pointer;
		}

  </style>
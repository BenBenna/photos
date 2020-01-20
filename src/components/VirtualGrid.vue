
<template>
	<Grid id="timeline" ref="grid">
		<template v-for="(item, index) in list">
			<!-- top spacer -->
			<span v-if="index == 0"
				:key="`filler-top-${index}`"
				ref="filler-top"
				class="grid-filler grid-filler--top"
				role="none" />

			<h2 v-if="index == 0 ||
					getFormatedDate(item.lastmod, 'MMMM YYYY') != getFormatedDate(list[index - 1].lastmod,'MMMM YYYY')"
				:key="item.lastmod"
				role="none"
				:class="['grid-title', index == 0 ? 'first-title' : '']">
				{{ getFormatedDate(item.lastmod, 'MMMM') }}
				<span>{{ getFormatedDate(item.lastmod, 'YYYY') }}</span>
			</h2>

			<!-- files list -->
			<component :is="component(item)"
				:key="item.fileid"
				:ref="`item-${index}`"
				:list="list"
				v-bind="props(item)" />

			<!-- bottom spacer -->
			<span v-if="index == list.length"
				ref="filler-bottom"
				:key="`filler-bottom-${index}`"
				class="grid-filler grid-filler--bottom"
				role="none" />
		</template>

		<!-- next page loading indicator -->
		<div v-if="loadingPage"
			key="grid-loading"
			class="grid-loading icon-loading"
			role="none" />
	</Grid>
</template>

<script>
import * as moment from 'moment'
import { requestTimeout, clearRequestTimeout } from '@essentials/request-timeout'
import Grid from './Grid'
import GridConfigMixin from '../mixins/GridConfig'

/**
 * Specifies the number of miliseconds during which to disable pointer events while a scroll is in progress.
 * This improves performance and makes scrolling smoother.
 */
export const DEFAULT_SCROLLING_RESET_TIME_INTERVAL = 150

export default {
	name: 'VirtualGrid',
	components: {
		Grid,
	},
	mixins: [GridConfigMixin],

	props: {
		list: {
			type: Array,
			default: () => ([]),
		},
		props: {
			type: Function,
			default: () => ({}),
		},
		component: {
			type: Function,
			required: true,
		},
		loadingPage: {
			type: Boolean,
			default: false,
		},
	},

	created() {
		window.addEventListener('scroll', this.onDocumentScroll)
	},
	mounted() {
		this.onDocumentScroll()
	},
	beforeDestroy() {
		window.removeEventListener('scroll', this.onDocumentScroll)
	},

	methods: {

		/**
		 * Request an animation frame and debounce the onScroll method
		 */
		debounceOnDocumentScroll() {
			if (this.debounceOnDocumentScrollRequest) {
				clearRequestTimeout(this.debounceOnDocumentScrollRequest)
			}

			this.debounceOnDocumentScrollRequest = requestTimeout(
				this.onDocumentScroll,
				DEFAULT_SCROLLING_RESET_TIME_INTERVAL,
			)
		},

		/**
		 * Handle document scroll
		 */
		onDocumentScroll() {
			if ((window.innerHeight + window.pageYOffset) >= document.getElementById('timeline').offsetHeight - 256) {
				this.$emit('bottomReached')
			}
		},

		getFormatedDate(string, format) {
			return moment(string).format(format)
		},

	},
}
</script>

<style lang="scss" scoped>
.grid-filler {
	// put the filler at the end of the row to put the next one into a new line
	grid-column-end: -1;
}

.grid-loading {
	grid-column: 1/-1;
	height: 88px;
}

.grid-title {
	grid-column: 1/-1;
	padding: 36px 0 12px 0;
	background:#fff;
	margin: 0;

	span {
		font-weight: normal;
	}

	&.first-title {
		padding: 0 0 12px 0;
	}
}
</style>

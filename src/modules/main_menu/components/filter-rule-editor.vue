<template>
	<section class="ffz--filter-rule-editor tw-elevation-1 tw-c-background-base tw-border tw-pd-y-05 tw-mg-y-05 tw-pd-r-1 tw-flex tw-flex-nowrap">
		<div class="tw-flex tw-flex-shrink-0 tw-align-items-center handle tw-c-text-alt-2">
			<h5 class="ffz-i-ellipsis-vert ffz-font-size-5" />
		</div>

		<div v-if="! component" class="tw-flex tw-flex-grow-1 tw-align-self-center tw-align-items-center">
			{{ t('setting.filters.missing', 'This rule of type "{type}" cannot be loaded. It may be from an add-on that is not loaded.', {type: editing && editing.type}) }}
		</div>
		<component
			v-else
			:is="component"
			v-model="editing"
			:type="type"
			:filters="filters"
			:context="context"
			:preview="preview"
		/>

		<div
			v-if="isShort && preview"
			class="tw-mg-l-1 tw-pd-x-1 tw-border-l tw-flex tw-align-items-center ffz--profile__icon tw-relative ffz-il-tooltip__container"
		>
			<figure :class="[passes ? 'ffz-i-ok' : 'ffz-i-cancel']" />
			<div class="ffz-il-tooltip ffz-il-tooltip--up ffz-il-tooltip--align-right">
				<span v-if="passes">
					{{ t('setting.filters.active', 'This rule matches.') }}
				</span>
				<span v-else>
					{{ t('setting.filters.inactive', 'This rule does not match.') }}
				</span>
			</div>
		</div>

		<div
			:class="[isShort ? '' : 'tw-mg-l-1']"
			class="tw-border-l tw-pd-l-1 tw-flex tw-flex-column tw-flex-wrap tw-justify-content-start tw-align-items-start"
		>
			<div v-if="! isShort && preview" class="tw-mg-b-1 tw-border-b tw-pd-b-1 tw-full-width tw-flex tw-justify-content-center ffz--profile__icon tw-relative ffz-il-tooltip__container">
				<figure :class="[passes ? 'ffz-i-ok' : 'ffz-i-cancel']" />
				<div class="ffz-il-tooltip ffz-il-tooltip--up ffz-il-tooltip--align-right">
					<span v-if="passes">
						{{ t('setting.filters.active', 'This rule matches.') }}
					</span>
					<span v-else>
						{{ t('setting.filters.inactive', 'This rule does not match.') }}
					</span>
				</div>
			</div>

			<template v-if="deleting">
				<button class="tw-button tw-button--text tw-relative ffz-il-tooltip__container" @click="deleting = false">
					<span class="tw-button__text ffz-i-cancel" />
					<div class="ffz-il-tooltip ffz-il-tooltip--up ffz-il-tooltip--align-right">
						{{ t('setting.cancel', 'Cancel') }}
					</div>
				</button>
				<button class="tw-button tw-button--text tw-relative ffz-il-tooltip__container" @click="$emit('delete')">
					<span class="tw-button__text ffz-i-trash" />
					<div class="ffz-il-tooltip ffz-il-tooltip--up ffz-il-tooltip--align-right">
						{{ t('setting.delete', 'Delete') }}
					</div>
				</button>
			</template>
			<template v-else>
				<button class="tw-button tw-button--text tw-relative ffz-il-tooltip__container" @click="deleting = true">
					<span class="tw-button__text ffz-i-trash" />
					<div class="ffz-il-tooltip ffz-il-tooltip--up ffz-il-tooltip--align-right">
						{{ t('setting.delete', 'Delete') }}
					</div>
				</button>
			</template>
		</div>
	</section>
</template>

<script>

import {deep_copy} from 'utilities/object';

export default {
	props: {
		value: Object,
		filters: Object,
		context: {
			type: Object,
			required: false
		},
		preview: {
			type: Boolean,
			required: false,
			default: true
		}
	},

	data() {
		return {
			editing: deep_copy(this.value),
			tester: null,
			deleting: false
		}
	},

	computed: {
		passes() {
			return this.preview && this.tester && this.tester(this.context);
		},

		type() {
			return this.filters[this.editing && this.editing.type]
		},

		component() {
			return this.type && this.type.editor;
		},

		isShort() {
			return ! this.component || (this.type && ! this.type.tall);
		}
	},

	watch: {
		editing: {
			handler() {
				this.tester = this.type.createTest(this.editing.data, this.filters);
				this.$emit('input', this.editing);
			},
			deep: true
		}
	},

	created() {
		this.tester = this.type && this.type.createTest(this.editing && this.editing.data, this.filters);
	}
}

</script>

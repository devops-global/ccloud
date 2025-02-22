<template>
	<div class="row"
		:class="{'disabled': loading.delete || loading.disable}"
		:data-id="user.id">
		<div class="avatar" :class="{'icon-loading-small': loading.delete || loading.disable || loading.wipe}">
			<img v-if="!loading.delete && !loading.disable && !loading.wipe"
				alt=""
				width="32"
				height="32"
				:src="generateAvatar(user.id, isDarkTheme)">
		</div>
		<!-- dirty hack to ellipsis on two lines -->
		<div class="name">
			<div class="displayName subtitle">
				<div :title="user.displayname.length > 20 ? user.displayname : ''" class="cellText">
					<strong>
						{{ user.displayname }}
					</strong>
				</div>
			</div>
			{{ user.id }}
		</div>
		<div />
		<div class="mailAddress">
			<div :title="user.email !== null && user.email.length > 20 ? user.email : ''" class="cellText">
				{{ user.email }}
			</div>
		</div>
		<div class="groups">
			{{ userGroupsLabels }}
		</div>
		<div v-if="subAdminsGroups.length > 0 && settings.isAdmin" class="subAdminsGroups">
			{{ userSubAdminsGroupsLabels }}
		</div>
		<div class="userQuota">
			<div class="quota">
				{{ userQuota }} ({{ usedSpace }})
				<progress class="quota-user-progress"
					:class="{'warn': usedQuota > 80}"
					:value="usedQuota"
					max="100" />
			</div>
		</div>
		<div v-if="showConfig.showLanguages" class="languages">
			{{ userLanguage.name }}
		</div>
		<div v-if="showConfig.showUserBackend || showConfig.showStoragePath" class="userBackend">
			<div v-if="showConfig.showUserBackend" class="userBackend">
				{{ user.backend }}
			</div>
			<div v-if="showConfig.showStoragePath" :title="user.storageLocation" class="storageLocation subtitle">
				{{ user.storageLocation }}
			</div>
		</div>
		<div v-if="showConfig.showLastLogin" :title="userLastLoginTooltip" class="lastLogin">
			{{ userLastLogin }}
		</div>
		<div class="managers">
			{{ user.manager }}
		</div>
		<div class="userActions">
			<UserRowActions v-if="canEdit && !loading.all"
				:actions="userActions"
				:edit="false"
				@update:edit="toggleEdit" />
		</div>
	</div>
</template>

<script>
import { getCurrentUser } from '@nextcloud/auth'

import ClickOutside from 'vue-click-outside'

import UserRowActions from './UserRowActions.vue'
import UserRowMixin from '../../mixins/UserRowMixin.js'

export default {
	name: 'UserRowSimple',
	components: {
		UserRowActions,
	},
	directives: {
		ClickOutside,
	},
	mixins: [UserRowMixin],
	props: {
		user: {
			type: Object,
			required: true,
		},
		loading: {
			type: Object,
			required: true,
		},
		showConfig: {
			type: Object,
			required: true,
		},
		userActions: {
			type: Array,
			required: true,
		},
		openedMenu: {
			type: Boolean,
			required: true,
		},
		subAdminsGroups: {
			type: Array,
			required: true,
		},
		settings: {
			type: Object,
			required: true,
		},
		isDarkTheme: {
			type: Boolean,
			required: true,
		},
	},
	computed: {
		userGroupsLabels() {
			return this.userGroups
				.map(group => group.name)
				.join(', ')
		},
		userSubAdminsGroupsLabels() {
			return this.userSubAdminsGroups
				.map(group => group.name)
				.join(', ')
		},
		usedSpace() {
			if (this.user.quota.used) {
				return t('settings', '{size} used', { size: OC.Util.humanFileSize(this.user.quota.used) })
			}
			return t('settings', '{size} used', { size: OC.Util.humanFileSize(0) })
		},
		canEdit() {
			return getCurrentUser().uid !== this.user.id || this.settings.isAdmin
		},
		userQuota() {
			let quota = this.user.quota.quota

			if (quota === 'default') {
				quota = this.settings.defaultQuota
				if (quota !== 'none') {
					// convert to numeric value to match what the server would usually return
					quota = OC.Util.computerFileSize(quota)
				}
			}

			// when the default quota is unlimited, the server returns -3 here, map it to "none"
			if (quota === 'none' || quota === -3) {
				return t('settings', 'Unlimited')
			} else if (quota >= 0) {
				return OC.Util.humanFileSize(quota)
			}
			return OC.Util.humanFileSize(0)
		},
	},
	methods: {
		toggleMenu() {
			this.$emit('update:openedMenu', !this.openedMenu)
		},
		hideMenu() {
			this.$emit('update:openedMenu', false)
		},
		toggleEdit() {
			this.$emit('update:editing', true)
		},
	},
}
</script>

<style lang="scss">
	.cellText {
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
}
	.icon-more {
		background-color: var(--color-main-background);
		border: 0;
	}
</style>

<!--
Nextcloud - Tasks

@author Raimund Schlüßler
@copyright 2018 Raimund Schlüßler <raimund.schluessler@mailbox.org>

This library is free software; you can redistribute it and/or
modify it under the terms of the GNU AFFERO GENERAL PUBLIC LICENSE
License as published by the Free Software Foundation; either
version 3 of the License, or any later version.

This library is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU AFFERO GENERAL PUBLIC LICENSE for more details.

You should have received a copy of the GNU Affero General Public
License along with this library.  If not, see <http://www.gnu.org/licenses/>.

-->

<template>
	<div id="content" class="app-tasks" @click="closeDetails($event)">
		<AppNavigation>
			<TheList />
			<AppNavigationSettings>
				<TheSettings />
			</AppNavigationSettings>
		</AppNavigation>

		<AppContent>
			<RouterView />
		</AppContent>

		<div id="app-sidebar" :class="{disappear: $route.params.taskId === undefined}">
			<RouterView name="details" />
		</div>
	</div>
</template>

<script>
import { mapState } from 'vuex'
import AppNavigation from '@nextcloud/vue/dist/Components/AppNavigation'
import AppContent from '@nextcloud/vue/dist/Components/AppContent'
import AppNavigationSettings from '@nextcloud/vue/dist/Components/AppNavigationSettings'
import TheList from './components/AppNavigation/List'
import TheSettings from './components/AppNavigation/TheSettings'
import client from './services/cdav.js'

export default {
	name: 'App',
	components: {
		TheSettings,
		TheList,
		AppNavigation,
		AppContent,
		AppNavigationSettings,
	},
	computed: {
		...mapState({
			calendars: state => state.calendars.calendars,
		}),
	},
	beforeMount() {
		// get calendars then get tasks
		client.connect({ enableCalDAV: true }).then(() => {
			this.$store.dispatch('getCalendars')
				.then((calendars) => {
					// No calendars? Create a new one!
					if (calendars.length === 0) {
						let color = '#0082C9'
						if (this.$OCA.Theming) {
							color = this.$OCA.Theming.color
						}
						this.$store.dispatch('appendCalendar', { displayName: this.$t('tasks', 'Tasks'), color })
							.then(() => {
								this.fetchTasks()
							})
					// else, let's get those tasks!
					} else {
						this.fetchTasks()
					}
				})
		})
	},
	methods: {
		/**
		 * Fetch the tasks of each calendar
		 */
		fetchTasks() {
			// wait for all calendars to have fetch their tasks
			Promise.all(this.calendars.map(calendar =>
				this.$store.dispatch('getTasksFromCalendar', { calendar: calendar, completed: false, related: null })
			)).then(results => {
				this.loading = false
				// console.log(results)
			})
		},

		/**
		 * Close the details view
		 *
		 * @param {Object} $event the event
		 */
		closeDetails($event) {
			if (!($event.target.closest('.reactive') || $event.target.classList.contains('reactive'))
			&& !$event.target.closest('#app-sidebar') && this.$route.params.taskId) {
				if (this.$route.params.calendarId) {
					this.$router.push({ name: 'calendars', params: { calendarId: this.$route.params.calendarId } })
				} else {
					this.$router.push({ name: 'collections', params: { collectionId: this.$route.params.collectionId } })
				}
			}
		},
	},
}
</script>

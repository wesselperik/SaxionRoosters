<template>
    <v-container id="week_holder">

        <v-flex row wrap v-if="this.scheduleWeek == null">
            <v-flex xs-12 class="text-xs-center">
                <v-progress-circular
                        :size="70"
                        :width="7"
                        color="primary"
                        indeterminate

                ></v-progress-circular>
            </v-flex>
        </v-flex>
        <!--TODO: make this better -->
        <ScheduleWeekDay v-else v-bind:day="day" v-for="day in this.scheduleWeek.days"/>

        <v-btn v-if="!isSaved"
               @click="buttonSave"
               color="primary"
               dark
               absolute
               bottom
               right
               fab>
            <v-icon>add</v-icon>
        </v-btn>
        <v-btn v-else
               @click="buttonUnSave"
               color="red"
               dark
               absolute
               bottom
               right
               fab>
            <v-icon>trash</v-icon>
        </v-btn>
    </v-container>
</template>

<script>

    import ScheduleWeekDay from './Items/ScheduleWeekDay'
    import ScheduleManager from '../../managers/ScheduleManager'
    import ScheduleIdentity from "../../models/ScheduleIdentity";
    import ScheduleDataManager from '../../managers/ScheduleDataManager'

    export default {
        name: "Schedule",
        components: {ScheduleWeekDay},
        mounted() {
            this.loadSchedule()

        },
        data() {
            return {
                scheduleWeek: null,
                scheduleIdentity: null
            }
        },
        computed: {
            isSaved: function () {
                let identities = ScheduleDataManager.getSavedScheduleIdentities()
                let found = false
                if (this.scheduleIdentity !== null) {
                    identities.forEach((identity) => {
                        if (identity.isTeacher()) {
                            // Check teacher codes
                            if (identity.teacherId === this.scheduleIdentity.teacherId) {
                                found = true
                            }
                        } else {
                            // Check group codes
                            if (identity.groupId === this.scheduleIdentity.groupId) {
                                found = true
                            }
                        }
                    })
                }
                return found
            }
        },
        methods: {
            buttonSave: function () {
                ScheduleDataManager.saveScheduleIdentity(this.scheduleIdentity)
            },

            buttonUnSave: function () {
                ScheduleDataManager.removeScheduleIdentity(this.scheduleIdentity)
            },
            loadSchedule: function () {
                let groupId = this.$route.query.group
                let teacherId = this.$route.query.teacher
                let weekOffset = this.$route.query.week

                this.scheduleIdentity = new ScheduleIdentity(groupId, teacherId)
                if (groupId == null && teacherId == null) {
                    // Fatal error, no schedule id given
                    console.error('No group or teacher ID given')
                } else {
                    // Load schedule
                    new ScheduleManager().getScheduleWeek(this.scheduleIdentity, weekOffset, (scheduleWeek) => {
                        this.scheduleWeek = scheduleWeek
                    })
                }
            }
        },
        watch: {
            '$route.query'() {
                this.loadSchedule()
            }
        }

    }
</script>

<style scoped lang="scss">


</style>
<template>
  <v-row class="justify-center">
    <v-col cols="12">
      <!-- Sessions Table -->
      <div>
        <v-data-table
          class="elevation-1"
          v-model="selectedPunch"
          :headers="headers"
          :items="punchData"
          item-key="id"
          :sortBy="['dateObjIn']"
          :sort-desc="[true]"
          :mobile-breakpoint="0"
          :single-select="true"
          :items-per-page="200"
          height="50vh"
          fixed-header
          dense
          show-select
          hide-default-footer
        >
          <template v-slot:top>
            <v-toolbar flat color="white">
              <v-toolbar-title v-if="!$vuetify.breakpoint.xsOnly"
                >Edit Sessions</v-toolbar-title
              >
              <v-spacer v-if="!$vuetify.breakpoint.xsOnly"></v-spacer>
              <!-- User Select -->
              <div>
                <v-card-text class="mt-6">
                  <v-select
                    dense
                    outlined
                    class="mx-auto"
                    v-model="userID"
                    :items="allUserIDs"
                    item-text="username"
                    item-value="id"
                    prepend-icon="mdi-account"
                    :label="
                      $vuetify.breakpoint.xsOnly == true ? '' : 'User'
                    "
                    @input="getUserSessions"
                  ></v-select>
                </v-card-text>
              </div>
              <v-divider
                v-if="!$vuetify.breakpoint.xsOnly"
                class="mx-4"
                inset
                vertical
              ></v-divider>
              <!-- Delete Session Button -->
              <div>
                <v-btn
                  :class="
                    !$vuetify.breakpoint.xsOnly == true
                      ? 'mx-4'
                      : 'mx-2'
                  "
                  :disabled="!selectedPunch[0]"
                  color="red"
                  @click="deletePunch()"
                >
                  {{
                    $vuetify.breakpoint.xsOnly == true
                      ? 'del'
                      : 'Delete'
                  }}
                </v-btn>
              </div>
              <v-divider
                v-if="!$vuetify.breakpoint.xsOnly"
                class="mx-4"
                inset
                vertical
              ></v-divider>

              <!-- Add Session button -->
              <div>
                <v-dialog
                  v-model="newSessionDialog"
                  persistent
                  max-width="600px"
                >
                  <template v-slot:activator="{ on, attrs }">
                    <v-btn
                      color="primary"
                      v-bind="attrs"
                      v-on="on"
                      class="mx-auto"
                      >{{
                        $vuetify.breakpoint.xsOnly == true
                          ? 'add'
                          : 'new session'
                      }}</v-btn
                    >
                  </template>
                  <v-card>
                    <v-card-title>
                      <span class="headline">New Session</span>
                    </v-card-title>
                    <v-card-text>
                      <v-container>
                        <v-row>
                          <v-col cols="12" sm="6" md="6">
                            <v-select
                              v-model="newPunchUserID"
                              :items="allUserIDs"
                              item-text="username"
                              item-value="id"
                              prepend-icon="mdi-account"
                              label="User ID"
                              single-line
                            ></v-select>
                          </v-col>
                          <v-col cols="12" sm="6" md="6">
                            <v-dialog
                              ref="dateDialog"
                              v-model="dateDialog"
                              :return-value.sync="newPunchDate"
                              width="290px"
                            >
                              <template
                                v-slot:activator="{ on, attrs }"
                              >
                                <v-text-field
                                  v-model="newPunchDate"
                                  label="Date of session"
                                  prepend-icon="mdi-calendar"
                                  readonly
                                  v-bind="attrs"
                                  v-on="on"
                                ></v-text-field>
                              </template>
                              <v-date-picker
                                v-model="newPunchDate"
                                @input="
                                  $refs.dateDialog.save(newPunchDate)
                                "
                                no-title
                                scrollable
                              >
                                <v-spacer></v-spacer>
                                <v-btn
                                  text
                                  color="primary"
                                  @click="dateDialog = false"
                                  >Cancel</v-btn
                                >
                              </v-date-picker>
                            </v-dialog>
                          </v-col>

                          <v-col cols="12" sm="6" md="6">
                            <v-dialog
                              ref="timeInDialog"
                              v-model="timeInDialog"
                              :close-on-content-click="false"
                              :return-value.sync="newPunchTimeIn"
                              width="290px"
                              persistent
                            >
                              <template
                                v-slot:activator="{ on, attrs }"
                              >
                                <v-text-field
                                  v-model="newPunchTimeIn"
                                  label="Time in"
                                  prepend-icon="mdi-clock-in"
                                  readonly
                                  v-bind="attrs"
                                  v-on="on"
                                ></v-text-field>
                              </template>
                              <v-time-picker v-model="newPunchTimeIn">
                                <v-spacer></v-spacer>
                                <v-btn
                                  text
                                  color="primary"
                                  @click="timeInDialog = false"
                                  >Cancel</v-btn
                                >
                                <v-btn
                                  text
                                  color="primary"
                                  @click="
                                    $refs.timeInDialog.save(
                                      newPunchTimeIn,
                                    )
                                  "
                                  >OK</v-btn
                                >
                              </v-time-picker>
                            </v-dialog>
                          </v-col>

                          <v-col cols="12" sm="6" md="6">
                            <v-dialog
                              ref="timeOutDialog"
                              v-model="timeOutDialog"
                              :close-on-content-click="false"
                              :return-value.sync="newPunchTimeOut"
                              width="290px"
                              persistent
                            >
                              <template
                                v-slot:activator="{ on, attrs }"
                              >
                                <v-text-field
                                  :disabled="!newPunchTimeIn"
                                  v-model="newPunchTimeOut"
                                  label="Time out"
                                  prepend-icon="mdi-clock-out"
                                  readonly
                                  v-bind="attrs"
                                  v-on="on"
                                ></v-text-field>
                              </template>
                              <v-time-picker
                                :min="newPunchTimeIn"
                                v-model="newPunchTimeOut"
                              >
                                <v-spacer></v-spacer>
                                <v-btn
                                  text
                                  color="primary"
                                  @click="timeOutDialog = false"
                                  >Cancel</v-btn
                                >
                                <v-btn
                                  text
                                  color="primary"
                                  @click="
                                    $refs.timeOutDialog.save(
                                      newPunchTimeOut,
                                    )
                                  "
                                  >OK</v-btn
                                >
                              </v-time-picker>
                            </v-dialog>
                          </v-col>
                        </v-row>
                      </v-container>
                    </v-card-text>
                    <v-card-actions>
                      <v-spacer></v-spacer>
                      <v-btn
                        color="blue darken-1"
                        text
                        @click="newSessionDialog = false"
                        >Close</v-btn
                      >
                      <v-btn
                        :disabled="
                          !newPunchTimeOut ||
                            !newPunchUserID ||
                            !newPunchDate ||
                            newPunchTimeOut.localeCompare(
                              newPunchTimeIn,
                            ) < 0
                        "
                        color="blue darken-1"
                        text
                        @click="
                          newSessionDialog = false;
                          makeNewSession();
                        "
                        >Save</v-btn
                      >
                    </v-card-actions>
                  </v-card>
                </v-dialog>
              </div>
            </v-toolbar>
          </template>
        </v-data-table>
      </div>

      <!-- Amount Earned Card -->
      <div class="d-flex">
        <v-card elevation="2" class="mx-auto mt-4">
          <v-card-text class="black--text">
            <div>Amount Earned: ${{ amountEarned.toFixed(2) }}</div>
          </v-card-text>
        </v-card>
      </div>
    </v-col>
  </v-row>
</template>

<script>
import _ from 'lodash';
import moment from 'moment';

export default {
  name: 'Clock',
  computed: {
    isLoggedIn() {
      return this.$store.getters.isLoggedIn;
    },
  },
  data() {
    return {
      newPunchUserID: null,
      newPunchDate: null,
      newPunchTimeIn: null,
      newPunchTimeOut: null,
      dateDialog: false,
      timeInDialog: false,
      timeOutDialog: false,
      newSessionDialog: false,

      selectedPunch: [],
      allUserIDs: [],
      userID: '',
      punchData: [],
      users: [],
      amountEarned: 0,
      headers: [
        { text: 'In Time', value: 'intime', groupable: false },
        { text: 'Out Time', value: 'outtime', groupable: false },
        { text: 'Date', value: 'date', groupable: false },
        { text: 'Hours', value: 'hours', groupable: false },
      ],
    };
  },
  mounted() {
    this.getAllUsers();
  },
  methods: {
    async makeNewSession() {
      let inDateParse = this.newPunchDate.split('-');
      let inTimeParse = this.newPunchTimeIn.split(':');
      let outTimeParse = this.newPunchTimeOut.split(':');
      this.newPunchDate = null;
      this.newPunchTimeIn = null;
      this.newPunchTimeOut = null;
      let inTime = new Date(
        inDateParse[0],
        inDateParse[1] - 1,
        inDateParse[2],
        inTimeParse[0],
        inTimeParse[1],
      );
      let outTime = new Date(
        inDateParse[0],
        inDateParse[1] - 1,
        inDateParse[2],
        outTimeParse[0],
        outTimeParse[1],
      );
      this.$axios.post('account/adminclock/newSession', {
        userID: this.newPunchUserID,
        inTime,
        outTime,
      });
    },
    async editSession() {
      let punch = this.selectedPunch[0];
      if (typeof punch != 'undefined')
        await this.$axios.post('account/adminclock/editSession', {
          id: punch.id,
          userid: punch.userid,
          intime: punch.intime,
          outtime: punch.outtime,
          date: punch.date,
          payrate: punch.payrate,
        });
    },
    async getAllUsers() {
      let users = await this.$axios.get('account/users/getAllUsers');
      this.users = users.data.users;
      this.users.forEach(user => {
        this.allUserIDs.push({
          username: user.username,
          id: user.id,
        });
      });
    },
    async getUserSessions() {
      let sessions = await this.$axios.get(
        'account/adminClock/getUserSessions',
        {
          params: {
            userID: this.userID,
          },
        },
      );
      this.amountEarned = 0; // TODO: rework earnings mechanism
      this.punchData = [];
      _.forEach(sessions.data, session => {
        session.dateObjIn = session.intime;
        session.date = moment(session.intime).format(
          'dddd, MMM Do YYYY',
        );

        if (session.outtime) {
          session.outtime = moment(session.outtime).format(
            'hh:mm:ss A',
          );
          let computedHours = new Date(null);
          computedHours.setSeconds(session.secondsworked);
          session.hours = computedHours.toISOString().substr(11, 8);
          this.amountEarned +=
            (session.secondsworked / 3600) * session.payrate;
        }
        session.intime = moment(session.intime).format('hh:mm:ss A');
      });

      this.punchData = sessions.data;
    },
    async deletePunch() {
      let index = this.punchData.findIndex(
        s => s.id === this.selectedPunch[0].id,
      );

      await this.$axios
        .post('account/adminClock/deleteSession', {
          data: {
            punchID: this.selectedPunch[0].id,
          },
        })
        .then(
          this.punchData.splice(index, 1),
          (this.selectedPunch = []),
        );
    },
  },
};
</script>

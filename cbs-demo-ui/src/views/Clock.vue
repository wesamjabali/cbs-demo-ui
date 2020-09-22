<template>
  <v-row class="justify-center">
    <v-col cols="12">
      <div>
        <v-data-table
          :loading="tableLoading"
          :items="tableLoading ? [{}] : punchData"
          class="elevation-1"
          :headers="headers"
          item-key="id"
          :sortBy="['dateObjIn']"
          :sort-desc="[true]"
          :mobile-breakpoint="0"
          height="50vh"
          fixed-header
          hide-default-footer
          dense
        >
          <template v-slot:top>
            <v-toolbar flat color="white">
              <v-toolbar-title v-if="!$vuetify.breakpoint.xsOnly"
                >View Sessions</v-toolbar-title
              >
              <v-spacer></v-spacer>
              <v-btn
                @click="prevPayPeriod"
                :disabled="punchData.length == 0"
                color="primary"
                >Previous</v-btn
              >
              <v-divider class="mx-4" inset vertical></v-divider>
              <v-btn
                @click="nextPayPeriod"
                :disabled="payPeriodNumber == latestPayPeriod"
                color="primary"
                >Next</v-btn
              >
            </v-toolbar>
          </template>
        </v-data-table>
      </div>
      <div class="d-flex mt-4">
        <v-btn
          :width="$vuetify.breakpoint.xsOnly ? '50%' : '200px'"
          @click="punchClock()"
          x-large
          :loading="buttonsLoading"
          :disabled="buttonsLoading"
          :color="clockStatus == 'out' ? 'green' : 'red'"
          class="mx-auto"
        >
          {{ clockStatus === 'out' ? 'Clock In' : 'Clock Out' }}
        </v-btn>
      </div>

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
    payPeriodEnd() {
      return moment(this.payPeriodBegin)
        .add(2, 'weeks')
        .subtract(1, 'second')
        .toDate();
    },
  },
  data() {
    return {
      clockStatus: '',
      punchData: [],
      tableLoading: true,
      buttonsLoading: true,
      originPayPeriodBegin: new Date('September 5, 2020 00:00:00'),
      payPeriodBegin: null,
      payPeriodNumber: 0,
      amountEarned: 0,
      latestPayPeriod: 0,
      headers: [
        { text: 'In Time', value: 'intime', groupable: false },
        { text: 'Out Time', value: 'outtime', groupable: false },
        { text: 'Date', value: 'date', groupable: false },
        { text: 'Hours', value: 'hours', groupable: false },
      ],
    };
  },
  mounted() {
    this.getClockStatus();
    this.getCurrentPayPeriod();
  },
  watch: {
    async payPeriodEnd() {
      await this.getSessions();
    },
  },
  methods: {
    async getCurrentPayPeriod() {
      this.payPeriodBegin = moment(
        this.originPayPeriodBegin,
      ).toDate();
      let now = moment();
      this.payPeriodNumber = Math.floor(
        moment
          .duration(now.diff(moment(this.originPayPeriodBegin)))
          .asWeeks() / 2,
      );
      this.latestPayPeriod = this.payPeriodNumber;

      this.goToPayPeriod(this.latestPayPeriod);

      this.tableLoading = false;
    },
    nextPayPeriod() {
      this.payPeriodNumber++;
      this.payPeriodBegin = moment(this.originPayPeriodBegin)
        .add(2 * this.payPeriodNumber, 'weeks')
        .toDate();
    },
    prevPayPeriod() {
      this.payPeriodNumber--;
      this.payPeriodBegin = moment(this.originPayPeriodBegin)
        .add(2 * this.payPeriodNumber, 'weeks')
        .toDate();
    },
    goToPayPeriod(goto = this.latestPayPeriod) {
      this.payPeriodNumber = goto;
      this.payPeriodBegin = moment(this.originPayPeriodBegin)
        .add(2 * this.payPeriodNumber, 'weeks')
        .toDate();
    },
    async getSessions() {
      const sessions = await this.$axios.get(
        'account/clock/getSessions',
        {
          params: {
            payPeriodBegin: this.payPeriodBegin,
            payPeriodEnd: this.payPeriodEnd,
          },
        },
      );
      this.amountEarned = 0;
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
    async getClockStatus() {
      try {
        const { data = {} } = await this.$axios.get(
          '/account/clock/status',
        );
        const clockStatus = data;
        this.clockStatus = clockStatus;
      } catch (error) {
        console.log('getClockStatus -> error', error);
      }
      this.buttonsLoading = false;
    },

    async punchClock() {
      this.buttonsLoading = true;

      try {
        await this.$axios.post('/account/clock/punch');
        this.getClockStatus();
      } catch (error) {
        console.log('punchClock -> error', error);
      }
      this.getCurrentPayPeriod();
    },
  },
};
</script>

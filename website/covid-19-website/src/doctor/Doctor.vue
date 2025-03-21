<template>
  <div class="doctor">
    <!-- ChatBox modal -->
    <v-dialog v-model="chatbox_modal" width="350px">
      <Chatbox />
    </v-dialog>
    <!-- end of ChatBox modal -->
    <v-row>
      <!-- date modal start -->
      <v-dialog v-model="date_dialog" width="600px">
        <v-card>
          <v-container>
            <v-row>
              <v-col cols="12" class="pa-5">
                <h1>Book an Appointment</h1>
                <v-date-picker
                  v-model="date"
                  :allowed-dates="allowedDates"
                  full-width
                  @change="onDateClick()"
                ></v-date-picker>
              </v-col>
              <!-- list -->
              <v-col cols="12">
                <v-autocomplete
                  :items="allAvailabilities"
                  v-model="available"
                  multiple
                  label="select "
                  small-chips
                  clearable
                  deletable-chips
                />
              </v-col>
              <v-col cols="6">
                <v-btn
                  color="error"
                  block
                  @click="date_dialog = false"
                  elevation="0"
                >
                  cancel
                </v-btn>
              </v-col>
              <v-col cols="6">
                <v-btn
                  block
                  color="success"
                  elevation="0"
                  @click="setAvailabilities()"
                >
                  Set
                </v-btn>
              </v-col>
            </v-row>
          </v-container>
        </v-card>
      </v-dialog>
      <!-- date modal end -->

      <v-col cols="12" md="6">
        <div>
          <v-btn
            class="white--text"
            style="font-size: 18px"
            color="blue lighten-2"
            block
            height="75px"
            @click="onPatientsClick()"
            >List Of Patients</v-btn
          >
        </div>
        <div class="my-6">
          <v-btn
            class="white--text"
            style="font-size: 18px"
            color="blue lighten-2"
            block
            height="75px"
            @click="listOfAppointments()"
            >List of Appointments</v-btn
          >
        </div>
        <div class="my-6">
          <v-btn
            class="white--text"
            style="font-size: 18px"
            color="blue lighten-2"
            block
            height="75px"
            @click="date_dialog = !date_dialog"
            >Update Availabilities</v-btn
          >
        </div>

        <apexchart :options="chartOptions" :series="series"></apexchart>
      </v-col>
      <v-col cols="12" md="6">
        <div
          style="background-color: rgba(256, 256, 256, 0.9)"
          class="pa-4 rounded-lg"
        >
          <h2>Availabilites:</h2>
          <div v-for="(item, i) in fetchAvailabilities" :key="i">
            {{
              item.SpecificDay.substr(0, 10) +
              " -- " +
              item.StartTime +
              "-" +
              item.EndTime
            }}

            <v-btn x-small @click="deleteAvailability(item)"> Delete </v-btn>
          </div>
        </div>
        <div style="margin-top: 50%; margin-left: 75%">
          <!-- <v-badge v-if="" class="mx-6" color="red" content="7" overlap> -->
          <v-btn @click="onChatBoxClick()" icon height="80px" width="80px">
            <v-icon color="blue darken-3" style="font-size: 80px">
              mdi-message-text
            </v-icon>
          </v-btn>
          <!-- </v-badge> -->
        </div>
      </v-col>
    </v-row>
  </div>
</template>

<script>
import moment from "moment";
import axios from "axios";
import Chatbox from "../components/ChatBox.vue";

import Swal from "sweetalert2";
export default {
  name: "Doctor",

  components: { Chatbox },

  data: function () {
    return {
      series: [44, 55],
      chartOptions: {
        chart: {
          width: 400,
          type: "pie",
        },
        labels: ["Infected", "Non-Infected"],
        responsive: [
          {
            breakpoint: 580,
            options: {
              chart: {
                width: 300,
              },
              legend: {
                position: "bottom",
              },
            },
          },
        ],
      },
      appointments: [],
      date_dialog: false,
      available: [],
      date: new Date(Date.now() - new Date().getTimezoneOffset() * 60000)
        .toISOString()
        .substr(0, 10),
      allAvailabilities: [],
      fetchAvailabilities: [],
      chatbox_modal: false,
    };
  },

  mounted() {
    this.init();
    this.$emit("img", "doctor");
  },

  methods: {
    chatBox() {
      this.chatbox_modal = !this.chatbox_modal;
    },
    init() {
      this.getMessages();
      this.getAvailabilities();
      this.listOfAvailabilities();
      this.getChartData();
    },

    // Get infected and non infected data
    async getChartData() {
      try {
        const DID = this.$store.state.user.UserID;
        // Get all users assigned to this doctor
        const response = await axios.post(`http://localhost:5000/users`, {
          Doctor: DID,
        });
        const patientList = response.data;
        let infectedCount = 0;
        let nonInfectedCount = 0;
        let totalCount = 0;
        let patientIDs = [];
        // Make a list of their IDs
        patientList.forEach((patient) => {
          ++totalCount;
          patientIDs.push(patient.UserID);
        });
        // For each ID get their latest health status and check if they have covid and calculate count
        for (let i = 0; i < patientIDs.length; ++i) {
          const latestHSResponse = await axios.get(
            `http://localhost:5000/healthstatus/${patientIDs[i]}`
          );
          console.log(JSON.stringify(latestHSResponse.data));
          const infected = latestHSResponse.data.Covid == 1;
          if (infected) {
            ++infectedCount;
          } else {
            ++nonInfectedCount;
          }
        }
        // Write the data to series
        this.series = [
          (infectedCount / totalCount) * 100,
          (nonInfectedCount / totalCount) * 100,
        ];
      } catch (err) {
        console.log(err);
      }
    },

    // Get all messages to and from this doctor's botchat
    async getMessages() {
      try {
        const ct = true;
        const DID = this.$store.state.user.UserID;
        let messages = [];
        let assignedPatients = [];
        while (ct) {
          // Get patients assigned to this doctor
          const assignedPatientsResponse = await axios.post(
            `http://localhost:5000/users`,
            { Doctor: DID }
          );
          assignedPatients = assignedPatientsResponse.data;
          // Get all messages sent to and from the users
          for (let i = 0; i < assignedPatients.length; ++i) {
            const messagesResponse = await axios.post(
              `http://localhost:5000/messages/${DID}/${assignedPatients[i].UserID}`
            );
            messages = messagesResponse.data;
            if (messages && messages.length > 0) {
              // Check if latest is read or not
              if (
                messages[messages.length - 1].ReceiveUserID ==
                  this.$store.state.user.UserID &&
                messages[messages.length - 1].State == "Sent"
              ) {
                // TODO: Show that there is new message to read
              }
            }
          }

          // Wait 2s before checking for new messages
          await new Promise((r) => setTimeout(r, 2000));
        }
      } catch (err) {
        console.log(err);
      }
    },

    // Get list of availabilities
    listOfAvailabilities() {
      this.allAvailabilities = [];
      var availabilities = [
        "09:00:00-10:00:00",
        "10:00:00-11:00:00",
        "11:00:00-12:00:00",
        "12:00:00-13:00:00",
        "13:00:00-14:00:00",
        "14:00:00-15:00:00",
        "15:00:00-16:00:00",
        "16:00:00-17:00:00",
      ];
      var dateTimes = [];
      for (var i = 0; i < this.fetchAvailabilities.length; i++) {
        const time =
          this.fetchAvailabilities[i].StartTime.toString() +
          "-" +
          this.fetchAvailabilities[i].EndTime.toString();
        if (
          this.date.substr(0, 10) ==
          this.fetchAvailabilities[i].SpecificDay.substr(0, 10)
        ) {
          dateTimes.push(time);
        }
      }
      for (let j = 0; j < availabilities.length; j++) {
        if (!dateTimes.includes(availabilities[j])) {
          this.allAvailabilities.push(availabilities[j]);
        }
      }
    },

    // Delete a doctor's availability
    deleteAvailability(item) {
      this.removeAvailability(
        item.DayOfWeek,
        item.StartTime,
        item.EndTime,
        item.SpecificDay.substr(0, 10)
      );
      this.init();
    },

    async addNotif(Message) {
      let d = new Date();
      let Time = d.toTimeString().split(" ")[0];
      const params = {
        Message,
        Recipient: this.userId,
        Read: 0,
        Time,
      };
      try {
        await axios.post(this.url + "notification", params);
        // alert("statuse added successfully");
        this.getStatuses();
        this.status_dialoge = false;
      } catch (err) {
        console.log("err", err);
        // alert("Failed ; add new status");
      }
    },

    // Click on a date to get availabilities
    onDateClick() {
      this.available = [];
      this.listOfAvailabilities();
    },

    //Set doctor's availabilities
    setAvailabilities() {
      for (var i = 0; i < this.available.length; i++) {
        var times = this.available[i].split("-");
        this.addAvailability(
          moment(this.date).format("dddd"),
          times[0],
          times[1],
          this.date
        );
      }
      this.getAvailabilities();
      this.listOfAvailabilities();
      this.date_dialog = false;
    },
    disAvail(item) {
      const found = this.appointments.findIndex((a) => {
        return a.Time == item.StartTime && a.Date == item.SpecificDay;
      });
      if (found > -1) return true;
      return false;
    },
    allowedDates() {
      return true;
    },
    listOfAppointments() {
      this.$router.push("/list-of-appointments");
    },

    // Get availabilities
    async getAvailabilities() {
      try {
        const res = await axios.get(
          `http://localhost:5000/availability/${this.$store.state.user.UserID}`
        );
        this.fetchAvailabilities = res.data;
      } catch (err) {
        console.log(err);
      }
    },

    // Add availability
    async addAvailability(DayOfWeek, StartTime, EndTime, SpecificDay) {
      try {
        await axios.post(`http://localhost:5000/availability`, {
          DID: this.$store.state.user.UserID,
          DayOfWeek: DayOfWeek,
          StartTime: StartTime,
          EndTime: EndTime,
          SpecificDay: SpecificDay,
        });
        Swal.fire({
          icon: "success",
          title: "success",
          text: "added successfully",
        });
      } catch (err) {
        Swal.fire({
          icon: "error",
          title: "error",
          text: "failed add availability",
        });
      }
    },

    // Remove availability
    async removeAvailability(DayOfWeek, StartTime, EndTime, SpecificDay) {
      try {
        await axios.post(`http://localhost:5000/deleteavailability`, {
          DID: this.$store.state.user.UserID,
          DayOfWeek: DayOfWeek,
          StartTime: StartTime,
          EndTime: EndTime,
          SpecificDay: SpecificDay,
        });
      } catch (err) {
        console.log(err);
      }
    },

    // Open patients list
    onPatientsClick() {
      this.$router.push("/doctor-patients-list");
    },
    onChatBoxClick() {
      this.$router.push("/doctor-patients-list");
    },
  },
};
</script>

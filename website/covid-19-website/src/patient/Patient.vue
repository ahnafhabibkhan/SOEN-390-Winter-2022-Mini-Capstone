<template>
  <v-container>
    <!-- ChatBox modal -->
    <v-dialog v-model="chatbox_modal" width="350px">
      <Chatbox />
    </v-dialog>

    <v-row>
      <!-- book modal start -->
      <v-dialog v-model="date_dialoge" width="500px">
        <v-card>
          <v-container>
            <ValidationObserver v-slot="{ handleSubmit }" ref="observer">
              <v-row>
                <v-col cols="12" class="pa-1">
                  <h2>Book an Appointment</h2>
                  <v-date-picker v-model="date" full-width></v-date-picker>
                </v-col>
                <!-- list -->
                <!-- if filteredAvailability exists this will displayed -->
                <v-col
                  cols="12"
                  class="py-0"
                  v-if="filteredAvailability.length > 0"
                >
                  select a date
                  <ValidationProvider rules="required" v-slot="{ errors }">
                    <v-radio-group v-model="available" dense hide-details>
                      <v-radio
                        v-for="(item, i) in filteredAvailability"
                        :key="i"
                        :color="disAvail(item) ? 'error' : ''"
                        :disabled="disAvail(item)"
                        :hide-details="errors.lenght == 0"
                        :error-messages="errors[0]"
                        :class="{
                          'text-decoration-line-through red--text':
                            disAvail(item),
                        }"
                        :label="
                          item.SpecificDay.substr(0, 10) +
                          ' -- ' +
                          item.StartTime +
                          ' - ' +
                          item.EndTime
                        "
                        :value="item"
                      ></v-radio>
                    </v-radio-group>
                  </ValidationProvider>
                </v-col>
                <v-col cols="12" class="py-0">
                  <h4 class="pt-1 my-0 pb-0">Level of Emergency:</h4>
                  <v-radio-group
                    v-model="emergencyLevel"
                    row
                    dense
                    :hide-details="true"
                  >
                    <v-radio label="High" value="High"> </v-radio>
                    <v-radio label="Medium" value="Medium"> </v-radio>
                    <v-radio label="Low" value="Low"> </v-radio>
                  </v-radio-group>
                </v-col>

                <v-col cols="6">
                  <v-btn
                    color="error"
                    block
                    @click="date_dialoge = false"
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
                    :disabled="!available"
                    @click="handleSubmit(bookAppoitment)"
                  >
                    book
                  </v-btn>
                </v-col>
              </v-row>
            </ValidationObserver>
          </v-container>
        </v-card>
      </v-dialog>

      <!-- book modal end -->

      <!-- status modal start -->
      <v-dialog v-model="status_dialoge" width="500px" v-if="status_dialoge">
        <v-card>
          <v-container>
            <!-- wrap all from with validation obs -->
            <ValidationObserver v-slot="{ handleSubmit }" ref="observer">
              <v-row>
                <v-col cols="12" class="pa-5">
                  <h1>update status</h1>
                  <!-- <v-switch
                  v-model="form.i_have_covid"
                  label="I have covid 19"
                ></v-switch> -->
                  <div>
                    <v-row>
                      <v-col cols="6">
                        <!-- wrap input with validaion comp -->
                        <ValidationProvider
                          rules="required|double"
                          v-slot="{ errors }"
                        >
                          <v-text-field
                            name="temp"
                            label="Temperature :"
                            filled
                            flat
                            v-model.number="form.Temperature"
                            dense
                            :hide-details="errors.lenght == 0"
                            :error-messages="errors[0]"
                          >
                            <template slot="append">C&deg;</template>
                          </v-text-field>
                        </ValidationProvider>
                      </v-col>
                      <v-col cols="6">
                        <ValidationProvider
                          rules="required|double"
                          v-slot="{ errors }"
                        >
                          <v-text-field
                            label="Weight :"
                            filled
                            flat
                            v-model.number="form.Weight"
                            dense
                            name="weight"
                            :hide-details="errors.lenght == 0"
                            :error-messages="errors[0]"
                          >
                            <template slot="append">Kg</template>
                          </v-text-field>
                        </ValidationProvider>
                      </v-col>
                      <v-col cols="6">
                        <v-checkbox
                          dense
                          hide-details
                          label="Headache"
                          :true-value="1"
                          :false-value="0"
                          v-model.number="form.Headache"
                        ></v-checkbox
                      ></v-col>
                      <v-col cols="6">
                        <v-checkbox
                          dense
                          hide-details
                          label="covid"
                          :true-value="1"
                          :false-value="0"
                          v-model.number="form.Covid"
                        ></v-checkbox
                      ></v-col>
                      <v-col cols="6">
                        <v-checkbox
                          dense
                          hide-details
                          label="BreathingIssues"
                          :true-value="1"
                          :false-value="0"
                          v-model.number="form.BreathingIssues"
                        ></v-checkbox
                      ></v-col>
                      <v-col cols="6">
                        <v-checkbox
                          dense
                          hide-details
                          label="Cough"
                          :true-value="1"
                          :false-value="0"
                          v-model.number="form.Cough"
                        ></v-checkbox
                      ></v-col>
                      <v-col cols="6">
                        <v-checkbox
                          label="LostTasteSmell"
                          dense
                          hide-details
                          v-model="form.LostTasteSmell"
                        >
                        </v-checkbox
                      ></v-col>
                      <v-col cols="6">
                        <v-checkbox
                          dense
                          hide-details
                          label="MusclePain"
                          :true-value="1"
                          :false-value="0"
                          v-model.number="form.MusclePain"
                        ></v-checkbox
                      ></v-col>
                      <v-col cols="6">
                        <v-checkbox
                          dense
                          hide-details
                          label="Diarrhea"
                          :true-value="1"
                          :false-value="0"
                          v-model.number="form.Diarrhea"
                        ></v-checkbox
                      ></v-col>
                      <v-col cols="6">
                        <v-checkbox
                          dense
                          hide-details
                          label="Vomitting"
                          :true-value="1"
                          :false-value="0"
                          v-model.number="form.Vomitting"
                        ></v-checkbox
                      ></v-col>
                      <v-col cols="6">
                        <v-checkbox
                          dense
                          hide-details
                          label="Nausea"
                          :true-value="1"
                          :false-value="0"
                          v-model.number="form.Nausea"
                        ></v-checkbox
                      ></v-col>
                      <v-col cols="6">
                        <v-checkbox
                          dense
                          hide-details
                          label="SoreThroat"
                          :true-value="1"
                          :false-value="0"
                          v-model.number="form.SoreThroat"
                        ></v-checkbox
                      ></v-col>
                    </v-row>
                  </div>
                </v-col>
                <v-col cols="12">
                  <ValidationProvider rules="required" v-slot="{ errors }">
                    <v-textarea
                      rows="3"
                      v-model="form.SympDescription"
                      label="SympDescription :"
                      :error-messages="errors[0]"
                      :hide-details="errors.length == 0"
                    >
                    </v-textarea>
                  </ValidationProvider>
                </v-col>

                <v-col cols="6">
                  <v-btn
                    type="submit"
                    block
                    color="success"
                    elevation="0"
                    @click="handleSubmit(addStatus)"
                    :disabled="edit_mode"
                  >
                    <v-icon left> mdi-plus </v-icon>
                    add
                  </v-btn>
                </v-col>
                <v-col cols="6">
                  <v-btn
                    type="submit"
                    block
                    color="success"
                    elevation="0"
                    @click="handleSubmit(updateStatus)"
                    :disabled="!edit_mode"
                  >
                    update
                  </v-btn>
                </v-col>
              </v-row>
            </ValidationObserver>
          </v-container>
        </v-card>
      </v-dialog>
      <!-- status modal end -->
      <v-col cols="12" md="6">
        <!-- buttons -->

        <div class="my-2 mx-3">
          <v-btn dark color="blue lighten-2" block large @click="myInfo"
            >My Info</v-btn
          >
        </div>
        <div class="mx-3">
          <v-btn dark color="blue lighten-2" block large @click="onAppointment"
            >Book an Appointment</v-btn
          >
        </div>
        <div class="my-2 mx-3">
          <v-btn dark color="blue lighten-2" block large @click="onUpdateStatus"
            >Update Status</v-btn
          >
        </div>

        <!-- status rows -->
        <v-row>
          <v-col v-if="isOverDue">
            <v-alert type="error" dense>
              Your updated status is overdue!
            </v-alert>
          </v-col>
          <v-col cols="12" class="my-0">
            <v-expansion-panels>
              <v-expansion-panel
                d
                v-for="(item, i) in statusesFiltered"
                :key="i"
              >
                <v-expansion-panel-header>
                  <span> Date: {{ item.fillOutDate.substr(0, 10) }} </span>
                  <span>
                    Updated Time: {{ item.lastUpdateTime.substr(0, 10) }}
                  </span>
                  <v-checkbox
                    label="Covid"
                    dense
                    readonly
                    hide-details
                    v-model="item.Covid"
                  >
                  </v-checkbox>

                  <v-btn @click="onEdit(item)" icon color="primary" small>
                    <v-icon> mdi-pencil </v-icon>
                  </v-btn>
                  <v-btn @click="deleteStatus(item)" icon color="error" small>
                    <v-icon> mdi-close </v-icon>
                  </v-btn>
                </v-expansion-panel-header>
                <v-expansion-panel-content>
                  <div
                    class="d-flex align-center justify-space-between flex-wrap"
                  >
                    <span> Temperature: {{ item.Temperature }} </span>
                    <span> Weight: {{ item.Weight }} </span>

                    <v-checkbox
                      label="BreathingIssues"
                      dense
                      hide-details
                      readonly
                      v-model="item.BreathingIssues"
                    >
                    </v-checkbox>
                    <v-checkbox
                      label="Cough"
                      dense
                      hide-details
                      readonly
                      v-model="item.Cough"
                    >
                    </v-checkbox>

                    <v-checkbox
                      label="Diarrhea"
                      dense
                      hide-details
                      readonly
                      v-model="item.Diarrhea"
                    >
                    </v-checkbox>
                    <v-checkbox
                      label="Headache"
                      dense
                      hide-details
                      readonly
                      v-model="item.Headache"
                    >
                    </v-checkbox>
                    <v-checkbox
                      label="LostTasteSmell"
                      dense
                      hide-details
                      readonly
                      v-model="item.LostTasteSmell"
                    >
                    </v-checkbox>
                    <v-checkbox
                      label="MusclePain"
                      dense
                      hide-details
                      readonly
                      v-model="item.MusclePain"
                    >
                    </v-checkbox>
                    <v-checkbox
                      label="SoreThroat"
                      dense
                      hide-details
                      readonly
                      v-model="item.SoreThroat"
                    />
                    <v-checkbox
                      label="Vomitting"
                      dense
                      hide-details
                      readonly
                      v-model="item.Vomitting"
                    >
                    </v-checkbox>
                    <v-checkbox
                      label="Nausea"
                      dense
                      hide-details
                      readonly
                      v-model="item.Nausea"
                    >
                    </v-checkbox>
                  </div>
                  <div>SympDescription: {{ item.SympDescription }}</div>
                </v-expansion-panel-content>
              </v-expansion-panel>
            </v-expansion-panels>
          </v-col>
          <v-col cols="12" v-if="statuses.length > 3">
            <v-btn
              elevation="0"
              text
              @click="show_more = !show_more"
              block
              small
            >
              {{ show_more ? "show less" : "show more" }}
            </v-btn>
          </v-col>
        </v-row>
      </v-col>

      <v-col cols="12" md="6" class="p2-6 mt-2">
        <!-- requested apppointments -->
        <div
          style="background-color: rgba(256, 256, 256, 0.5)"
          class="pa-4 rounded-lg mb-5"
        >
          <h2>Requested Appointments :</h2>
          <div v-for="(item, i) in appointments" :key="i">
            {{ item.Date.substr(0, 10) + " -- " + item.Time }}

            <v-btn x-small @click="deleteAppointment(item)"> cancel </v-btn>
          </div>
        </div>
        <!-- approved apppointments -->
        <div
          style="background-color: rgba(256, 256, 256, 0.5)"
          class="pa-4 rounded-lg mt-5"
        >
          <h2>Approved Appointments :</h2>
          <div v-for="(item, i) in approved" :key="i">
            {{ item.Date.substr(0, 10) + " -- " + item.Time }}

            <v-btn x-small @click="deleteApproved(item)"> cancel </v-btn>
          </div>
        </div>
        <!-- Doctor's Requested apppointments -->
        <div
          style="background-color: rgba(256, 256, 256, 0.5)"
          class="pa-4 rounded-lg mt-5"
        >
          <h2>Doctor's Requested Appointments:</h2>
          <div v-for="(item, i) in doctorRequestedAppointments" :key="i">
            {{ item.Date.substr(0, 10) + " -- " + item.Time }}

            <v-btn x-small @click="approveRequested(item)"> Approve </v-btn>
            <v-btn class="mx-2" x-small @click="deleteAppointment(item)">
              Refuse
            </v-btn>
          </div>
        </div>
      </v-col>
    </v-row>
    <!-- To open the chatbox but also to mark red if there is a notification  -->
    <div class="chatbox-css">
      <v-badge :value="flag" class="mx-6" color="red" overlap>
        <v-btn @click="openChatBoxModal()" icon height="80px" width="80px">
          <v-icon color="blue darken-3" style="font-size: 80px">
            mdi-message-text
          </v-icon>
        </v-btn>
      </v-badge>
    </div>
  </v-container>
</template>
<script>
import axios from "axios";
import Swal from "sweetalert2";
import Chatbox from "../components/ChatBox.vue";
//import validation
import { ValidationProvider, extend, ValidationObserver } from "vee-validate";
// import needed rules for validate
import { required, numeric, double } from "vee-validate/dist/rules";
// extend  rules
extend("required", {
  ...required,
  message: "This field is required!",
});
extend("numeric", {
  ...numeric,
  message: "Please enter a number",
});
extend("double", {
  ...double,
  message: "Please enter a number",
});
// end
export default {
  name: "Patient",
  // register as component
  components: { ValidationProvider, ValidationObserver, Chatbox },
  data() {
    return {
      url: "http://localhost:5000/",
      edit_mode: false,
      show_more: false,
      date_dialoge: false,
      status_dialoge: false,
      date: new Date(Date.now() - new Date().getTimezoneOffset() * 60000)
        .toISOString()
        .substr(0, 10),
      statuses: [],
      form_default: {
        fillOutDate: "2022-03-17",
        lastUpdateTime: "15:30:30",
        Weight: null,
        SympDescription: null,
        Temperature: null,
        BreathingIssues: 0,
        Cough: 0,
        Covid: 0,
        LostTasteSmell: 0,
        MusclePain: 0,
        Diarrhea: 0,
        Vomitting: 0,
        Nausea: 0,
        Headache: 0,
        SoreThroat: 0,
      },
      appointments: [],
      form: null,
      availabilities: [],
      available: null,
      emergencyLevel: "Low",
      approved: [],
      doctorRequestedAppointments: [],
      chatbox_modal: false,
      notification: false,
      flag: false,
    };
  },
  created() {
    this.getMessages();
    this.notification;
  },

  methods: {
    openChatBoxModal() {
      this.chatbox_modal = !this.chatbox_modal;
      //console.log(event);
      this.notification = true;
    },
    disAvail(item) {
      const found = this.appointments.findIndex((a) => {
        return a.Time == item.StartTime && a.Date == item.SpecificDay;
      });
      if (found > -1) return true;
      return false;
    },
    onEdit(item) {
      this.edit_mode = true;
      this.status_dialoge = true;
      this.form = Object.assign({}, item);
    },
    allowedDates() {
      return true;
    },
    async onUpdateStatus() {
      // first we check if today status exist or not
      let d = new Date();
      const today = d.toISOString().split("T")[0];
      const i = this.statuses.findIndex((item) => {
        return item.fillOutDate.split("T")[0] === today;
      });
      // if exist this alert displayed
      if (i > -1) {
        Swal.fire({
          icon: "error",
          title: "error",
          text: "today status exist already , You can edit it",
        });
        return;
      }
      this.form = Object.assign({}, this.form_default);
      this.status_dialoge = !this.status_dialoge;
    },
    myInfo() {
      this.$router.push("/profile");
    },
    async getStatuses() {
      try {
        console.log("request fired");
        const res = await axios.get(this.url + `healthstatuses/${this.userId}`);
        console.log(res);
        this.statuses = res.data;
      } catch (err) {
        // alert("error ; get statuses");
        console.log("err", err);
      }
    },
    async addStatus() {
      let d = new Date();
      this.form.fillOutDate = d.toISOString().split("T")[0];
      this.form.lastUpdateTime = d.toLocaleTimeString().substr(0, 8);
      try {
        this.form.PID = this.userId;
        await axios.post(this.url + "healthstatus", this.form);
        // alert("statuse added successfully");
        this.getStatuses();
        this.status_dialoge = false;
      } catch (err) {
        console.log("err", err);
        // alert("Failed ; add new status");
      }
    },
    async addNotif(Message, Recipient) {
      let d = new Date();
      let Time = d.toTimeString().split(" ")[0];
      const params = {
        Message,
        Recipient,
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
    async updateStatus() {
      const PID = this.userId;
      const date = this.form.fillOutDate.substr(0, 10);
      this.form.fillOutDate = this.form.fillOutDate.substr(0, 10);
      try {
        await axios.put(this.url + `healthstatus/${PID}/${date}`, this.form);
        // alert("statuse updated successfully");
        this.status_dialoge = false;
        this.edit_mode = false;
        this.getStatuses();
      } catch (err) {
        console.log("err", err);
        // alert("Failed ; add new status");
      }
    },
    async deleteStatus(status) {
      Swal.fire({
        title: "Are you sure?",
        text: "You won't be able to revert this!",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        confirmButtonText: "Yes, delete it!",
      })
        .then((result) => {
          if (result.isConfirmed) {
            // del status
            try {
              const id = this.userId;
              status.PID = id;
              axios.delete(
                this.url +
                  `healthstatus/${id}/${status.fillOutDate.split("T")[0]}`
              );
            } catch (err) {
              console.log("err", err);
            }
            Swal.fire("Deleted!", "Your file has been deleted.", "success");
          }
        })
        .then(() => {
          this.getStatuses();
        });
    },
    async bookAppoitment() {
      const did = this.doctorId;
      const pid = this.userId;
      const covidStatusInt = await axios.get(
        `http://localhost:5000/healthstatus/${pid}`
      );
      var levelOfEmergency = 0;
      if (this.emergencyLevel == "High") {
        levelOfEmergency = 2;
      } else if (this.emergencyLevel == "Medium") {
        levelOfEmergency = 1;
      }
      let params = {
        DID: did,
        PID: pid,
        Time: this.available.StartTime,
        Date: this.available.SpecificDay.substr(0, 10),
        RequestedBy: "P",
        LevelOfEmergency: levelOfEmergency,
        Priority: covidStatusInt.data.Covid,
      };
      try {
        await axios.post(this.url + "appointmentrequest", params);
        this.addNotif(
          `an appointment [${params.Date + "-" + params.Time}] is requested`,
          this.userId
        );
        this.addNotif(
          `an appointment [${params.Date + "-" + params.Time}] is requested`,
          did
        );
        Swal.fire({
          icon: "success",
          title: "success",
          text: "The Appointment Request Booked Successfully*",
        });
        this.available = null;
        this.date_dialoge = false;
        this.getAppointments();
      } catch (err) {
        console.log("err", err);
        alert("Failed ; book appointment");
      }
    },
    async deleteAppointment(item) {
      Swal.fire({
        title: "Are you sure?",
        text: "delete appointment ?",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        confirmButtonText: "Yes, delete it!",
      })
        .then((result) => {
          if (result.isConfirmed) {
            item.Date = item.Date.substr(0, 10);
            delete item.RequestedBy;
            try {
              axios.post(this.url + "deleteappointmentrequest", item);

              // window.location.reload();
              this.addNotif(
                `an appointment[${item.Date}] deleted`,
                this.userId
              );
            } catch (err) {
              console.log("err", err);
              alert("Failed ; delete appointment");
            }
            Swal.fire("Deleted!", "Your file has been deleted.", "success");
          }
        })
        .then(() => {
          this.getAppointments();
          this.getDoctorAppointmentRequests();
        });
    },
    async deleteApproved(item) {
      Swal.fire({
        title: "Are you sure?",
        text: "You won't be able to revert this!",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        confirmButtonText: "Yes, delete it!",
      }).then((result) => {
        if (result.isConfirmed) {
          item.Date = item.Date.substr(0, 10);
          delete item.RequestedBy;
          try {
            axios.post(this.url + "deleteappointment", item);
            this.addNotif(
              `an approver appointment[${item.Date}] was deleted`,
              item.DID
            );
            this.getApproved();
          } catch (err) {
            console.log("err", err);
            alert("Failed ; delete appointment");
          }
          Swal.fire("Deleted!", "Your file has been deleted.", "success");
        }
      });
    },
    async getAppointments() {
      const id = this.userId;
      try {
        const res = await axios.post(this.url + `appointmentrequests`, {
          PID: id,
        });
        this.appointments = res.data;
      } catch (err) {
        console.log("err", err);
        alert("Failed ; get appointment");
      }
    },
    async getApproved() {
      const id = this.userId;
      try {
        const res = await axios.post(this.url + `appointments`, {
          PID: id,
        });
        this.approved = res.data;
      } catch (err) {
        console.log("err", err);
        alert("Failed ; get appointment");
      }
    },
    async getAvailabilities() {
      const id = this.doctorId;
      try {
        const res = await axios.get(this.url + `availability/${id}`);
        this.availabilities = res.data;
      } catch (err) {
        console.log("err", err);
        alert("Failed ; get appointment");
      }
    },
    async onAppointment() {
      await this.getAvailabilities();
      this.date_dialoge = true;
    },

    // Get appointment requests made by this doctor
    async getDoctorAppointmentRequests() {
      try {
        const res = await axios.post(
          `http://localhost:5000/appointmentrequests`,
          {
            DID: this.doctorId,
            PID: this.userId,
            RequestedBy: "D",
          }
        );
        this.doctorRequestedAppointments = res.data;
        // we send recived data to store to be saved there
        // this.$store.commit('setNotifs',res.data)
        // this.$store.commit('setNotifs',['notif1','notif2'])
      } catch (err) {
        console.log(err);
      }
    },
    //Accept requested appointment
    approveRequested(item) {
      Swal.fire({
        title: "Are you sure?",
        text: "Approve this appointment?",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        confirmButtonText: "Yes, Approve it!",
      }).then((result) => {
        if (result.isConfirmed) {
          this.approveAppointment(
            item.PID,
            item.DID,
            item.Date.substr(0, 10),
            item.Time,
            item.LevelOfEmergency,
            item.Priority
          );
          // window.location.reload();
          this.init();
        }
      });
    },
    // Approve an appointment
    async approveAppointment(PID, DID, Date, Time, LevelOfEmergency, Priority) {
      try {
        this.cancelAppointmentRequest(PID, DID, Date, Time);
        await axios.post(`http://localhost:5000/appointment`, {
          PID: PID,
          DID: DID,
          Date: Date,
          Time: Time,
          LevelOfEmergency: LevelOfEmergency,
          Priority: Priority,
        });
        this.addNotif(
          `an appoitment on [${Date + "-" + Time}]approved by patient`,
          DID
        );
      } catch (err) {
        console.log(err);
      }
    },
    // Cancel an appointment from the doctor
    async cancelAppointmentRequest(PID, DID, Date, Time) {
      try {
        await axios.post(`http://localhost:5000/deleteappointmentrequest`, {
          PID: PID,
          DID: DID,
          Date: Date,
          Time: Time,
        });
        this.addNotif(
          `the appointment[${Date + "-" + Time}] reuest was deleted`,
          DID
        );
      } catch (err) {
        console.log(err);
      }
    },
    async getMessages() {
      // updating the messages in patient side
      try {
        const ct = true;
        var count = 0;
        while (ct) {
          const userResponse = await axios.post(`http://localhost:5000/users`, {
            UserID: this.$store.state.user.UserID,
          });

          // Getting users ID
          const user = userResponse.data[0];
          let idToUse = user.Doctor;
          console.log(
            "Getting messages between " +
              this.$store.state.user.UserID +
              " and " +
              idToUse
          );
          const messagesResponse = await axios.get(
            `http://localhost:5000/messages/${this.$store.state.user.UserID}/${idToUse}`
          );

          //Getting the array of message exchange between Docter and patient
          this.messages = messagesResponse.data;
          if (this.messages && this.messages.length > 0) {
            // Check if latest is read or not
            if (
              this.messages[this.messages.length - 1].ReceiveUserID ==
                this.$store.state.user.UserID &&
              this.messages[this.messages.length - 1].State == "Sent" &&
              this.notification
            ) {
              // if there's a new message flag will be true
              console.log("There's a new message");
              this.flag = true;
              if (count == 0) {
                count++;
              }

              // verifying the state of the message
              if (this.messages[this.messages.length - 1].State != "Sent") {
                this.notification = false;
              }

              //when the chatbox is open change the flag to false
              if (this.chatbox_modal == true && count > 0) {
                await axios.put(
                  `http://localhost:5000/message/${
                    this.messages[this.messages.length - 1].ID
                  }`,
                  {
                    State: "Read",
                  }
                );
                this.flag = false;
                count = 0;
              }
              //console.log("Value inside:", this.flag);
              // TODO: Show that there is new message to read
            }
          }

          // Wait 2s before checking for new messages
          await new Promise((r) => setTimeout(r, 2000));
        }
      } catch (err) {
        console.log(err);
      }
    },
    init() {
      this.getApproved();
      this.getStatuses();
      this.getAppointments();
      this.getDoctorAppointmentRequests();
      this.form = Object.assign({}, this.form_default);
    },
  },
  computed: {
    statusesFiltered() {
      if (this.show_more === false) {
        return this.statuses.slice(0, 3);
      }
      return this.statuses;
    },
    userId() {
      // return 3;
      return this.$store.state.user.UserID;
    },
    doctorId() {
      // return 4;
      return this.$store.state.user.Doctor;
    },
    filteredAvailability() {
      return this.availabilities.filter((item) => {
        return item.SpecificDay.substr(0, 10) == this.date;
      });
    },
    // check if user is overdue and if yes show alert to him
    isOverDue() {
      if (this.statuses.length == 0) return false;
      // get last status
      const last_status = this.statuses[0];
      const last_status_date = +new Date(last_status.fillOutDate);
      // get today time stamp
      const today = +new Date();
      console.log(today, last_status_date);
      //compare
      const diff = today - last_status_date;
      console.log(diff > 24 * 60 * 60 * 1000);
      if (diff > 24 * 60 * 60 * 1000 && last_status.Covid) {
        return true;
      }
      return false;
    },
    getNotificationValue() {
      return this.notification;
    },
  },

  mounted() {
    this.init();
    this.$emit("img", "patient");
  },
};
</script>

<style>
.chatbox-css {
  float: right;
}
</style>

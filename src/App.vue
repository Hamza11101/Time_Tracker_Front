

<template>
  <h1>{{ timestamp }}</h1>
  <div class="buttons">
    <button class="StopButton" @click="stop">Stop</button>
    <button class="StartButton" @click="play">start new</button>
    <a class="timer">{{ zfill(hour) }}:{{ zfill(min) }}:{{ zfill(sec) }}</a>
  </div>

  <div>
    <span class="periodNumber"></span>
    <span></span>
    <span></span>
  </div>


    <div class="cla1" v-for="period in periods" :key="period.id">
      <span class="periods">period {{ period.id}} </span>
      <span class="timing">{{ period.StarTime }}-{{ period.EndTime }}</span>
      <span class="StayedTime">{{ period.PeriodTime }}</span>
    </div>
  <div class="Total">Total: {{ total }}</div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import http from "./Utils/http";
export default defineComponent({
  name: "app",
  components: {},

  data() {
    return {
      name: "hamza",
      timestamp: "",
      sec: 0,
      min: 0,
      hour: 0,
      stope: false,
      timer: null,
      intervalList: [],
      startedat: "",
      finishedat: "",
      PeriodTime: "",
      periods: null,
      total: "00:00:00",
    };
  },
  created() {
    setInterval(this.getNow, 1000);
  },

  methods: {
    changethename(name: string) {
      this.name = name;
    },

    getNow: function () {
      const monthNames = [
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December",
      ];
      const today = new Date();
      const date =
        today.getFullYear() +
        "-" +
        monthNames[today.getMonth() + 1] +
        "-" +
        today.getDate();
      const dateTime = date;
      this.timestamp = dateTime;
    },

    zfill(number: any) {
      return number.toString().padStart(2, 0);
    },
    play() {
      var hours: any = new Date().getHours().toString().padStart(2, "0");
      var minutes: any = new Date().getMinutes().toString().padStart(2, "0");
      this.startedat = "" + hours + ":" + minutes;
      console.log(this.startedat);
      this.stope = false;
      if (this.timer === null) {
        this.playing();
        var time = setInterval(() => this.playing(), 1000);
      } else {
        clearInterval(this.timer);
        this.timer = null;
      }
    },
    playing() {
      if (this.stope == false) {
        this.sec++;
        if (this.sec >= 59) {
          this.sec = 0;
          this.min++;
        }
        if (this.min >= 59) {
          this.min = 0;
          this.hour++;
        }
      }
    },

    stop() {
      var hours: any = new Date().getHours().toString().padStart(2, "0");
      var minutes: any = new Date().getMinutes().toString().padStart(2, "0");
      this.finishedat = "" + hours + ":" + minutes;
      console.log(this.finishedat);

      this.stope = true;
      if (this.timer !== null) {
        this.timer = null;
      }
      const ti =
        this.hour.toString().padStart(2, "0") +
        ":" +
        this.min.toString().padStart(2, "0") +
        ":" +
        this.sec.toString().padStart(2, "0");
      this.PeriodTime = ti;
      http.post("/time-tracker/time/", {
        StarTime: this.startedat,

        EndTime: this.finishedat,

        PeriodTime: this.PeriodTime,
      }).then(()=>this.getAllTime());

      clearInterval();
      this.sec = 0;
      this.min = 0;
      this.hour = 0;
    },

 async   getAllTime() {
    await  http.get("/time-tracker/time/").then((res) => {
        let result = res.data.Tims.sort().reverse();
        this.periods=result
        console.log(this.periods);
        
        let arr = res.data.Tims.map((s: any) => {
          return s.PeriodTime;
        });
        this.totalTime(arr);
      });
    },
    totalTime(arr: any) {
      let sum = "";
      if (arr != null) {
        for (let i = 0; i < arr.length; i++) {
          if (i == 0) {
            sum = arr[i];
            continue;
          } else {
            var a = sum.split(":");
            var seconds = +a[0] * 60 * 60 + +a[1] * 60 + +a[2];
            var b = arr[i].split(":");
            var seconds2 = +b[0] * 60 * 60 + +b[1] * 60 + +b[2];
            var date = new Date(1970, 0, 1);
            date.setSeconds(seconds + seconds2);
            sum = date.toTimeString().replace(/.*(\d{2}:\d{2}:\d{2}).*/, "$1");
          }
        }
      }
      console.log(sum);
      this.total = sum;
    },
  },
});
</script>

<style>
.StopButton {
  font-size: 15px;
  color: red;
  display: inline;
  margin-right: 20px;
  border-color: red;
  border-radius: 5px;
}
.StartButton {
  font-size: 15px;

  color: green;
  border-color: green;
  border-radius: 5px;
}
.buttons {
  display: flex;
  justify-content: right;
  margin-right: 10%;
}
.timer {
  margin-left: 10px;
  font-weight: bold;
  color: green;
  font-size: 25px;
}
.cla1 {
  padding: 2px 16px;
  background: #fff;
  display: block;
  margin: 20px 10px;
  border-left: 8px solid #fff;
}
.periods {
  float: left;
}
.timing {
  display: flex;
  justify-content: center;
}

.StayedTime {
  float: right;
}
.Total {
  float: right;
  font-weight: bold;
}
</style>

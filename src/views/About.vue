<template>
  <div class="about">
    <h1>This is an about page</h1>
    <br />
    <h3>{{ result }}</h3>
    <p>{{ error }}</p>
    <br />
    <button @click="getValues">Refresh</button>
    <br />
    <div class="box" v-if="plcInfos">
      PLC MAC
      <p>{{ plcInfos }}</p>
    </div>
    <div class="box" v-if="lauf">Laufvariable: {{ lauf }}</div>
    <div class="box" v-if="message">Message: {{ message }}</div>
    <div class="box" v-if="toggle">Toggle: {{ toggle }}</div>
  </div>
</template>
<script>
import axios from "axios";
export default {
  name: "About",
  data() {
    return {
      result: "nothing",
      error: "no error",
      lauf: null,
      message: null,
      toggle: null,
      plcInfos: null
    };
  },
  created() {
    this.getValues();
    axios
      .post(
        "https://192.168.1.18/wbm/configtools.php",
        {
          csrfToken: "",
          renewSession: true,
          aDeviceParams: {
            "0": {
              name: "get_actual_eth_config",
              parameter: ["X1", "mac-address"],
              sudo: true,
              multiline: false,
              timeout: 12000,
              dataId: 0
            }
          }
        },
        {
          headers: {
            "Content-Type": "text/plain"
          }
        }
      )
      .then(res => {
        if (res.data.aDeviceResponse[0].resultString.match(/00:30:DE/gi)) {
          this.plcInfos = res.data.aDeviceResponse[0].resultString;
        } else {
          this.error = "Messkoffer nicht gefunden!";
        }
      });
  },
  methods: {
    getValues() {
      axios
        .post(
          "https://192.168.1.18/webvisu/webvisu.htm",
          "|0|3|0|4|154|2|1|1|4|168|81|8|2|4|160|1|0|"
        )
        .then(res => {
          let temp = res.data;
          temp = temp.substring(1, temp.length - 1);
          temp = temp.split("|");
          this.lauf = temp[0];
          this.message = temp[1];
          this.toggle = temp[2];
        })
        .catch(err => (this.error = err));
    }
  }
};
</script>
<style>
.box {
  border: 1px solid grey;
  border-radius: 10px;
  min-width: 200px;
  max-width: 500px;
  margin: 25px auto;
}
</style>

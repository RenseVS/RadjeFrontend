<template>
  <div
    style="
      display: flex;
      justify-content: center;
      align-items: center;
      text-align: center;
      min-height: 100vh;
      background-image: url('https://media.istockphoto.com/id/539199982/nl/foto/blur-dark-bar-or-cafe-at-night.jpg?s=170667a&w=0&k=20&c=mKyTZI4plJn-sRvpX-SmwcyC0mfsS3P1tvzNxVOBnwk=');
      background-repeat: no-repeat;
      background-attachment: fixed;
      background-size: cover;
    "
  >
    <FortuneWheel
      ref="foo"
      style="width: 500px"
      borderColor="#584b43"
      :borderWidth="6"
      :prizes="prizes"
      :duration="duration"
      :verify="isVerifiedCanvas"
      @rotateStart="onCanvasRotateStart"
      @rotateEnd="onRotateEnd"
    />
  </div>
</template>

<script>
import audioFile from "@/assets/Victory.mp3";
import axios from "axios";
import FortuneWheel from "vue-fortune-wheel";
import "vue-fortune-wheel/lib/vue-fortune-wheel.css";

export default {
  components: {
    FortuneWheel,
  },
  sockets: {
    connect() {
      console.log("client connected to server");
    },
    receive(data) {
      console.log("Data received from .Net Core: ", data);
      this.duration = data * 1000;
      this.$refs.foo.onClickBtn();
    },
  },
  data() {
    return {
      canvasVerify: true, // Whether the turntable in canvas mode is enabled for verification
      duration: 10000,
      prizes: [
        {
          id: 1, //* The unique id of each prize, an integer greater than 0
          name: "Blue", // Prize name, display value when type is canvas (this parameter is not needed when type is image)
          value: "blue", //* Prize value, return value after spinning
          bgColor: "#45ace9", // Background color (no need for this parameter when type is image)
          color: "#ffffff", // Font color (this parameter is not required when type is image)
          probability: 30, //* Probability, up to 4 decimal places (the sum of the probabilities of all prizes
        },
        {
          id: 2,
          name: "Red",
          value: "red",
          bgColor: "#dd3832",
          color: "#ffffff",
          probability: 40,
        },
        {
          id: 3,
          name: "Yellow",
          value: "yellow",
          bgColor: "#fef151",
          color: "#ffffff",
          probability: 30,
        },
      ],
    };
  },
  computed: {
    isVerifiedCanvas: function () {
      return this.canvasVerify;
    },
  },
  methods: {
    onImageRotateStart() {
      console.log("onRotateStart");
    },
    onCanvasRotateStart(rotate) {
      console.log(rotate);
      console.log(this.isVerifiedCanvas);
      if (this.isVerifiedCanvas) {
        const verified = true; // true: the test passed the verification, false: the test failed the verification
        this.DoServiceVerify(verified, 2000).then((verifiedRes) => {
          if (verifiedRes) {
            console.log("Verification passed, start to rotate");
            rotate(); // Call the callback, start spinning
            this.canvasVerify = false; // Turn off verification mode
          } else {
            alert("Failed verification");
          }
        });
        return;
      }
      console.log("onCanvasRotateStart");
    },
    onRotateEnd(prize) {
      const audio = new Audio(audioFile);
      audio.play();
      const config = {
        headers: {},
      };
      //alert(prize.value);
      axios
        .post(
          "https://localhost:7021/WeatherForecast/PostResult",
          {
            color: prize.value,
          },
          config
        )
        .then((response) => {
          console.log("Response:", response.data);
        })
        .catch((error) => {
          console.error("Error:", error);
        });
    },
    // Simulate the request back-end interface, verified: whether to pass the verification, duration: delay time
    DoServiceVerify(verified, duration) {
      return new Promise((resove) => {
        setTimeout(() => {
          resove(verified);
        }, duration);
      });
    },
  },
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

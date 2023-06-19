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
    <div class="container">{{ this.userText }} {{ this.user }}</div>
    <FortuneWheel
      ref="foo"
      style="width: 500px"
      borderColor="#584b43"
      :borderWidth="6"
      :prizes="prizes"
      :duration="duration"
      :verify="isVerifiedCanvas"
      btnText="Stips Cafe"
      :fontSize="60"
      @rotateStart="onCanvasRotateStart"
      @rotateEnd="onRotateEnd"
    />
  </div>
</template>

<script>
import audioFile from "@/assets/Victory.mp3";
import Challenger from "@/assets/Challenger.mp3";
import tick from "@/assets/Tick.mp3";
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
      this.userText = "Spinning User:";
      this.duration = data * 1000;
      this.$refs.foo.onClickBtn();
    },
    receiveuser(data) {
      this.userText = "Next User:";
      this.user = data;
      const audio = new Audio(Challenger);
      audio.play();
    },
  },
  mounted() {
    const fooComponent = this.$refs.foo;
    const fortuneWheelBoxElement = fooComponent.$refs.fortuneWheelBox;

    const checkRotation = () => {
      // Get the current rotation in degrees
      const currentRotation = this.getRotationDegrees(fortuneWheelBoxElement);

      // Calculate the difference in rotation from the last trigger
      const rotationDifference = Math.abs(currentRotation - this.lastRotation);
      // Check if the rotation has changed by 35 degrees or more
      if (rotationDifference >= 360 / this.prizes.length) {
        const audio = new Audio(tick);
        audio.play();
        // Trigger your method in the main view
        this.$emit("rotationTriggered");

        // Update the lastRotation variable
        this.lastRotation = currentRotation;
      }

      // Continue checking the rotation on the next frame
      requestAnimationFrame(checkRotation);
    };

    // Start checking the rotation
    checkRotation();
  },
  data() {
    return {
      userText: "",
      user: "",
      lastRotation: 0,
      canvasVerify: true, // Whether the turntable in canvas mode is enabled for verification
      duration: 10000,
      prizes: [
        {
          id: 1, //* The unique id of each prize, an integer greater than 0
          name: "💧", // Prize name, display value when type is canvas (this parameter is not needed when type is image)
          value: "blue", //* Prize value, return value after spinning
          bgColor: "#1e2f97", // Background color (no need for this parameter when type is image)
          color: "#ffffff", // Font color (this parameter is not required when type is image)
          probability: 15, //* Probability, up to 4 decimal places (the sum of the probabilities of all prizes
        },
        {
          id: 2,
          name: "🍺",
          value: "yellow",
          bgColor: "#fbe106",
          color: "#ffffff",
          probability: 15,
        },
        {
          id: 3,
          name: "💧",
          value: "blue",
          bgColor: "#1e2f97",
          color: "#ffffff",
          probability: 15,
        },
        {
          id: 4, //* The unique id of each prize, an integer greater than 0
          name: "🍺", // Prize name, display value when type is canvas (this parameter is not needed when type is image)
          value: "yellow", //* Prize value, return value after spinning
          bgColor: "#fbe106", // Background color (no need for this parameter when type is image)
          color: "#ffffff", // Font color (this parameter is not required when type is image)
          probability: 25, //* Probability, up to 4 decimal places (the sum of the probabilities of all prizes
        },
        {
          id: 5,
          name: "💧",
          value: "blue",
          bgColor: "#1e2f97",
          color: "#ffffff",
          probability: 15,
        },
        {
          id: 6,
          name: "🍺",
          value: "yellow",
          bgColor: "#fbe106",
          color: "#ffffff",
          probability: 15,
        },
      ],
      prizeId: {},
    };
  },
  computed: {
    isVerifiedCanvas: function () {
      return this.canvasVerify;
    },
  },
  methods: {
    getRotationDegrees(element) {
      const matrix = getComputedStyle(element).getPropertyValue("transform");
      const matrixValues = matrix.split("(")[1].split(")")[0].split(",");
      const a = matrixValues[0];
      const b = matrixValues[1];
      const angle = Math.round(Math.atan2(b, a) * (180 / Math.PI));
      return angle < 0 ? angle + 360 : angle;
    },
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
            prize: prize.name,
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

<style>
.container {
  display: inline-block;
  position: absolute;
  z-index: 100;
  width: 100%;
  height: 100vh;
  color: white;
  font-size: x-large;
}
</style>
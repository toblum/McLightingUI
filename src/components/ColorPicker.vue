<template>
    <v-container>
        <img ref="color_wheel_image" src="../assets/color-wheel.png" style="display: none" />
        <canvas ref="canvas" id="color_wheel_canvas" v-on:click="onSelectColor" v-on:touchmove="onSelectColor">
        </canvas>
    </v-container>
</template>

<script>
export default {
  name: "ColorPicker",

  data() {
    return {
      canvas: null,
      context: null
    };
  },

  methods: {
    componentToHex(c) {
      return ("0" + Number(c).toString(16)).slice(-2).toUpperCase();
    },

    rgbToHex(rgb) {
      return (
        this.componentToHex(rgb[0]) +
        this.componentToHex(rgb[1]) +
        this.componentToHex(rgb[2])
      );
    },

    onSelectColor(event) {
      var pos;
      event.preventDefault();

      if (event.type === "touchmove") {
        var el = event.target;

        pos = {
          x: Math.round(event.targetTouches[0].pageX - el.offsetLeft),
          y: Math.round(event.targetTouches[0].pageY - el.offsetTop)
        };
      }
      if (event.type === "click") {
        pos = {
          x: Math.round(event.offsetX),
          y: Math.round(event.offsetY)
        };
      }

      var color = this.context.getImageData(pos.x, pos.y, 1, 1).data;
      var hex_color = this.rgbToHex(color);
      var result = {
        hex: hex_color,
        r: color[0],
        g: color[1],
        b: color[2],
        a: color[3]
      };
      this.$emit("selected", result);

      // console.log("e", event.type, pos, event, color, hex_color);
    },

    drawWheel() {
      var centerX = this.canvas.width / 2;
      var centerY = this.canvas.height / 2;
      // console.log(centerX, centerY, this.canvas.width, this.canvas.height);
      var innerRadius = this.canvas.width / 4.5;
      var outerRadius = (this.canvas.width - 10) / 2;
      // outer border
      this.context.beginPath();
      // outer circle
      this.context.arc(centerX, centerY, outerRadius, 0, 2 * Math.PI, false);
      // draw the outer border: (gets drawn around the circle!)
      this.context.lineWidth = 4;
      this.context.strokeStyle = "#000000";
      this.context.stroke();
      this.context.closePath();

      // fill with beautiful colors
      // taken from here: http://stackoverflow.com/questions/18265804/building-a-color-wheel-in-html5
      for (var angle = 0; angle <= 360; angle += 1) {
        var startAngle = (angle - 2) * Math.PI / 180;
        var endAngle = angle * Math.PI / 180;
        this.context.beginPath();
        this.context.moveTo(centerX, centerY);
        this.context.arc(
          centerX,
          centerY,
          outerRadius,
          startAngle,
          endAngle,
          false
        );
        this.context.closePath();
        this.context.fillStyle = "hsl(" + angle + ", 100%, 50%)";
        this.context.fill();
        this.context.closePath();
      }

      // inner border
      this.context.beginPath();
      // this.context.arc(centerX, centerY, radius, startAngle, endAngle, counterClockwise);
      this.context.arc(centerX, centerY, innerRadius, 0, 2 * Math.PI, false);
      // fill the center
      var my_gradient = this.context.createLinearGradient(0, 0, 170, 0);
      my_gradient.addColorStop(0, "black");
      my_gradient.addColorStop(1, "white");

      this.context.fillStyle = my_gradient;
      this.context.fillStyle = "white";
      this.context.fill();

      // draw the inner line
      this.context.lineWidth = 2;
      this.context.strokeStyle = "#000000";
      this.context.stroke();
      this.context.closePath();
    },
    drawWheelImage() {
      var image = new Image();
      var that = this;
      image.onload = function() {
        that.context.drawImage(image, 0, 0, 400, 400);
      };
      image.src = this.$refs.color_wheel_image.src;
    }
  },
  mounted() {
    this.canvas = this.$refs.canvas;
    this.canvas.width = 400;
    this.canvas.height = 400;

    this.context = this.canvas.getContext("2d");

    // this.drawWheel();
    this.drawWheelImage();
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  #color_wheel_canvas {
    user-select: none;
    width: 100%;
    max-width: 400px;
    -webkit-user-select: none;
    -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
    -moz-user-select: none;
  }
</style>

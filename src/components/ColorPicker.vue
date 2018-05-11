<template>
    <v-container>
        <canvas ref="canvas" id="color_wheel_canvas" class="mx-auto" v-on:click="onSelectColor" v-on:touchmove="onSelectColor"></canvas>

        <div id="color_wheel_legend" class="elevation-6" vxxxif="color_hex" v-bind:style="{'background-color': '#' + color_hex}">
          Color: #{{ color_hex }}<br/>
          R: {{ color.r || "" }}, G: {{ color.g || "" }}, B: {{ color.b || "" }}
        </div>
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

  props: ["prop_color"],
  computed: {
    color: function() {
      if (this.prop_color) {
        return JSON.parse(JSON.stringify(this.prop_color));
      } else {
        return {r: 0, g: 0, b: 0};
      }
    },
    color_hex: function() {
      if (this.color) {
        return this.rgbToHex([this.color.r, this.color.g, this.color.b]);
      } else {
        return "";
      }
    }
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
    },

    /**
     * Draws color ring
     */
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

    /**
     * Draws color circle
     */
    drawCircle() {
      let radius = this.canvas.width / 2;
      let image = this.context.createImageData(2 * radius, 2 * radius);
      let data = image.data;

      for (let x = -radius; x < radius; x++) {
        for (let y = -radius; y < radius; y++) {
          let [r, phi] = this.xy2polar(x, y);

          if (r > radius) {
            // skip all (x,y) coordinates that are outside of the circle
            continue;
          }

          let deg = this.rad2deg(phi);

          // Figure out the starting index of this pixel in the image data array.
          let rowLength = 2 * radius;
          let adjustedX = x + radius; // convert x from [-50, 50] to [0, 100] (the coordinates of the image data array)
          let adjustedY = y + radius; // convert y from [-50, 50] to [0, 100] (the coordinates of the image data array)
          let pixelWidth = 4; // each pixel requires 4 slots in the data array
          let index = (adjustedX + (adjustedY * rowLength)) * pixelWidth;

          let hue = deg;
          let saturation = r / radius;
          let value = 1.0;

          let [red, green, blue] = this.hsv2rgb(hue, saturation, value);
          let alpha = 255;

          data[index] = red;
          data[index + 1] = green;
          data[index + 2] = blue;
          data[index + 3] = alpha;
        }
      }
      this.context.putImageData(image, 0, 0);
    },
    xy2polar(x, y) {
      let r = Math.sqrt(x * x + y * y);
      let phi = Math.atan2(y, x);
      return [r, phi];
    },
    rad2deg(rad) {
      // rad in [-π, π] range
      // return degree in [0, 360] range
      return ((rad + Math.PI) / (2 * Math.PI)) * 360;
    },
    hsv2rgb(hue, saturation, value) {
      let chroma = value * saturation;
      let hue1 = hue / 60;
      let x = chroma * (1 - Math.abs((hue1 % 2) - 1));
      let r1, g1, b1;

      if (hue1 >= 0 && hue1 <= 1) {
        ([r1, g1, b1] = [chroma, x, 0]);
      } else if (hue1 >= 1 && hue1 <= 2) {
        ([r1, g1, b1] = [x, chroma, 0]);
      } else if (hue1 >= 2 && hue1 <= 3) {
        ([r1, g1, b1] = [0, chroma, x]);
      } else if (hue1 >= 3 && hue1 <= 4) {
        ([r1, g1, b1] = [0, x, chroma]);
      } else if (hue1 >= 4 && hue1 <= 5) {
        ([r1, g1, b1] = [x, 0, chroma]);
      } else if (hue1 >= 5 && hue1 <= 6) {
        ([r1, g1, b1] = [chroma, 0, x]);
      }

      let m = value - chroma;
      let [r, g, b] = [r1 + m, g1 + m, b1 + m];

      // Change r,g,b values from [0,1] to [0,255]
      return [255 * r, 255 * g, 255 * b];
    }
  },
  mounted() {
    this.canvas = this.$refs.canvas;
    this.canvas.width = 400;
    this.canvas.height = 400;

    this.context = this.canvas.getContext("2d");

    this.drawCircle();
    // this.drawWheel();
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  #color_wheel_canvas {
    user-select: none;
    width: 100%;
    max-width: 400px;
    display: block;
    -webkit-user-select: none;
    -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
    -moz-user-select: none;
  }
  #color_wheel_legend {
    margin-top: 2rem;
    padding: 0.8rem;
    text-align: center;
  }
</style>

<template>
  <div class="slider-wrapper">
    <canvas :id="'slider_' + id" width="60" height="270"></canvas>
  </div>
</template>

<script>
import { onMounted } from "vue";

export default {
  name: "Slider",
  props: {
    id: {
      type: Number,
      default: 0,
    },
    data: {
      type: Number,
      default: 0,
    },
  },
  setup(props, { emit }) {
    let c = null;
    let ctx = null;
    let isDragged = false;
    let X = 0;
    let Y = 0;

    const handle = {
      x: 0,
      y: 0,
      radius: 9,
      color: "rgba(200, 0, 200, 0.8)",
      init() {
        this.x = X;
        this.y = Y + 250;
        ctx.beginPath();
        ctx.arc(this.x + 30, this.y, this.radius, 0, Math.PI * 2, false);
        ctx.closePath();
        ctx.fillStyle = this.color;
        ctx.fill();
      },
      draw() {
        ctx.beginPath();
        ctx.arc(this.x + 30, this.y, this.radius, 0, Math.PI * 2, false);
        ctx.closePath();
        ctx.fillStyle = this.color;
        ctx.fill();
      },
      update(event) {
        this.y = event.y;
      },
    };

    function drawScale() {
      ctx.strokeStyle = "rgba(255,255,255,1)";
      ctx.lineWidth = 1;
      for (let i = Y + 50; i < Y + 260; i += 20) {
        if (i === 150) {
          ctx.save();
          ctx.strokeStyle = "rgba(255,0,100,1)";
          ctx.beginPath();
          ctx.moveTo(X + 5, i);
          ctx.lineTo(X + 55, i);
          ctx.stroke();
          ctx.restore();
          i = 170;
        }
        ctx.beginPath();
        ctx.moveTo(X + 10, i);
        ctx.lineTo(X + 50, i);
        ctx.stroke();
      }
    }
    function drawValue(value) {
      if (value === 0) {
        // eslint-disable-next-line no-param-reassign
        value = "0.00";
      }

      if (value === 1) {
        // eslint-disable-next-line no-param-reassign
        value = "1.00";
      }

      ctx.font = "16px Arial";
      ctx.fillStyle = "rgba(255,255,255,1)";
      ctx.fillText(value, 14, 30);
    }
    function drawBody() {
      ctx.fillStyle = "rgba(27,27,27,1.0)";
      ctx.fillRect(X, Y, ctx.canvas.width, ctx.canvas.height);
      drawScale();
      ctx.fillRect(X + 24, Y + 49, X + 12, Y + 202);
    }
    function drawSliderInit() {
      ctx.fillStyle = "rgba(27,27,27,1.0)";
      ctx.fillRect(X, Y, ctx.canvas.width, ctx.canvas.height);
      c.style.cursor = "pointer";
      drawBody();
      drawValue(props.data);
      handle.init();
    }

    function mouseToCoord(mousePos) {
      if (mousePos <= 251 || mousePos >= 49) {
        return (mousePos - 250) / (50 - 250);
      }
      return 0;
    }

    onMounted(() => {
      c = document.getElementById(`slider_${props.id}`);
      ctx = c.getContext("2d");
      X = c.width - 60;
      Y = c.height - 270;

      c.addEventListener("mousemove", (event) => {
        if (isDragged) {
          if (event.y > Y + 49 && event.y < Y + 251) {
            ctx.clearRect(0, 0, 60, 280);
            drawBody();
            ctx.fillStyle = "rgba(149, 7, 64, 1)";
            ctx.beginPath();
            ctx.rect(X + 27, event.y, X + 6, 252 - event.y);
            ctx.closePath();
            ctx.fill();
            handle.update(event);
            handle.draw();
            const tempValue = mouseToCoord(event.y);
            drawValue(tempValue);
            emit("getSlider", tempValue);
          }
        }
      });

      c.addEventListener("mousedown", () => {
        isDragged = true;
      });

      c.addEventListener("mouseup", () => {
        isDragged = false;
      });

      drawSliderInit();
    });

    return {
      c,
      ctx,
      handle,
      drawSliderInit,
      drawScale,
      drawBody,
      drawValue,
    };
  },
};
</script>

<style>
#slider {
  border: 1px solid wheat;
}
</style>

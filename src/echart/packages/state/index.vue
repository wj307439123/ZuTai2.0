<template>
  <div :class="b()"
       :style="styleSizeName"
       @click="handleClick">
    <img :style="[styleImgName,styleSizeName]"
         :src="dataChart.value"
         :class="b({'rotate': rotate})"
         draggable="false" />
  </div>
</template>

<script>
import create from "../../create";
export default create({
  name: "state",
  computed: {
    styleImgName () {
      //console.log(11111111111111111)
      return Object.assign(
        (() => {
          if (this.rotate) {
            return {
              animationDuration: this.duration / 1000 + "s"
            };
          }
          return {};
        })(),
        {
          opacity: this.option.opacity || 1
        }
      );
    },
    duration () {
      return this.option.duration || 3000;
    },
    rotate () {
      return this.option.rotate;
    }
  },
  methods: {
    handleClick () {
      this.clickFormatter && this.clickFormatter({
        data: this.dataChart
      });
    }
  }
});
</script>


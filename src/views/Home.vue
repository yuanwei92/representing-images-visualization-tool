<template>
  <div class="home">
    <b-container class="px-0" fluid>
      <b-row no-gutters>
        <b-col md="4">
          <div id="content">
            <div class="p-4">
              <h1 class="title">Representing Images in Binary</h1>
              <p>
                An interactive tool to visualize how images are represented
                using binary.
              </p>
              <b-form-group
                label-cols="4"
                label-cols-lg="4"
                label-size="sm"
                label="Width"
                label-for="input-sm"
              >
                <b-input-group size="sm" append="px">
                  <b-form-input
                    v-model.number="resoWidthInput"
                    min="1"
                    max="50"
                    type="number"
                    size="sm"
                  ></b-form-input>
                </b-input-group>
              </b-form-group>
              <b-form-group
                label-cols="4"
                label-cols-lg="4"
                label-size="sm"
                label="Height"
                label-for="input-sm"
              >
                <b-input-group size="sm" append="px">
                  <b-form-input
                    v-model.number="resoHeightInput"
                    min="1"
                    max="50"
                    type="number"
                    size="sm"
                  ></b-form-input>
                </b-input-group>
              </b-form-group>
              <b-form-group
                label-cols="4"
                label-cols-lg="4"
                label-size="sm"
                label="Colour Depth"
                label-for="input-sm"
              >
                <b-form-select
                  v-model="colourDepthInput"
                  class="mb-3"
                  size="sm"
                >
                  <b-form-select-option :value="1">
                    1 bit
                  </b-form-select-option>
                  <b-form-select-option :value="2">
                    2 bits
                  </b-form-select-option>
                  <b-form-select-option :value="3">
                    3 bits
                  </b-form-select-option>
                  <b-form-select-option :value="24">
                    24 bits
                  </b-form-select-option>
                </b-form-select>
              </b-form-group>
              <div>
                <div>Resolution: {{ `${resoWidth} x ${resoHeight} px` }}</div>
                <div>Total pixels: {{ resoWidth * resoHeight }}</div>
                <div>Number of colours: {{ 2 ** colourDepth }}</div>
              </div>
              <div class="mt-3">
                <b-button variant="primary" @click="onClickGenerate">
                  Generate image
                </b-button>
              </div>

              <p class="mt-3"></p>

              <div>
                <b-table-simple
                  v-if="colourDepthInput != 24"
                  class="text-center"
                >
                  <b-thead head-variant="dark">
                    <b-tr>
                      <b-th>Binary</b-th>
                      <b-th>Colour</b-th>
                    </b-tr>
                  </b-thead>
                  <b-tbody>
                    <b-tr v-for="colour in colours" :key="colour.binary">
                      <b-td>{{ colour.binary }}</b-td>
                      <b-td>{{ colour.colour }}</b-td>
                    </b-tr>
                  </b-tbody>
                </b-table-simple>
                <div v-else>
                  <p>
                    24-bit colour also known as true colour. It can display more
                    than 16 million colours which is approximately the number of
                    individual colours the human eye can distinguish.
                  </p>
                  <div class="d-flex align-items-center mb-3">
                    <div>Pick a colour:</div>
                    <input class="ml-3 " type="color" v-model="selectColour" />
                  </div>
                  <div>
                    <textarea v-model="selectedBinary" class="w-100">
                    </textarea>
                  </div>
                </div>
              </div>
            </div>
            <div id="footer" class="px-3">
              Created by
              <a class="footer-link" href="https://csplayground.io">
                csplayground.io
              </a>
              |
              <a class="footer-link" href="https://twitter.com/yuanwei92"
                >Twitter</a
              >
              |
              <a
                class="footer-link"
                href="https://github.com/yuanwei92/representing-images-visualization-tool"
                >Github</a
              >
            </div>
          </div>
        </b-col>

        <b-col md="4">
          <div id="ace-editor"></div>
        </b-col>

        <b-col md="4">
          <div id="canvas-container">
            <div ref="p5-container" id="p5-container"></div>
          </div>
        </b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script>
import ace from "ace-builds/src-noconflict/ace";
import p5 from "p5";
import _ from "lodash";
import { palette } from "@/ts/palette";
import "ace-builds/webpack-resolver";

export default {
  name: "Home",

  data() {
    return {
      editor: null,
      p5Instance: null,
      resoWidthInput: 10,
      resoHeightInput: 10,
      resoWidth: 10,
      resoHeight: 10,
      canvasWidth: 0,
      canvasHeight: 0,
      colourDepth: 1,
      colourDepthInput: 1,
      grid: [],
      selectColour: "#000000"
    };
  },

  created() {
    window.addEventListener("resize", this.onWindowResize);
  },

  mounted() {
    this.onMounted();
  },

  destroyed() {
    window.removeEventListener("resize", this.onWindowResize);
  },

  computed: {
    pixelWidth() {
      const largerValue =
        this.resoWidth > this.resoHeight ? this.resoWidth : this.resoHeight;
      return this.canvasWidth / largerValue;
    },

    colours() {
      return palette[this.colourDepthInput];
    },

    selectedBinary() {
      const hexString = this.selectColour.slice(1);
      let array = hexString.match(/.{1,2}/g);
      let binary = "";

      array = array.map(hex => {
        return parseInt(hex, 16)
          .toString(2)
          .padStart(8, "0");
      });
      array.forEach(hex => {
        binary += hex;
      });
      return binary;
    }
  },

  methods: {
    onMounted() {
      this.generateGrid();
      this.initEditor();
      this.initp5();
    },

    initEditor() {
      this.editor = ace.edit("ace-editor", {
        mode: "ace/mode/text",
        selectionStyle: "text",
        fontSize: 16,
        theme: "ace/theme/eclipse"
      });
    },

    initp5() {
      this.canvasWidth = this.$refs["p5-container"].clientWidth;
      this.canvasHeight = this.$refs["p5-container"].clientHeight;
      this.p5Instance = new p5(sketch => {
        sketch.setup = () => {
          sketch.createCanvas(this.canvasWidth, this.canvasWidth);
        };

        sketch.draw = () => {
          sketch.background(0);
          sketch.fill(255);
          for (let column = 0; column < this.resoWidth; column++) {
            for (let row = 0; row < this.resoHeight; row++) {
              sketch.fill(255, 236, 217);

              const pixel = this.grid[row][column];
              this.setSketchFillColour(sketch, pixel);

              sketch.stroke(0);
              sketch.rect(
                column * this.pixelWidth,
                row * this.pixelWidth,
                this.pixelWidth - 0.1,
                this.pixelWidth - 0.1
              );
            }
          }
        };
      }, "p5-container");
    },

    onClickGenerate() {
      this.resoWidth = this.resoWidthInput;
      this.resoHeight = this.resoHeightInput;
      this.colourDepth = this.colourDepthInput;
      this.generateGrid();

      const value = this.editor.getValue();
      const re = new RegExp(`.{1,${this.colourDepth}}`, "g");
      const pixels = value.match(re);
      const grid = _.chunk(pixels, this.resoWidth);

      for (let index = 0; index < this.grid.length; index++) {
        this.grid[index] = grid[index] ?? this.grid[index];
      }
    },

    setSketchFillColour(sketch, pixel) {
      if (pixel == undefined) {
        return;
      }

      if (this.colourDepth <= 3) {
        const decimal = parseInt(pixel, 2);
        const colour = palette[this.colourDepth][decimal];
        sketch.fill(colour.rgb[0], colour.rgb[1], colour.rgb[2]);
      } else {
        const rgb = pixel.match(/.{1,8}/g);
        const additional = rgb.length % 3;

        for (let index = 0; index < additional; index++) {
          rgb.pop();
        }
        rgb.map(bin => {
          return parseInt(bin.padStart(8, "0"), 2);
        });
        sketch.fill(rgb[0], rgb[1], rgb[2]);
      }
    },

    generateGrid() {
      this.grid = new Array(this.resoHeight);
      for (let index = 0; index < this.resoHeight; index++) {
        this.grid[index] = new Array(this.resoWidth);
      }
    },

    onWindowResize() {
      this.canvasWidth = this.$refs["p5-container"].clientWidth;
      this.canvasHeight = this.$refs["p5-container"].clientHeight;
      this.p5Instance.resizeCanvas(this.canvasWidth, this.canvasWidth);
    }
  }
};
</script>

<style lang="less" scoped>
#content {
  height: calc(100vh - 40px);
  overflow: auto;
}

#footer {
  font-size: 14px;
  display: flex;
  align-items: center;
  position: absolute;
  color: white;
  background-color: rgb(65, 65, 65);
  height: 40px;
  width: 100%;
  bottom: 0;
  white-space: pre-wrap;
}

.footer-link {
  color: white;
}

#ace-editor {
  height: 100vh;
  width: 100%;
  overflow: auto;
}

#canvas-container {
  padding: 20px;
  height: 100vh;
  background-color: grey;
}

#p5-container {
  width: 100%;
  height: 100%;
}

.title {
  font-size: 2em;
  font-weight: 600;
}
</style>

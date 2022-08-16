<template>
  <div id="app">
    <img id="img" crossorigin="*" src="https://img2.baidu.com/it/u=3635204433,939208923&fm=253&fmt=auto&app=138&f=JPEG?w=375&h=500" />
    APP
    <div class="canvas-box"></div>
  </div>
</template>

<script>
import EscPosEncoder from '../lib/esc-pos-encoder'
const Dither = require('canvas-dither');
const Flatten = require('canvas-flatten');
const encoder = new EscPosEncoder()

export default {
  name: 'App',
  methods: {
    initHandler() {
      const img = document.querySelector('#img')
      const { width ,height } = img.getBoundingClientRect()
      console.log(width ,height)
      const scrw = 200
      let scrh = Math.ceil(height * scrw / width)
      scrh = scrh + 8 - (scrh % 8)
      console.log(scrw ,scrh, 222)
      // this.createCanvas(img, width, height)
      // this.createCanvas(img, width, height, 'bayer')
      // this.createCanvas(img, width, height, 'floydsteinberg')
      // this.createCanvas(img, width, height, 'atkinson')
      this.getArrryBuffer(scrw ,scrh, this.createCanvas(img, scrw ,scrh))
      this.getArrryBuffer(scrw ,scrh, this.createCanvas(img, scrw ,scrh, 'bayer'))
      this.getArrryBuffer(scrw ,scrh, this.createCanvas(img, scrw ,scrh, 'floydsteinberg'))
      this.getArrryBuffer(scrw ,scrh, this.createCanvas(img, scrw ,scrh, 'atkinson'))
      const result = encoder.image(img, scrw ,scrh).encode()
      console.log(result, 'result', Array.prototype.filter.call(result, Boolean))
    },
    createCanvas(element, width, height, algorithm = 'threshold', threshold = 128) {
      const canvas = document.createElement('canvas');
      canvas.width = width;
      canvas.height = height;
      const context = canvas.getContext('2d');
      context.drawImage(element, 0, 0, width, height);
      let image = context.getImageData(0, 0, width, height);

      image = Flatten.flatten(image, [0xff, 0xff, 0xff]);

      switch (algorithm) {
        case 'threshold': image = Dither.threshold(image, threshold); break;
        case 'bayer': image = Dither.bayer(image, threshold); break;
        case 'floydsteinberg': image = Dither.floydsteinberg(image); break;
        case 'atkinson': image = Dither.atkinson(image); break;
      }
      console.log(image, 'imagedata')
      context.putImageData(image, 0,0)
      document.querySelector('.canvas-box').appendChild(canvas)
      return image
    },
    getArrryBuffer(width, height, image) {
        const getPixel = (x, y) => x < width && y < height ? (image.data[((width * y) + x) * 4] > 0 ? 0 : 1) : 0;

        const getColumnData = (width, height) => {
          const data = [];

          for (let s = 0; s < Math.ceil(height / 24); s++) {
            const bytes = new Uint8Array(width * 3);

            for (let x = 0; x < width; x++) {
              for (let c = 0; c < 3; c++) {
                for (let b = 0; b < 8; b++) {
                  bytes[(x * 3) + c] |= getPixel(x, (s * 24) + b + (8 * c)) << (7 - b);
                }
              }
            }

            data.push(bytes);
          }

          return data;
        };
        const queuedList = []
        const _queue = (value) => {
          value.forEach((item) => queuedList.push(item));
        }
        _queue([
            0x1b, 0x33, 0x24,
        ]);

        getColumnData(width, height).forEach((bytes) => {
          console.log('bytes', bytes)
          _queue([
            0x1b, 0x2a, 0x21,
            (width) & 0xff, (((width) >> 8) & 0xff),
            bytes,
            0x0a,
          ]);
        });

        _queue([
          0x1b, 0x32,
        ]);
        console.log('queuedList', queuedList)
      return queuedList
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.initHandler()
    })
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

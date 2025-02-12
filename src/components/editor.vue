<template>
  <div class="editor">
    <div
      class="canvas"
      @dblclick="dblclick"
    >
      <img
        ref="image"
        :alt="data.name"
        :src="data.url"
        @loadstart="start"
        @load="start"
      >
    </div>
    <div
      v-if="cropper"
      class="toolbar"
      @click="click"
    >
      <button
        class="toolbar__button"
        data-action="move"
        title="Move (M)"
      >
        <span class="fa fa-arrows" />
      </button>
      <button
        class="toolbar__button"
        data-action="crop"
        title="Crop (C)"
      >
        <span class="fa fa-crop" />
      </button>
      <button
        class="toolbar__button"
        data-action="zoom-in"
        title="Zoom In (I)"
      >
        <span class="fa fa-search-plus" />
      </button>
      <button
        class="toolbar__button"
        data-action="zoom-out"
        title="Zoom Out (O)"
      >
        <span class="fa fa-search-minus" />
      </button>
      <button
        class="toolbar__button"
        data-action="rotate-left"
        title="Rotate Left (L)"
      >
        <span class="fa fa-rotate-left" />
      </button>
      <button
        class="toolbar__button"
        data-action="rotate-right"
        title="Rotate Right (R)"
      >
        <span class="fa fa-rotate-right" />
      </button>
      <button
        class="toolbar__button"
        data-action="flip-horizontal"
        title="Flip Horizontal (H)"
      >
        <span class="fa fa-arrows-h" />
      </button>
      <button
        class="toolbar__button"
        data-action="flip-vertical"
        title="Flip Vertical (V)"
      >
        <span class="fa fa-arrows-v" />
      </button>
    </div>
  </div>
</template>

<script>
import Cropper from 'cropperjs';

export default {
  name: 'Editor',

  props: {
    data: {
      type: Object,
      default: () => ({}),
    },
  },

  data() {
    return {
      canvasData: null,
      cropBoxData: null,
      croppedData: null,
      cropper: null,
    };
  },

  mounted() {
    window.addEventListener('keydown', (this.onKeydown = this.keydown.bind(this)));
  },

  beforeDestroy() {
    window.removeEventListener('keydown', this.onKeydown);
    this.stop();
  },

  methods: {
    click({ target }) {
      const { cropper } = this;
      const action = target.getAttribute('data-action') || target.parentElement.getAttribute('data-action');

      switch (action) {
        case 'move':
        case 'crop':
          cropper.setDragMode(action);
          break;

        case 'zoom-in':
          cropper.zoom(0.1);
          break;

        case 'zoom-out':
          cropper.zoom(-0.1);
          break;

        case 'rotate-left':
          cropper.rotate(-90);
          break;

        case 'rotate-right':
          cropper.rotate(90);
          break;

        case 'flip-horizontal':
          cropper.scaleX(-cropper.getData().scaleX || -1);
          break;

        case 'flip-vertical':
          cropper.scaleY(-cropper.getData().scaleY || -1);
          break;
      }
    },

keydown(e) {
  const keyMap = {
    90: 'z', // Z
    46: 'delete', // Delete
    13: 'enter', // Enter
    27: 'escape', // Escape
    37: 'arrowleft', // Left Arrow
    38: 'arrowup', // Up Arrow
    39: 'arrowright', // Right Arrow
    40: 'arrowdown', // Down Arrow
    67: 'c', // C
    77: 'm', // M
    73: 'i', // I
    79: 'o', // O
    76: 'l', // L
    82: 'r', // R
    72: 'h', // H
    86: 'v', // V
  };

  const key = keyMap[e.keyCode];

  if (!key) return;

  switch (key) {
    case 'z':
      if (e.ctrlKey) {
        e.preventDefault();
        this.restore();
      }
      break;

    case 'delete':
      this.reset();
      break;

    case 'enter':
      this.crop();
      break;

    case 'escape':
      this.clear();
      break;

    case 'arrowleft':
      e.preventDefault();
      this.cropper.move(-1, 0);
      break;

    case 'arrowup':
      e.preventDefault();
      this.cropper.move(0, -1);
      break;

    case 'arrowright':
      e.preventDefault();
      this.cropper.move(1, 0);
      break;

    case 'arrowdown':
      e.preventDefault();
      this.cropper.move(0, 1);
      break;

    case 'c':
      this.cropper.setDragMode('crop');
      break;

    case 'm':
      this.cropper.setDragMode('move');
      break;

    case 'i':
      this.cropper.zoom(0.1);
      break;

    case 'o':
      this.cropper.zoom(-0.1);
      break;

    case 'l':
      this.cropper.rotate(-90);
      break;

    case 'r':
      this.cropper.rotate(90);
      break;

    case 'h':
      this.cropper.scaleX(-this.cropper.getData().scaleX || -1);
      break;

    case 'v':
      this.cropper.scaleY(-this.cropper.getData().scaleY || -1);
      break;
      }
    },

    dblclick(e) {
      if (e.target.className.indexOf('cropper-face') >= 0) {
        e.preventDefault();
        e.stopPropagation();
        this.crop();
      }
    },

    start() {
      const { data } = this;

      if (data.cropped || this.cropper) {
        return;
      }

      this.cropper = new Cropper(this.$refs.image, {
        autoCrop: false,
        dragMode: 'move',
        background: false,

        ready: () => {
          if (this.croppedData) {
            this.cropper
              .crop()
              .setData(this.croppedData)
              .setCanvasData(this.canvasData)
              .setCropBoxData(this.cropBoxData);

            this.croppedData = null;
            this.canvasData = null;
            this.cropBoxData = null;
          }
        },

        crop: ({ detail }) => {
          if (detail.width > 0 && detail.height > 0 && !data.cropping) {
            this.update({
              cropping: true,
            });
          }
        },
      });
    },

    stop() {
      if (this.cropper) {
        this.cropper.destroy();
        this.cropper = null;
      }
    },

    crop() {
      const { cropper, data } = this;

      if (data.cropping) {
        this.croppedData = cropper.getData();
        this.canvasData = cropper.getCanvasData();
        this.cropBoxData = cropper.getCropBoxData();
        this.update({
          cropped: true,
          cropping: false,
          previousUrl: data.url,
          url: cropper.getCroppedCanvas(data.type === 'image/png' ? {} : {
            fillColor: '#fff',
          }).toDataURL(data.type),
        });
        this.stop();
      }
    },

    clear() {
      if (this.data.cropping) {
        this.cropper.clear();
        this.update({
          cropping: false,
        });
      }
    },

    restore() {
      if (this.data.cropped) {
        this.update({
          cropped: false,
          previousUrl: '',
          url: this.data.previousUrl,
        });
      }
    },

    reset() {
      this.stop();
      this.update({
        cropped: false,
        cropping: false,
        loaded: false,
        name: '',
        previousUrl: '',
        type: '',
        url: '',
      });
    },

    update(data) {
      Object.assign(this.data, data);
    },
  },
};
</script>

<style scoped>
.editor {
  height: 100%;
}

.canvas {
  align-items: center;
  display: flex;
  height: 100%;
  justify-content: center;

  & > img {
    max-height: 100%;
    max-width: 100%;
  }
}

.toolbar {
  background-color: rgba(0, 0, 0, .5);
  bottom: 1rem;
  color: #fff;
  height: 2rem;
  left: 50%;
  margin-left: -8rem;
  position: absolute;
  width: 16rem;
  z-index: 2015;
}

.toolbar__button {
  background-color: transparent;
  border-width: 0;
  color: #fff;
  cursor: pointer;
  display: block;
  float: left;
  font-size: .875rem;
  height: 2rem;
  text-align: center;
  width: 2rem;

  &:focus {
    outline: none;
  }

  &:hover {
    background-color: #0074d9;
    color: #fff;
  }
}
</style>

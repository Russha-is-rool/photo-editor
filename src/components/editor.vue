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

// Дальше идёт код который исправил ChatGPT  
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
  keydown(e) {
    const { cropper } = this;
    if (!cropper) return;

    const key = e.key.toLowerCase(); // Приводим к нижнему регистру для одной и той же клавиши

    // Обработка для специальных клавиш, которые могут отличаться
    switch (true) {
      case e.key === 'Enter': // Завершить обрезку
        this.crop();
        break;
      case e.key === 'Escape': // Очистить область обрезки
        this.clear();
        break;
      case e.key === 'Delete': // Удалить изображение
        this.reset();
        break;

      case e.key === 'ArrowLeft': // Движение влево
        e.preventDefault();
        cropper.move(-1, 0);
        break;
      case e.key === 'ArrowUp': // Движение вверх
        e.preventDefault();
        cropper.move(0, -1);
        break;
      case e.key === 'ArrowRight': // Движение вправо
        e.preventDefault();
        cropper.move(1, 0);
        break;
      case e.key === 'ArrowDown': // Движение вниз
        e.preventDefault();
        cropper.move(0, 1);
        break;

      case key === 'c': // Включить режим обрезки
        cropper.setDragMode('crop');
        break;
      case key === 'm': // Включить режим перемещения
        cropper.setDragMode('move');
        break;

      case key === 'i': // Увеличить
        cropper.zoom(0.1);
        break;
      case key === 'o': // Уменьшить
        cropper.zoom(-0.1);
        break;

      case key === 'l': // Повернуть влево
        cropper.rotate(-90);
        break;
      case key === 'r': // Повернуть вправо
        cropper.rotate(90);
        break;

      case key === 'h': // Отразить по горизонтали
        cropper.scaleX(-cropper.getData().scaleX || -1);
        break;
      case key === 'v': // Отразить по вертикали
        cropper.scaleY(-cropper.getData().scaleY || -1);
        break;

      case key === 'z' && e.ctrlKey: // Отмена (Ctrl + Z)
        e.preventDefault();
        this.restore();
        break;
    }
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
          url: cropper.getCroppedCanvas(data.type === 'image/png' ? {} : { fillColor: '#fff' }).toDataURL(data.type),
        });
        this.stop();
      }
    },

    clear() {
      if (this.data.cropping) {
        this.cropper.clear();
        this.update({ cropping: false });
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

<template lang="html">
  <div class="d-flex">
  </div>
</template>
<script>
import { render } from "vue";

var overlayRef = null;

// const DraggableValue = defineComponent((props, context) => {
//   return h(
//     `div`,
//     context.attrs,
//     context.$slots.default({
//       value: props?.value,
//     })
//   );
// });

export default {
  props: {
    values: {
      type: Array,
      default: function () {
        return [];
      },
    },
  },
  data() {
    return {
      valueRefs: {},
    };
  },
  mounted() {
    this.initOverlayIfNotPresent();
    this.initValues();
  },
  watch: {
    values() {
      this.initValues();
    },
  },
  methods: {
    initOverlayIfNotPresent() {
      if (overlayRef) {
        return;
      }
      overlayRef = document.createElement("div");
      overlayRef.style.height = "100%";
      overlayRef.style.width = "100%";
      overlayRef.style.position = "fixed";
      overlayRef.style.top = "0";
      overlayRef.style.right = "0";
      overlayRef.style.backgroundColor = "#00ffff2b";
      overlayRef.style.zIndex = -1000;
      document.body.appendChild(overlayRef);
    },
    initValues() {
      this.values.forEach((value) => {
        let dragging = false;
        // console.log(this._.appContext);
        const draggableValueNode = this.$slots.default({
          value,
        })[0];
        draggableValueNode.appContext = this._.appContext;
        const draggableValueElement = document.createElement("div");
        render(draggableValueNode, draggableValueElement);
        // const draggableValue = new DraggableValue({
        //   propsData: { value },
        // });
        // draggableValue.$slots.default = this.$slots.default;
        // draggableValue.$mount();
        this.$el.appendChild(draggableValueElement);
        const onDraggedElementMove = (event) => {
          if (!dragging) {
            return;
          }
          draggableValueElement.style.left = `${event.x}px`;
          draggableValueElement.style.top = `${event.y}px`;
        };
        draggableValueElement.onmousedown = (event) => {
          dragging = true;

          const heightBeforeDragging = draggableValueElement.clientHeight;
          const widthBeforeDragging = draggableValueElement.clientWidth;

          this.$el.removeChild(draggableValueElement);

          draggableValueElement.style.height = `${heightBeforeDragging}px`;
          draggableValueElement.style.width = `${widthBeforeDragging}px`;
          draggableValueElement.style.position = "absolute";
          draggableValueElement.style.left = `${event.x}px`;
          draggableValueElement.style.top = `${event.y}px`;

          overlayRef.style.zIndex = 1000;
          overlayRef.appendChild(draggableValueElement);
          overlayRef.addEventListener("mousemove", onDraggedElementMove, true);
        };
        draggableValueElement.onmouseup = () => {
          overlayRef.removeEventListener(
            "mousemove",
            onDraggedElementMove,
            true
          );
          overlayRef.removeChild(draggableValueElement);
          overlayRef.style.zIndex = -1000;

          draggableValueElement.style.height = null;
          draggableValueElement.style.width = null;
          draggableValueElement.style.position = null;
          draggableValueElement.style.left = null;
          draggableValueElement.style.top = null;

          this.$el.appendChild(draggableValueElement);

          dragging = false;
        };
      });
    },
  },
};
</script>
<style lang="css">
</style>
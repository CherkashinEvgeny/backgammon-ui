<template lang="html">
  <div class="d-flex">
  </div>
</template>
<script>
import { render } from "vue";

var overlayRef = null;

const renderSlot = (slot, { props, context }) => {
  const rootElement = document.createElement("div");
  const vNodes = slot(props);
  const elements = [];
  vNodes.forEach((vNode) => {
    vNode.appContext = context;
    const element = document.createElement("div");
    elements.push(element);
    render(vNode, element);
    rootElement.appendChild(element);
  });
  const destroy = () => {
    elements.forEach((element) => {
      render(null, element);
    });
  };
  return { element: rootElement, destroy };
};

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
        const { element } = renderSlot(this.$slots.default, {
          props: { value },
          context: this._.appContext,
        });
        this.$el.appendChild(element);
        const onDraggedElementMove = (event) => {
          if (!dragging) {
            return;
          }
          element.style.left = `${event.x}px`;
          element.style.top = `${event.y}px`;
        };
        element.onmousedown = (event) => {
          dragging = true;

          const heightBeforeDragging = element.clientHeight;
          const widthBeforeDragging = element.clientWidth;

          this.$el.removeChild(element);

          element.style.height = `${heightBeforeDragging}px`;
          element.style.width = `${widthBeforeDragging}px`;
          element.style.position = "absolute";
          element.style.left = `${event.x}px`;
          element.style.top = `${event.y}px`;

          overlayRef.style.zIndex = 1000;
          overlayRef.appendChild(element);
          overlayRef.addEventListener("mousemove", onDraggedElementMove, true);
        };
        element.onmouseup = () => {
          overlayRef.removeEventListener(
            "mousemove",
            onDraggedElementMove,
            true
          );
          overlayRef.removeChild(element);
          overlayRef.style.zIndex = -1000;

          element.style.height = null;
          element.style.width = null;
          element.style.position = null;
          element.style.left = null;
          element.style.top = null;

          this.$el.appendChild(element);

          dragging = false;
        };
      });
    },
  },
};
</script>
<style lang="css">
</style>
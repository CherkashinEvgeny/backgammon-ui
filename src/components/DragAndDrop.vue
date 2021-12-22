<template lang="html">
  <div :class="classes">
  </div>
</template>
<script>
import { render } from "vue";

let overlayRef = null;

let context = {
  containers: null,
  draggingValueRef: null,
};

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

const insertChildAtIndex = (container, child, index) => {
  if (!index) index = 0;
  if (index >= container.children.length) {
    container.appendChild(child);
  } else {
    container.insertBefore(child, container.children[index]);
  }
};

const isInBox = ({ x, y }, { left, right, top, bottom }) => {
  return left < x && x < right && top < y && y < bottom;
};

const indetBox = ({ left, right, top, bottom }, indent) => {
  return {
    left: left - indent,
    right: right + indent,
    top: top - indent,
    bottom: bottom + indent,
  };
};

export default {
  props: {
    context: {
      type: Object,
      default: function () {
        return {
          containers: null,
          draggingValueRef: null,
        };
      },
    },
    orientation: {
      type: String,
      validator: function (value) {
        return ["vertical", "horizontal"].indexOf(value) !== -1;
      },
      default: "vertical",
    },
    indent: {
      type: Number,
      default: 20,
    },
    dropPosition: {
      type: Function,
    },
    values: {
      type: Array,
      default: function () {
        return [];
      },
    },
  },
  data() {
    return {
      valueRefs: [],
      valueElements: new Map(),
      placeholder: null,
      dropPositionFunc:
        this.dropPosition ||
        (({ elementPosition, containerBox, valuesBoxes }) => {
          if (
            !isInBox(elementPosition, indetBox(containerBox, this.indent))
          ) {
            return null;
          }
          let position = 0;
          valuesBoxes?.every((valueBox) => {
            position += 1;
            if (this.orientation === "vertical") {
              const isLowerThanValueBox = valueBox.top < elementPosition.y;
              if (isLowerThanValueBox) {
                position += 1;
              }
              return isLowerThanValueBox;
            } else if (this.orientation === "horizontal") {
              const isMoreRightThanValueBox = valueBox.left < elementPosition.x;
              if (isMoreRightThanValueBox) {
                position += 1;
              }
              return isMoreRightThanValueBox;
            }
            return false;
          });
          return position;
        }),
    };
  },
  mounted() {
    this.initOverlayIfNotPresent();
    this.registerContainer();
    this.setValueRefs();
  },
  watch: {
    values() {
      this.setValueRefs();
    },
  },
  computed: {
    classes() {
      if (this.orientation === "vertical") {
        return ["d-flex", "flex-column"].join(" ");
      }
      if (this.orientation === "horizontal") {
        return ["d-flex", "flex-row"].join(" ");
      }
      return ["d-flex"].join(" ");
    },
  },
  methods: {
    registerContainer() {
      if (!context.containers) {
        context.containers = [];
      }
      context.containers.push(this);
    },
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
    setValueRefs() {
      this.valueRefs = this.values?.map((value) => ({ value }));
      this.updateView();
    },
    updateView() {
      this.updateElements();
      this.$el.innerHTML = "";
      this.valueRefs?.forEach((value) => {
        this.$el.appendChild(this.valueElements.get(value));
      });
    },
    updateElements() {
      const oldValueElements = this.valueElements;
      this.valueElements = new Map();
      this.valueRefs?.forEach((valueRef) => {
        if (oldValueElements?.has(valueRef)) {
          this.valueElements.set(valueRef, oldValueElements?.get(valueRef));
        } else {
          this.valueElements.set(valueRef, this.createElement(valueRef));
        }
      });
    },
    createElement(valueRef) {
      const { element, destroy } = renderSlot(this.$slots.default, {
        props: valueRef,
        context: this._.appContext,
      });
      this.$el.appendChild(element);
      const onElementMove = (event) => {
        event.preventDefault();
        element.style.top = `${event.y - element.clientHeight / 2}px`;
        element.style.left = `${event.x - element.clientWidth / 2}px`;

        context.containers?.forEach((container) => {
          container.onElementMove(event);
        });
      };
      element.addEventListener("mousedown", (event) => {
        event.preventDefault();
        context.draggingValueRef = valueRef;
        const heightBeforeDragging = element.clientHeight;
        const widthBeforeDragging = element.clientWidth;

        this.valueRefs?.splice(this.valueRefs?.indexOf(valueRef), 1);
        this.updateView();
        this.valuesChanged();

        element.style.height = `${heightBeforeDragging}px`;
        element.style.width = `${widthBeforeDragging}px`;
        element.style.position = "absolute";
        element.style.top = `${event.y - heightBeforeDragging / 2}px`;
        element.style.left = `${event.x - widthBeforeDragging / 2}px`;

        overlayRef.style.zIndex = 1000;
        overlayRef.appendChild(element);
        overlayRef.addEventListener("mousemove", onElementMove, true);

        context.containers?.forEach((container) => {
          container.onElementDrag(event);
        });
      });
      element.addEventListener("mouseup", (event) => {
        event.preventDefault();
        overlayRef.removeEventListener("mousemove", onElementMove, true);
        overlayRef.removeChild(element);
        overlayRef.style.zIndex = -1000;

        destroy();

        context.containers?.forEach((container) => {
          container.onElementDrop(event);
        });

        context.draggingValueRef = null;
      });
      return element;
    },
    showPlaceholder(position) {
      if (this.placeholder && this.placeholder.position === position) {
        return;
      }
      this.removePlaceholder();
      if (position === null || position === undefined) {
        return;
      }
      let element = null;
      let destroy = null;
      if (this.$slots.placeholder) {
        const renderedSlot = renderSlot(this.$slots.placeholder, {
          props: context.draggingValueRef,
          context: this._.appContext,
        });
        element = renderedSlot.element;
        destroy = renderedSlot.destroy;
      } else {
        element = document.createElement("div");
        element.style.height = "20px";
        element.style.width = "20px";
        element.style.background = "#ccc"
        destroy = () => {};
      }
      insertChildAtIndex(this.$el, element, position);
      this.placeholder = { element, destroy, position };
    },
    removePlaceholder() {
      if (this.placeholder) {
        this.$el.removeChild(this.placeholder.element);
        this.placeholder.destroy();
      }
    },
    onElementDrag(event) {
      const containerBox = this.$el.getBoundingClientRect();
      const valuesBoxes = this.valueRefs?.map((valueRef) =>
        this.valueElements?.get(valueRef)?.getBoundingClientRect()
      );
      const position = this.dropPositionFunc({
        elementPosition: event,
        containerBox,
        valuesBoxes,
      });
      this.showPlaceholder(position);
    },
    onElementMove(event) {
      const containerBox = this.$el.getBoundingClientRect();
      const valuesBoxes = this.valueRefs?.map((valueRef) =>
        this.valueElements?.get(valueRef)?.getBoundingClientRect()
      );
      const position = this.dropPositionFunc({
        elementPosition: event,
        containerBox,
        valuesBoxes,
      });
      this.showPlaceholder(position);
    },
    onElementDrop(event) {
      if (!this.placeholder) {
        return;
      }
      const placeholderBox = this.placeholder.element.getBoundingClientRect();
      if (!isInBox(event, indetBox(placeholderBox, this.indent))) {
        return;
      }
      this.removePlaceholder();
      this.valueRefs.splice(
        this.placeholder?.position,
        0,
        context.draggingValueRef
      );
      this.updateView();
      this.valuesChanged();
    },
    valuesChanged() {
      this.$emit(
        "update:values",
        this.valueRefs?.map((valueRef) => valueRef?.value)
      );
    },
  },
};
</script>
<style lang="css">
</style>
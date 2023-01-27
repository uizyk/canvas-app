<template>
  <div id="container">
    <section id="side-menu" ref="sideMenu">
      <h2>Shapes</h2>
      <div id="shapes">
        <img
          id="imgCircle"
          src="./assets/shapes/circle.svg"
          draggable="true"
          data-shape="circle"
          @dragstart="handleDragStart"
          @dragend="handleDrop"
        />
        <img
          id="imgSquare"
          src="./assets/shapes/square.svg"
          draggable="true"
          data-shape="square"
          @dragstart="handleDragStart"
          @dragend="handleDrop"
        />
        <img
          id="imgHexagon"
          src="./assets/shapes/hexagon.svg"
          draggable="true"
          data-shape="hexagon"
          @dragstart="handleDragStart"
          @dragend="handleDrop"
        />
        <img
          id="imgTriangle"
          src="./assets/shapes/triangle.svg"
          draggable="true"
          data-shape="triangle"
          @dragstart="handleDragStart"
          @dragend="handleDrop"
        />
      </div>
      <div id="color-picker">
        <p v-if="selectedShapeName !== ''" type="color">Change Color</p>
        <input
          v-if="selectedShapeName !== ''"
          type="color"
          v-model="selectedColor"
          @input="handleColorChange"
        />
      </div>
    </section>
    <section id="canvas" ref="canvas">
      <v-stage
        :config="configCanvas"
        @mousedown="handleStageMouseDown"
        @touchstart="handleStageMouseDown"
        >`
        <v-layer ref="layer">
          <v-circle
            v-for="shape in filteredCircleShapes"
            :x="shape.x"
            :y="shape.y"
            :scaleX="shape.scaleX"
            :scaleY="shape.scaleY"
            :config="configCircle"
            :name="shape.name"
            :key="shape.id"
            :fill="shape.fill"
            @dblclick="removeShape(shape)"
            @transformend="handleTransformEnd"
          >
          </v-circle>
          <v-rect
            v-for="shape in filteredSquareShapes"
            :x="shape.x"
            :y="shape.y"
            :scaleX="shape.scaleX"
            :scaleY="shape.scaleY"
            :config="configSquare"
            :name="shape.name"
            :key="shape.id"
            :fill="shape.fill"
            @dblclick="removeShape(shape)"
            @transformend="handleTransformEnd"
          >
          </v-rect>
          <v-regular-polygon
            v-for="shape in filteredHexagonShapes"
            :x="shape.x"
            :y="shape.y"
            :scaleX="shape.scaleX"
            :scaleY="shape.scaleY"
            :config="configHexagon"
            :name="shape.name"
            :key="shape.id"
            :fill="shape.fill"
            @dblclick="removeShape(shape)"
            @transformend="handleTransformEnd"
          >
          </v-regular-polygon>
          <v-regular-polygon
            v-for="shape in filteredTriangleShapes"
            :x="shape.x"
            :y="shape.y"
            :scaleX="shape.scaleX"
            :scaleY="shape.scaleY"
            :config="configTriangle"
            :name="shape.name"
            :key="shape.id"
            :fill="shape.fill"
            @dblclick="removeShape(shape)"
            @transformend="handleTransformEnd"
          ></v-regular-polygon>
          <v-transformer ref="transformer" />
        </v-layer>
      </v-stage>
    </section>
  </div>
</template>

<script>
export default {
  data() {
    return {
      selectedColor: "#000000",
      shapes: [],
      selectedShapeName: "",
      configCanvas: {
        name: "canvas",
        width: window.innerWidth,
        height: 1200,
      },
      configCircle: {
        radius: 80,
        draggable: true,
        transformer: {
          enabledAnchors: [
            "top-left",
            "top-right",
            "bottom-left",
            "bottom-right",
          ],
        },
      },
      configSquare: {
        width: 150,
        height: 150,
        draggable: true,
      },
      configHexagon: {
        sides: 6,
        radius: 80,
        draggable: true,
      },
      configTriangle: {
        sides: 3,
        radius: 80,
        draggable: true,
      },
    };
  },
  computed: {
    filteredCircleShapes() {
      return this.shapes.filter((shape) => shape.type === "circle");
    },
    filteredSquareShapes() {
      return this.shapes.filter((shape) => shape.type === "square");
    },
    filteredHexagonShapes() {
      return this.shapes.filter((shape) => shape.type === "hexagon");
    },
    filteredTriangleShapes() {
      return this.shapes.filter((shape) => shape.type === "triangle");
    },
  },
  methods: {
    // ADD SHAPE
    handleDragStart(e) {
      const shapeType = e.target.getAttribute("data-shape");
      this.currentShape = shapeType;
    },
    handleDrop(e) {
      e.preventDefault();
      const newShape = {
        name: Date.now().toString(),
        id: Date.now(),
        x: e.clientX,
        y: e.clientY,
        scaleX: 1,
        scaleY: 1,
        fill: "dodgerblue",
        type: this.currentShape,
      };
      this.shapes.push(newShape);
      console.log(this.$refs.sideMenu.clientWidth);
    },
    // DELETE SHAPE
    removeShape(shape) {
      const index = this.shapes.indexOf(shape);
      this.shapes.splice(index, 1);
      this.$refs.transformer.getNode().nodes([])
    },
    // TRANSFORM SHAPE
    handleTransformEnd(e) {
      const shape = this.shapes.find(
        (shape) => shape.name === this.selectedShapeName
      );

      shape.x = e.target.x();
      shape.y = e.target.y();
      shape.rotation = e.target.rotation();
      shape.scaleX = e.target.scaleX();
      shape.scaleY = e.target.scaleY();
    },
    handleStageMouseDown(e) {
      // clicked on stage - clear selection
      if (e.target === e.target.getStage()) {
        this.selectedShapeName = "";
        this.updateTransformer();
        return;
      }
      // clicked on transformer - do nothing
      const clickedOnTransformer =
        e.target.getParent().className === "Transformer";
      if (clickedOnTransformer) {
        return;
      }
      // find clicked rect by its name
      const name = e.target.name();
      const shape = this.shapes.find((shape) => shape.name === name);
      if (shape) {
        this.selectedShapeName = name;
      } else {
        this.selectedShapeName = "";
      }
      this.updateTransformer();
      console.log(e.target.name());
    },
    updateTransformer() {
      // here we need to manually attach or detach Transformer node
      const transformerNode = this.$refs.transformer.getNode();
      const stage = transformerNode.getStage();
      const { selectedShapeName } = this;
      const selectedNode = stage.findOne("." + selectedShapeName);
      // do nothing if selected node is already attached
      if (selectedNode === transformerNode.node()) {
        return;
      }
      if (selectedNode) {
        // attach to another node
        transformerNode.nodes([selectedNode]);
      } else {
        // remove transformer
        transformerNode.nodes([]);
      }
    },
    // COLOR CHANGE
    handleColorChange(e) {
      const newColor = e.target.value;
      // Find the selected shape in the shapes array
      const selectedShape = this.shapes.find(
        (shape) => shape.name === this.selectedShapeName
      );
      // Update the shape's fill property
      selectedShape.fill = newColor;
    },
  },
  mounted() {
    this.configCanvas.width = this.$refs.canvas.clientWidth;
    this.configCanvas.height = this.$refs.canvas.clientHeight;
  },
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
}

#container {
  display: flex;
}

#side-menu {
  /* min-width: 30vw; */
  background-color: rgb(243, 225, 255);
  position: absolute;
  z-index: 10;
  display: flex;
  flex-direction: column;
  width: 10vw;
  height: 100vh;
  align-items: center;
}

#side-menu #shapes {
  display: flex;
  flex-wrap: wrap;
  padding-top: 20px;
  justify-content: center;
}

#side-menu h2 {
  color: black;
  padding-top: 10px;
  font-size: 2em;
  text-decoration: underline;
}


#shapes img {
  width: 50px;
  height: 50px;
  padding: 8px;
}

#color-picker {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-top: 30px;
}

#color-picker p {
  color: black;
}
</style>

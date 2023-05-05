<script setup lang="ts">
import { Bodies, Body, Composite, Engine, Mouse, MouseConstraint, Render, Runner } from "matter-js";
import { ref } from "vue";

const container = ref<HTMLCanvasElement>();
const screenWidth = window.innerWidth;
const screenHeight = window.innerHeight;

function startAimation() {
  if (!container.value) return;

  container.value.innerHTML = "";

  const engine = Engine.create({
    gravity: {
      y: 1,
    },
  });

  const render = Render.create({
    element: container.value,
    engine: engine,
    options: {
      width: screenWidth,
      height: screenHeight,
      wireframes: false,
      background: "white",
    },
  });

  const boundsOptions = { isStatic: true, render: { visible: false } };
  const ground = Bodies.rectangle(screenWidth / 2, screenHeight + 30, screenWidth, 60, boundsOptions);
  const leftSide = Bodies.rectangle(0 - 30, screenHeight / 2, 60, screenHeight, boundsOptions);

  const rightSide = Bodies.rectangle(screenWidth + 30, screenHeight / 2, 60, screenHeight, boundsOptions);

  const circles = [];
  const columnOffset = 30;
  const rowOffset = 30;

  for (let x = 0; x < 20; x++) {
    for (let y = 0; y < x; y++) {
      //const offset = x % 2 === 0 ? 0 : 10;
      circles.push(
        Bodies.circle(
          screenWidth / 2 + y * columnOffset - (x * columnOffset) / 2 + columnOffset / 2,
          100 + x * rowOffset,
          5,
          {
            isStatic: true,
            friction: 0,
          }
        )
      );
    }
  }

  circles.shift();
  circles.shift();
  circles.shift();

  Promise.all(
    Array.from({ length: 20 }, (_, colIndex) => {
      for (let index = 0; index < 500; index++) {
        circles.push(
          Bodies.circle(screenWidth / 2 - 60 + colIndex * 10, -50 * index, 1, {
            friction: 0,
            restitution: 0.7 + 0.2 * Math.random(),
          })
        );
      }
    })
  );

  const floor = Bodies.rectangle(200, 800, screen.width, 10, {
    isStatic: true,
  });

  const borderLeft = Bodies.rectangle(screenWidth / 3.6, 400, 10, 640, {
    isStatic: true,
  });
  Body.rotate(borderLeft, 0.465);

  const borderRight = Bodies.rectangle(screenWidth - screenWidth / 3.6, 400, 10, 640, {
    isStatic: true,
  });
  Body.rotate(borderRight, -0.465);

  const walls = Bodies.rectangle(screenWidth / 2, 750, 2, 100, {
    isStatic: true,
  });

  // add all of the bodies to the world
  Composite.add(engine.world, [
    ground,
    leftSide,
    rightSide,
    ...circles,
    floor,
    walls,
    borderLeft,
    borderRight,
  ]);

  // run the renderer
  Render.run(render);

  // create runner
  const runner = Runner.create();

  // add mouse control
  var mouse = Mouse.create(render.canvas),
    mouseConstraint = MouseConstraint.create(engine, {
      mouse: mouse,
      constraint: {
        stiffness: 0.2,
        render: {
          visible: false,
        },
      },
    });
  Composite.add(engine.world, mouseConstraint);
  render.mouse = mouse;

  // run the engine
  Runner.run(runner, engine);
}
</script>

<template>
  <button @click="startAimation">START</button>
  <div ref="container" class="canvas"></div>
</template>

<style>
body {
  width: 100%;
  height: 100vh;
  overflow: hidden;
}

.canvas {
  width: 100%;
  height: 100%;
}

.ui {
  opacity: 80%;
  display: flex;
  justify-content: space-around;
  position: absolute;
  bottom: 10%;
  left: 0;
  width: 100%;
}
</style>

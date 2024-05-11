<script>
  // @ts-nocheck
  import { onMount, onDestroy } from "svelte";

  import { Stage, Layer, Rect, Circle } from "svelte-konva";

  const LEFT_KEY = 37;
  const RIGHT_KEY = 39;
  const UP_KEY = 38;
  const DOWN_KEY = 40;
  const MULTIPLIER = 25;
  const BLOCK_SIZE = 20;
  const STAGE_WIDTH = BLOCK_SIZE * MULTIPLIER; // 500px
  const STAGE_HEIGHT = BLOCK_SIZE * MULTIPLIER; // 500px
  const TARGET_FILL = "red";
  const BLOCK_FILL = "#8D7328";
  const CANVAS_FILL = "#72B76A";

  let is_target_visible = false;

  let target = { x: 0, y: 0 };
  let head = { x: 0, y: 0 };
  let tails = [];
  let direction = "right";
  let game_over = false;
  let interval;
  let started = false;

  let canvas_config = {
    width: STAGE_WIDTH,
    height: STAGE_HEIGHT,
    x: 0,
    y: 0,
    fill: CANVAS_FILL,
  };

  const get_random_value = () => {
    const items = [];
    for (let i = 0; i < MULTIPLIER; i++) {
      if (!items.length) {
        items.push(0);
      } else {
        items.push(items[items.length - 1] + BLOCK_SIZE);
      }
    }
    return items[Math.floor(Math.random() * items.length)];
  };

  const get_random_coordinate = () => {
    const coordinate = {};
    coordinate.x = get_random_value();
    coordinate.y = get_random_value();
    return coordinate;
  };

  const spawn_snake = () => {
    head = get_random_coordinate();
  };

  const update_tail = (prev) => {
    if (tails.length) {
      const items = [prev];

      const [head, ...tail] = tails;

      let last = head;

      for (let i = 0; i < tail.length; i++) {
        items.push({
          x: last.x,
          y: last.y,
        });
        last = tail[i];
      }

      tails = items;
    }
  };

  const update = () => {
    if (game_over) {
      reset_game();
      start_game();
      return;
    }

    const prev = { ...head };

    if (direction === "right") {
      head = {
        x: head.x + BLOCK_SIZE,
        y: head.y,
      };
    } else if (direction === "down") {
      head = {
        x: head.x,
        y: head.y + BLOCK_SIZE,
      };
    } else if (direction === "up") {
      head = {
        x: head.x,
        y: head.y - BLOCK_SIZE,
      };
    } else if (direction === "left") {
      head = {
        x: head.x - BLOCK_SIZE,
        y: head.y,
      };
    }

    if (head.x === target.x && head.y === target.y) {
      tails = [{ ...target }, ...tails];
      spawn_target();
    }

    update_tail(prev);

    // check if the head hits the wall
    if (
      head.x < 0 ||
      head.y < 0 ||
      head.x > STAGE_WIDTH ||
      head.y > STAGE_HEIGHT
    ) {
      game_over = true;
    }

    if (tails.length < 4) return;

    for (let i = 3; i < tails.length; i++) {
      const tail = tails[i];
      if (head.x === tail.x && head.y === tail.y) {
        game_over = true;
      }
    }
  };

  const spawn_target = () => {
    target = get_random_coordinate();
    is_target_visible = true;
  };

  const handle_keydown = (event) => {
    const keyPressed = event.keyCode;

    if (keyPressed === RIGHT_KEY && direction !== "left") direction = "right";
    if (keyPressed === LEFT_KEY && direction !== "right") direction = "left";
    if (keyPressed === UP_KEY && direction !== "down") direction = "up";
    if (keyPressed === DOWN_KEY && direction !== "up") direction = "down";

    if (
      [RIGHT_KEY, LEFT_KEY, UP_KEY, DOWN_KEY].includes(keyPressed) &&
      !started
    ) {
      return start_interval();
    }
  };

  const reset_game = () => {
    clearInterval(interval);
    target = { x: 0, y: 0 };
    head = { x: 0, y: 0 };
    tails = [];
    is_target_visible = false;
    game_over = false;
    started = false;
  };

  const start_interval = () => {
    started = true;
    interval = setInterval(() => update(), 100);
  };

  const start_game = () => {
    spawn_snake();
    spawn_target();
  };

  const perform_cleanup = () => {
    clearInterval(interval);
  };

  onMount(() => start_game());

  onDestroy(() => perform_cleanup());
</script>

<svelte:window on:keydown={handle_keydown} />

<Stage config={{ width: STAGE_WIDTH, height: STAGE_HEIGHT }}>
  <Layer>
    <Rect config={canvas_config} />
  </Layer>
  <Layer>
    {#if is_target_visible}
      <Rect
        config={{
          ...target,
          width: BLOCK_SIZE,
          height: BLOCK_SIZE,
          fill: TARGET_FILL,
        }}
      />
    {/if}

    <Rect
      config={{
        ...head,
        width: BLOCK_SIZE,
        height: BLOCK_SIZE,
        fill: BLOCK_FILL,
      }}
    />

    {#each tails as tail}
      <Rect
        config={{
          ...tail,
          width: BLOCK_SIZE,
          height: BLOCK_SIZE,
          fill: BLOCK_FILL,
        }}
      />
    {/each}
  </Layer>
</Stage>

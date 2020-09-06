<script>
  let eyeColor = "black";
  let leftEyeX = 115;
  let rightEyeX = 135;
  let leftEyeY = 100;
  let rightEyeY = 100;
  let leftEyeRadius = 5;
  let rightEyeRadius = 5;

  let bodyColor = "#5f517e";
  let svg = null;
  let mouseOffset = null;
  let selectedPincherIndex;
  let pinchers = [
    {
      type: "M",
      cx: 100,
      cy: 100,
      ca: 120,
      cb: 120,
    },
    {
      type: "Q",
      cx: 125,
      cy: 70,
    },
    {
      type: "Q",
      cx: 150,
      cy: 100,
    },
    {
      type: "Q",
      cx: 150,
      cy: 200,
    },
    {
      type: "Q",
      cx: 130,
      cy: 230,
    },
    {
      type: "Q",
      cx: 100,
      cy: 200,
    },
  ];
  let bodyPath = "";

  $: {
    bodyPath = "";
    pinchers.forEach(({ type, cx, cy }) => {
      switch (type) {
        case "M":
          bodyPath = `${bodyPath} ${type} ${cx} ${cy}`;
          break;
        case "Q":
          bodyPath = `${bodyPath} ${type} ${cx + 5} ${cy + 5} ${cx} ${cy}`;
          break;
        case "Z":
          break;
      }
    });
    bodyPath = `${bodyPath} Z`;
  }

  function makeDraggable(e) {
    svg = e.target;
  }

  function startDrag(e, index) {
    selectedPincherIndex = index;
    const { cx, cy } = pinchers[selectedPincherIndex];
    mouseOffset = getMousePosition(e);
    mouseOffset.x -= cx;
    mouseOffset.y -= cy;
  }

  function drag(e) {
    if (typeof selectedPincherIndex === "number") {
      e.preventDefault();
      let mouseCoordinates = getMousePosition(e);

      pinchers[selectedPincherIndex] = {
        ...pinchers[selectedPincherIndex],
        cx: mouseCoordinates.x - mouseOffset.x,
        cy: mouseCoordinates.y - mouseOffset.y,
      };
    }
  }

  function endDrag() {
    selectedPincherIndex = null;
  }

  function getMousePosition(e) {
    let CTM = svg.getScreenCTM();
    return {
      x: (e.clientX - CTM.e) / CTM.a,
      y: (e.clientY - CTM.f) / CTM.d,
    };
  }

  let inner, thiz;

  function exportBlob() {
    var svgData = thiz.outerHTML;
    var preface = '<?xml version="1.0" standalone="no"?>\r\n';
    var svgBlob = new Blob([preface, svgData], {
      type: "image/svg+xml;charset=utf-8",
    });
    var svgUrl = URL.createObjectURL(svgBlob);
    var downloadLink = document.createElement("a");
    downloadLink.href = svgUrl;
    downloadLink.download = name;
    document.body.appendChild(downloadLink);
    downloadLink.click();
    document.body.removeChild(downloadLink);
  }
</script>

<style>
  .container {
    display: grid;
    grid-template-columns: 1fr 1fr;
  }
  .blob {
    background-color: #222;
  }

  .settings {
    display: grid;
    grid-template-columns: 1fr 2fr;
    align-items: center;
    justify-items: start;
  }

  .settings .title {
    grid-column: span 2;
  }

  input[type="color"] {
    width: 100%;
    height: 100%;
  }

  .pincher {
    fill: red;
    fill-opacity: 0.7;
    stroke-opacity: 0;
    stroke-width: 5;
    cursor: move;
  }
</style>

<div class="container">
  <section class="settings">
    <h3 class="title">👁 Eyes</h3>
    <p>Color</p>
    <input type="color" bind:value={eyeColor} />
    <!-- Left Eye -->
    <h4 class="title">⏪ Left</h4>
    <p>X Offset</p>
    <input type="range" bind:value={leftEyeX} min="100" max="130" />

    <p>Y Offset</p>
    <input type="range" bind:value={leftEyeY} min="100" max="130" />

    <p>Size</p>
    <input type="range" bind:value={leftEyeRadius} min="3" max="9" />

    <!-- Right Eye -->
    <h4 class="title">⏩ Right</h4>
    <p>X Offset</p>
    <input type="range" bind:value={rightEyeX} min="120" max="150" />

    <p>Y Offset</p>
    <input type="range" bind:value={rightEyeY} min="100" max="130" />

    <p>Size</p>
    <input type="range" bind:value={rightEyeRadius} min="3" max="9" />

    <h3 class="title">🦶 Body</h3>

    <p>Color</p>
    <input type="color" bind:value={bodyColor} />
  </section>

  <section class="blob">
    <svg
      bind:this={thiz}
      xmlns="http://www.w3.org/2000/svg"
      viewBox="0 0 200 300"
      on:load={makeDraggable}
      on:mousemove={drag}
      on:mouseup={endDrag}
      on:mouseleave={endDrag}>
      <!-- Body -->
      <path stroke="white" stroke-width="1" fill={bodyColor} d={bodyPath} />

      {#each pinchers as pincher, i}
        <circle
          class="pincher"
          on:mousedown={(e) => {
            startDrag(e, i);
          }}
          cx={pincher.cx}
          cy={pincher.cy}
          r="3" />
      {/each}

      <!-- Eyes -->
      <!-- Left eye -->
      <circle cx={leftEyeX} cy={leftEyeY} r={leftEyeRadius} fill="white" />
      <circle
        cx={leftEyeX}
        cy={leftEyeY}
        r={leftEyeRadius - 3}
        fill={eyeColor} />

      <!-- Right eye -->
      <circle cx={rightEyeX} cy={rightEyeY} r={rightEyeRadius} fill="white" />
      <circle
        cx={rightEyeX}
        cy={rightEyeY}
        r={rightEyeRadius - 3}
        fill={eyeColor} />

      <path
        fill="white"
        stroke="white"
        stroke-width="1"
        d="
            m 115 130 
            c 0 0, 10 5, 20 0
          " />
    </svg>
  </section>

  <button on:click={exportBlob}>Export SVG</button>
</div>

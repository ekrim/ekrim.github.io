---
layout: default
title: About
permalink: /
---

I’m a low-level, math-driven systems engineer with a PhD in Machine Learning and 7 years of production C++ experience at Apple. I love the challenge of making models run fast and efficiently on real hardware and I’ve shipped optimized ML features across multiple platforms. Earlier this year I shifted my full attention to model inference, combining my ML background with years of shipping performant systems at scale. I’ve been deep in GPU optimization ever since.

Prior to Apple, I worked on vision, tracking, and generative AI problems, implementing papers and writing training pipelines (some trace of that era is on my Github). My PhD was on unsupervised and active learning, but included some reinforcement and deep learning.

**Seeking opportunities!**

I'm now actively seeking roles in SF or remote, preferably in the domain of AI/ML. Please reach out via email.

- [Resume](/assets/Kriminger_Resume.pdf)
- [Github](https://github.com/ekrim)

<div id="matmul-viz" style="margin-top: 60px; text-align: center;">
  <h3 style="margin-bottom: 4px;">CUDA Tiled Matmul</h3>
</div>

<script src="https://d3js.org/d3.v7.min.js"></script>
<script>
// Tiled Matrix Multiplication Visualization - CTA-level view
(function() {
  const width = Math.min(1000, window.innerWidth - 40);
  const height = 400;

  // Matrix dimensions (total)
  const M = 12, K = 12, N = 12;

  // Block dimensions (what one CTA computes)
  const BM = 6, BN = 6, BK = 4;

  // Thread output dimensions
  const TM = 2, TN = 2;

  // Adjust cell size and spacing for mobile
  const isMobile = width < 600;
  const cellSize = isMobile ? 12 : 20;
  const spacing = isMobile ? 20 : 40;

  const svg = d3.select("#matmul-viz")
    .append("svg")
    .attr("width", width)
    .attr("height", height)
    .attr("viewBox", `0 0 ${width} ${height}`)
    .attr("style", "max-width: 100%; height: auto;");

  // Colors
  const colorA = "#ff6b6b"; // Red for A
  const colorB = "#4ecdc4"; // Blue for B
  const colorC = "#9b59b6"; // Purple for C

  // Position matrices
  const matrixAX = isMobile ? 10 : 50;
  const matrixAY = 100;
  const matrixBX = matrixAX + K * cellSize + spacing;
  const matrixBY = matrixAY;
  const matrixCX = matrixBX + N * cellSize + spacing;
  const matrixCY = matrixAY;

  // Create Matrix A
  const matrixAGroup = svg.append("g")
    .attr("transform", `translate(${matrixAX}, ${matrixAY})`);

  matrixAGroup.append("text")
    .attr("x", (K * cellSize) / 2)
    .attr("y", -10)
    .attr("text-anchor", "middle")
    .attr("font-size", isMobile ? "14px" : "18px")
    .attr("font-weight", "bold")
    .text("A");

  matrixAGroup.append("rect")
    .attr("class", "matrix-a-full")
    .attr("x", 0)
    .attr("y", 0)
    .attr("width", K * cellSize)
    .attr("height", M * cellSize)
    .attr("fill", colorA)
    .attr("opacity", 0.1)
    .attr("stroke", "#333")
    .attr("stroke-width", 2);

  // Create Matrix B
  const matrixBGroup = svg.append("g")
    .attr("transform", `translate(${matrixBX}, ${matrixBY})`);

  matrixBGroup.append("text")
    .attr("x", (N * cellSize) / 2)
    .attr("y", -10)
    .attr("text-anchor", "middle")
    .attr("font-size", isMobile ? "14px" : "18px")
    .attr("font-weight", "bold")
    .text("B");

  matrixBGroup.append("rect")
    .attr("class", "matrix-b-full")
    .attr("x", 0)
    .attr("y", 0)
    .attr("width", N * cellSize)
    .attr("height", K * cellSize)
    .attr("fill", colorB)
    .attr("opacity", 0.1)
    .attr("stroke", "#333")
    .attr("stroke-width", 2);

  // Create Matrix C
  const matrixCGroup = svg.append("g")
    .attr("transform", `translate(${matrixCX}, ${matrixCY})`);

  matrixCGroup.append("text")
    .attr("x", (N * cellSize) / 2)
    .attr("y", -10)
    .attr("text-anchor", "middle")
    .attr("font-size", isMobile ? "14px" : "18px")
    .attr("font-weight", "bold")
    .text("C = A * B");

  matrixCGroup.append("rect")
    .attr("class", "matrix-c-full")
    .attr("x", 0)
    .attr("y", 0)
    .attr("width", N * cellSize)
    .attr("height", M * cellSize)
    .attr("fill", colorC)
    .attr("opacity", 0.1)
    .attr("stroke", "#333")
    .attr("stroke-width", 2);

  // Draw block divisions on C (4 blocks: 2x2 grid)
  matrixCGroup.append("line")
    .attr("x1", 0).attr("y1", M * cellSize / 2)
    .attr("x2", N * cellSize).attr("y2", M * cellSize / 2)
    .attr("stroke", "#666").attr("stroke-width", 1).attr("stroke-dasharray", "3,3");

  matrixCGroup.append("line")
    .attr("x1", N * cellSize / 2).attr("y1", 0)
    .attr("x2", N * cellSize / 2).attr("y2", M * cellSize)
    .attr("stroke", "#666").attr("stroke-width", 1).attr("stroke-dasharray", "3,3");

  // Active block in C (bottom-left)
  const blockCX = 0;
  const blockCY = M * cellSize / 2;

  const blockCRect = matrixCGroup.append("rect")
    .attr("class", "block-c")
    .attr("x", blockCX)
    .attr("y", blockCY)
    .attr("width", BN * cellSize)
    .attr("height", BM * cellSize)
    .attr("fill", colorC)
    .attr("opacity", 0)
    .attr("stroke", "#333")
    .attr("stroke-width", 2);

  // Block tiles for A and B (will be animated)
  const blockATile = matrixAGroup.append("rect")
    .attr("class", "block-a-tile")
    .attr("x", 0)
    .attr("y", blockCY)
    .attr("width", BK * cellSize)
    .attr("height", BM * cellSize)
    .attr("fill", colorA)
    .attr("opacity", 0)
    .attr("stroke", "#333")
    .attr("stroke-width", 2);

  const blockBTile = matrixBGroup.append("rect")
    .attr("class", "block-b-tile")
    .attr("x", blockCX)
    .attr("y", 0)
    .attr("width", BN * cellSize)
    .attr("height", BK * cellSize)
    .attr("fill", colorB)
    .attr("opacity", 0)
    .attr("stroke", "#333")
    .attr("stroke-width", 2);

  // Thread output square in C
  const threadCSquare = matrixCGroup.append("rect")
    .attr("class", "thread-c")
    .attr("x", blockCX)
    .attr("y", blockCY)
    .attr("width", TN * cellSize)
    .attr("height", TM * cellSize)
    .attr("fill", colorC)
    .attr("opacity", 0)
    .attr("stroke", "#000")
    .attr("stroke-width", 1.5);

  // Thread registers (partial column from A, partial row from B)
  const threadARect = matrixAGroup.append("rect")
    .attr("class", "thread-a")
    .attr("x", 0)
    .attr("y", blockCY)
    .attr("width", cellSize)
    .attr("height", TM * cellSize)
    .attr("fill", colorA)
    .attr("opacity", 0)
    .attr("stroke", "#000")
    .attr("stroke-width", 1.5);

  const threadBRect = matrixBGroup.append("rect")
    .attr("class", "thread-b")
    .attr("x", blockCX)
    .attr("y", 0)
    .attr("width", TN * cellSize)
    .attr("height", cellSize)
    .attr("fill", colorB)
    .attr("opacity", 0)
    .attr("stroke", "#000")
    .attr("stroke-width", 1.5);

  // Status text
  const statusText = svg.append("text")
    .attr("x", width / 2)
    .attr("y", 30)
    .attr("text-anchor", "middle")
    .attr("font-size", isMobile ? "12px" : "24px")
    .attr("font-weight", "normal")
    .text("");

  // Animation state
  let phase = 0; // 0: load tiles, 1-BK: compute steps, then repeat
  let tileK = 0; // Which tile along K dimension (0, 1, 2)
  const numKTiles = K / BK; // 3 tiles

  function animate() {
    let duration = 1000;

    if (phase === 0) {
      // Phase 0: Load tiles from global to shared memory
      duration = 4000; // Stay longer on this phase (4 seconds)
      statusText.text("Loading tiles from global to shared memory");

      blockATile
        .transition().duration(duration)
        .attr("opacity", 0.4);

      blockBTile
        .transition().duration(duration)
        .attr("opacity", 0.4);

      // Fade in C block tile after a delay
      blockCRect
        .transition().delay(duration * 0.5).duration(duration * 0.5)
        .attr("opacity", 0.3);

      threadCSquare.attr("opacity", 0);
      threadARect.attr("opacity", 0);
      threadBRect.attr("opacity", 0);

      phase++;
    } else if (phase === 1) {
      // Phase 1: Start computing - show thread output and initial registers
      statusText.text("One thread's output: accumulate outer products in registers");

      // C block should already be visible from phase 0, just ensure it's at 0.3
      threadCSquare
        .attr("opacity", 0.7);

      threadARect
        .attr("x", tileK * BK * cellSize)
        .attr("y", blockCY)
        .attr("opacity", 0.7);

      threadBRect
        .attr("x", blockCX)
        .attr("y", tileK * BK * cellSize)
        .attr("opacity", 0.7);

      phase++;
      // Skip the setTimeout and immediately continue to next phase
      setTimeout(animate, 0);
      return;
    } else if (phase >= 2 && phase < 2 + BK) {
      // Phases 2 to (2+BK-1): Slide thread registers through the K dimension
      const k = phase - 2;

      threadARect
        .transition().duration(duration)
        .attr("x", tileK * BK * cellSize + k * cellSize);

      threadBRect
        .transition().duration(duration)
        .attr("y", tileK * BK * cellSize + k * cellSize);

      phase++;
    } else {
      // After sliding, hide thread rectangles before moving tiles
      threadARect.attr("opacity", 0);
      threadBRect.attr("opacity", 0);

      // Move to next tile
      tileK++;

      if (tileK >= numKTiles) {
        // Finished all tiles, pause at the end then reset
        setTimeout(() => {
          // Reset everything to initial state
          tileK = 0;
          phase = 0;

          blockATile
            .attr("x", 0)
            .attr("opacity", 0);

          blockBTile
            .attr("y", 0)
            .attr("opacity", 0);

          threadCSquare.attr("opacity", 0);

          threadARect
            .attr("x", 0)
            .attr("opacity", 0);

          threadBRect
            .attr("y", 0)
            .attr("opacity", 0);

          statusText.text("");

          // Restart animation
          setTimeout(animate, duration);
        }, duration);
        return; // Exit early to pause
      } else {
        // Move block tiles to next K position
        blockATile
          .transition().duration(duration)
          .attr("x", tileK * BK * cellSize);

        blockBTile
          .transition().duration(duration)
          .attr("y", tileK * BK * cellSize);

        phase = 1; // Go back to loading registers
      }
    }

    setTimeout(animate, duration);
  }

  // Start animation
  animate();
})();
</script>
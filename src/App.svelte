<script>
  let isLoading = false
  let canvasHeight = 0
  let canvasWidth = 0
</script>

<main>
  <section>
    {#if isLoading}
      <div class="text-loading">加载中. . .</div>
    {:else}
      <div class="coat">
        <canvas id="canvas-apple-anime" {canvasHeight} {canvasWidth} />
      </div>
    {/if}
  </section>
</main>

<style>
  main {
    height: 100%;
    text-align: center;
    margin: 0 auto;
    background-color: #000;
    color: #fff;
  }

  .text-loading {
    font-size: 20px;
    font-weight: 700;
    padding-top: 4rem;
  }

  .coat {
    width: 100%;
    height: 100%;
  }
</style>

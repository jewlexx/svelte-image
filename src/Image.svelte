<script lang="ts">
  import { decode } from 'blurhash';
  import Waypoint from 'svelte-waypoint';

  export let c = ''; // deprecated
  export let alt = '';
  export let width: number | undefined;
  export let height: number | undefined;
  export let usemap = '';
  export let src = '';
  export let srcset = '';
  export let srcsetWebp = '';
  export let ratio = '100%';
  export let blur = true;
  export let sizes = '(max-width: 1000px) 100vw, 1000px';
  export let offset = 0;
  export let threshold = 1.0;
  export let lazy = true;
  export let wrapperClass = '';
  export let placeholderClass = '';
  export let blurhash = null;
  export let blurhashSize = null;

  let className = '';
  export { className as class };

  let loaded = !lazy;

  function load(img) {
    img.onload = () => (loaded = true);
  }

  function decodeBlurhash(canvas) {
    const pixels = decode(blurhash, blurhashSize.width, blurhashSize.height);
    const ctx = canvas.getContext('2d');
    const imageData = ctx.createImageData(
      blurhashSize.width,
      blurhashSize.height,
    );
    imageData.data.set(pixels);
    ctx.putImageData(imageData, 0, 0);
  }
</script>

<Waypoint
  class={wrapperClass}
  style="min-height: 100px; width: 100%;"
  once
  {threshold}
  {offset}
  disabled={!lazy}
>
  <div class:loaded style="position: relative; width: 100%;">
    <div style="position: relative; overflow: hidden;">
      <div style="width:100%;padding-bottom:{ratio};" />
      {#if blurhash}
        <canvas
          class="placeholder"
          use:decodeBlurhash
          width={blurhashSize.width}
          height={blurhashSize.height}
        />
      {:else}
        <img class="placeholder {placeholderClass}" class:blur {src} {alt} />
      {/if}
      <picture>
        <source type="image/webp" srcset={srcsetWebp} {sizes} />
        <source {srcset} {sizes} />
        <img
          {src}
          use:load
          class="main {c} {className}"
          {alt}
          {width}
          {height}
          {usemap}
        />
      </picture>
    </div>
  </div>
</Waypoint>

<style>
  img,
  canvas {
    object-position: center;
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    will-change: opacity;
  }

  .blur {
    filter: blur(15px);
    transition: opacity 1200ms;
  }

  .placeholder {
    opacity: 1;
    width: 100%;
    height: 100%;
    transition: opacity 1200ms ease-out;
    transition-delay: 0.4s;
  }

  .main {
    opacity: 0;
    transition: opacity 1200ms ease-out;
    transition-delay: 0.4s;
  }

  .loaded .placeholder {
    opacity: 0;
  }

  .loaded .main {
    opacity: 1;
  }
</style>

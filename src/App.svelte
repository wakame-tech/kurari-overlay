<script lang="ts">
  let images = import.meta.glob<false, string, { default: string }>(
    "./assets/*.png",
  );
  let files: FileList | null = null;
  let value: string = "20";
  let selected: string = Object.keys(images)[0];

  let image: HTMLImageElement | null = null;
  let overlay: HTMLImageElement | null = null;
  let resultDataUrl: string | null = null;

  const readAsDataURL = (blob: Blob) =>
    new Promise((resolve) => {
      const reader = new FileReader();
      reader.onload = () => resolve(reader.result);
      reader.readAsDataURL(blob);
    });

  const loadImage = (url: string) =>
    new Promise<HTMLImageElement>((resolve) => {
      const image = new Image();
      image.onload = () => resolve(image);
      image.src = url;
    });

  const drawImages = (ratio: number) => {
    if (!image || !overlay) return;

    const canvas = document.getElementById("canvas")! as HTMLCanvasElement;
    canvas.width = image.width;
    canvas.height = image.height;

    const ctx = canvas.getContext("2d")!;
    ctx.drawImage(image, 0, 0);

    const scale = overlay.height / image.height / ratio;
    const [width, height] = [overlay.width / scale, overlay.height / scale];
    const top = image.height - height;
    const left = overlay.src.includes("left") ? 0 : image.width - width;
    ctx.drawImage(overlay, left, top, width, height);
    resultDataUrl = canvas.toDataURL();
  };

  const handleScaleChange = (event: Event) => {
    const value = (event.target as HTMLInputElement).value;
    const ratio = parseInt(value) / 100;
    drawImages(ratio);
  };

  $: if (files) {
    (async () => {
      const dataUrl = await readAsDataURL(files[0]);
      image = await loadImage(dataUrl as string);

      const { default: overlayUrl } = await images[selected]();
      overlay = await loadImage(overlayUrl);

      const ratio = parseInt(value) / 100;
      drawImages(ratio);
    })();
  }
</script>

<main>
  <h2>kurari-overlay</h2>
  <p>くらりちゃんを写真に重ねることが出来ます</p>
  <div>
    <label for="image">Upload a picture:</label>
    <input type="file" id="image" accept="image/png, image/jpeg" bind:files />
  </div>
  <div>
    <label for="overlay">Select an overlay:</label>
    <select bind:value={selected}>
      {#each Object.keys(images) as image}
        <option value={image}>{image}</option>
      {/each}
    </select>
  </div>
  <div>
    <label for="scale">Scale(%): {value}%</label>
    <input
      type="range"
      min="10"
      max="100"
      step="5"
      bind:value
      on:change={handleScaleChange}
    />
  </div>

  <canvas id="canvas"></canvas>
  {#if resultDataUrl}
    <img class="preview" src={resultDataUrl} alt="result" />
  {/if}
</main>

<style>
  #canvas {
    display: none;
  }

  .preview {
    max-width: 100%;
  }
</style>

<script lang="ts">
  import { onMount } from "svelte";
  import Konva from "konva";
  import { tw } from "twind";

  const sceneWidth: number = 1080;
  const sceneHeight: number = 1080;
  const frameHeight: number = 180;

  let stage: Konva.Stage;
  let exportStage: Konva.Stage;
  let layer: Konva.Layer;
  let uploadImage: Konva.Image;
  let previewImage: (event: Event) => void;
  let previewText: (newText: string) => void;
  let ImageTr: Konva.Transformer;

  const getDate = () => {
    const now = new Date();
    const date = now.getDate();
    const month =
      now.getMonth().toString().length === 1
        ? `0${now.getMonth()}`
        : now.getMonth();
    const year = (now.getFullYear() + 543).toString().slice(2);
    return `${date}\\${month}\\${year}`;
  };

  let line1: string = "Fujifilm X-T20 + XF18mm";
  let line2: string = "1/200s f/5.6 ISO-800";
  let line3: string = "Thailand";
  let line4: string = getDate();
  let line5: string = "Max180643";

  $: previewText?.(`${line1}\n${line2}\n${line3}\n${line4}\n${line5}`);

  class ImageSwitcher {
    constructor(private image: Konva.Image, private layer: Konva.Layer) {}
    switchImage(url: string) {
      const img = new Image();
      img.onload = () => {
        this.image.image(img);
        this.layer.batchDraw();
      };
      ImageTr?.remove();
      ImageTr = addTransformer(this.layer, this.image);
      img.src = url;
    }
  }

  const addTransformer = (layer: Konva.Layer, el: Konva.Image) => {
    const MIN_WIDTH = 20;
    const tr = new Konva.Transformer({
      nodes: [el],
      padding: 0,
      rotateEnabled: true,
      enabledAnchors: ["top-left", "top-right", "bottom-left", "bottom-right"],
      boundBoxFunc: (oldBox, newBox) => {
        if (newBox.width < MIN_WIDTH) {
          return oldBox;
        }
        return newBox;
      },
    });
    layer.add(tr);
    el.on("transform", () => {
      el.setAttrs({
        width: Math.max(el.width() * el.scaleX(), MIN_WIDTH),
        height: Math.max(el.height() * el.scaleY(), MIN_WIDTH),
        scaleX: 1,
        scaleY: 1,
      });
    });

    return tr;
  };

  const eventResizeHandler = () => {
    const container = document.getElementById("canvasParent");
    const c_width = container.offsetWidth;
    const scale = c_width / sceneWidth;
    stage!.width(sceneWidth * scale);
    stage!.height(sceneHeight * scale);
    stage.scale({ x: scale, y: scale });
  };
  window.addEventListener("resize", eventResizeHandler);

  onMount(async () => {
    stage = new Konva.Stage({
      container: "canvasEditor",
      width: sceneWidth,
      height: sceneHeight,
    });

    previewImage = (e) => {
      const input = e.target as HTMLInputElement;
      const url = URL.createObjectURL(input.files[0]);
      uploadImageSwitcher.switchImage(url);
    };

    previewText = (newText) => {
      const textShape = creditText;
      textShape.text(newText);
      layer.batchDraw();
    };

    layer = new Konva.Layer({ fill: "#ffffff" });
    stage.add(layer);

    // Static background
    const background = new Konva.Rect({
      x: 0,
      y: 0,
      width: stage.width(),
      height: stage.height(),
      fill: "#f6f6f6",
      zOffset: -1,
      listening: false,
    });
    layer.add(background);

    // Upload image
    uploadImage = new Konva.Image({
      x: 0,
      y: frameHeight,
      image: undefined as any,
      draggable: true,
      scale: { x: 0.5, y: 0.5 },
    });
    layer.add(uploadImage);
    const uploadImageSwitcher = new ImageSwitcher(uploadImage, layer);

    // Top frame
    const topFrame = new Konva.Rect({
      x: 0,
      y: 0,
      width: stage.width(),
      height: frameHeight,
      fill: "#ffffff",
      listening: false,
    });
    layer.add(topFrame);

    // Bottom frame
    const bottomFrame = new Konva.Rect({
      x: 0,
      y: 900,
      width: stage.width(),
      height: frameHeight,
      fill: "#ffffff",
      listening: false,
    });
    layer.add(bottomFrame);

    // Credit
    const creditText = new Konva.Text({
      x: 20,
      y: 921,
      width: 1040,
      fontSize: 24,
      fontFamily: "Itim",
      lineHeight: 1.2,
      fill: "#000000",
      text: "",
      align: "right",
    });
    layer.add(creditText);

    eventResizeHandler();

    exportStage = new Konva.Stage({
      container: "export",
      width: sceneWidth,
      height: sceneHeight,
      scale: { x: 1, y: 1 },
    });
  });

  const downloadURI = (uri, name) => {
    const link = document.createElement("a");
    link.download = name;
    link.href = uri;
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
  };

  const download = () => {
    ImageTr.hide();
    const exportLayer = layer.clone({ listening: false });
    exportStage.add(exportLayer);

    const dataURL = exportStage.toDataURL({
      mimeType: "image/jpeg",
    });
    downloadURI(dataURL, "ig-frame-download");
    exportStage.removeChildren();

    setTimeout(() => {
      ImageTr.show();
    }, 300);
  };
</script>

<main class={tw`h-screen flex flex-col items-center bg-blue-50`}>
  <h1 class={tw`text(5xl) my-4`}>IG-Frame</h1>
  <div
    class={tw`w-full flex flex-col lg:flex-row w-[96vw] lg:w-[80vw] h-3/5 mt-4`}
  >
    <div
      id="canvasParent"
      class={tw`w-full lg:w-1/2 text(center) flex items-start mb-4 drop-shadow-2xl`}
    >
      <div id="canvasEditor" class={tw`w-full h-full`} />
    </div>

    <div class={tw`w-full lg:w-1/2 pl-0 lg:pl-4 flex-col items-start`}>
      <div class={tw`my-2`}>
        line 1 <input
          type="text"
          bind:value={line1}
          class={tw`mt-1 p-1 focus:ring-blue-300 focus:border-blue-300 block w-full shadow-sm border border-gray-300 rounded-md`}
        />
      </div>
      <div class={tw`my-2`}>
        line 2 <input
          type="text"
          bind:value={line2}
          class={tw`mt-1 p-1 focus:ring-blue-300 focus:border-blue-300 block w-full shadow-sm border border-gray-300 rounded-md`}
        />
      </div>
      <div class={tw`my-2`}>
        line 3 <input
          type="text"
          bind:value={line3}
          class={tw`mt-1 p-1 focus:ring-blue-300 focus:border-blue-300 block w-full shadow-sm border border-gray-300 rounded-md`}
        />
      </div>
      <div class={tw`my-2`}>
        line 4 <input
          type="text"
          bind:value={line4}
          class={tw`mt-1 p-1 focus:ring-blue-300 focus:border-blue-300 block w-full shadow-sm border border-gray-300 rounded-md`}
        />
      </div>
      <div class={tw`my-2`}>
        line 5 <input
          type="text"
          bind:value={line5}
          class={tw`mt-1 p-1 focus:ring-blue-300 focus:border-blue-300 block w-full shadow-sm border border-gray-300 rounded-md`}
        />
      </div>
      <div class={tw`my-2`}>
        Upload Image <br /><input
          class={tw`text-base mt-1`}
          type="file"
          id="background"
          accept="image/*"
          on:change={(event) => previewImage(event)}
        />
      </div>
      <div class={tw`mt-6 mb-16 lg:mb-0`}>
        <button
          class={tw`w-full py-4 px-8 rounded bg-blue-100 hover:bg-blue-200 text-2xl font-bold border`}
          on:click={() => download()}>Save</button
        >
      </div>
    </div>
  </div>

  <div id="export" class={tw`hidden`} />
</main>

<style>
  * {
    font-family: "Itim", cursive;
    font-feature-settings: "liga" 0;
  }
</style>

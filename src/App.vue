<script setup>

import { ref, onMounted } from "vue"
import { useOruga } from "@oruga-ui/oruga-next";

const oruga = useOruga();

//-- Data
const line1 = ref("");
const line2 = ref("lancer");
const line3 = ref("Ars Moriendi");
const episode = ref("episode:");
const title = ref("Title text");
const alignment = ref("right");

const canvasElement = ref(null);
const context = ref(null);
let settings = {}

onMounted(() => {
  context.value = canvasElement.value?.getContext('2d') || undefined;
  render(context.value, config(settings));
})

function draw(){
  render(context.value, config(settings))
}

function config(settings) {

  // 4:3 @ 900x675
  settings.canvasWidth = 900;
  settings.canvasHeight = 650;
  settings.leftMargin = settings.canvasWidth * 1 / 15;
  settings.rightBoundary = settings.canvasWidth - settings.leftMargin;

  settings.smHeadSize = settings.canvasHeight * 0.188;
  settings.lgHeadSize = settings.canvasHeight * 0.308;
  settings.epSize = settings.canvasHeight * 0.095;
  settings.titleSize = settings.canvasHeight * 0.095;
  settings.maxWidth = settings.rightBoundary - settings.leftMargin;

  return settings;
}

function render(ctx, config) {
  if (!ctx) {
    return;
  }

  ctx.clearRect(0, 0, config.canvasWidth, config.canvasHeight)


  let topSquash = 0.62,
    midSquash = 0.62,
    botSquash = 1,  // fit to maxWidth
    epSquash = 0.76,
    titleSquash = 0.74;

  // Drawing white text with white stroke and neon shadow
  ctx.fillStyle = "#FFFFFF";
  ctx.strokeStyle = "#FFFFFF";
  ctx.textBaseline = "top";
  ctx.shadowColor = "orange";
  ctx.shadowBlur = 6;

  // Top two lines
  ctx.font = `900 ${config.smHeadSize}px "Times New Roman",serif`;
  const text1 = line1.value.toUpperCase();
  addFittedText(ctx, config, text1, config.canvasHeight * 0.06667, topSquash);
  const text2 = line2.value.toUpperCase();
  addFittedText(ctx, config, text2, config.canvasHeight * 0.21926, midSquash);

  // Bigger third line
  const text3 = line3.value.toUpperCase();
  ctx.font = `900 ${config.lgHeadSize}px "Times New Roman",serif`;
  addFittedText(ctx, config, text3, config.canvasHeight * 0.35852, botSquash);

  // Change font for EPISODE: label
  ctx.font = `700 ${config.epSize}px sans-serif`;
  const textEp = episode.value.toUpperCase();
  addFittedText(ctx, config, textEp,  config.canvasHeight * 0.62963, epSquash);

  // Change font for Title
  ctx.font = `600 ${config.titleSize}px "Times New Roman",serif`;
  const textTitle = title.value.toUpperCase();
  addFittedText(ctx, config, textTitle, config.canvasHeight * 0.78519, titleSquash, alignment.value);

}

function addFittedText(ctx, config, text, y, squash, align = 'left') {
  let x;

  if (align == "right") {
    ctx.textAlign = "right";
    x = config.rightBoundary;
  }
  else if (align == "left") {
    ctx.textAlign = "left";
    x = config.leftMargin
  }
  else if (align == "center") {
    ctx.textAlign = "center";
    x = (config.rightBoundary + config.leftMargin) / 2;
  }
  else { x = align }

  let toDraw = text.split('\n');
  if (toDraw.length > 1) {
    ctx.textBaseline = "middle";
  }
  for (let n in toDraw) {
    let mWidth = ctx.measureText('M').width;
    // Use maximum available space (if set)
    // or squashed width by ratio
    let widthCalc = ctx.measureText(toDraw[n]).width;
    if (widthCalc * squash >= config.maxWidth) {
      widthCalc = config.maxWidth;
    } else {
      widthCalc = widthCalc * squash;
    }
    // we're not stroking text for now, weird artifacts.
    // ctx.strokeText(toDraw[n], x, y+(n*mWidth), widthCalc);
    ctx.fillText(toDraw[n], x, y + (n * mWidth), widthCalc);
  }

  // reset to "reasonable" defaults
  ctx.textBaseline = "top";
  ctx.textAlign = "left";
}

function download() {
  // Convert our canvas to a data URL
  let canvasUrl = canvasElement.value.toDataURL();
  // Create an anchor, and set the href value to our data URL
  const createEl = document.createElement('a');
  createEl.href = canvasUrl;

  // This is the name of our downloaded file
  createEl.download = "lancer-title-card.png";

  // Click the download button, causing a download, and then remove it
  createEl.click();
  createEl.remove();
}

</script>

<template>
  <section class="hero is-small is-link is-bold">
    <div class="hero-body">
      <p class="title">LANCER Title Card Generator</p>
    </div>
  </section>
  <section class="section">
    <div class="container">
      <div class="columns">
        <div class="column">
          <o-field horizontal label="Line 1">
            <o-input v-model="line1" expanded @blur="draw"/>
          </o-field>
          <o-field horizontal label="Line 2">
            <o-input v-model="line2" expanded @blur="draw"/>
          </o-field>
          <o-field horizontal label="Line 3">
            <o-input v-model="line3" expanded @blur="draw"/>
          </o-field>
          <o-field horizontal label="Subtitle">
            <o-input v-model="episode" expanded @blur="draw"/>
          </o-field>
          <o-field horizontal label="Title">
            <o-input type="textarea" v-model="title" expanded @blur="draw"/>
          </o-field>
          <o-field horizontal label="Alignment">
            <o-select v-model="alignment" expanded @blur="draw">
              <option value="left">Left</option>
              <option value="center">Center</option>
              <option value="right">Right</option>
            </o-select>
          </o-field>
          <o-button label="Save" variant="primary" expanded @click="download" />
        </div>
        <div class="column">
          <div class="canvas-container">
            <canvas id="canvas" ref="canvasElement" width="900" height="675"></canvas>
          </div>
        </div>
      </div>
    </div>
  </section>
  <footer class="footer">
    <div class="content has-text-centered">
      <p>
        <strong>LANCER Title Card Generator</strong> by <a href="https://github.com/kuenaimaku">Kyle Humphrey</a>.<br />
        Inspired by <a href="https://kychou.net/eva-title/">Evangelion Title Card Generator</a> by Kuan-Yen Chou.
      </p>
    </div>
  </footer>
</template>

<style scoped></style>

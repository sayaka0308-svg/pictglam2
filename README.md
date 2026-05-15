# pictglam2
<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<title>ピクトグラム制作ツール（3作品＋保存機能）</title>

<!-- PDF生成ライブラリ -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

<style>
  body { font-family: sans-serif; }
  .title-row {
    display: flex;
    justify-content: space-between;
    width: 1200px;
    margin-bottom: 10px;
  }
  .title-row div {
    width: 400px;
    text-align: center;
    font-size: 22px;
    font-weight: bold;
  }
  .grid-row {
    display: flex;
    gap: 10px;
  }
</style>
</head>
<body>

<h2>美術Ⅰ課題：ピクトグラム制作ツール</h2>

<div class="title-row">
  <div>歩く</div>
  <div>走る</div>
  <div>跳ぶ</div>
</div>

<div class="grid-row">
  <canvas id="grid0" width="400" height="400" style="border:1px solid black;"></canvas>
  <canvas id="grid1" width="400" height="400" style="border:1px solid black;"></canvas>
  <canvas id="grid2" width="400" height="400" style="border:1px solid black;"></canvas>
</div>

<br>
<canvas id="palette" width="1200" height="40" style="border:1px solid black;"></canvas>

<br><br>
<button id="saveBtn" style="font-size:18px; padding:10px 20px;">PNGで保存</button>
<button id="pdfBtn" style="font-size:18px; padding:10px 20px;">PDFで保存</button>

<script>
// ====== 基本設定 ======
const GRID = 5;
const SIZE = 80;
const TILE_COUNT = 30;  // tile0.png〜tile29.png
const WORK_COUNT = 3;   // 歩く・走る・跳ぶ

let selected = 0;

// 3作品分のデータ
let grids = [];
let rots = [];
let lastX = [-1, -1, -1];
let lastY = [-1, -1, -1];

for (let w = 0; w < WORK_COUNT; w++) {
  grids[w] = [];
  rots[w] = [];
  for (let y = 0; y < GRID; y++) {
    grids[w][y] = [];
    rots[w][y] = [];
    for (let x = 0; x < GRID; x++) {
      grids[w][y][x] = 0;
      rots[w][y][x] = 0;
    }
  }
}

// ====== Canvas ======
const gridCanvas = [
  document.getElementById("grid0"),
  document.getElementById("grid1"),
  document.getElementById("grid2")
];
const gctx = gridCanvas.map(c => c.getContext("2d"));

const paletteCanvas = document.getElementById("palette");
const pctx = paletteCanvas.getContext("2d");

// ====== タイル画像読み込み ======
const tileImages = [];

for (let i = 0; i < TILE_COUNT; i++) {
  const img = new Image();
  img.src = `tile${i}.png`;  // tile0.png〜tile29.png

  img.onload = () => drawAll();
  img.onerror = () => {
    console.warn(`画像が読み込めません: tile${i}.png`);
    drawAll();
  };

  tileImages.push(img);
}

// ====== グリッド描画 ======
function drawGrid(w) {
  const ctx = gctx[w];
  ctx.clearRect(0, 0, 400, 400);

  for (let y = 0; y < GRID; y++) {
    for (let x = 0; x < GRID; x++) {
      const tileIndex = grids[w][y][x];
      const angle = rots[w][y][x];

      ctx.save();
      ctx.translate(x * SIZE + SIZE / 2, y * SIZE + SIZE / 2);
      ctx.rotate(angle * Math.PI / 180);

      const img = tileImages[tileIndex];
      if (img && img.complete && img.naturalWidth > 0) {
        ctx.drawImage(img, -SIZE / 2, -SIZE / 2, SIZE, SIZE);
      } else {
        ctx.fillStyle = "#ccc";
        ctx.fillRect(-SIZE / 2, -SIZE / 2, SIZE, SIZE);
      }

      ctx.restore();

      ctx.strokeStyle = "black";
      ctx.strokeRect(x * SIZE, y * SIZE, SIZE, SIZE);
    }
  }
}

// ====== パレット描画 ======
function drawPalette() {
  pctx.clearRect(0, 0, 1200, 40);

  const cellWidth = 40;

  for (let i = 0; i < TILE_COUNT; i++) {
    const x = i * cellWidth;
    const img = tileImages[i];

    if (img && img.complete && img.naturalWidth > 0) {
      pctx.drawImage(img, x, 0, cellWidth, 40);
    } else {
      pctx.fillStyle = "#eee";
      pctx.fillRect(x, 0, cellWidth, 40);
    }

    if (i === selected) {
      pctx.strokeStyle = "red";
      pctx.lineWidth = 3;
      pctx.strokeRect(x + 2, 2, cellWidth - 4, 36);
    } else {
      pctx.strokeStyle = "black";
      pctx.strokeRect(x, 0, cellWidth, 40);
    }
  }
}

// ====== 全描画 ======
function drawAll() {
  for (let w = 0; w < WORK_COUNT; w++) drawGrid(w);
  drawPalette();
}

// ====== グリッドクリック ======
gridCanvas.forEach((canvas, w) => {
  canvas.addEventListener("click", (e) => {
    const rect = canvas.getBoundingClientRect();
    const x = Math.floor((e.clientX - rect.left) / SIZE);
    const y = Math.floor((e.clientY - rect.top) / SIZE);

    if (x >= 0 && x < GRID && y >= 0 && y < GRID) {
      grids[w][y][x] = selected;
      lastX[w] = x;
      lastY[w] = y;
      drawGrid(w);
    }
  });
});

// ====== パレットクリック ======
paletteCanvas.addEventListener("click", (e) => {
  const rect = paletteCanvas.getBoundingClientRect();
  const cellWidth = 40;
  const x = Math.floor((e.clientX - rect.left) / cellWidth);

  if (x >= 0 && x < TILE_COUNT) {
    selected = x;
    drawPalette();
  }
});

// ====== Rキーで回転 ======
document.addEventListener("keydown", (e) => {
  if (e.key === "r" || e.key === "R") {
    for (let w = 0; w < WORK_COUNT; w++) {
      if (lastX[w] >= 0 && lastY[w] >= 0) {
        rots[w][lastY[w]][lastX[w]] =
          (rots[w][lastY[w]][lastX[w]] + 90) % 360;
        drawGrid(w);
      }
    }
  }
});

// ====== 共通：タイトル＋3グリッドを1枚に描く関数 ======
function renderToCombinedCanvas() {
  const out = document.createElement("canvas");
  out.width = 1200;
  out.height = 450;
  const octx = out.getContext("2d");

  // 背景白
  octx.fillStyle = "white";
  octx.fillRect(0, 0, out.width, out.height);

  // タイトル
  octx.font = "28px sans-serif";
  octx.textAlign = "center";
  octx.fillStyle = "black";
  octx.fillText("歩く", 200, 30);
  octx.fillText("走る", 600, 30);
  octx.fillText("跳ぶ", 1000, 30);

  // グリッド
  octx.drawImage(gridCanvas[0], 0, 50);
  octx.drawImage(gridCanvas[1], 400, 50);
  octx.drawImage(gridCanvas[2], 800, 50);

  return out;
}

// ====== PNG保存 ======
document.getElementById("saveBtn").addEventListener("click", () => {
  const out = renderToCombinedCanvas();
  try {
    const dataURL = out.toDataURL("image/png");
    const link = document.createElement("a");
    link.download = "pictogram.png";
    link.href = dataURL;
    link.click();
  } catch (err) {
    console.error(err);
    alert("PNG保存に失敗しました。\nブラウザのセキュリティ設定や file:/// での実行が原因の可能性があります。");
  }
});

// ====== PDF保存 ======
document.getElementById("pdfBtn").addEventListener("click", () => {
  const out = renderToCombinedCanvas();
  let imgData;
  try {
    imgData = out.toDataURL("image/png");
  } catch (err) {
    console.error(err);
    alert("PDF用画像の生成に失敗しました。\nブラウザのセキュリティ設定や file:/// での実行が原因の可能性があります。");
    return;
  }

  try {
    const { jsPDF } = window.jspdf;
    const pdf = new jsPDF({
      orientation: "landscape",
      unit: "px",
      format: [1200, 500]
    });

    pdf.addImage(imgData, "PNG", 0, 0, 1200, 450);
    pdf.save("pictogram.pdf");
  } catch (err) {
    console.error(err);
    alert("PDF保存に失敗しました。");
  }
});

// 初期描画
drawAll();
</script>
</body>
</html>

<!DOCTYPE html>
<html>
<head>
  <title></title>
  <meta charset="utf-8">
  <style>
    * {
      margin: 0 0;
      padding: 0 0;
      box-sizing: border-box;
    }

    html, body {
      height: 100%;
    }

    textarea {
      margin: 0 auto;
      height: 300px;
      width: 100%;
      resize: none;
      padding: 5px 10px;
      font-size: 16px;
    }

    .input-warp {
      width: 500px;
      position: absolute;
      top: 20px;
      left: 20px;
      background: #FFFFFF;
      z-index: 2;
      opacity: 0.4;
    }

    .content-warp {
      position: absolute;
      top: 0;
      left: 0;
      overflow: auto;
      height: 100%;
      background: #f2f2f2;
      width: 100%;
    }

    .content-warp::-webkit-scrollbar {
      width: 0;
      height: 0;
    }

    .svg-warp {
      width: 20000px;
      height: 20000px;
      z-index: 0;
    }

    .util-warp {
      position: absolute;
      right: 20px;
      top: 20px;
      z-index: 1;
    }

    button {
      cursor: pointer;
    }
  </style>
</head>
<body>
<div class="input-warp">
  <span>use tab or two space for set childNode</span>
  <textarea title="" id="text" placeholder="input anything"></textarea>
</div>
<div class="util-warp">
  <button onclick="handleMagnify()">放大</button>
  <span id="zoomText">100%</span>
  <button onclick="handleMinify()">缩小</button>
  <button onclick="handlePicture()">打印</button>
</div>
<div onmousedown="handlerDragMoveEvent(event)" class="content-warp">
  <div id="svgcontent" class="svg-warp"></div>
</div>
<script src="../dist/code2mind.min.js"></script>
<script>
const text = document.getElementById('text');
const zoomTextEl = document.getElementById('zoomText');
const svgcontent = document.getElementById('svgcontent');
let timer = null;
// text.value = '大厦快捷的等会撒即可的和';
// text.value =`a
// b
//   b-1
//     b3
//       b5
//       b6
//     b4
//   b-2
// c`;
text.value = `哈佛学习法
哈佛商学院的超级学习法
\t模块学习
\t\t模块学习回顾法的好处
\t\t\t查缺补漏
\t\t\t巩固知识、加深记忆
\t\t\t知识系统化
\t\t\t深化知识
\t\t\t\t深化知识
\t\t\t\t深化知识
\t\t\t\t深化知识
\t\t\t\t深化知识
\t\t学习步骤
\t\t\t看电影
\t\t\t看课本
\t\t\t写一篇回顾性文章
\t\t\t经常翻看
\t实践性模拟演练法
\t\t情节设计环节
\t\t组织实施环节
\tSMART原则法
\t\tS=specific具体的
\t\tM=measurble可度量的
\t\tR=relevant和执行者有关的
\t\tT=time-blound有时间限制的
\t案例教学法
\t高强度课堂发言学习法
\t提前学习法
\t\t好处
\t\t\t被动学习变主动学习
\t\t\t有助于培养开放式思维，有助于培养包容的态度
\t\t\t提前在脑中形成完整的知识体系
\t\t\t把"补漏时间"保存自学时间
\t\t\t有助于积累大量经验
\t\tSQ3R法
\t\t\tsurvey浏览，用时1分钟
\t\t\tquestion发问，一边读一边提问
\t\t\tread阅读
\t\t\trecite复述
\t\t\treview复习
如何把时间花到重要的事情上
\t未来视角给现在的自己定目标
\t找到最重要的事
\t\t四象限法则
\t\t找到5-10个目标
\t\t保留最多5个目标
\t保留最多2个目标/年
\t将目标拆分成可以每天执行的微习惯
\t每天记录定期复盘
读书法
\t精读
\t速读
\t批注法
\t读书笔记
哈佛学生
\t对自己要求高，不安于现状
\t强大的目标感，清楚自己要什么
\t不断努力、不断向上、不断督促自己
三步精进法
\t自励心
\t\t短期榜样、长期榜样
\t\t每天对自己说yes，告诉别人自己都可以，我也一定可以
\t\t学会自知、自控、管理自己的情绪
\t规划力
\t\t分析现状、合理定目标
\t\t\t内向
\t\t\t\t你的学习能力（记忆力、理解力、观察力）
\t\t\t\t你的学习心态
\t\t\t\t你的学习方法
\t\t\t外向
\t\t\t\t工作方面
\t\t分步规划
\t\t\t即时
\t\t\t\t每天、todolist
\t\t\t近期
\t\t\t\t以每周、每月为维度
\t\t\t中长期
\t\t\t\t半年、一年或者更长
\t执行力
\t\t番茄工作法
\t\t与世隔绝法
\t\t自我鼓励法
\t\t同伴激励法`;
text.addEventListener('input', () => {
  if (timer) clearTimeout(timer);
  timer = setTimeout(() => {
    handler(text.value);
  }, 200);
});
text.addEventListener('focus', () => {
  text.parentElement.style.opacity = '1';
});
text.addEventListener('blur', () => {
  text.parentElement.style.opacity = '0.4';
});
text.addEventListener('keydown', (e) => {
  if (e.code === 'Tab') {
    e.preventDefault();
  }
});
text.addEventListener('keyup', (e) => {
  if (e.code === 'Tab') {
    e.preventDefault();
    const currentEl = e.target;
    const startAt = currentEl.selectionStart;
    currentEl.value = currentEl.value.substring(0, startAt) + '  ' + currentEl.value.substring(startAt);
    currentEl.selectionStart = startAt + 2;
    currentEl.selectionEnd = startAt + 2;
  }
});
let zoom = 1;
const codemind = new Code2Mind({
  el: '#svgcontent',
});
handler(text.value);

function handler (value) {
  codemind.render(value);
  handleToMiddle();
}

function handlerDragMoveEvent (event) {
  if (event.button === 0) {
    const downX = event.pageX;
    const downY = event.pageY;
    let x = 0;
    let y = 0;
    const startX = svgcontent.parentElement.scrollLeft;
    const startY = svgcontent.parentElement.scrollTop;
    const move = (moveEvent) => {
      x = moveEvent.pageX - downX;
      y = moveEvent.pageY - downY;
      svgcontent.parentElement.scrollLeft = startX - x;
      svgcontent.parentElement.scrollTop = startY - y;
    };
    const end = () => {
      document.removeEventListener('mousemove', move);
      document.removeEventListener('mouseup', end);
    };
    document.addEventListener('mousemove', move);
    document.addEventListener('mouseup', end);
  }
}

function handleToMiddle () {
  const height = svgcontent.offsetHeight;
  const width = svgcontent.offsetWidth;
  const px = (width - svgcontent.parentElement.offsetWidth) / 2;
  const ph = (height - svgcontent.parentElement.offsetHeight) / 2;
  svgcontent.parentElement.scrollTo(px, ph);
}

function handleMagnify () {
  if (zoom < 2) {
    zoom += 0.1;
    svgcontent.style.transform = `scale(${ zoom })`;
  }
  zoomTextEl.innerText = (zoom * 100).toFixed(0) + '%';
}

function handleMinify () {
  if (zoom > 0.4) {
    zoom -= 0.1;
    svgcontent.style.transform = `scale(${ zoom })`;
  }
  zoomTextEl.innerText = (zoom * 100).toFixed(0) + '%';
}

function handlePicture () {
  codemind.picture(2, 10);
}
</script>
</body>
</html>

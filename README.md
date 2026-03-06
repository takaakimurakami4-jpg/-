<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <base target="_top">
  <script src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>
  <style>
    :root { --primary: #2c3e50; --accent: #3498db; --success: #27ae60; --error: #e74c3c; }
    body { font-family: 'Helvetica Neue', Arial, "Hiragino Kaku Gothic ProN", sans-serif; background: #f4f7f6; margin: 0; padding: 20px; color: #333; }
    .container { max-width: 800px; margin: 0 auto; background: #fff; padding: 30px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
    h1 { text-align: center; color: var(--primary); border-bottom: 2px solid var(--accent); padding-bottom: 10px; }
    .status-bar { display: flex; justify-content: space-between; margin-bottom: 20px; font-weight: bold; color: #666; }
    .question-box { font-size: 1.2rem; line-height: 1.6; margin-bottom: 25px; padding: 15px; background: #fafafa; border-radius: 8px; }
    .option { display: block; width: 100%; padding: 15px; margin-bottom: 12px; border: 2px solid #ddd; border-radius: 8px; background: #fff; cursor: pointer; text-align: left; font-size: 1rem; transition: 0.2s; }
    .option:hover:not(:disabled) { background: #f0f8ff; border-color: var(--accent); }
    .option:disabled { cursor: default; }
    .correct { background: #d4edda !important; border-color: var(--success) !important; color: #155724; }
    .incorrect { background: #f8d7da !important; border-color: var(--error) !important; color: #721c24; }
    #feedback { margin-top: 20px; padding: 20px; border-radius: 8px; display: none; line-height: 1.6; animation: fadeIn 0.3s; }
    .feedback-correct { background: #e9f7ef; border-left: 5px solid var(--success); }
    .feedback-error { background: #fdf2f2; border-left: 5px solid var(--error); }
    #next-btn { display: block; width: 100%; padding: 15px; background: var(--primary); color: #fff; border: none; border-radius: 8px; font-size: 1.1rem; cursor: pointer; margin-top: 20px; display: none; }
    #result-screen { text-align: center; display: none; }
    @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
  </style>
</head>
<body>

<div class="container">
  <h1>🧪 高校化学 究極演習</h1>
  
  <div id="quiz-area">
    <div class="status-bar">
      <span id="progress">第 1 問 / -- 問</span>
      <span id="score-display">正解数: 0</span>
    </div>
    
    <div class="question-box" id="question">読み込み中...</div>
    <div id="options-area"></div>
    
    <div id="feedback"></div>
    <button id="next-btn" onclick="showNext()">次の問題へ進む</button>
  </div>

  <div id="result-screen">
    <h2>演習終了！</h2>
    <p id="final-score" style="font-size: 1.5rem;"></p>
    <button class="option" style="text-align:center" onclick="location.reload()">もう一度挑戦する</button>
  </div>
</div>

<script>
// 【受験化学 全範囲網羅データ】
const masterQuizData = [
  // --- 理論化学 ---
  { q: "蒸気圧降下において、不揮発性の溶質を溶かした溶液の蒸気圧は、純溶媒に比べてどうなるか？", a: ["低くなる", "高くなる", "変化しない"], ok: 0, ex: "溶質粒子が溶媒の蒸発を妨げるため、蒸気圧は低くなります。" },
  { q: "浸透圧について、溶液のモル濃度を $C$、絶対温度を $T$、気体定数を $R$ としたとき、浸透圧 $\\Pi$ を表す式（ファントホッフの法則）は？", a: ["$\\Pi = CRT$", "$\\Pi = CR/T$", "$\\Pi = RT/C$"], ok: 0, ex: "希薄溶液の浸透圧は気体の状態方程式と類似した形になります。" },
  { q: "鉛蓄電池において、放電したときに正極（$PbO_2$）の質量はどう変化するか？", a: ["増加する", "減少する", "変化しない"], ok: 0, ex: "$PbO_2 + 4H^+ + SO_4^{2-} + 2e^- \\rightarrow PbSO_4 + 2H_2O$ となり、硫酸鉛(II)が沈着するため増加します。" },
  
  // --- 無機化学 ---
  { q: "アンモニアソーダ法（ソルベー法）において、副産物として得られ、道路の融雪剤などに再利用される物質は？", a: ["塩化カルシウム", "塩化ナトリウム", "炭酸カルシウム"], ok: 0, ex: "$CaCl_2$ が副産物として生じます。" },
  { q: "濃硝酸中にアルミニウムや鉄を入れた際、表面に緻密な酸化被膜ができ、反応が止まる現象を何というか？", a: ["不動態", "合金化", "アモルファス"], ok: 0, ex: "Fe, Co, Ni, Al, Cr などで見られる現象です。" },
  { q: "銅イオン $Cu^{2+}$ を含む水溶液に少量のアンモニア水を加えると青白色沈殿が生じるが、さらに過剰に加えた時に生じる錯イオンの色は？", a: ["深青色", "無色", "赤褐色"], ok: 0, ex: "$[Cu(NH_3)_4]^{2+}$（テトラアンミン銅(II)イオン）の生成によるものです。" },

  // --- 有機化学 ---
  { q: "アセチレンに水を付加反応させ、水銀(II)塩を触媒として得られる主生成物は？", a: ["アセトアルデヒド", "ビニルアルコール", "エチレングリコール"], ok: 0, ex: "一度ビニルアルコールになりますが、不安定なため直ちにアセトアルデヒドに変わります。" },
  { q: "サリチル酸に無水酢酸を反応させて得られる、解熱鎮痛剤として知られる物質は？", a: ["アセチルサリチル酸", "サリチル酸メチル", "アセトアニリド"], ok: 0, ex: "ヒドロキシ基がアセチル化され、アスピリン（アセチルサリチル酸）となります。" },
  { q: "α-アミノ酸の検出に用いられ、反応すると赤紫〜青紫色を呈する試薬は？", a: ["ニンヒドリン試薬", "フェーリング液", "ヨウ素液"], ok: 0, ex: "アミノ基を持つ物質の代表的な検出反応です。" }
  
  // ※ ここにさらに数百問追加しても動作は安定します。
];

let currentIdx = 0;
let score = 0;
let shuffledData = [];

function init() {
  shuffledData = masterQuizData.sort(() => Math.random() - 0.5);
  showQuestion();
}

function showQuestion() {
  const item = shuffledData[currentIdx];
  document.getElementById('progress').innerText = `第 ${currentIdx + 1} 問 / 全 ${shuffledData.length} 問`;
  document.getElementById('question').innerHTML = item.q;
  
  const optsArea = document.getElementById('options-area');
  optsArea.innerHTML = '';
  document.getElementById('feedback').style.display = 'none';
  document.getElementById('next-btn').style.display = 'none';

  item.a.forEach((text, i) => {
    const btn = document.createElement('button');
    btn.className = 'option';
    btn.innerHTML = text;
    btn.onclick = () => checkAnswer(i, item.ok, item.ex, btn);
    optsArea.appendChild(btn);
  });
  
  if(window.MathJax) MathJax.typeset();
}

function checkAnswer(selected, correct, explanation, btn) {
  const allBtns = document.querySelectorAll('.option');
  allBtns.forEach(b => b.disabled = true);
  
  const feedback = document.getElementById('feedback');
  feedback.style.display = 'block';

  if (selected === correct) {
    score++;
    btn.classList.add('correct');
    feedback.className = 'feedback-correct';
    feedback.innerHTML = `<strong>✅ 正解！</strong><br>${explanation}`;
  } else {
    btn.classList.add('incorrect');
    allBtns[correct].classList.add('correct');
    feedback.className = 'feedback-error';
    feedback.innerHTML = `<strong>❌ 不正解...</strong><br>正解は「${shuffledData[currentIdx].a[correct]}」です。<br>${explanation}`;
  }
  
  document.getElementById('score-display').innerText = `正解数: ${score}`;
  document.getElementById('next-btn').style.display = 'block';
  if(window.MathJax) MathJax.typeset();
}

function showNext() {
  currentIdx++;
  if (currentIdx < shuffledData.length) {
    showQuestion();
  } else {
    showResult();
  }
}

function showResult() {
  document.getElementById('quiz-area').style.display = 'none';
  const resultScreen = document.getElementById('result-screen');
  resultScreen.style.display = 'block';
  document.getElementById('final-score').innerText = `あなたのスコア: ${score} / ${shuffledData.length}`;
}

init();
</script>

</body>
</html>

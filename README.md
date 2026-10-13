[code_artifact (2).html](https://github.com/user-attachments/files/33378492/code_artifact.2.html)
<!DOCTYPE html>
<html lang="zh-HK">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>時序小火車：尋日、今日、聽日（五關生活篇）</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    /* 特教與4歲幼兒低刺激、高清晰度人體工學規範 */
    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "PingFang HK", "Noto Sans HK", "Microsoft JhengHei", sans-serif;
      touch-action: manipulation;
      user-select: none;
      -webkit-user-select: none;
      -webkit-touch-callout: none;
      background-color: #F8FAFC;
    }

    /* 溫和高亮微動動畫 (避免強光閃爍，柔和吸引注意力) */
    @keyframes calmGlow {
      0%, 100% {
        transform: scale(1);
        box-shadow: 0 4px 18px rgba(245, 158, 11, 0.2);
      }
      50% {
        transform: scale(1.02);
        box-shadow: 0 8px 26px rgba(245, 158, 11, 0.4);
      }
    }

    .active-carriage {
      animation: calmGlow 2.6s ease-in-out infinite;
      border-color: #F59E0B !important;
      background-color: #FFFDF0 !important;
    }

    /* 拖曳中的手勢反饋 */
    .dragging-card {
      opacity: 0.45;
      transform: scale(1.05);
      cursor: grabbing;
      box-shadow: 0 16px 32px rgba(0, 0, 0, 0.12);
      z-index: 50;
    }

    .touch-proxy {
      position: fixed;
      pointer-events: none;
      z-index: 9999;
      transform: translate(-50%, -50%) scale(1.08);
      filter: drop-shadow(0 14px 22px rgba(0, 0, 0, 0.2));
      will-change: transform, left, top;
    }

    /* 輕微晃動表示重試 (溫和不具處罰感) */
    @keyframes softNudge {
      0%, 100% { transform: translateX(0); }
      30% { transform: translateX(-5px); }
      70% { transform: translateX(5px); }
    }
    .nudge-retry {
      animation: softNudge 0.32s ease-in-out;
    }

    /* 綠色大打勾成功標記 */
    .checkmark-pop {
      animation: popIn 0.38s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
    }
    @keyframes popIn {
      0% { transform: scale(0.3) rotate(-10deg); opacity: 0; }
      100% { transform: scale(1) rotate(0); opacity: 1; }
    }
  </style>
</head>
<body class="min-h-screen text-slate-800 flex flex-col justify-between overflow-x-hidden p-2.5 sm:p-4 select-none">

  <!-- 頂部：導師工具列、關卡切換器及仿說指引 -->
  <header class="w-full max-w-6xl mx-auto flex flex-col gap-2.5 pb-2.5 border-b border-slate-200">
    <div class="flex flex-wrap items-center justify-between gap-2">
      <!-- 標題與情景描述 -->
      <div class="flex items-center gap-2">
        <span class="text-3xl" role="img" aria-label="train">🚂</span>
        <div>
          <h1 class="text-lg sm:text-2xl font-black text-slate-800 tracking-wide flex items-center gap-2">
            時序小火車
            <span id="level-badge" class="text-xs px-2.5 py-0.5 rounded-full bg-amber-500 text-white font-extrabold">第 1 關</span>
          </h1>
          <p id="level-title-desc" class="text-xs sm:text-sm text-slate-500 font-bold">生活與遊戲篇</p>
        </div>
      </div>

      <!-- 關卡選擇器 (1-5 關快速切換，便於導師因材施教) -->
      <div class="flex items-center gap-1.5 bg-slate-100 p-1.5 rounded-2xl border border-slate-200" id="level-selector-group">
        <button data-level="0" class="lvl-btn px-3 py-1.5 text-xs sm:text-sm font-black rounded-xl bg-amber-500 text-white shadow-sm transition-all">關卡 1</button>
        <button data-level="1" class="lvl-btn px-3 py-1.5 text-xs sm:text-sm font-black rounded-xl text-slate-600 hover:bg-slate-200 transition-all">關卡 2</button>
        <button data-level="2" class="lvl-btn px-3 py-1.5 text-xs sm:text-sm font-black rounded-xl text-slate-600 hover:bg-slate-200 transition-all">關卡 3</button>
        <button data-level="3" class="lvl-btn px-3 py-1.5 text-xs sm:text-sm font-black rounded-xl text-slate-600 hover:bg-slate-200 transition-all">關卡 4</button>
        <button data-level="4" class="lvl-btn px-3 py-1.5 text-xs sm:text-sm font-black rounded-xl text-slate-600 hover:bg-slate-200 transition-all">關卡 5</button>
      </div>

      <!-- 操作按鈕：重聽發音與重試本關 -->
      <div class="flex items-center gap-2">
        <button id="btn-replay-audio" class="p-2 sm:px-3.5 bg-amber-100 hover:bg-amber-200 active:scale-95 text-amber-950 rounded-2xl flex items-center gap-1.5 font-black text-xs sm:text-sm transition-all border border-amber-300 shadow-sm">
          <svg class="w-5 h-5 text-amber-700" fill="currentColor" viewBox="0 0 20 20">
            <path fill-rule="evenodd" d="M9.383 3.076A1 1 0 0110 4v12a1 1 0 01-1.617.794L4.82 13H2a1 1 0 01-1-1V8a1 1 0 011-1h2.82l3.564-3.794A1 1 0 019.383 3.076zm5.825 2.131a1 1 0 011.414 0 9.972 9.972 0 010 14.108 1 1 0 01-1.414-1.414 7.972 7.972 0 000-11.28 1 1 0 010-1.414zm-2.828 2.828a1 1 0 011.414 0 5.986 5.986 0 010 8.465 1 1 0 01-1.414-1.414 3.986 3.986 0 000-5.637 1 1 0 010-1.414z" clip-rule="evenodd"></path>
          </svg>
          <span>重聽語音 🔊</span>
        </button>
        <button id="btn-reset-level" class="p-2 sm:px-3 bg-slate-100 hover:bg-slate-200 active:scale-95 text-slate-700 rounded-2xl text-xs sm:text-sm font-bold transition-all border border-slate-200">
          重玩此關
        </button>
      </div>
    </div>

    <!-- 導師個別訓練口語鷹架提點板 (醒目、低感官刺激、提供概念引導) -->
    <div id="tutor-prompt-card" class="bg-blue-50 border-2 border-blue-200 rounded-2xl px-4 py-2.5 flex items-center justify-between gap-3 text-blue-900 shadow-sm">
      <div class="flex items-center gap-2.5">
        <span class="px-2 py-0.5 text-xs font-black bg-blue-600 text-white rounded-md tracking-wider whitespace-nowrap">導師仿說引導</span>
        <span id="tutor-speech-prompt" class="text-sm sm:text-base font-bold text-blue-950">指住中間車廂「今日」，問幼兒「今日做咩呀？」</span>
      </div>
      <!-- 導師節奏控制手動前進按鈕 (非倒數強制) -->
      <button id="btn-next-step" class="hidden whitespace-nowrap px-3.5 py-1.5 bg-emerald-600 hover:bg-emerald-700 active:scale-95 text-white font-extrabold text-xs sm:text-sm rounded-xl shadow transition-all">
        繼續下一步 ➔
      </button>
    </div>
  </header>

  <!-- 中間主要工作區：時間軸三節車廂 -->
  <main class="w-full max-w-6xl mx-auto flex-1 flex flex-col justify-around py-2">
    
    <!-- 語音文字即時看板 (超大字型、嚴格 3-5 字) -->
    <div class="text-center my-1.5">
      <div id="current-spoken-phrase" class="inline-flex items-center gap-2.5 px-6 py-2.5 bg-amber-50 rounded-full border-2 border-amber-300 shadow-sm transition-all duration-300">
        <span class="w-3.5 h-3.5 rounded-full bg-amber-500 animate-ping"></span>
        <span id="instruction-text" class="text-2xl sm:text-3xl font-black text-amber-950 tracking-wider">今日玩積木</span>
      </div>
    </div>

    <!-- 軌道與車廂區域 -->
    <div class="relative w-full my-2">
      <!-- 鐵路軌道底紋 -->
      <div class="absolute left-4 right-4 top-1/2 -translate-y-1/2 h-3.5 bg-slate-300 rounded-full flex justify-between px-8 items-center -z-0">
        <div class="w-full h-1 bg-slate-400"></div>
      </div>

      <!-- 三節火車車廂 (尋日 -> 今日 -> 聽日) -->
      <div class="grid grid-cols-3 gap-2 sm:gap-4 md:gap-6 relative z-10">
        
        <!-- 左車廂：尋日 (上一站 - 過去) -->
        <div id="carriage-yesterday" data-slot="yesterday" class="carriage relative flex flex-col items-center bg-white border-4 border-slate-300 rounded-3xl p-2 sm:p-4 shadow-md transition-all duration-300 min-h-[200px] sm:min-h-[250px] md:min-h-[270px]">
          <!-- 站點標籤 -->
          <div class="w-full flex items-center justify-between border-b border-slate-150 pb-1.5 mb-1.5">
            <span class="px-2 py-0.5 text-xs sm:text-sm font-bold bg-slate-100 text-slate-500 rounded-lg">上一站</span>
            <span class="text-lg sm:text-2xl font-black text-slate-700">尋日</span>
          </div>

          <!-- 放置區 -->
          <div class="drop-zone w-full flex-1 flex flex-col items-center justify-center border-2 border-dashed border-slate-300 rounded-2xl bg-slate-50/70 p-2 transition-colors relative">
            <div class="placeholder-hint flex flex-col items-center text-slate-400 text-center pointer-events-none">
              <svg class="w-9 h-9 mb-1 opacity-40 text-slate-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v3m0 0v3m0-3h3m-3 0H9m12 0a9 9 0 11-18 0 9 9 0 0118 0z"></path>
              </svg>
              <span class="text-xs sm:text-sm font-bold">放入尋日事件</span>
            </div>
            <!-- 正確放置後掛載的卡片容器 -->
            <div class="placed-card-container hidden w-full h-full flex flex-col items-center justify-center"></div>
          </div>

          <!-- 輪子裝飾 -->
          <div class="w-full flex justify-around mt-1.5">
            <div class="w-6 h-6 sm:w-8 sm:h-8 bg-slate-700 rounded-full border-4 border-slate-300 flex items-center justify-center shadow-inner"><div class="w-2 h-2 bg-slate-200 rounded-full"></div></div>
            <div class="w-6 h-6 sm:w-8 sm:h-8 bg-slate-700 rounded-full border-4 border-slate-300 flex items-center justify-center shadow-inner"><div class="w-2 h-2 bg-slate-200 rounded-full"></div></div>
          </div>
        </div>

        <!-- 中間車廂：今日 (現在站 - 粗黃色外框醒目標示) -->
        <div id="carriage-today" data-slot="today" class="carriage relative flex flex-col items-center bg-amber-50/50 border-4 border-amber-400 active-carriage rounded-3xl p-2 sm:p-4 shadow-lg transition-all duration-300 min-h-[200px] sm:min-h-[250px] md:min-h-[270px]">
          <!-- 站點標籤 -->
          <div class="w-full flex items-center justify-between border-b border-amber-200 pb-1.5 mb-1.5">
            <span class="px-2 py-0.5 text-xs sm:text-sm font-black bg-amber-400 text-amber-950 rounded-lg shadow-sm">★ 現在站</span>
            <span class="text-xl sm:text-3xl font-black text-amber-900">今日</span>
          </div>

          <!-- 放置區 -->
          <div class="drop-zone w-full flex-1 flex flex-col items-center justify-center border-2 border-dashed border-amber-400 rounded-2xl bg-amber-100/50 p-2 transition-colors relative">
            <div class="placeholder-hint flex flex-col items-center text-amber-700 text-center pointer-events-none">
              <svg class="w-9 h-9 mb-1 opacity-70 text-amber-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M12 9v3m0 0v3m0-3h3m-3 0H9m12 0a9 9 0 11-18 0 9 9 0 0118 0z"></path>
              </svg>
              <span class="text-xs sm:text-sm font-bold text-amber-800">放入今日事件</span>
            </div>
            <!-- 正確放置後掛載的卡片容器 -->
            <div class="placed-card-container hidden w-full h-full flex flex-col items-center justify-center"></div>
          </div>

          <!-- 輪子裝飾 -->
          <div class="w-full flex justify-around mt-1.5">
            <div class="w-6 h-6 sm:w-8 sm:h-8 bg-amber-800 rounded-full border-4 border-amber-300 flex items-center justify-center shadow-inner"><div class="w-2 h-2 bg-amber-100 rounded-full"></div></div>
            <div class="w-6 h-6 sm:w-8 sm:h-8 bg-amber-800 rounded-full border-4 border-amber-300 flex items-center justify-center shadow-inner"><div class="w-2 h-2 bg-amber-100 rounded-full"></div></div>
          </div>
        </div>

        <!-- 右車廂：聽日 (下一站 - 未來) -->
        <div id="carriage-tomorrow" data-slot="tomorrow" class="carriage relative flex flex-col items-center bg-white border-4 border-slate-300 rounded-3xl p-2 sm:p-4 shadow-md transition-all duration-300 min-h-[200px] sm:min-h-[250px] md:min-h-[270px]">
          <!-- 站點標籤 -->
          <div class="w-full flex items-center justify-between border-b border-slate-150 pb-1.5 mb-1.5">
            <span class="px-2 py-0.5 text-xs sm:text-sm font-bold bg-slate-100 text-slate-500 rounded-lg">下一站</span>
            <span class="text-lg sm:text-2xl font-black text-slate-700">聽日</span>
          </div>

          <!-- 放置區 -->
          <div class="drop-zone w-full flex-1 flex flex-col items-center justify-center border-2 border-dashed border-slate-300 rounded-2xl bg-slate-50/70 p-2 transition-colors relative">
            <div class="placeholder-hint flex flex-col items-center text-slate-400 text-center pointer-events-none">
              <svg class="w-9 h-9 mb-1 opacity-40 text-slate-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v3m0 0v3m0-3h3m-3 0H9m12 0a9 9 0 11-18 0 9 9 0 0118 0z"></path>
              </svg>
              <span class="text-xs sm:text-sm font-bold">放入聽日事件</span>
            </div>
            <!-- 正確放置後掛載的卡片容器 -->
            <div class="placed-card-container hidden w-full h-full flex flex-col items-center justify-center"></div>
          </div>

          <!-- 輪子裝飾 -->
          <div class="w-full flex justify-around mt-1.5">
            <div class="w-6 h-6 sm:w-8 sm:h-8 bg-slate-700 rounded-full border-4 border-slate-300 flex items-center justify-center shadow-inner"><div class="w-2 h-2 bg-slate-200 rounded-full"></div></div>
            <div class="w-6 h-6 sm:w-8 sm:h-8 bg-slate-700 rounded-full border-4 border-slate-300 flex items-center justify-center shadow-inner"><div class="w-2 h-2 bg-slate-200 rounded-full"></div></div>
          </div>
        </div>

      </div>
    </div>

    <!-- 底部操作區：動態載入之高清晰高對比實物事件圖卡 (支援拖曳或點選) -->
    <div class="mt-3 pt-2.5 border-t-2 border-slate-100">
      <div class="flex items-center justify-between mb-2">
        <p class="text-xs sm:text-sm font-bold text-slate-500">
          💡 操作模式：可以<span class="text-amber-600 font-extrabold">直接拖上去</span>，或<span class="text-blue-600 font-extrabold">點一下卡片再點車廂</span>
        </p>
        <span id="selected-status-badge" class="hidden text-xs font-bold text-blue-700 bg-blue-100 px-3 py-1 rounded-full animate-pulse">
          已選取，請點擊目標車廂
        </span>
      </div>

      <div id="cards-container" class="grid grid-cols-3 gap-3 sm:gap-6 max-w-4xl mx-auto">
        <!-- 5關的卡片將由 JS 根據關卡動態渲染 -->
      </div>
    </div>
  </main>

  <!-- 完成單關或全關成就展示視窗 (溫和自信激勵) -->
  <div id="completion-modal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm hidden flex items-center justify-center p-4 z-50">
    <div class="bg-white rounded-3xl p-6 sm:p-8 max-w-lg w-full text-center shadow-2xl border-4 border-amber-300 transform transition-all scale-95 opacity-0 duration-300" id="completion-modal-box">
      <div class="text-5xl sm:text-6xl mb-2 animate-bounce" id="modal-emoji">🎉</div>
      <h2 class="text-2xl sm:text-3xl font-black text-slate-800 mb-2" id="modal-title">好叻呀！完成本關喇！</h2>
      <p class="text-slate-600 text-sm sm:text-base font-bold mb-4" id="modal-desc">尋日、今日、聽日，時序小火車出發囉！</p>
      
      <!-- 溫和回顧小火車示意 -->
      <div id="modal-train-preview" class="bg-amber-50 rounded-2xl p-4 border border-amber-200 mb-6 flex items-center justify-around">
        <!-- 動態填入三節車廂回顧 -->
      </div>

      <div class="flex gap-3 justify-center">
        <button id="modal-replay-btn" class="w-1/2 py-3 bg-slate-100 hover:bg-slate-200 active:scale-95 text-slate-700 font-black text-sm sm:text-base rounded-2xl transition-transform border border-slate-200">
          再玩一次 🔄
        </button>
        <button id="modal-next-btn" class="w-1/2 py-3 bg-amber-500 hover:bg-amber-600 active:scale-95 text-white font-black text-sm sm:text-base rounded-2xl shadow-lg transition-transform">
          下一關 🚀
        </button>
      </div>
    </div>
  </div>

  <script>
    /* ==========================================================================
       1. 溫和音效合成器 (Web Audio API - 無需外源音訊，正向五聲音階低刺激)
       ========================================================================== */
    class CalmSoundController {
      constructor() {
        this.ctx = null;
      }

      initCtx() {
        if (!this.ctx) {
          const AudioContext = window.AudioContext || window.webkitAudioContext;
          this.ctx = new AudioContext();
        }
        if (this.ctx.state === 'suspended') {
          this.ctx.resume();
        }
      }

      // 溫和正向雙音 (C5 -> G5)
      playSoftSuccess() {
        this.initCtx();
        const now = this.ctx.currentTime;
        
        const osc1 = this.ctx.createOscillator();
        const gain1 = this.ctx.createGain();
        osc1.type = 'sine';
        osc1.frequency.setValueAtTime(523.25, now);
        gain1.gain.setValueAtTime(0.001, now);
        gain1.gain.exponentialRampToValueAtTime(0.18, now + 0.04);
        gain1.gain.exponentialRampToValueAtTime(0.0001, now + 0.4);
        osc1.connect(gain1);
        gain1.connect(this.ctx.destination);
        osc1.start(now);
        osc1.stop(now + 0.4);

        const osc2 = this.ctx.createOscillator();
        const gain2 = this.ctx.createGain();
        osc2.type = 'sine';
        osc2.frequency.setValueAtTime(783.99, now + 0.1);
        gain2.gain.setValueAtTime(0.001, now + 0.1);
        gain2.gain.exponentialRampToValueAtTime(0.22, now + 0.15);
        gain2.gain.exponentialRampToValueAtTime(0.0001, now + 0.65);
        osc2.connect(gain2);
        gain2.connect(this.ctx.destination);
        osc2.start(now + 0.1);
        osc2.stop(now + 0.65);
      }

      // 關卡完成輕快小調
      playCelebrationFanfare() {
        this.initCtx();
        const notes = [523.25, 659.25, 783.99, 1046.50];
        const start = this.ctx.currentTime;
        notes.forEach((freq, idx) => {
          const osc = this.ctx.createOscillator();
          const gain = this.ctx.createGain();
          osc.type = 'triangle';
          osc.frequency.setValueAtTime(freq, start + (idx * 0.13));
          gain.gain.setValueAtTime(0.001, start + (idx * 0.13));
          gain.gain.exponentialRampToValueAtTime(0.16, start + (idx * 0.13) + 0.04);
          gain.gain.exponentialRampToValueAtTime(0.0001, start + (idx * 0.13) + 0.45);
          osc.connect(gain);
          gain.connect(this.ctx.destination);
          osc.start(start + (idx * 0.13));
          osc.stop(start + (idx * 0.13) + 0.5);
        });
      }
    }

    const soundManager = new CalmSoundController();

    /* ==========================================================================
       2. 廣東話精準口語發音模組 (嚴格3-5字限制，支援zh-HK)
       ========================================================================== */
    function speakCantonese(phrase, onEndCallback) {
      if (!('speechSynthesis' in window)) {
        if (onEndCallback) onEndCallback();
        return;
      }

      window.speechSynthesis.cancel();
      const utterance = new SpeechSynthesisUtterance(phrase);
      utterance.lang = 'zh-HK';
      utterance.rate = 0.85; // 專為4歲幼童調緩語速，促進辨析仿說
      utterance.pitch = 1.1;

      const voices = window.speechSynthesis.getVoices();
      const hkVoice = voices.find(v => v.lang === 'zh-HK' || v.lang.includes('yue') || v.lang === 'zh_HK');
      if (hkVoice) {
        utterance.voice = hkVoice;
      }

      if (onEndCallback) {
        utterance.onend = onEndCallback;
      }
      window.speechSynthesis.speak(utterance);
    }

    if ('speechSynthesis' in window) {
      window.speechSynthesis.onvoiceschanged = () => {};
    }

    /* ==========================================================================
       3. 完整五個關卡情景設定與純淨SVG圖形庫
       ========================================================================== */
    const ALL_LEVELS = [
      // 第 1 關：生活與遊戲篇
      {
        id: 0,
        badge: '第 1 關',
        theme: '生活與遊戲篇',
        steps: [
          {
            targetKey: 'today',
            phrase: '今日玩積木',
            tutorHint: '今日係依家嘅意思！引導仿說：「今日玩積木」',
            affirmation: '係今日！今日係依家！',
            activeCarriageId: 'carriage-today'
          },
          {
            targetKey: 'yesterday',
            phrase: '尋日落大雨',
            tutorHint: '尋日係過咗去！引導仿說：「尋日落大雨」',
            affirmation: '係尋日！尋日過咗去！',
            activeCarriageId: 'carriage-yesterday'
          },
          {
            targetKey: 'tomorrow',
            phrase: '聽日食蛋糕',
            tutorHint: '聽日係未到㗎！引導仿說：「聽日食蛋糕」',
            affirmation: '係聽日！聽日仲未到！',
            activeCarriageId: 'carriage-tomorrow'
          }
        ],
        cards: [
          {
            type: 'today',
            title: '積木',
            subtitle: '玩積木',
            voice: '今日玩積木',
            svg: `
              <svg class="w-full h-full drop-shadow-sm" viewBox="0 0 120 120" fill="none">
                <rect x="20" y="66" width="36" height="34" rx="5" fill="#EF4444"/>
                <rect x="24" y="70" width="28" height="26" rx="3" fill="#F87171"/>
                <circle cx="30" cy="76" r="3" fill="#FEE2E2"/>
                <rect x="62" y="66" width="38" height="34" rx="5" fill="#F59E0B"/>
                <rect x="66" y="70" width="30" height="26" rx="3" fill="#FBBF24"/>
                <circle cx="74" cy="76" r="3" fill="#FEF3C7"/>
                <path d="M40 64H80L60 28L40 64Z" fill="#3B82F6"/>
                <path d="M46 60H74L60 35L46 60Z" fill="#60A5FA"/>
              </svg>
            `
          },
          {
            type: 'yesterday',
            title: '雨傘',
            subtitle: '落大雨',
            voice: '尋日落大雨',
            svg: `
              <svg class="w-full h-full drop-shadow-sm" viewBox="0 0 120 120" fill="none">
                <circle cx="28" cy="24" r="3.5" fill="#38BDF8"/>
                <circle cx="92" cy="26" r="3.5" fill="#38BDF8"/>
                <path d="M60 22V16" stroke="#475569" stroke-width="4" stroke-linecap="round"/>
                <path d="M18 64C18 40.8 36.8 22 60 22C83.2 22 102 40.8 102 64C102 65 101.2 66 100 66C88 66 84 59 74 59C64 59 60 66 60 66C60 66 56 59 46 59C36 59 32 66 20 66C18.8 66 18 65 18 64Z" fill="#0284C7"/>
                <path d="M38 27C44.5 37 47 50 46 59C36 59 32 66 20 66C18.8 66 18 65 18 64C18 40.8 36.8 22 60 22C51.5 22 43.8 24 38 27Z" fill="#38BDF8"/>
                <path d="M60 64V94C60 100 54 103 49 99" stroke="#334155" stroke-width="4.5" stroke-linecap="round"/>
              </svg>
            `
          },
          {
            type: 'tomorrow',
            title: '蛋糕',
            subtitle: '食蛋糕',
            voice: '聽日食蛋糕',
            svg: `
              <svg class="w-full h-full drop-shadow-sm" viewBox="0 0 120 120" fill="none">
                <ellipse cx="60" cy="98" rx="42" ry="8" fill="#CBD5E1"/>
                <rect x="25" y="66" width="70" height="30" rx="6" fill="#F472B6"/>
                <path d="M25 74C29 78 33 78 37 74C41 70 45 70 49 74C53 78 57 78 61 74C65 70 69 70 73 74C77 78 81 78 85 74C89 70 93 70 95 74V66H25V74Z" fill="#FFF1F2"/>
                <circle cx="38" cy="63" r="5" fill="#E11D48"/>
                <circle cx="60" cy="63" r="5" fill="#E11D48"/>
                <circle cx="82" cy="63" r="5" fill="#E11D48"/>
                <rect x="57" y="38" width="6" height="22" rx="2" fill="#38BDF8"/>
                <path d="M60 22C64 27 65 32 60 36C55 32 56 27 60 22Z" fill="#F59E0B"/>
              </svg>
            `
          }
        ]
      },

      // 第 2 關：戶外與校園篇
      {
        id: 1,
        badge: '第 2 關',
        theme: '戶外與校園篇',
        steps: [
          {
            targetKey: 'today',
            phrase: '今日去公園',
            tutorHint: '今日係依家嘅意思！引導仿說：「今日去公園」',
            affirmation: '係今日！今日去公園！',
            activeCarriageId: 'carriage-today'
          },
          {
            targetKey: 'yesterday',
            phrase: '尋日返學校',
            tutorHint: '尋日係過咗去！引導仿說：「尋日返學校」',
            affirmation: '係尋日！尋日返學校！',
            activeCarriageId: 'carriage-yesterday'
          },
          {
            targetKey: 'tomorrow',
            phrase: '聽日去游水',
            tutorHint: '聽日係未到㗎！引導仿說：「聽日去游水」',
            affirmation: '係聽日！聽日去游水！',
            activeCarriageId: 'carriage-tomorrow'
          }
        ],
        cards: [
          {
            type: 'today',
            title: '公園滑梯',
            subtitle: '去公園',
            voice: '今日去公園',
            svg: `
              <svg class="w-full h-full drop-shadow-sm" viewBox="0 0 120 120" fill="none">
                <!-- 戶外公園滑梯與綠草 -->
                <path d="M15 105C35 100 85 100 105 105" stroke="#10B981" stroke-width="6" stroke-linecap="round"/>
                <!-- 階梯與支架 -->
                <path d="M30 102V40" stroke="#64748B" stroke-width="5" stroke-linecap="round"/>
                <path d="M22 56H38M22 72H38M22 88H38" stroke="#F59E0B" stroke-width="4" stroke-linecap="round"/>
                <!-- 平台 -->
                <rect x="28" y="36" width="18" height="8" rx="2" fill="#3B82F6"/>
                <!-- 滑道波浪 -->
                <path d="M42 42C56 42 62 82 98 96" stroke="#EF4444" stroke-width="8" stroke-linecap="round"/>
                <path d="M44 42C58 42 64 82 96 94" stroke="#F87171" stroke-width="4" stroke-linecap="round"/>
              </svg>
            `
          },
          {
            type: 'yesterday',
            title: '書包學校',
            subtitle: '返學校',
            voice: '尋日返學校',
            svg: `
              <svg class="w-full h-full drop-shadow-sm" viewBox="0 0 120 120" fill="none">
                <!-- 可愛雙肩小書包 -->
                <rect x="30" y="42" width="60" height="58" rx="14" fill="#3B82F6"/>
                <path d="M46 42V30C46 25 74 25 74 30V42" stroke="#1D4ED8" stroke-width="5" stroke-linecap="round"/>
                <!-- 前袋 -->
                <rect x="38" y="66" width="44" height="26" rx="8" fill="#60A5FA"/>
                <!-- 扣帶與名牌 -->
                <rect x="54" y="60" width="12" height="12" rx="3" fill="#FBBF24"/>
                <circle cx="60" cy="66" r="2" fill="#78350F"/>
              </svg>
            `
          },
          {
            type: 'tomorrow',
            title: '游水泳圈',
            subtitle: '去游水',
            voice: '聽日去游水',
            svg: `
              <svg class="w-full h-full drop-shadow-sm" viewBox="0 0 120 120" fill="none">
                <!-- 泳圈與水波 -->
                <path d="M18 100C30 96 45 104 60 100C75 96 90 104 102 100" stroke="#38BDF8" stroke-width="5" stroke-linecap="round"/>
                <!-- 救生圈主體 -->
                <ellipse cx="60" cy="58" rx="42" ry="32" fill="#F43F5E"/>
                <ellipse cx="60" cy="58" rx="20" ry="14" fill="#F8FAFC"/>
                <!-- 條紋裝飾 -->
                <path d="M38 34L30 42M82 34L90 42M30 74L38 82M90 74L82 82" stroke="#FFFFFF" stroke-width="9" stroke-linecap="round"/>
              </svg>
            `
          }
        ]
      },

      // 第 3 關：飲食與生活篇
      {
        id: 2,
        badge: '第 3 關',
        theme: '飲食與生活篇',
        steps: [
          {
            targetKey: 'today',
            phrase: '今日食蘋果',
            tutorHint: '今日係依家嘅意思！引導仿說：「今日食蘋果」',
            affirmation: '係今日！今日食蘋果！',
            activeCarriageId: 'carriage-today'
          },
          {
            targetKey: 'yesterday',
            phrase: '尋日飲橙汁',
            tutorHint: '尋日係過咗去！引導仿說：「尋日飲橙汁」',
            affirmation: '係尋日！尋日飲橙汁！',
            activeCarriageId: 'carriage-yesterday'
          },
          {
            targetKey: 'tomorrow',
            phrase: '聽日食雪糕',
            tutorHint: '聽日係未到㗎！引導仿說：「聽日食雪糕」',
            affirmation: '係聽日！聽日食雪糕！',
            activeCarriageId: 'carriage-tomorrow'
          }
        ],
        cards: [
          {
            type: 'today',
            title: '紅蘋果',
            subtitle: '食蘋果',
            voice: '今日食蘋果',
            svg: `
              <svg class="w-full h-full drop-shadow-sm" viewBox="0 0 120 120" fill="none">
                <!-- 蘋果葉與柄 -->
                <path d="M60 22C62 30 58 40 56 46" stroke="#78350F" stroke-width="4.5" stroke-linecap="round"/>
                <path d="M60 26C72 20 80 28 76 36C68 36 62 30 60 26Z" fill="#22C55E"/>
                <!-- 圓潤紅蘋果 -->
                <path d="M60 48C45 32 20 42 22 72C24 96 50 102 60 98C70 102 96 96 98 72C100 42 75 32 60 48Z" fill="#DC2626"/>
                <path d="M35 56C30 64 30 76 36 84" stroke="#F87171" stroke-width="4" stroke-linecap="round"/>
              </svg>
            `
          },
          {
            type: 'yesterday',
            title: '鮮橙汁',
            subtitle: '飲橙汁',
            voice: '尋日飲橙汁',
            svg: `
              <svg class="w-full h-full drop-shadow-sm" viewBox="0 0 120 120" fill="none">
                <!-- 杯身與飲管 -->
                <path d="M72 16L62 36" stroke="#EF4444" stroke-width="5" stroke-linecap="round"/>
                <path d="M30 38H90L82 98C82 102 78 106 74 106H46C42 106 38 102 38 98L30 38Z" fill="#E2E8F0"/>
                <path d="M33 46H87L80 96C80 99 77 102 74 102H46C43 102 40 99 40 96L33 46Z" fill="#FB923C"/>
                <!-- 橙片裝飾 -->
                <circle cx="86" cy="40" r="14" fill="#F97316"/>
                <circle cx="86" cy="40" r="10" fill="#FED7AA"/>
              </svg>
            `
          },
          {
            type: 'tomorrow',
            title: '甜筒雪糕',
            subtitle: '食雪糕',
            voice: '聽日食雪糕',
            svg: `
              <svg class="w-full h-full drop-shadow-sm" viewBox="0 0 120 120" fill="none">
                <!-- 甜筒餅乾 -->
                <path d="M42 66L60 106L78 66Z" fill="#D97706"/>
                <path d="M48 76L72 76M54 88L66 88" stroke="#B45309" stroke-width="3" stroke-linecap="round"/>
                <!-- 雙色雪糕球 -->
                <circle cx="60" cy="52" r="24" fill="#EC4899"/>
                <circle cx="50" cy="42" r="6" fill="#F472B6"/>
                <circle cx="60" cy="30" r="7" fill="#EF4444"/>
              </svg>
            `
          }
        ]
      },

      // 第 4 關：家庭與活動篇
      {
        id: 3,
        badge: '第 4 關',
        theme: '家庭與活動篇',
        steps: [
          {
            targetKey: 'today',
            phrase: '今日畫公仔',
            tutorHint: '今日係依家嘅意思！引導仿說：「今日畫公仔」',
            affirmation: '係今日！今日畫公仔！',
            activeCarriageId: 'carriage-today'
          },
          {
            targetKey: 'yesterday',
            phrase: '尋日去超市',
            tutorHint: '尋日係過咗去！引導仿說：「尋日去超市」',
            affirmation: '係尋日！尋日去超市！',
            activeCarriageId: 'carriage-yesterday'
          },
          {
            targetKey: 'tomorrow',
            phrase: '聽日睇醫生',
            tutorHint: '聽日係未到㗎！引導仿說：「聽日睇醫生」',
            affirmation: '係聽日！聽日睇醫生！',
            activeCarriageId: 'carriage-tomorrow'
          }
        ],
        cards: [
          {
            type: 'today',
            title: '彩色彩筆',
            subtitle: '畫公仔',
            voice: '今日畫公仔',
            svg: `
              <svg class="w-full h-full drop-shadow-sm" viewBox="0 0 120 120" fill="none">
                <!-- 畫板圖畫與彩色筆 -->
                <rect x="22" y="32" width="76" height="58" rx="8" fill="#F1F5F9" stroke="#94A3B8" stroke-width="4"/>
                <!-- 畫中的笑臉太陽 -->
                <circle cx="44" cy="58" r="12" fill="#FBBF24"/>
                <path d="M40 56H41M47 56H48M40 62C42 65 46 65 48 62" stroke="#78350F" stroke-width="2" stroke-linecap="round"/>
                <!-- 傾斜畫筆 -->
                <path d="M70 82L92 38L98 42L76 86Z" fill="#3B82F6"/>
                <path d="M68 86L70 82L76 86Z" fill="#1E293B"/>
              </svg>
            `
          },
          {
            type: 'yesterday',
            title: '超市購物車',
            subtitle: '去超市',
            voice: '尋日去超市',
            svg: `
              <svg class="w-full h-full drop-shadow-sm" viewBox="0 0 120 120" fill="none">
                <!-- 購物手推車手柄與籃子 -->
                <path d="M22 32H34L46 76H86L96 44H38" stroke="#0284C7" stroke-width="5" stroke-linecap="round" stroke-linejoin="round"/>
                <!-- 網格 -->
                <path d="M52 46V74M66 46V74M80 46V74" stroke="#38BDF8" stroke-width="3" stroke-linecap="round"/>
                <!-- 輪子 -->
                <circle cx="48" cy="92" r="7" fill="#334155"/>
                <circle cx="82" cy="92" r="7" fill="#334155"/>
              </svg>
            `
          },
          {
            type: 'tomorrow',
            title: '醫生聽診器',
            subtitle: '睇醫生',
            voice: '聽日睇醫生',
            svg: `
              <svg class="w-full h-full drop-shadow-sm" viewBox="0 0 120 120" fill="none">
                <!-- 聽診器耳塞與導管 -->
                <path d="M38 24V46C38 60 52 70 60 70C68 70 82 60 82 46V24" stroke="#059669" stroke-width="5" stroke-linecap="round"/>
                <path d="M60 70V86" stroke="#059669" stroke-width="5" stroke-linecap="round"/>
                <!-- 聽診頭 -->
                <circle cx="60" cy="94" r="12" fill="#E2E8F0" stroke="#059669" stroke-width="4"/>
                <circle cx="60" cy="94" r="5" fill="#10B981"/>
              </svg>
            `
          }
        ]
      },

      // 第 5 關：綜合挑戰篇
      {
        id: 4,
        badge: '第 5 關',
        theme: '綜合挑戰篇',
        steps: [
          {
            targetKey: 'today',
            phrase: '今日砌拼圖',
            tutorHint: '今日係依家嘅意思！引導仿說：「今日砌拼圖」',
            affirmation: '係今日！今日砌拼圖！',
            activeCarriageId: 'carriage-today'
          },
          {
            targetKey: 'yesterday',
            phrase: '尋日去沙灘',
            tutorHint: '尋日係過咗去！引導仿說：「尋日去沙灘」',
            affirmation: '係尋日！尋日去沙灘！',
            activeCarriageId: 'carriage-yesterday'
          },
          {
            targetKey: 'tomorrow',
            phrase: '聽日去搭車',
            tutorHint: '聽日係未到㗎！引導仿說：「聽日去搭車」',
            affirmation: '係聽日！聽日去搭車！',
            activeCarriageId: 'carriage-tomorrow'
          }
        ],
        cards: [
          {
            type: 'today',
            title: '四色拼圖',
            subtitle: '砌拼圖',
            voice: '今日砌拼圖',
            svg: `
              <svg class="w-full h-full drop-shadow-sm" viewBox="0 0 120 120" fill="none">
                <!-- 經典幼兒色彩拼圖塊 -->
                <path d="M26 26H58V40C62 40 66 44 66 48C66 52 62 56 58 56V58H26V26Z" fill="#3B82F6"/>
                <path d="M58 26H94V58H80C80 62 76 66 72 66C68 66 64 62 64 58H58V26Z" fill="#EF4444"/>
                <path d="M26 58H58V94H26V80C30 80 34 76 34 72C34 68 30 64 26 64V58Z" fill="#F59E0B"/>
                <path d="M58 58H94V94H58V58Z" fill="#10B981"/>
              </svg>
            `
          },
          {
            type: 'yesterday',
            title: '沙灘玩具',
            subtitle: '去沙灘',
            voice: '尋日去沙灘',
            svg: `
              <svg class="w-full h-full drop-shadow-sm" viewBox="0 0 120 120" fill="none">
                <!-- 沙桶、鏟子與沙堆 -->
                <ellipse cx="60" cy="100" rx="42" ry="8" fill="#FDE68A"/>
                <!-- 水桶 -->
                <path d="M36 50L44 94H76L84 50H36Z" fill="#EC4899"/>
                <path d="M34 50C34 40 86 40 86 50" stroke="#DB2777" stroke-width="4"/>
                <!-- 黃色沙鏟 -->
                <path d="M80 34L94 88" stroke="#F59E0B" stroke-width="6" stroke-linecap="round"/>
                <rect x="76" y="28" width="12" height="10" rx="3" fill="#FBBF24"/>
              </svg>
            `
          },
          {
            type: 'tomorrow',
            title: '黃色巴士',
            subtitle: '去搭車',
            voice: '聽日去搭車',
            svg: `
              <svg class="w-full h-full drop-shadow-sm" viewBox="0 0 120 120" fill="none">
                <!-- 幼兒大巴車身 -->
                <rect x="20" y="38" width="80" height="52" rx="12" fill="#FACC15"/>
                <!-- 車窗 -->
                <rect x="28" y="46" width="18" height="18" rx="4" fill="#38BDF8"/>
                <rect x="52" y="46" width="18" height="18" rx="4" fill="#38BDF8"/>
                <rect x="76" y="46" width="18" height="18" rx="4" fill="#38BDF8"/>
                <!-- 車燈與輪子 -->
                <circle cx="94" cy="74" r="4" fill="#EF4444"/>
                <circle cx="36" cy="92" r="9" fill="#1E293B"/>
                <circle cx="84" cy="92" r="9" fill="#1E293B"/>
              </svg>
            `
          }
        ]
      }
    ];

    /* ==========================================================================
       4. 狀態管理與關卡渲染邏輯
       ========================================================================== */
    let currentLevelIdx = 0;
    let currentStepIdx = 0;
    let selectedItemCard = null;
    const completedItems = new Set();

    // DOM 元素引用
    const instructionText = document.getElementById('instruction-text');
    const tutorSpeechPrompt = document.getElementById('tutor-speech-prompt');
    const selectedStatusBadge = document.getElementById('selected-status-badge');
    const cardsContainer = document.getElementById('cards-container');
    const levelBadge = document.getElementById('level-badge');
    const levelTitleDesc = document.getElementById('level-title-desc');
    const btnNextStep = document.getElementById('btn-next-step');
    const completionModal = document.getElementById('completion-modal');
    const completionModalBox = document.getElementById('completion-modal-box');
    const modalEmoji = document.getElementById('modal-emoji');
    const modalTitle = document.getElementById('modal-title');
    const modalDesc = document.getElementById('modal-desc');
    const modalTrainPreview = document.getElementById('modal-train-preview');
    const btnReplayAudio = document.getElementById('btn-replay-audio');
    const btnResetLevel = document.getElementById('btn-reset-level');
    const modalReplayBtn = document.getElementById('modal-replay-btn');
    const modalNextBtn = document.getElementById('modal-next-btn');

    // 載入關卡卡片與重置畫布
    function loadLevel(levelIndex, autoSpeak = true) {
      currentLevelIdx = levelIndex;
      currentStepIdx = 0;
      completedItems.clear();
      clearSelection();
      btnNextStep.classList.add('hidden');

      const lvlData = ALL_LEVELS[currentLevelIdx];

      // 更新頂部標籤
      levelBadge.textContent = lvlData.badge;
      levelTitleDesc.textContent = lvlData.theme;

      // 更新關卡選取器樣式
      document.querySelectorAll('.lvl-btn').forEach((btn, idx) => {
        if (idx === currentLevelIdx) {
          btn.className = "lvl-btn px-3 py-1.5 text-xs sm:text-sm font-black rounded-xl bg-amber-500 text-white shadow-sm transition-all";
        } else {
          btn.className = "lvl-btn px-3 py-1.5 text-xs sm:text-sm font-black rounded-xl text-slate-600 hover:bg-slate-200 transition-all";
        }
      });

      // 清空三大車廂放置區
      document.querySelectorAll('.carriage').forEach(carriage => {
        const placeholder = carriage.querySelector('.placeholder-hint');
        const placedContainer = carriage.querySelector('.placed-card-container');
        placeholder.classList.remove('hidden');
        placedContainer.classList.add('hidden');
        placedContainer.innerHTML = '';
      });

      // 重新生成 3 張操作圖卡 (保持大面積、去背、高對比)
      cardsContainer.innerHTML = '';
      lvlData.cards.forEach(card => {
        const cardEl = document.createElement('div');
        cardEl.id = `card-${card.type}`;
        cardEl.setAttribute('data-type', card.type);
        cardEl.setAttribute('data-voice', card.voice);
        cardEl.className = "item-card bg-white border-2 border-slate-200 hover:border-amber-400 rounded-3xl p-2.5 sm:p-5 flex flex-col items-center justify-center shadow-md cursor-grab active:cursor-grabbing transform transition-all duration-200 hover:shadow-lg touch-none";
        
        cardEl.innerHTML = `
          <div class="w-16 h-16 sm:w-24 sm:h-24 md:w-28 md:h-28 flex items-center justify-center pointer-events-none">
            ${card.svg}
          </div>
          <span class="mt-1 text-sm sm:text-xl font-black text-slate-800 tracking-wide pointer-events-none">${card.title}</span>
          <span class="text-xs font-semibold text-slate-400 pointer-events-none">${card.subtitle}</span>
        `;
        cardsContainer.appendChild(cardEl);
      });

      // 重新綁定事件
      initTouchAndDragForCurrentCards();

      // 更新當前步驟焦點與語音
      updateGameStepVisuals(autoSpeak);
    }

    function updateGameStepVisuals(playVoice = true) {
      const lvlData = ALL_LEVELS[currentLevelIdx];

      if (currentStepIdx >= lvlData.steps.length) {
        showCelebration();
        return;
      }

      btnNextStep.classList.add('hidden');
      const step = lvlData.steps[currentStepIdx];

      // 更新畫面看板文字與導師指引
      instructionText.textContent = step.phrase;
      tutorSpeechPrompt.textContent = step.tutorHint;

      // 高亮焦點車廂
      document.querySelectorAll('.carriage').forEach(el => el.classList.remove('active-carriage'));
      const activeCarriage = document.getElementById(step.activeCarriageId);
      if (activeCarriage) {
        activeCarriage.classList.add('active-carriage');
      }

      if (playVoice) {
        speakCantonese(step.phrase);
      }
    }

    /* 正確放置處理：溫和提示音、綠勾號與口語仿說鷹架 */
    function handleCorrectPlacement(cardType, cardElement, targetCarriageElement) {
      const lvlData = ALL_LEVELS[currentLevelIdx];
      const step = lvlData.steps[currentStepIdx];

      soundManager.playSoftSuccess();

      // 車廂內放入複製卡片
      const placeholder = targetCarriageElement.querySelector('.placeholder-hint');
      const placedContainer = targetCarriageElement.querySelector('.placed-card-container');
      placeholder.classList.add('hidden');
      placedContainer.classList.remove('hidden');

      const cardClone = cardElement.cloneNode(true);
      cardClone.id = `${cardClone.id}-placed`;
      cardClone.className = "w-full h-full flex flex-col items-center justify-center p-1 pointer-events-none relative";

      // 綠色大打勾成功標記
      const checkBadge = document.createElement('div');
      checkBadge.className = 'checkmark-pop absolute top-1 right-2 bg-emerald-500 text-white rounded-full p-1 shadow-md';
      checkBadge.innerHTML = `<svg class="w-5 h-5 sm:w-6 sm:h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 13l4 4L19 7"></path></svg>`;
      cardClone.appendChild(checkBadge);

      placedContainer.innerHTML = '';
      placedContainer.appendChild(cardClone);

      // 原卡片半透明
      cardElement.classList.add('opacity-20', 'pointer-events-none', 'scale-90');
      completedItems.add(cardType);
      clearSelection();

      // 觸發仿說引導提示面板
      instructionText.textContent = step.affirmation;
      tutorSpeechPrompt.textContent = `導師引導仿說：「${step.affirmation} 一齊講一次。」`;
      btnNextStep.classList.remove('hidden');

      speakCantonese(step.affirmation, () => {
        // 提供短暫自動過渡，導師亦可隨時點擊手動按鈕
        setTimeout(() => {
          if (currentStepIdx < lvlData.steps.length - 1) {
            currentStepIdx++;
            updateGameStepVisuals(true);
          } else {
            showCelebration();
          }
        }, 1100);
      });
    }

    // 放置錯誤時溫和回彈與微幅搖晃（無負面音效）
    function handleIncorrectPlacement(cardElement) {
      cardElement.classList.add('nudge-retry');
      setTimeout(() => {
        cardElement.classList.remove('nudge-retry');
      }, 350);

      const step = ALL_LEVELS[currentLevelIdx].steps[currentStepIdx];
      speakCantonese(step.phrase);
    }

    /* ==========================================================================
       5. 雙模式操作：觸控拖曳 (Touch Drag) 與 點擊選取 (Tap-to-Place)
       ========================================================================== */
    function clearSelection() {
      if (selectedItemCard) {
        selectedItemCard.classList.remove('ring-4', 'ring-blue-400', 'bg-blue-50');
      }
      selectedItemCard = null;
      selectedStatusBadge.classList.add('hidden');
    }

    function handleCardClick(cardEl) {
      const type = cardEl.getAttribute('data-type');
      if (completedItems.has(type)) return;

      soundManager.initCtx();

      if (selectedItemCard === cardEl) {
        clearSelection();
        return;
      }

      clearSelection();
      selectedItemCard = cardEl;
      cardEl.classList.add('ring-4', 'ring-blue-400', 'bg-blue-50');
      selectedStatusBadge.classList.remove('hidden');

      const voice = cardEl.getAttribute('data-voice');
      speakCantonese(voice);
    }

    function handleCarriageClick(carriageEl) {
      if (!selectedItemCard) return;

      const step = ALL_LEVELS[currentLevelIdx].steps[currentStepIdx];
      const targetSlot = carriageEl.getAttribute('data-slot');
      const cardType = selectedItemCard.getAttribute('data-type');

      if (targetSlot === step.targetKey && cardType === step.targetKey) {
        handleCorrectPlacement(cardType, selectedItemCard, carriageEl);
      } else {
        handleIncorrectPlacement(selectedItemCard);
        clearSelection();
      }
    }

    let touchDragProxy = null;
    let activeDraggedCard = null;

    function initTouchAndDragForCurrentCards() {
      const cards = cardsContainer.querySelectorAll('.item-card');

      cards.forEach(card => {
        card.addEventListener('click', () => handleCardClick(card));

        card.addEventListener('pointerdown', (e) => {
          const type = card.getAttribute('data-type');
          if (completedItems.has(type)) return;

          soundManager.initCtx();
          activeDraggedCard = card;

          const rect = card.getBoundingClientRect();
          touchDragProxy = card.cloneNode(true);
          touchDragProxy.id = "active-touch-proxy";
          touchDragProxy.classList.add('touch-proxy');
          touchDragProxy.style.width = `${rect.width}px`;
          touchDragProxy.style.height = `${rect.height}px`;
          touchDragProxy.style.left = `${e.clientX}px`;
          touchDragProxy.style.top = `${e.clientY}px`;

          document.body.appendChild(touchDragProxy);
          card.classList.add('dragging-card');
        });
      });
    }

    window.addEventListener('pointermove', (e) => {
      if (!touchDragProxy || !activeDraggedCard) return;
      e.preventDefault();
      touchDragProxy.style.left = `${e.clientX}px`;
      touchDragProxy.style.top = `${e.clientY}px`;
    });

    window.addEventListener('pointerup', (e) => {
      if (!activeDraggedCard) return;

      const step = ALL_LEVELS[currentLevelIdx].steps[currentStepIdx];
      const cardType = activeDraggedCard.getAttribute('data-type');

      if (touchDragProxy) {
        touchDragProxy.remove();
        touchDragProxy = null;
      }
      activeDraggedCard.classList.remove('dragging-card');

      const elemBelow = document.elementFromPoint(e.clientX, e.clientY);
      const targetCarriage = elemBelow ? elemBelow.closest('.carriage') : null;

      if (targetCarriage) {
        const slot = targetCarriage.getAttribute('data-slot');
        if (slot === step.targetKey && cardType === step.targetKey) {
          handleCorrectPlacement(cardType, activeDraggedCard, targetCarriage);
        } else {
          handleIncorrectPlacement(activeDraggedCard);
        }
      }

      activeDraggedCard = null;
    });

    // 車廂點擊監聽 (全域只掛載一次)
    document.querySelectorAll('.carriage').forEach(carriage => {
      carriage.addEventListener('click', () => handleCarriageClick(carriage));
    });

    /* ==========================================================================
       6. 關卡完成展示與下一關推進
       ========================================================================== */
    function showCelebration() {
      soundManager.playCelebrationFanfare();
      const isFinalLevel = currentLevelIdx === ALL_LEVELS.length - 1;
      const lvlData = ALL_LEVELS[currentLevelIdx];

      modalEmoji.textContent = isFinalLevel ? '🏆' : '🎉';
      modalTitle.textContent = isFinalLevel ? '太叻喇！完成五個時序關卡！' : `好叻呀！完成${lvlData.badge}！`;
      modalDesc.textContent = isFinalLevel ? '尋日、今日、聽日，全部都掌握好喇！' : `${lvlData.theme}，時序火車出發囉！`;

      // 產生三節車廂回顧小圖示
      modalTrainPreview.innerHTML = `
        <div class="flex flex-col items-center">
          <span class="text-xs font-bold text-slate-500">尋日</span>
          <span class="text-xs font-bold text-slate-700 mt-1">${lvlData.cards.find(c=>c.type==='yesterday').title}</span>
        </div>
        <span class="text-slate-300 font-black">➔</span>
        <div class="flex flex-col items-center bg-white px-3 py-1 rounded-xl shadow-sm border border-amber-300">
          <span class="text-xs font-black text-amber-600">今日</span>
          <span class="text-xs font-bold text-amber-900 mt-1">${lvlData.cards.find(c=>c.type==='today').title}</span>
        </div>
        <span class="text-slate-300 font-black">➔</span>
        <div class="flex flex-col items-center">
          <span class="text-xs font-bold text-slate-500">聽日</span>
          <span class="text-xs font-bold text-slate-700 mt-1">${lvlData.cards.find(c=>c.type==='tomorrow').title}</span>
        </div>
      `;

      modalNextBtn.textContent = isFinalLevel ? '從第 1 關再玩' : '下一關 🚀';

      completionModal.classList.remove('hidden');
      setTimeout(() => {
        completionModalBox.classList.remove('scale-95', 'opacity-0');
        completionModalBox.classList.add('scale-100', 'opacity-100');
      }, 40);

      speakCantonese('好叻呀！尋日、今日、聽日！');
    }

    function closeModal() {
      completionModalBox.classList.add('scale-95', 'opacity-0');
      completionModalBox.classList.remove('scale-100', 'opacity-100');
      setTimeout(() => {
        completionModal.classList.add('hidden');
      }, 200);
    }

    /* ==========================================================================
       7. 按鈕控制與初始化
       ========================================================================== */
    window.addEventListener('DOMContentLoaded', () => {
      // 關卡切換按鈕綁定
      document.querySelectorAll('.lvl-btn').forEach(btn => {
        btn.addEventListener('click', (e) => {
          soundManager.initCtx();
          const targetLvl = parseInt(e.currentTarget.getAttribute('data-level'), 10);
          loadLevel(targetLvl, true);
        });
      });

      // 導師手動前進按鈕
      btnNextStep.addEventListener('click', () => {
        const lvlData = ALL_LEVELS[currentLevelIdx];
        if (currentStepIdx < lvlData.steps.length - 1) {
          currentStepIdx++;
          updateGameStepVisuals(true);
        } else {
          showCelebration();
        }
      });

      // 重聽語音
      btnReplayAudio.addEventListener('click', () => {
        soundManager.initCtx();
        const lvlData = ALL_LEVELS[currentLevelIdx];
        if (currentStepIdx < lvlData.steps.length) {
          speakCantonese(lvlData.steps[currentStepIdx].phrase);
        } else {
          speakCantonese('好叻呀！尋日、今日、聽日！');
        }
      });

      // 重玩本關
      btnResetLevel.addEventListener('click', () => {
        soundManager.initCtx();
        loadLevel(currentLevelIdx, true);
      });

      // Modal 再玩一次
      modalReplayBtn.addEventListener('click', () => {
        closeModal();
        loadLevel(currentLevelIdx, true);
      });

      // Modal 下一關
      modalNextBtn.addEventListener('click', () => {
        closeModal();
        const nextIdx = (currentLevelIdx + 1) % ALL_LEVELS.length;
        loadLevel(nextIdx, true);
      });

      // 初始加載第 1 關 (靜態不自動突發播放語音，待點擊後自然啟動)
      loadLevel(0, false);

      // 初次觸碰激活音訊上下文
      window.addEventListener('pointerdown', () => soundManager.initCtx(), { once: true });
    });
  </script>
</body>
</html>

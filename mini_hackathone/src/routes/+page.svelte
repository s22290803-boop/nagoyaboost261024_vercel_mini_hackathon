<script>
  import { onMount } from 'svelte';

  // おみくじの結果データ
  const results = [
    {
      title: '超大吉 🌟',
      desc: '最高の1日になります！やりたいことに挑戦しましょう。',
      bgColor: 'linear-gradient(135deg, #2b1055 0%, #7597de 100%)',
      accentColor: '#ffd700',
      soundFreq: 880 // 効果音の周波数 (高音)
    },
    {
      title: '大吉 🎉',
      desc: '運気は絶好調！素晴らしいチャンスが巡ってきそうです。',
      bgColor: 'linear-gradient(135deg, #1e3c72 0%, #2a5298 100%)',
      accentColor: '#00f2fe',
      soundFreq: 660
    },
    {
      title: '吉 🌸',
      desc: '穏やかで幸運な1日。日常の小さな幸せを見つけられます。',
      bgColor: 'linear-gradient(135deg, #134e5e 0%, #71b280 100%)',
      accentColor: '#4facfe',
      soundFreq: 523
    },
    {
      title: '中吉 🍀',
      desc: '努力が実を結ぶ日。焦らず着実に進めましょう。',
      bgColor: 'linear-gradient(135deg, #3a1c71 0%, #d76d77 100%)',
      accentColor: '#ff0844',
      soundFreq: 440
    },
    {
      title: '小吉 🌱',
      desc: 'ささやかな幸せが訪れる予感。笑顔で過ごしましょう。',
      bgColor: 'linear-gradient(135deg, #200122 0%, #6f0000 100%)',
      accentColor: '#ffb199',
      soundFreq: 392
    },
    {
      title: '末吉 🕊️',
      desc: 'これから運気が上がっていきます。マイペースが吉。',
      bgColor: 'linear-gradient(135deg, #0f2027 0%, #203a43 100%)',
      accentColor: '#a8c0ff',
      soundFreq: 330
    }
  ];

  let slots = ['❓', '❓', '❓'];
  let isSpinning = false;
  let currentResult = null;
  let currentBg = 'linear-gradient(135deg, #0f172a 0%, #1e293b 100%)';
  let audioCtx = null;

  // Web Audio API によるWeb標準効果音の再生（外部ファイル不要）
  function playSound(freq, duration = 0.15, type = 'sine') {
    try {
      if (!audioCtx) {
        const AudioContext = window.AudioContext || window.webkitAudioContext;
        audioCtx = new AudioContext();
      }
      if (audioCtx.state === 'suspended') {
        audioCtx.resume();
      }
      const osc = audioCtx.createOscillator();
      const gain = audioCtx.createGain();
      
      osc.type = type;
      osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
      
      gain.gain.setValueAtTime(0.1, audioCtx.currentTime);
      gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + duration);

      osc.connect(gain);
      gain.connect(audioCtx.destination);

      osc.start();
      osc.stop(audioCtx.currentTime + duration);
    } catch (e) {
      console.error(e);
    }
  }

  // スロットを回す処理
  function startSlot() {
    if (isSpinning) return;
    
    isSpinning = true;
    currentResult = null;

    let counter = 0;
    const interval = setInterval(() => {
      // ドラム回転中のランダム表示
      slots = slots.map(() => results[Math.floor(Math.random() * results.length)].title.split(' ')[0]);
      playSound(300 + (counter % 5) * 50, 0.05, 'square');
      counter++;

      if (counter > 25) { // 約2.5秒後に停止
        clearInterval(interval);
        
        // 最終結果の決定
        const finalIndex = Math.floor(Math.random() * results.length);
        const selected = results[finalIndex];
        
        const shortTitle = selected.title.split(' ')[0];
        slots = [shortTitle, shortTitle, shortTitle];
        
        currentResult = selected;
        currentBg = selected.bgColor;
        isSpinning = false;

        // ファンファーレ風の効果音再生
        setTimeout(() => playSound(selected.soundFreq, 0.2, 'triangle'), 0);
        setTimeout(() => playSound(selected.soundFreq * 1.25, 0.2, 'triangle'), 150);
        setTimeout(() => playSound(selected.soundFreq * 1.5, 0.4, 'triangle'), 300);
      }
    }, 100);
  }
</script>

<main style="background: {currentBg}">
  <div class="card">
    <h1 class="title">🎰 スロットおみくじ 🎰</h1>

    <!-- スロットディスプレイ -->
    <div class="slot-container">
      {#each slots as slot}
        <div class="slot-box" class:spinning={isSpinning}>
          <span>{slot}</span>
        </div>
      {/each}
    </div>

    <!-- スタートボタン -->
    <button class="spin-btn" on:click={startSlot} disabled={isSpinning}>
      {isSpinning ? 'シャッフル中...' : 'おみくじを引く！'}
    </button>

    <!-- 結果表示エリア -->
    {#if currentResult}
      <div class="result-box" style="border-color: {currentResult.accentColor}">
        <h2 style="color: {currentResult.accentColor}">{currentResult.title}</h2>
        <p>{currentResult.desc}</p>
      </div>
    {/if}
  </div>
</main>

<style>
  :global(body) {
    margin: 0;
    font-family: 'Helvetica Neue', Arial, sans-serif;
    color: #f8fafc;
  }

  main {
    min-height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 20px;
    transition: background 0.8s ease;
    box-sizing: border-box;
  }

  .card {
    background: rgba(15, 23, 42, 0.75);
    backdrop-filter: blur(12px);
    border: 1px solid rgba(255, 255, 255, 0.1);
    border-radius: 24px;
    padding: 32px 24px;
    width: 100%;
    max-width: 440px;
    text-align: center;
    box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.5);
  }

  .title {
    font-size: 1.5rem;
    margin-bottom: 24px;
    letter-spacing: 0.05em;
  }

  .slot-container {
    display: flex;
    justify-content: center;
    gap: 12px;
    margin-bottom: 28px;
  }

  .slot-box {
    width: 90px;
    height: 90px;
    background: rgba(30, 41, 59, 0.9);
    border: 2px solid rgba(255, 255, 255, 0.15);
    border-radius: 16px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 1.25rem;
    font-weight: bold;
    box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.3);
  }

  .slot-box.spinning {
    animation: pulse 0.2s infinite alternate;
  }

  @keyframes pulse {
    from { transform: translateY(-2px); }
    to { transform: translateY(2px); }
  }

  .spin-btn {
    width: 100%;
    padding: 16px;
    font-size: 1.1rem;
    font-weight: bold;
    color: #ffffff;
    background: linear-gradient(135deg, #6366f1 0%, #a855f7 100%);
    border: none;
    border-radius: 12px;
    cursor: pointer;
    box-shadow: 0 4px 14px rgba(99, 102, 241, 0.4);
    transition: all 0.2s ease;
  }

  .spin-btn:hover:not(:disabled) {
    transform: translateY(-2px);
    box-shadow: 0 6px 20px rgba(99, 102, 241, 0.6);
  }

  .spin-btn:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }

  .result-box {
    margin-top: 24px;
    padding: 20px;
    background: rgba(30, 41, 59, 0.8);
    border-radius: 16px;
    border: 2px solid;
    animation: fadeIn 0.5s ease-out;
  }

  .result-box h2 {
    margin: 0 0 8px 0;
    font-size: 1.5rem;
  }

  .result-box p {
    margin: 0;
    font-size: 0.95rem;
    color: #cbd5e1;
    line-height: 1.5;
  }

  @keyframes fadeIn {
    from { opacity: 0; transform: translateY(10px); }
    to { opacity: 1; transform: translateY(0); }
  }
</style>

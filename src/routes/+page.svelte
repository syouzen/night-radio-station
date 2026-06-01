<script lang="ts">
  import { browser } from "$app/environment";
  import { onMount } from "svelte";

  type StoryPack = {
    id: string;
    title: string;
    description: string;
    tone: string;
    unlockHint: string;
  };

  type Letter = {
    id: string;
    packId: StoryPack["id"];
    order: number;
    author: string;
    subject: string;
    body: string;
    mood: string;
  };

  const storyPacks: StoryPack[] = [
    {
      id: "first-night",
      title: "첫 번째 밤",
      description: "잠들지 못한 도시가 조심스럽게 주파수에 기대는 시작 사연입니다.",
      tone: "힐링",
      unlockHint: "처음부터 열림"
    }
  ];

  const incomingLetters: Letter[] = [
    {
      id: "first-night-taxi-minu",
      packId: "first-night",
      order: 1,
      author: "택시 기사 민우",
      subject: "오늘도 03:12에 같은 손님을 태웠습니다",
      body: "목적지는 늘 비어 있는 극장입니다. 요금은 낡은 동전 세 개. 내일도 같은 시간에 틀어주세요.",
      mood: "불안"
    },
    {
      id: "first-night-rooftop-haerin",
      packId: "first-night",
      order: 2,
      author: "옥상 정원사 해린",
      subject: "전파가 식물 잎을 흔들어요",
      body: "당신 방송이 나오면 죽은 줄 알았던 달맞이꽃이 다시 폅니다. 주파수를 바꾸지 말아 주세요.",
      mood: "따뜻함"
    },
    {
      id: "first-night-hidden-city",
      packId: "first-night",
      order: 3,
      author: "익명 청취자",
      subject: "97.3 아래에 다른 도시가 있습니다",
      body: "잡음 사이로 들리는 종소리를 따라가면, 지도에 없는 정류장 이름이 반복됩니다. 당신도 들었나요?",
      mood: "미스터리"
    },
    {
      id: "first-night-store-jun",
      packId: "first-night",
      order: 4,
      author: "편의점 야간 알바 준",
      subject: "손님 없는 시간에만 광고가 들립니다",
      body: "분명 방송을 껐는데도 카운터 라디오에서 누군가 잃어버린 물건을 사고 있습니다.",
      mood: "기묘함"
    },
    {
      id: "first-night-bridge-sora",
      packId: "first-night",
      order: 5,
      author: "늦은 귀가의 소라",
      subject: "다리 위 가로등이 하나씩 켜졌어요",
      body: "사연을 보낸 뒤 집까지 가는 길이 덜 무서웠습니다. 이 도시는 아직 깨어 있네요.",
      mood: "안도"
    }
  ];

  let secondsOnline = $state(0);
  let signal = $state(42);
  let listeners = $state(7);
  let reputation = $state(0);
  let stories = $state(0);
  let antennaLevel = $state(1);
  let transmitterLevel = $state(1);
  let letters = $state<Letter[]>([incomingLetters[0]]);
  let selectedLetter = $state<Letter>(incomingLetters[0]);
  let stationLog = $state("첫 사연이 접수되었습니다. 주파수는 아직 좁지만 방송은 살아 있습니다.");

  const saveKey = "night-radio-station-state";

  const currentFrequency = $derived((91.7 + antennaLevel * 1.4).toFixed(1));
  const selectedStoryPack = $derived(storyPacks.find((pack) => pack.id === selectedLetter.packId) ?? storyPacks[0]);
  const nextLetterIn = $derived(18 - (secondsOnline % 18));
  const nextLetterProgress = $derived(Math.round(((18 - nextLetterIn) / 18) * 100));
  const antennaCost = $derived(antennaLevel * 3);
  const transmitterCost = $derived(transmitterLevel * 2);
  const canTuneAntenna = $derived(reputation >= antennaCost);
  const canWarmTransmitter = $derived(stories >= transmitterCost);
  const antennaProgress = $derived(Math.min(100, Math.round((reputation / antennaCost) * 100)));
  const transmitterProgress = $derived(Math.min(100, Math.round((stories / transmitterCost) * 100)));
  const broadcastTime = $derived(`${Math.floor(secondsOnline / 60)}:${String(secondsOnline % 60).padStart(2, "0")}`);

  function normalizeLetter(letter: Partial<Letter>) {
    return incomingLetters.find((incomingLetter) => incomingLetter.id === letter.id || incomingLetter.subject === letter.subject) ?? incomingLetters[0];
  }

  function addLetter() {
    const next = incomingLetters[letters.length % incomingLetters.length];
    letters = [next, ...letters].slice(0, 6);
    selectedLetter = next;
    reputation += 1;
    stories += 1;
    signal = Math.min(100, signal + 4 + antennaLevel);
    listeners += 2 + transmitterLevel;
    stationLog = `${next.author}의 사연이 도착했습니다. 평판과 이야기가 1씩 늘었습니다.`;
  }

  function tuneAntenna() {
    if (!canTuneAntenna) return;
    reputation -= antennaCost;
    antennaLevel += 1;
    signal = Math.min(100, signal + 12);
    stationLog = `안테나 Lv.${antennaLevel} 조율 완료. 더 먼 밤의 주파수를 잡습니다.`;
  }

  function warmTransmitter() {
    if (!canWarmTransmitter) return;
    stories -= transmitterCost;
    transmitterLevel += 1;
    listeners += 8;
    stationLog = `송신기 Lv.${transmitterLevel} 예열 완료. 새 청취자 8명이 주파수에 머뭅니다.`;
  }

  function resetStation() {
    secondsOnline = 0;
    signal = 42;
    listeners = 7;
    reputation = 0;
    stories = 0;
    antennaLevel = 1;
    transmitterLevel = 1;
    letters = [incomingLetters[0]];
    selectedLetter = incomingLetters[0];
    stationLog = "방송국 기록을 지우고 첫 사연부터 다시 송출합니다.";
    if (browser) localStorage.removeItem(saveKey);
  }

  onMount(() => {
    if (browser) {
      const saved = localStorage.getItem(saveKey);
      if (saved) {
        const state = JSON.parse(saved);
        secondsOnline = state.secondsOnline ?? secondsOnline;
        signal = state.signal ?? signal;
        listeners = state.listeners ?? listeners;
        reputation = state.reputation ?? reputation;
        stories = state.stories ?? stories;
        antennaLevel = state.antennaLevel ?? antennaLevel;
        transmitterLevel = state.transmitterLevel ?? transmitterLevel;
        letters = Array.isArray(state.letters) ? state.letters.map(normalizeLetter) : letters;
        selectedLetter = letters[0] ?? incomingLetters[0];
      }
    }

    const interval = window.setInterval(() => {
      secondsOnline += 1;
      signal = Math.max(18, Math.min(100, signal + (Math.random() > 0.45 ? 1 : -1)));
      listeners = Math.max(1, listeners + (Math.random() > 0.55 ? 1 : 0));

      if (secondsOnline > 0 && secondsOnline % 18 === 0) {
        addLetter();
      }

      if (browser) {
        localStorage.setItem(
          saveKey,
          JSON.stringify({
            secondsOnline,
            signal,
            listeners,
            reputation,
            stories,
            antennaLevel,
            transmitterLevel,
            letters
          })
        );
      }
    }, 1000);

    return () => window.clearInterval(interval);
  });
</script>

<main class="station-shell" aria-label="Night Radio Station">
  <section class="pixel-scene" aria-labelledby="station-title">
    <div class="scene-sky" aria-hidden="true">
      <span></span><span></span><span></span><span></span>
    </div>

    <div class="studio-room">
      <div class="wall-light" aria-hidden="true"></div>
      <div class="window" aria-hidden="true">
        <span></span><span></span><span></span>
      </div>
      <div class="poster" aria-hidden="true">FM</div>
      <div class="shelf" aria-hidden="true">
        <span></span><span></span><span></span>
      </div>
      <div class="host" role="img" aria-label="심야 DJ 캐릭터">
        <div class="host-head"></div>
        <div class="host-body"></div>
      </div>
      <button type="button" class="radio-object" onclick={warmTransmitter} disabled={!canWarmTransmitter} aria-label={`송신기 예열 Lv.${transmitterLevel}`}>
        <span class="antenna"></span>
        <span class="radio-face"></span>
      </button>
      <button type="button" class="letter-box" onclick={tuneAntenna} disabled={!canTuneAntenna} aria-label={`안테나 조율 Lv.${antennaLevel}`}>
        <span></span>
      </button>
      <div class="desk" aria-hidden="true"></div>
    </div>
  </section>

  <section class="status-panel" aria-label="방송 상태">
    <div class="title-row">
      <div>
        <p class="eyebrow">pixel midnight broadcast</p>
        <h1 id="station-title">Night Radio Station</h1>
      </div>
      <div class="on-air"><span aria-hidden="true"></span>ON AIR</div>
    </div>

    <dl class="metrics-grid">
      <div>
        <dt>주파수</dt>
        <dd>FM {currentFrequency}</dd>
      </div>
      <div>
        <dt>신호</dt>
        <dd>{signal}%</dd>
      </div>
      <div>
        <dt>청취자</dt>
        <dd>{listeners}</dd>
      </div>
      <div>
        <dt>송출</dt>
        <dd>{broadcastTime}</dd>
      </div>
    </dl>

    <dl class="resource-strip" aria-label="성장 자원" aria-live="polite">
      <div>
        <dt>평판</dt>
        <dd>{reputation}</dd>
      </div>
      <div>
        <dt>이야기</dt>
        <dd>{stories}</dd>
      </div>
    </dl>

    <p class="station-log" aria-live="polite">{stationLog}</p>

    <p class="action-hint">편지함은 신호를 넓히고, 라디오는 더 많은 청취자를 부릅니다.</p>

    <div class="actions" aria-label="방송국 성장 행동">
      <button type="button" disabled={!canTuneAntenna} onclick={tuneAntenna}>
        편지함 확인 Lv.{antennaLevel}
        <span>평판 {reputation}/{antennaCost}</span>
        <span class="progress-track" aria-hidden="true"><span style={`width: ${antennaProgress}%`}></span></span>
      </button>
      <button type="button" disabled={!canWarmTransmitter} onclick={warmTransmitter}>
        송신기 예열 Lv.{transmitterLevel}
        <span>이야기 {stories}/{transmitterCost}</span>
        <span class="progress-track" aria-hidden="true"><span style={`width: ${transmitterProgress}%`}></span></span>
      </button>
    </div>

    <button type="button" class="reset-button" onclick={resetStation}>처음 방송부터 다시 시작</button>
  </section>

  <section class="letter-panel" aria-label="도착한 사연">
    <div class="letter-header">
      <h2>도착한 사연</h2>
      <p>다음 사연까지 {nextLetterIn}초</p>
    </div>

    <div class="letter-timer" role="timer" aria-label={`다음 사연까지 ${nextLetterIn}초`}>
      <span class="progress-track" aria-hidden="true"><span style={`width: ${nextLetterProgress}%`}></span></span>
    </div>

    <div class="letter-list" role="list">
      {#each letters as letter (letter.id)}
        <button type="button" class:active={selectedLetter.id === letter.id} onclick={() => (selectedLetter = letter)}>
          <span>{letter.author}</span>
          {letter.subject}
        </button>
      {/each}
    </div>

    <article class="letter-card" aria-live="polite">
      <p>{selectedStoryPack.title} · {selectedLetter.mood}</p>
      <h2>{selectedLetter.subject}</h2>
      <span>{selectedLetter.body}</span>
    </article>
  </section>
</main>

<style>
  :global(*) {
    box-sizing: border-box;
  }

  :global(body) {
    margin: 0;
    min-width: 320px;
    color: #f7e9c7;
    background: #15111d;
    font-family: "Courier New", ui-monospace, monospace;
    image-rendering: pixelated;
  }

  button {
    font: inherit;
  }

  .station-shell {
    width: min(430px, 100%);
    min-height: 100vh;
    margin: 0 auto;
    padding: 0.75rem;
    background: linear-gradient(#20172a, #15111d 52%, #0d0b12);
  }

  .pixel-scene,
  .status-panel,
  .letter-panel {
    border: 4px solid #4b3149;
    box-shadow: 0 0 0 4px #120d18;
    background: #20172a;
  }

  .pixel-scene {
    position: relative;
    height: 260px;
    overflow: hidden;
  }

  .scene-sky {
    position: absolute;
    inset: 0;
    background: linear-gradient(#0c1228 0 45%, #2d1d34 45% 100%);
  }

  .scene-sky span {
    position: absolute;
    width: 4px;
    height: 4px;
    background: #f9df8f;
  }

  .scene-sky span:nth-child(1) { top: 22px; left: 42px; }
  .scene-sky span:nth-child(2) { top: 46px; right: 60px; }
  .scene-sky span:nth-child(3) { top: 72px; left: 180px; }
  .scene-sky span:nth-child(4) { top: 36px; right: 152px; }

  .studio-room {
    position: absolute;
    right: 18px;
    bottom: 18px;
    left: 18px;
    height: 190px;
    border: 4px solid #6b3f55;
    background: linear-gradient(#3a263f 0 66%, #2a1c2f 66% 100%);
  }

  .wall-light {
    position: absolute;
    top: 16px;
    left: 18px;
    width: 44px;
    height: 22px;
    background: #f1a45f;
    box-shadow: 0 0 0 4px #4b3149, 0 0 32px #f1a45f;
  }

  .window {
    position: absolute;
    top: 18px;
    right: 22px;
    width: 78px;
    height: 54px;
    border: 4px solid #916070;
    background: #111a35;
  }

  .window span {
    position: absolute;
    bottom: 10px;
    width: 10px;
    background: #27365b;
  }

  .window span:nth-child(1) { left: 12px; height: 18px; }
  .window span:nth-child(2) { left: 32px; height: 28px; }
  .window span:nth-child(3) { left: 52px; height: 12px; }

  .poster {
    position: absolute;
    top: 62px;
    left: 28px;
    width: 38px;
    height: 46px;
    padding-top: 12px;
    border: 4px solid #6b3f55;
    color: #20172a;
    background: #f9df8f;
    text-align: center;
    font-weight: 700;
  }

  .shelf {
    position: absolute;
    top: 84px;
    right: 26px;
    width: 92px;
    height: 10px;
    background: #7a4b4f;
  }

  .shelf span {
    display: inline-block;
    width: 12px;
    height: 22px;
    margin-left: 8px;
    transform: translateY(-20px);
    background: #b8675c;
  }

  .host {
    position: absolute;
    bottom: 50px;
    left: 120px;
    width: 54px;
    height: 82px;
  }

  .host-head {
    width: 42px;
    height: 36px;
    margin: 0 auto;
    border: 4px solid #442638;
    background: #d58b6a;
  }

  .host-head::before,
  .host-head::after {
    position: absolute;
    top: 14px;
    width: 6px;
    height: 6px;
    content: "";
    background: #20172a;
  }

  .host-head::before { left: 18px; }
  .host-head::after { right: 18px; }

  .host-body {
    width: 54px;
    height: 42px;
    border: 4px solid #442638;
    background: #6b5bb9;
  }

  .radio-object,
  .letter-box {
    position: absolute;
    border: 4px solid #442638;
    cursor: pointer;
  }

  .radio-object {
    right: 104px;
    bottom: 52px;
    width: 68px;
    height: 54px;
    background: #b8675c;
  }

  .antenna {
    position: absolute;
    top: -34px;
    left: 28px;
    width: 4px;
    height: 34px;
    background: #d6c08a;
  }

  .radio-face {
    position: absolute;
    inset: 12px;
    border: 4px solid #442638;
    background: #f9df8f;
  }

  .letter-box {
    right: 26px;
    bottom: 56px;
    width: 48px;
    height: 42px;
    background: #4f8f80;
  }

  .letter-box span {
    display: block;
    width: 24px;
    height: 12px;
    margin: 10px auto;
    background: #f7e9c7;
  }

  .radio-object:disabled,
  .letter-box:disabled {
    cursor: not-allowed;
    filter: grayscale(0.65);
    opacity: 0.55;
  }

  .desk {
    position: absolute;
    right: 46px;
    bottom: 26px;
    left: 88px;
    height: 24px;
    border: 4px solid #442638;
    background: #7a4b4f;
  }

  .status-panel,
  .letter-panel {
    margin-top: 0.75rem;
    padding: 0.75rem;
  }

  .title-row,
  .letter-header {
    display: flex;
    align-items: flex-start;
    justify-content: space-between;
    gap: 0.75rem;
  }

  .eyebrow,
  .letter-header p,
  dt,
  .actions span,
  .action-hint,
  .letter-list span,
  .letter-card p {
    color: #c7a77b;
    font-size: 0.72rem;
  }

  .station-log {
    margin: 0 0 0.45rem;
    border: 3px solid #4b3149;
    background: #15111d;
    color: #ffcf91;
    padding: 0.45rem;
    font-size: 0.78rem;
    line-height: 1.45;
  }

  .eyebrow,
  h1,
  h2,
  p {
    margin-top: 0;
  }

  h1 {
    margin-bottom: 0;
    font-size: 1.2rem;
    line-height: 1;
  }

  h2 {
    margin-bottom: 0.5rem;
    font-size: 0.95rem;
  }

  .on-air {
    display: inline-flex;
    align-items: center;
    gap: 0.35rem;
    padding: 0.35rem 0.45rem;
    border: 3px solid #7f383e;
    color: #ffcf91;
    background: #3a1724;
    font-size: 0.8rem;
  }

  .on-air span {
    width: 8px;
    height: 8px;
    background: #ff6b4a;
  }

  .metrics-grid,
  .resource-strip {
    display: grid;
    gap: 0.4rem;
  }

  .metrics-grid {
    grid-template-columns: repeat(4, minmax(0, 1fr));
    margin: 0.75rem 0 0.45rem;
  }

  .resource-strip {
    grid-template-columns: repeat(2, minmax(0, 1fr));
    margin: 0 0 0.55rem;
  }

  .metrics-grid div,
  .resource-strip div,
  .letter-card {
    border: 3px solid #4b3149;
    background: #15111d;
    padding: 0.45rem;
  }

  dd {
    margin: 0.15rem 0 0;
    font-size: 0.9rem;
    font-weight: 700;
  }

  .action-hint {
    margin: 0 0 0.45rem;
    line-height: 1.45;
  }

  .actions,
  .letter-list {
    display: grid;
    gap: 0.45rem;
  }

  .actions button,
  .letter-list button,
  .reset-button {
    border: 3px solid #6b3f55;
    color: #f7e9c7;
    background: #2a1c2f;
    cursor: pointer;
    text-align: left;
  }

  .actions button,
  .reset-button {
    padding: 0.5rem;
  }

  .reset-button {
    width: 100%;
    margin-top: 0.45rem;
    color: #c7a77b;
  }

  .letter-list button {
    padding: 0.45rem;
  }

  .letter-timer {
    margin-bottom: 0.45rem;
  }

  .actions button:hover:not(:disabled),
  .letter-list button:hover,
  .letter-list button.active,
  .reset-button:hover {
    border-color: #f1a45f;
    background: #3a263f;
  }

  .actions button:disabled {
    cursor: not-allowed;
    opacity: 0.5;
  }

  .actions span,
  .letter-list span {
    display: block;
    margin-top: 0.2rem;
  }

  .progress-track {
    height: 8px;
    border: 2px solid #4b3149;
    background: #15111d;
  }

  .progress-track span {
    height: 100%;
    margin: 0;
    background: #f1a45f;
  }

  .letter-card {
    margin-top: 0.5rem;
  }

  .letter-card h2 {
    margin-bottom: 0.5rem;
    line-height: 1.35;
  }

  .letter-card span {
    display: block;
    color: #ead7ad;
    line-height: 1.6;
  }

  @media (max-width: 380px) {
    .station-shell {
      padding: 0.5rem;
    }

    .metrics-grid {
      grid-template-columns: repeat(2, minmax(0, 1fr));
    }
  }
</style>

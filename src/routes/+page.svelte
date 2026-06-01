<script lang="ts">
  import { browser } from "$app/environment";
  import { onMount } from "svelte";

  type Letter = {
    author: string;
    subject: string;
    body: string;
    mood: string;
  };

  const incomingLetters: Letter[] = [
    {
      author: "택시 기사 민우",
      subject: "오늘도 03:12에 같은 손님을 태웠습니다",
      body: "목적지는 늘 비어 있는 극장입니다. 요금은 낡은 동전 세 개. 내일도 같은 시간에 틀어주세요.",
      mood: "불안"
    },
    {
      author: "옥상 정원사 해린",
      subject: "전파가 식물 잎을 흔들어요",
      body: "당신 방송이 나오면 죽은 줄 알았던 달맞이꽃이 다시 폅니다. 주파수를 바꾸지 말아 주세요.",
      mood: "따뜻함"
    },
    {
      author: "익명 청취자",
      subject: "97.3 아래에 다른 도시가 있습니다",
      body: "잡음 사이로 들리는 종소리를 따라가면, 지도에 없는 정류장 이름이 반복됩니다. 당신도 들었나요?",
      mood: "미스터리"
    },
    {
      author: "편의점 야간 알바 준",
      subject: "손님 없는 시간에만 광고가 들립니다",
      body: "분명 방송을 껐는데도 카운터 라디오에서 누군가 잃어버린 물건을 사고 있습니다.",
      mood: "기묘함"
    },
    {
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

  const saveKey = "night-radio-station-state";

  const currentFrequency = $derived((91.7 + antennaLevel * 1.4).toFixed(1));
  const nextLetterIn = $derived(18 - (secondsOnline % 18));
  const canTuneAntenna = $derived(reputation >= antennaLevel * 3);
  const canWarmTransmitter = $derived(stories >= transmitterLevel * 2);

  function addLetter() {
    const next = incomingLetters[letters.length % incomingLetters.length];
    letters = [next, ...letters].slice(0, 6);
    selectedLetter = next;
    reputation += 1;
    stories += 1;
    signal = Math.min(100, signal + 4 + antennaLevel);
    listeners += 2 + transmitterLevel;
  }

  function tuneAntenna() {
    if (!canTuneAntenna) return;
    reputation -= antennaLevel * 3;
    antennaLevel += 1;
    signal = Math.min(100, signal + 12);
  }

  function warmTransmitter() {
    if (!canWarmTransmitter) return;
    stories -= transmitterLevel * 2;
    transmitterLevel += 1;
    listeners += 8;
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
        letters = state.letters ?? letters;
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

<main class="station-shell" aria-label="Night Radio Station control room">
  <section class="hero-panel" aria-labelledby="station-title">
    <div>
      <p class="eyebrow">midnight public broadcast</p>
      <h1 id="station-title">Night Radio Station</h1>
      <p class="station-copy">
        03:00 이후에도 잠들지 못한 도시를 위해 송출 중입니다. 주파수를 넓히고, 사연을 받고,
        잡음 아래 숨은 이야기를 찾으세요.
      </p>
    </div>
    <div class="on-air" aria-label="Broadcast status">
      <span class="pulse" aria-hidden="true"></span>
      ON AIR
    </div>
  </section>

  <section class="dashboard" aria-label="Station dashboard">
    <article class="radio-card primary-card">
      <div class="frequency-row">
        <span>FM {currentFrequency}</span>
        <strong>{signal}%</strong>
      </div>
      <div class="dial" aria-hidden="true">
        <div class="dial-needle" style={`transform: rotate(${signal * 1.8 - 90}deg)`}></div>
      </div>
      <p class="hint">다음 사연까지 {nextLetterIn}초. 신호가 좋아질수록 더 먼 동네가 응답합니다.</p>
    </article>

    <article class="radio-card metrics-card">
      <h2>방송 지표</h2>
      <dl class="metrics-grid">
        <div>
          <dt>청취자</dt>
          <dd>{listeners}</dd>
        </div>
        <div>
          <dt>평판</dt>
          <dd>{reputation}</dd>
        </div>
        <div>
          <dt>이야기 조각</dt>
          <dd>{stories}</dd>
        </div>
        <div>
          <dt>송출 시간</dt>
          <dd>{Math.floor(secondsOnline / 60)}:{String(secondsOnline % 60).padStart(2, "0")}</dd>
        </div>
      </dl>
    </article>

    <article class="radio-card upgrades-card">
      <h2>장비 정비</h2>
      <button type="button" disabled={!canTuneAntenna} onclick={tuneAntenna}>
        안테나 조율 Lv.{antennaLevel}
        <span>평판 {antennaLevel * 3} 필요</span>
      </button>
      <button type="button" disabled={!canWarmTransmitter} onclick={warmTransmitter}>
        송신기 예열 Lv.{transmitterLevel}
        <span>이야기 {transmitterLevel * 2} 필요</span>
      </button>
    </article>
  </section>

  <section class="story-grid" aria-label="Incoming listener letters">
    <article class="radio-card inbox-card">
      <h2>도착한 사연</h2>
      <div class="letter-list" role="list">
        {#each letters as letter}
          <button
            type="button"
            class:active={selectedLetter.subject === letter.subject}
            onclick={() => (selectedLetter = letter)}
          >
            <span>{letter.author}</span>
            {letter.subject}
          </button>
        {/each}
      </div>
    </article>

    <article class="radio-card letter-card" aria-live="polite">
      <p class="mood">{selectedLetter.mood}</p>
      <h2>{selectedLetter.subject}</h2>
      <p>{selectedLetter.body}</p>
      <footer>from {selectedLetter.author}</footer>
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
    color: #f3ead7;
    background:
      radial-gradient(circle at top left, rgba(196, 112, 63, 0.18), transparent 32rem),
      linear-gradient(145deg, #10131c 0%, #171019 48%, #090a0f 100%);
    font-family:
      Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  }

  button {
    font: inherit;
  }

  .station-shell {
    width: min(1120px, 100%);
    margin: 0 auto;
    padding: 2rem;
  }

  .hero-panel,
  .radio-card {
    border: 1px solid rgba(230, 196, 132, 0.18);
    background: rgba(15, 18, 28, 0.76);
    backdrop-filter: blur(18px);
  }

  .hero-panel {
    display: flex;
    align-items: flex-start;
    justify-content: space-between;
    gap: 1.5rem;
    padding: 1.5rem;
    border-radius: 1.25rem;
  }

  .eyebrow,
  .mood,
  .hint,
  footer,
  dt {
    color: #b9aa8f;
  }

  .eyebrow,
  .mood {
    margin: 0 0 0.5rem;
    font-size: 0.78rem;
    letter-spacing: 0.16em;
    text-transform: uppercase;
  }

  h1,
  h2,
  p {
    margin-top: 0;
  }

  h1 {
    margin-bottom: 0.75rem;
    font-size: clamp(2rem, 7vw, 4.75rem);
    line-height: 0.95;
  }

  h2 {
    margin-bottom: 1rem;
    font-size: 1rem;
  }

  .station-copy {
    max-width: 46rem;
    margin-bottom: 0;
    color: #d9c9ab;
  }

  .on-air {
    display: inline-flex;
    align-items: center;
    gap: 0.5rem;
    min-width: max-content;
    padding: 0.55rem 0.75rem;
    border: 1px solid rgba(255, 125, 85, 0.4);
    border-radius: 999px;
    color: #ffd3bc;
    background: rgba(118, 39, 32, 0.38);
    font-weight: 700;
    letter-spacing: 0.08em;
  }

  .pulse {
    width: 0.55rem;
    height: 0.55rem;
    border-radius: 999px;
    background: #ff7d55;
    box-shadow: 0 0 1rem #ff7d55;
  }

  .dashboard,
  .story-grid {
    display: grid;
    grid-template-columns: 1.2fr 1fr 1fr;
    gap: 1rem;
    margin-top: 1rem;
  }

  .story-grid {
    grid-template-columns: 0.9fr 1.6fr;
  }

  .radio-card {
    min-width: 0;
    padding: 1.25rem;
    border-radius: 1rem;
  }

  .frequency-row {
    display: flex;
    align-items: baseline;
    justify-content: space-between;
    gap: 1rem;
    font-size: clamp(1.75rem, 5vw, 3.5rem);
    font-weight: 800;
    letter-spacing: -0.05em;
  }

  .frequency-row strong {
    color: #f6b982;
    font-size: 1.25rem;
    letter-spacing: 0;
  }

  .dial {
    position: relative;
    height: 5rem;
    margin: 1.25rem 0;
    overflow: hidden;
    border-bottom: 1px solid rgba(230, 196, 132, 0.18);
  }

  .dial::before {
    position: absolute;
    right: 8%;
    bottom: -7.5rem;
    left: 8%;
    height: 15rem;
    content: "";
    border: 1px solid rgba(246, 185, 130, 0.3);
    border-radius: 50%;
  }

  .dial-needle {
    position: absolute;
    bottom: 0;
    left: 50%;
    width: 2px;
    height: 4.5rem;
    background: #ff7d55;
    transform-origin: bottom;
    transition: transform 0.5s ease;
  }

  .metrics-grid {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 1rem;
    margin: 0;
  }

  dt {
    font-size: 0.78rem;
  }

  dd {
    margin: 0.2rem 0 0;
    font-size: 1.5rem;
    font-weight: 750;
  }

  .upgrades-card {
    display: grid;
    align-content: start;
    gap: 0.75rem;
  }

  .upgrades-card button,
  .letter-list button {
    width: 100%;
    border: 1px solid rgba(230, 196, 132, 0.2);
    border-radius: 0.8rem;
    color: #f8ead0;
    background: rgba(246, 185, 130, 0.08);
    cursor: pointer;
  }

  .upgrades-card button {
    display: grid;
    gap: 0.25rem;
    padding: 0.8rem;
    text-align: left;
  }

  .upgrades-card button:hover:not(:disabled),
  .letter-list button:hover,
  .letter-list button.active {
    border-color: rgba(246, 185, 130, 0.55);
    background: rgba(246, 185, 130, 0.16);
  }

  .upgrades-card button:disabled {
    cursor: not-allowed;
    opacity: 0.45;
  }

  .upgrades-card span,
  .letter-list span {
    display: block;
    color: #b9aa8f;
    font-size: 0.78rem;
  }

  .letter-list {
    display: grid;
    gap: 0.6rem;
  }

  .letter-list button {
    padding: 0.75rem;
    text-align: left;
  }

  .letter-card {
    min-height: 15rem;
  }

  .letter-card h2 {
    font-size: clamp(1.3rem, 4vw, 2rem);
  }

  .letter-card p:not(.mood) {
    color: #e8d9bf;
    font-size: 1.05rem;
    line-height: 1.7;
  }

  footer {
    margin-top: 2rem;
  }

  @media (max-width: 820px) {
    .station-shell {
      padding: 1rem;
    }

    .hero-panel,
    .dashboard,
    .story-grid {
      grid-template-columns: 1fr;
    }

    .hero-panel {
      display: grid;
    }
  }
</style>

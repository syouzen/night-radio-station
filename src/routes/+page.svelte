<script lang="ts">
  import { browser } from "$app/environment";
  import { onMount } from "svelte";

  type StoryPack = {
    id: string;
    title: string;
    description: string;
    tone: string;
    unlockHint: string;
    unlock: {
      listeners: number;
      signal: number;
      stories: number;
    };
  };

  type Letter = {
    id: string;
    packId: StoryPack["id"];
    order: number;
    author: string;
    subject: string;
    body: string;
    mood: string;
    characterId: string;
    sequence: string;
    djComment: string;
  };

  const storyPacks: StoryPack[] = [
    {
      id: "first-night",
      title: "첫 번째 밤",
      description: "잠들지 못한 도시가 조심스럽게 주파수에 기대는 시작 사연입니다.",
      tone: "힐링",
      unlockHint: "처음부터 열림",
      unlock: { listeners: 0, signal: 0, stories: 0 }
    },
    {
      id: "rooftop-garden",
      title: "옥상 정원",
      description: "낡은 건물 옥상에서 식물과 밤공기를 돌보는 청취자들의 사연 묶음입니다.",
      tone: "힐링",
      unlockHint: "청취자 18명, 신호 55%, 이야기 4개 필요",
      unlock: { listeners: 18, signal: 55, stories: 4 }
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
      mood: "불안",
      characterId: "taxi-minu",
      sequence: "첫 운행",
      djComment: "민우 님, 같은 시간에 같은 길을 도는 밤도 언젠가 목적지를 바꿉니다. 오늘은 극장 앞까지 같이 켜둘게요."
    },
    {
      id: "first-night-rooftop-haerin",
      packId: "first-night",
      order: 2,
      author: "옥상 정원사 해린",
      subject: "전파가 식물 잎을 흔들어요",
      body: "당신 방송이 나오면 죽은 줄 알았던 달맞이꽃이 다시 폅니다. 주파수를 바꾸지 말아 주세요.",
      mood: "따뜻함",
      characterId: "gardener-haerin",
      sequence: "첫 발아",
      djComment: "해린 님, 오늘 주파수는 그대로 둘게요. 달맞이꽃이 듣는 밤이라면 우리도 조용히 볼륨을 낮추겠습니다."
    },
    {
      id: "first-night-hidden-city",
      packId: "first-night",
      order: 3,
      author: "익명 청취자",
      subject: "97.3 아래에 다른 도시가 있습니다",
      body: "잡음 사이로 들리는 종소리를 따라가면, 지도에 없는 정류장 이름이 반복됩니다. 당신도 들었나요?",
      mood: "미스터리",
      characterId: "hidden-city-listener",
      sequence: "첫 제보",
      djComment: "익명 청취자님, 종소리는 아직 희미합니다. 대신 잡음을 조금 더 넓혀서 그 정류장 이름을 받아보겠습니다."
    },
    {
      id: "first-night-store-jun",
      packId: "first-night",
      order: 4,
      author: "편의점 야간 알바 준",
      subject: "손님 없는 시간에만 광고가 들립니다",
      body: "분명 방송을 껐는데도 카운터 라디오에서 누군가 잃어버린 물건을 사고 있습니다.",
      mood: "기묘함",
      characterId: "night-store-jun",
      sequence: "첫 야간 근무",
      djComment: "준 님, 꺼진 라디오까지 듣는 손님이라면 분명 잃어버린 물건보다 잃어버린 시간을 찾는 중일 겁니다."
    },
    {
      id: "first-night-bridge-sora",
      packId: "first-night",
      order: 5,
      author: "늦은 귀가의 소라",
      subject: "다리 위 가로등이 하나씩 켜졌어요",
      body: "사연을 보낸 뒤 집까지 가는 길이 덜 무서웠습니다. 이 도시는 아직 깨어 있네요.",
      mood: "안도",
      characterId: "bridge-sora",
      sequence: "첫 귀가",
      djComment: "소라 님, 집까지 닿는 가로등 하나를 더 켜둔 셈으로 생각할게요. 다음 다리도 같이 건너요."
    },
    {
      id: "rooftop-garden-dalsoo",
      packId: "rooftop-garden",
      order: 1,
      author: "옥상 관리인 달수",
      subject: "물탱크 옆 작은 화분을 맡았습니다",
      body: "퇴근길마다 한 컵씩 물을 줍니다. 방송에서 비 소리가 나오면 잎이 조금 더 곧게 서는 것 같습니다.",
      mood: "잔잔함",
      characterId: "rooftop-dalsoo",
      sequence: "첫 물주기",
      djComment: "달수 님, 오늘 예보에는 비가 없지만 방송국에서 작은 빗소리를 섞어 보내겠습니다."
    },
    {
      id: "rooftop-garden-mira",
      packId: "rooftop-garden",
      order: 2,
      author: "새벽 배송원 미라",
      subject: "옥상 난간에 매달린 리본을 봤어요",
      body: "매일 다른 색으로 바뀌는 리본입니다. 오늘은 노란색이었고, 이상하게 피곤함이 덜했습니다.",
      mood: "위로",
      characterId: "delivery-mira",
      sequence: "첫 리본",
      djComment: "미라 님, 노란 리본은 새벽에도 도착하는 햇빛일지 모릅니다. 다음 배송길에도 주파수를 열어둘게요."
    },
    {
      id: "rooftop-garden-seoho",
      packId: "rooftop-garden",
      order: 3,
      author: "라디오 수리공 서호",
      subject: "낡은 스피커에서 흙냄새가 납니다",
      body: "주파수를 맞추면 잡음 사이로 분갈이하는 소리가 들립니다. 고장이라기보다 누군가 돌보고 있는 소리 같습니다.",
      mood: "기묘한 평온",
      characterId: "repair-seoho",
      sequence: "첫 수리",
      djComment: "서호 님, 고장이 아니라면 다행입니다. 밤에도 무언가 자라고 있다는 증거니까요."
    },
    {
      id: "rooftop-garden-yeon",
      packId: "rooftop-garden",
      order: 4,
      author: "잠 못 드는 연",
      subject: "안테나 그림자가 화단까지 닿았어요",
      body: "그 그림자 아래 앉아 있으면 오늘 못 한 말을 내일 해도 괜찮을 것 같습니다. 고마워요, 계속 틀어줘서.",
      mood: "회복",
      characterId: "sleepless-yeon",
      sequence: "첫 그림자",
      djComment: "연 님, 내일 해도 되는 말은 오늘 밤 우리가 지켜둘게요. 안테나 그림자 아래에서 잠시 쉬어가세요."
    },
    {
      id: "rooftop-garden-haerin-bloom",
      packId: "rooftop-garden",
      order: 5,
      author: "옥상 정원사 해린",
      subject: "달맞이꽃이 두 번째 편지를 피웠어요",
      body: "처음 사연을 보낸 뒤 화분을 옥상으로 옮겼습니다. 오늘은 꽃잎 안쪽에 작은 주파수 번호가 보였습니다.",
      mood: "연결",
      characterId: "gardener-haerin",
      sequence: "두 번째 발아",
      djComment: "해린 님, 첫 밤의 꽃이 옥상까지 올라왔군요. 그 번호를 따라가면 다음 사연도 피어날 겁니다."
    }
  ];

  let secondsOnline = $state(0);
  let signal = $state(42);
  let listeners = $state(7);
  let reputation = $state(0);
  let stories = $state(0);
  let antennaLevel = $state(1);
  let transmitterLevel = $state(1);
  let receivedLetterCount = $state(1);
  let receivedLetterIds = $state<Letter["id"][]>([incomingLetters[0].id]);
  let unlockedStoryPackIds = $state<StoryPack["id"][]>([storyPacks[0].id]);
  let announcedStoryPackIds = $state<StoryPack["id"][]>([storyPacks[0].id]);
  let completedStoryPackIds = $state<StoryPack["id"][]>([]);
  let unlockedPackNotice = $state<StoryPack | null>(null);
  let completedPackNotice = $state<StoryPack | null>(null);
  let hasLoadedState = $state(false);
  let letters = $state<Letter[]>([incomingLetters[0]]);
  let selectedLetter = $state<Letter>(incomingLetters[0]);
  let stationLog = $state("첫 사연이 접수되었습니다. 주파수는 아직 좁지만 방송은 살아 있습니다.");

  const saveKey = "night-radio-station-state";

  const currentFrequency = $derived((91.7 + antennaLevel * 1.4).toFixed(1));
  const unlockedStoryPacks = $derived(storyPacks.filter((pack) => isStoryPackUnlocked(pack)));
  const nextStoryPack = $derived(storyPacks.find((pack) => !isStoryPackUnlocked(pack)));
  const nextStoryPackProgress = $derived(nextStoryPack ? storyPackProgress(nextStoryPack) : 100);
  const availableLetters = $derived(incomingLetters.filter((letter) => unlockedStoryPackIds.includes(letter.packId)));
  const selectedStoryPack = $derived(storyPacks.find((pack) => pack.id === selectedLetter.packId) ?? storyPacks[0]);
  const completedStoryPacks = $derived(storyPacks.filter((pack) => isStoryPackComplete(pack)));
  const selectedCharacterLetters = $derived(incomingLetters.filter((letter) => letter.characterId === selectedLetter.characterId));
  const selectedReceivedCharacterLetters = $derived(selectedCharacterLetters.filter((letter) => receivedLetterIds.includes(letter.id)));
  const selectedPackReceivedCount = $derived(storyPackReceivedCount(selectedStoryPack));
  const selectedPackLetterCount = $derived(storyPackLetters(selectedStoryPack).length);
  const nextLetterIn = $derived(18 - (secondsOnline % 18));
  const nextLetterProgress = $derived(Math.round(((18 - nextLetterIn) / 18) * 100));
  const antennaCost = $derived(antennaLevel * 3);
  const transmitterCost = $derived(transmitterLevel * 2);
  const canTuneAntenna = $derived(reputation >= antennaCost);
  const canWarmTransmitter = $derived(stories >= transmitterCost);
  const antennaProgress = $derived(Math.min(100, Math.round((reputation / antennaCost) * 100)));
  const transmitterProgress = $derived(Math.min(100, Math.round((stories / transmitterCost) * 100)));
  const broadcastTime = $derived(`${Math.floor(secondsOnline / 60)}:${String(secondsOnline % 60).padStart(2, "0")}`);
  const signalMood = $derived(signal >= 72 ? "clear" : signal >= 44 ? "warm" : "thin");
  const listenerLightCount = $derived(Math.min(8, Math.max(2, Math.ceil(listeners / 4))));
  const scenePulse = $derived(Math.min(1, Math.max(0.35, signal / 100 + transmitterLevel * 0.04)));
  const sceneGlow = $derived(`${Math.round(14 + scenePulse * 26)}px`);
  const lightOpacity = $derived((0.38 + scenePulse * 0.48).toFixed(2));
  const antennaReach = $derived(Math.min(62, 28 + antennaLevel * 8));
  const isRooftopGardenUnlocked = $derived(unlockedStoryPackIds.includes("rooftop-garden"));
  const isRooftopGardenComplete = $derived(completedStoryPackIds.includes("rooftop-garden"));
  const sceneStatus = $derived(
    `현재 방송국은 ${signalMood === "clear" ? "선명한" : signalMood === "warm" ? "따뜻한" : "희미한"} 신호로 송출 중입니다. 도시 창문 ${listenerLightCount}개가 켜져 있고 안테나는 Lv.${antennaLevel}, 송신기는 Lv.${transmitterLevel}입니다.${isRooftopGardenUnlocked ? " 창가에는 옥상 정원 화분이 놓여 있습니다." : ""}${isRooftopGardenComplete ? " 화분에는 완결된 사연을 닮은 노란 꽃이 피었습니다." : ""}`
  );

  function savedNumber(value: unknown, fallback: number) {
    return typeof value === "number" && Number.isFinite(value) ? value : fallback;
  }

  $effect(() => {
    if (!hasLoadedState) return;

    const freshPack = storyPacks.find((pack) => !isStoryPackUnlocked(pack) && hasStoryPackRequirements(pack));
    if (!freshPack) return;

    unlockedStoryPackIds = [...unlockedStoryPackIds, freshPack.id];
    if (announcedStoryPackIds.includes(freshPack.id)) return;

    announcedStoryPackIds = [...announcedStoryPackIds, freshPack.id];
    unlockedPackNotice = freshPack;
    stationLog = `${freshPack.title} 사연 묶음이 열렸습니다. 새 밤의 편지가 편성표에 들어왔습니다.`;
    saveStationState();
  });

  $effect(() => {
    if (!hasLoadedState) return;

    const completedPack = storyPacks.find((pack) => isStoryPackComplete(pack) && !completedStoryPackIds.includes(pack.id));
    if (!completedPack) return;

    completedStoryPackIds = [...completedStoryPackIds, completedPack.id];
    completedPackNotice = completedPack;
    stationLog = `${completedPack.title} 사연 묶음이 완성되었습니다. DJ가 마지막 코멘트를 편성표에 남겼습니다.`;
    saveStationState();
  });

  function normalizeLetter(letter: Partial<Letter>) {
    return incomingLetters.find((incomingLetter) => incomingLetter.id === letter.id || incomingLetter.subject === letter.subject) ?? incomingLetters[0];
  }

  function isStoryPackUnlocked(pack: StoryPack) {
    return unlockedStoryPackIds.includes(pack.id);
  }

  function hasStoryPackRequirements(pack: StoryPack) {
    return listeners >= pack.unlock.listeners && signal >= pack.unlock.signal && stories >= pack.unlock.stories;
  }

  function storyPackProgress(pack: StoryPack) {
    const listenerProgress = pack.unlock.listeners === 0 ? 100 : Math.min(100, Math.round((listeners / pack.unlock.listeners) * 100));
    const signalProgress = pack.unlock.signal === 0 ? 100 : Math.min(100, Math.round((signal / pack.unlock.signal) * 100));
    const storyProgress = pack.unlock.stories === 0 ? 100 : Math.min(100, Math.round((stories / pack.unlock.stories) * 100));
    return Math.round((listenerProgress + signalProgress + storyProgress) / 3);
  }

  function orderedLetters(availablePackIds = unlockedStoryPackIds) {
    return incomingLetters
      .filter((letter) => availablePackIds.includes(letter.packId))
      .sort((a, b) => storyPacks.findIndex((pack) => pack.id === a.packId) - storyPacks.findIndex((pack) => pack.id === b.packId) || a.order - b.order);
  }

  function knownLetterIds(ids: Letter["id"][]) {
    return Array.from(new Set(ids.filter((id) => incomingLetters.some((letter) => letter.id === id))));
  }

  function migratedReceivedLetterIds(count: number, visibleLetters: Letter[], availablePackIds = unlockedStoryPackIds) {
    return knownLetterIds([...orderedLetters(availablePackIds).slice(0, count).map((letter) => letter.id), ...visibleLetters.map((letter) => letter.id)]);
  }

  function nextAvailableLetter() {
    return orderedLetters().find((letter) => !receivedLetterIds.includes(letter.id)) ?? availableLetters[receivedLetterCount % availableLetters.length] ?? incomingLetters[0];
  }

  function storyPackLetters(pack: StoryPack) {
    return incomingLetters.filter((letter) => letter.packId === pack.id);
  }

  function storyPackReceivedCount(pack: StoryPack) {
    return storyPackLetters(pack).filter((letter) => receivedLetterIds.includes(letter.id)).length;
  }

  function isStoryPackComplete(pack: StoryPack) {
    const packLetters = storyPackLetters(pack);
    return packLetters.length > 0 && packLetters.every((letter) => receivedLetterIds.includes(letter.id));
  }

  function saveStationState() {
    if (!browser) return;

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
        receivedLetterCount,
        receivedLetterIds,
        unlockedStoryPackIds,
        announcedStoryPackIds,
        completedStoryPackIds,
        letters
      })
    );
  }

  function addLetter() {
    const next = nextAvailableLetter();
    receivedLetterCount += 1;
    if (!receivedLetterIds.includes(next.id)) receivedLetterIds = [...receivedLetterIds, next.id];
    letters = [next, ...letters].slice(0, 6);
    selectedLetter = next;
    reputation += 1;
    stories += 1;
    signal = Math.min(100, signal + 4 + antennaLevel);
    listeners += 2 + transmitterLevel;
    stationLog = `${next.author}의 사연이 도착했습니다. DJ 코멘트: ${next.djComment}`;
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
    receivedLetterCount = 1;
    receivedLetterIds = [incomingLetters[0].id];
    unlockedStoryPackIds = [storyPacks[0].id];
    announcedStoryPackIds = [storyPacks[0].id];
    completedStoryPackIds = [];
    unlockedPackNotice = null;
    completedPackNotice = null;
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
        secondsOnline = savedNumber(state.secondsOnline, secondsOnline);
        signal = savedNumber(state.signal, signal);
        listeners = savedNumber(state.listeners, listeners);
        reputation = savedNumber(state.reputation, reputation);
        stories = savedNumber(state.stories, stories);
        antennaLevel = savedNumber(state.antennaLevel, antennaLevel);
        transmitterLevel = savedNumber(state.transmitterLevel, transmitterLevel);
        letters = Array.isArray(state.letters) ? state.letters.map(normalizeLetter) : letters;
        receivedLetterCount = state.receivedLetterCount ?? Math.max(letters.length, 1);
        unlockedStoryPackIds = Array.isArray(state.unlockedStoryPackIds)
          ? state.unlockedStoryPackIds.filter((id: string) => storyPacks.some((pack) => pack.id === id))
          : storyPacks.filter(hasStoryPackRequirements).map((pack) => pack.id);
        receivedLetterIds = Array.isArray(state.receivedLetterIds)
          ? knownLetterIds(state.receivedLetterIds)
          : migratedReceivedLetterIds(receivedLetterCount, letters, unlockedStoryPackIds);
        if (!receivedLetterIds.includes(incomingLetters[0].id)) receivedLetterIds = [incomingLetters[0].id, ...receivedLetterIds];
        if (!unlockedStoryPackIds.includes(storyPacks[0].id)) unlockedStoryPackIds = [storyPacks[0].id, ...unlockedStoryPackIds];
        announcedStoryPackIds = Array.isArray(state.announcedStoryPackIds)
          ? state.announcedStoryPackIds.filter((id: string) => storyPacks.some((pack) => pack.id === id))
          : unlockedStoryPackIds;
        completedStoryPackIds = Array.isArray(state.completedStoryPackIds)
          ? state.completedStoryPackIds.filter((id: string) => storyPacks.some((pack) => pack.id === id))
          : storyPacks.filter(isStoryPackComplete).map((pack) => pack.id);
        selectedLetter = letters[0] ?? incomingLetters[0];
      }
    }

    hasLoadedState = true;

    const interval = window.setInterval(() => {
      secondsOnline += 1;
      signal = Math.max(18, Math.min(100, signal + (Math.random() > 0.45 ? 1 : -1)));
      listeners = Math.max(1, listeners + (Math.random() > 0.55 ? 1 : 0));

      if (secondsOnline > 0 && secondsOnline % 18 === 0) {
        addLetter();
      }

      saveStationState();
    }, 1000);

    return () => window.clearInterval(interval);
  });
</script>

<main class="station-shell" aria-label="Night Radio Station">
  <section
    class={`pixel-scene signal-${signalMood}${isRooftopGardenUnlocked ? " has-rooftop" : ""}${isRooftopGardenComplete ? " rooftop-complete" : ""}`}
    style={`--signal-pulse: ${scenePulse}; --scene-glow: ${sceneGlow}; --light-opacity: ${lightOpacity}; --antenna-reach: ${antennaReach}px; --listener-lights: ${listenerLightCount};`}
    aria-labelledby="station-title"
    aria-describedby="scene-status"
  >
    <p id="scene-status" class="sr-only">{sceneStatus}</p>
    <div class="scene-sky" aria-hidden="true">
      <span></span><span></span><span></span><span></span>
    </div>
    <div class="moon" aria-hidden="true"></div>
    <div class="roofline" aria-hidden="true"><span></span><span></span><span></span><span></span></div>
    <div class="city-lights" aria-hidden="true">
      {#each Array(listenerLightCount) as _, index}
        <span style={`left: ${8 + index * 46}px; height: ${10 + (index % 3) * 8}px;`}></span>
      {/each}
    </div>
    <div class="signal-rings" aria-hidden="true">
      <span></span><span></span><span></span>
    </div>

    <div class="studio-room">
      <div class="wall-light" aria-hidden="true"></div>
      <div class="studio-grid" aria-hidden="true"></div>
      <div class="window" aria-hidden="true">
        <span></span><span></span><span></span><span></span><span></span><span></span>
      </div>
      {#if isRooftopGardenUnlocked}
        <div class="rooftop-pot" aria-hidden="true">
          <span></span><span></span><span></span>
        </div>
      {/if}
      <div class="poster" aria-hidden="true">FM</div>
      <div class="shelf" aria-hidden="true">
        <span></span><span></span><span></span>
      </div>
      <div class="host" role="img" aria-label="심야 DJ 캐릭터">
        <div class="host-head"></div>
        <div class="host-body"></div>
      </div>
      <div class="mic-stand" aria-hidden="true"><span></span></div>
      <button type="button" class="radio-object" onclick={warmTransmitter} disabled={!canWarmTransmitter} aria-label={`송신기 예열 Lv.${transmitterLevel}`}>
        <span class="antenna"></span>
        <span class="radio-wave"></span>
        <span class="radio-face"></span>
        <span class="radio-knobs"></span>
      </button>
      <button type="button" class="letter-box" onclick={tuneAntenna} disabled={!canTuneAntenna} aria-label={`안테나 조율 Lv.${antennaLevel}`}>
        <span></span>
        <span class="letter-flag"></span>
      </button>
      <div class="desk" aria-hidden="true"><span></span><span></span><span></span></div>
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

    <div class="pack-status" aria-label="사연 묶음 해금 상태">
      <span>{unlockedStoryPacks.length}/{storyPacks.length}개 사연 묶음 열림</span>
      {#if nextStoryPack}
        <span>다음: {nextStoryPack.title} · {nextStoryPack.unlockHint}</span>
        <span
          class="progress-track"
          role="progressbar"
          aria-label={`${nextStoryPack.title} 해금 진행도`}
          aria-valuemin="0"
          aria-valuemax="100"
          aria-valuenow={nextStoryPackProgress}
        ><span style={`width: ${nextStoryPackProgress}%`}></span></span>
      {:else}
        <span>현재 준비된 모든 사연 묶음이 열렸습니다.</span>
      {/if}
    </div>

    {#if unlockedPackNotice}
      <div class="unlock-notice" role="status" aria-live="polite">
        <span>새 사연 묶음 해금</span>
        <strong>{unlockedPackNotice.title}</strong>
        <small>{unlockedPackNotice.description}</small>
      </div>
    {/if}

    {#if completedPackNotice}
      <div class="completion-notice" role="status" aria-live="polite">
        <span>사연 묶음 완성</span>
        <strong>{completedPackNotice.title}</strong>
        <small>이 묶음의 모든 사연이 방송 기록에 남았습니다.</small>
      </div>
    {/if}

    <div class="pack-collection" role="list" aria-label="사연 묶음 보관함">
      {#each storyPacks as pack (pack.id)}
        <div class:unlocked={isStoryPackUnlocked(pack)} class:complete={isStoryPackComplete(pack)} class="pack-card" role="listitem">
          <div>
            <span>{isStoryPackComplete(pack) ? "완성" : isStoryPackUnlocked(pack) ? "열림" : "잠김"}</span>
            <strong>{pack.title}</strong>
          </div>
          <p>{pack.description}</p>
          <small>{isStoryPackUnlocked(pack) ? `${storyPackReceivedCount(pack)}/${storyPackLetters(pack).length}개 사연 수신` : pack.unlockHint}</small>
          <span class="progress-track" aria-hidden="true"><span style={`width: ${isStoryPackUnlocked(pack) ? Math.round((storyPackReceivedCount(pack) / storyPackLetters(pack).length) * 100) : storyPackProgress(pack)}%`}></span></span>
        </div>
      {/each}
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
      <p>{selectedStoryPack.title} · {selectedLetter.mood} · {selectedLetter.sequence}</p>
      <h2>{selectedLetter.subject}</h2>
      <span>{selectedLetter.body}</span>
      <blockquote class="dj-comment">DJ 코멘트: {selectedLetter.djComment}</blockquote>
      <div class="story-thread" aria-label={`${selectedLetter.author} 연결 사연`}>
        <strong>연결된 사연 {selectedReceivedCharacterLetters.length}/{selectedCharacterLetters.length}</strong>
        {#each selectedCharacterLetters as threadLetter (threadLetter.id)}
          <span class:heard={receivedLetterIds.includes(threadLetter.id)}>{receivedLetterIds.includes(threadLetter.id) ? threadLetter.sequence : "아직 도착하지 않은 후속 사연"}</span>
        {/each}
      </div>
      <p class="pack-progress-copy">{selectedStoryPack.title} 기록 {selectedPackReceivedCount}/{selectedPackLetterCount}</p>
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
    background:
      radial-gradient(circle at 50% 0, rgba(75, 49, 73, 0.34), transparent 280px),
      repeating-linear-gradient(0deg, rgba(255, 255, 255, 0.025) 0 1px, transparent 1px 4px),
      #15111d;
    font-family: "Courier New", ui-monospace, monospace;
    image-rendering: pixelated;
  }

  button {
    font: inherit;
  }

  button:focus-visible {
    outline: 3px solid #f9df8f;
    outline-offset: 3px;
  }

  .sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    white-space: nowrap;
  }

  .station-shell {
    width: min(430px, 100%);
    min-height: 100vh;
    margin: 0 auto;
    padding: 0.75rem;
    background:
      linear-gradient(90deg, transparent 0 8px, rgba(249, 223, 143, 0.035) 8px 10px, transparent 10px 100%),
      linear-gradient(#20172a, #15111d 52%, #0d0b12);
  }

  .pixel-scene,
  .status-panel,
  .letter-panel {
    border: 4px solid #4b3149;
    box-shadow: 0 0 0 4px #120d18, inset 0 0 0 3px rgba(249, 223, 143, 0.06);
    background: #20172a;
  }

  .pixel-scene {
    --signal-pulse: 0.5;
    --antenna-reach: 36px;
    --listener-lights: 2;
    position: relative;
    height: 260px;
    overflow: hidden;
  }

  .pixel-scene::before,
  .pixel-scene::after {
    position: absolute;
    inset: 0;
    z-index: 4;
    content: "";
    pointer-events: none;
  }

  .pixel-scene::before {
    border: 3px solid rgba(249, 223, 143, 0.08);
  }

  .pixel-scene::after {
    background: repeating-linear-gradient(0deg, rgba(255, 255, 255, 0.035) 0 1px, transparent 1px 5px);
    opacity: 0.45;
    mix-blend-mode: screen;
  }

  .scene-sky {
    position: absolute;
    inset: 0;
    background: linear-gradient(#0c1228 0 42%, #2d1d34 42% 100%);
    transition: background 0.4s ease;
  }

  .pixel-scene.signal-warm .scene-sky {
    background: linear-gradient(#111a35 0 42%, #3a263f 42% 100%);
  }

  .pixel-scene.signal-clear .scene-sky {
    background: linear-gradient(#17234c 0 42%, #3d2e58 42% 100%);
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

  .moon {
    position: absolute;
    top: 24px;
    right: 108px;
    width: 22px;
    height: 22px;
    background: #f7e9c7;
    box-shadow: -8px 4px 0 #0c1228, 0 0 18px rgba(249, 223, 143, 0.36);
  }

  .roofline {
    position: absolute;
    right: 0;
    bottom: 196px;
    left: 0;
    height: 42px;
    background: linear-gradient(transparent 0 18px, #191425 18px 100%);
  }

  .roofline span {
    position: absolute;
    bottom: 0;
    width: 34px;
    background: #100c18;
  }

  .roofline span:nth-child(1) { left: 18px; height: 18px; }
  .roofline span:nth-child(2) { left: 86px; height: 30px; }
  .roofline span:nth-child(3) { right: 92px; height: 22px; }
  .roofline span:nth-child(4) { right: 28px; height: 34px; }

  .city-lights {
    position: absolute;
    right: 14px;
    bottom: 204px;
    left: 14px;
    height: 36px;
  }

  .city-lights span {
    position: absolute;
    bottom: 0;
    width: 10px;
    background: #f9df8f;
    box-shadow: 0 0 var(--scene-glow) #f1a45f;
    opacity: var(--light-opacity);
  }

  .signal-rings {
    position: absolute;
    top: 62px;
    right: 74px;
    width: 74px;
    height: 74px;
    pointer-events: none;
  }

  .signal-rings span {
    position: absolute;
    border: 3px solid rgba(249, 223, 143, 0.18);
    animation: signal-flicker 2.4s steps(2, end) infinite;
  }

  .signal-rings span:nth-child(1) { inset: 0; }
  .signal-rings span:nth-child(2) { inset: 11px; animation-delay: 0.2s; }
  .signal-rings span:nth-child(3) { inset: 22px; animation-delay: 0.4s; }

  .pixel-scene.signal-clear .signal-rings span {
    border-color: rgba(249, 223, 143, 0.36);
  }

  .studio-room {
    position: absolute;
    right: 18px;
    bottom: 18px;
    left: 18px;
    height: 190px;
    border: 4px solid #6b3f55;
    background:
      linear-gradient(90deg, rgba(249, 223, 143, 0.05) 0 4px, transparent 4px 100%),
      linear-gradient(#3a263f 0 66%, #2a1c2f 66% 100%);
    background-size: 18px 100%, auto;
  }

  .studio-room::after {
    position: absolute;
    right: 0;
    bottom: 58px;
    left: 0;
    height: 4px;
    content: "";
    background: #6b3f55;
  }

  .studio-grid {
    position: absolute;
    inset: 0;
    background:
      linear-gradient(transparent 0 118px, rgba(18, 13, 24, 0.26) 118px 122px, transparent 122px),
      repeating-linear-gradient(90deg, transparent 0 23px, rgba(249, 223, 143, 0.04) 23px 25px);
    pointer-events: none;
  }

  .studio-room > * {
    z-index: 1;
  }

  .studio-room::after,
  .studio-grid {
    z-index: 0;
  }

  .wall-light {
    position: absolute;
    top: 16px;
    left: 18px;
    width: 44px;
    height: 22px;
    background: #f1a45f;
    box-shadow: 0 0 0 4px #4b3149, 0 0 var(--scene-glow) #f1a45f;
    animation: light-breathe 3.2s steps(3, end) infinite;
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

  .window span:nth-child(1) { left: 10px; height: 18px; }
  .window span:nth-child(2) { left: 26px; height: 28px; background: #f9df8f; }
  .window span:nth-child(3) { left: 42px; height: 12px; }
  .window span:nth-child(4) { left: 58px; height: 24px; background: #f1a45f; }
  .window span:nth-child(5) { left: 18px; height: 8px; bottom: 30px; }
  .window span:nth-child(6) { left: 50px; height: 10px; bottom: 32px; background: #f9df8f; }

  .pixel-scene.signal-thin .window span:nth-child(n + 4),
  .pixel-scene.signal-warm .window span:nth-child(5) {
    opacity: 0.28;
  }

  .rooftop-pot {
    position: absolute;
    top: 64px;
    right: 42px;
    width: 34px;
    height: 22px;
    border: 4px solid #442638;
    background: #7a4b4f;
  }

  .rooftop-pot span {
    position: absolute;
    bottom: 14px;
    width: 8px;
    background: #4f8f80;
  }

  .rooftop-pot span:nth-child(1) { left: 4px; height: 14px; }
  .rooftop-pot span:nth-child(2) { left: 13px; height: 22px; }
  .rooftop-pot span:nth-child(3) { right: 4px; height: 16px; }

  .pixel-scene.rooftop-complete .rooftop-pot span::after {
    position: absolute;
    top: -7px;
    left: 1px;
    width: 6px;
    height: 6px;
    content: "";
    background: #f9df8f;
    box-shadow: 0 0 var(--scene-glow) #f1a45f;
  }

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
    animation: host-idle 2.8s steps(2, end) infinite;
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
    background: linear-gradient(90deg, #6b5bb9 0 50%, #52489a 50% 100%);
  }

  .mic-stand {
    position: absolute;
    bottom: 72px;
    left: 184px;
    width: 26px;
    height: 46px;
  }

  .mic-stand::before {
    position: absolute;
    top: 0;
    left: 8px;
    width: 14px;
    height: 22px;
    border: 4px solid #442638;
    content: "";
    background: #c7a77b;
  }

  .mic-stand span {
    position: absolute;
    bottom: 0;
    left: 14px;
    width: 4px;
    height: 28px;
    background: #442638;
  }

  .mic-stand span::after {
    position: absolute;
    bottom: 0;
    left: -10px;
    width: 24px;
    height: 4px;
    content: "";
    background: #442638;
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
    top: calc(0px - var(--antenna-reach));
    left: 28px;
    width: 4px;
    height: var(--antenna-reach);
    background: #d6c08a;
  }

  .antenna::after {
    position: absolute;
    top: -6px;
    left: -4px;
    width: 12px;
    height: 8px;
    content: "";
    background: #f9df8f;
    box-shadow: 0 0 var(--scene-glow) #f1a45f;
  }

  .radio-wave {
    position: absolute;
    top: -18px;
    left: 14px;
    width: 40px;
    height: 18px;
    border-top: 4px solid rgba(249, 223, 143, 0.52);
    animation: wave-skip 1.6s steps(2, end) infinite;
  }

  .radio-face {
    position: absolute;
    inset: 12px;
    border: 4px solid #442638;
    background: repeating-linear-gradient(90deg, #f9df8f 0 5px, #f1a45f 5px 8px);
  }

  .radio-knobs {
    position: absolute;
    right: 8px;
    bottom: 8px;
    width: 8px;
    height: 8px;
    background: #442638;
    box-shadow: -14px 0 0 #442638;
  }

  .letter-box {
    right: 26px;
    bottom: 56px;
    width: 48px;
    height: 42px;
    background: #4f8f80;
  }

  .letter-box > span:first-child {
    display: block;
    width: 24px;
    height: 12px;
    margin: 10px auto;
    background: #f7e9c7;
  }

  .letter-flag {
    position: absolute;
    top: -16px;
    right: 6px;
    width: 14px;
    height: 14px;
    margin: 0;
    background: #ffcf91;
    opacity: var(--light-opacity);
    animation: mail-blink 2s steps(2, end) infinite;
  }

  .letter-box:disabled .letter-flag {
    opacity: 0.2;
    animation: none;
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
    background: linear-gradient(#9a5d55 0 45%, #7a4b4f 45% 100%);
  }

  .desk span {
    position: absolute;
    top: 6px;
    width: 20px;
    height: 6px;
    background: #2a1c2f;
  }

  .desk span:nth-child(1) { left: 14px; }
  .desk span:nth-child(2) { left: 48px; background: #f1a45f; }
  .desk span:nth-child(3) { right: 16px; }

  .status-panel,
  .letter-panel {
    margin-top: 0.75rem;
    padding: 0.75rem;
    background:
      linear-gradient(90deg, rgba(249, 223, 143, 0.04) 0 3px, transparent 3px 100%),
      #20172a;
    background-size: 16px 100%, auto;
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
    background:
      repeating-linear-gradient(90deg, rgba(255, 207, 145, 0.08) 0 5px, transparent 5px 12px),
      #15111d;
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
    box-shadow: 0 0 10px #ff6b4a;
    animation: on-air-pulse 1.4s steps(2, end) infinite;
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
    position: relative;
    border: 3px solid #4b3149;
    background: linear-gradient(135deg, rgba(249, 223, 143, 0.05), transparent 34%), #15111d;
    padding: 0.45rem;
  }

  .metrics-grid div::after,
  .resource-strip div::after,
  .letter-card::after {
    position: absolute;
    right: 4px;
    bottom: 4px;
    width: 8px;
    height: 8px;
    content: "";
    background: rgba(249, 223, 143, 0.16);
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
  .letter-list,
  .pack-status,
  .pack-collection {
    display: grid;
    gap: 0.45rem;
  }

  .actions button,
  .letter-list button,
  .reset-button {
    border: 3px solid #6b3f55;
    box-shadow: inset 3px 0 0 rgba(249, 223, 143, 0.08), inset -3px -3px 0 rgba(18, 13, 24, 0.42);
    color: #f7e9c7;
    background: linear-gradient(90deg, rgba(79, 143, 128, 0.12), transparent 42%), #2a1c2f;
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

  .letter-timer,
  .pack-status,
  .unlock-notice,
  .completion-notice,
  .pack-collection {
    margin-bottom: 0.45rem;
  }

  .pack-status,
  .unlock-notice,
  .completion-notice,
  .pack-card {
    border: 3px solid #4b3149;
    background: #15111d;
    color: #c7a77b;
    padding: 0.45rem;
    font-size: 0.72rem;
    line-height: 1.45;
  }

  .unlock-notice,
  .completion-notice {
    border-color: #f1a45f;
    color: #ffcf91;
    background: #2a1c2f;
  }

  .completion-notice {
    border-color: #f9df8f;
    box-shadow: inset 0 0 0 2px #4f8f80;
  }

  .unlock-notice span,
  .unlock-notice strong,
  .unlock-notice small,
  .completion-notice span,
  .completion-notice strong,
  .completion-notice small,
  .pack-card span,
  .pack-card strong,
  .pack-card small {
    display: block;
  }

  .pack-card.unlocked {
    border-color: #4f8f80;
  }

  .pack-card.complete {
    border-color: #f9df8f;
    color: #ffcf91;
  }

  .pack-card div {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 0.45rem;
  }

  .pack-card p {
    margin: 0.3rem 0;
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
    display: block;
    height: 8px;
    border: 2px solid #4b3149;
    background: #15111d;
  }

  .progress-track span {
    display: block;
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

  .dj-comment {
    margin: 0.65rem 0;
    border-left: 4px solid #f1a45f;
    padding: 0.45rem 0 0.45rem 0.55rem;
    color: #ffcf91;
    background: #20172a;
    line-height: 1.55;
  }

  .story-thread {
    display: grid;
    gap: 0.28rem;
    margin: 0.55rem 0;
    border: 3px solid #4b3149;
    padding: 0.45rem;
    background: #20172a;
  }

  .story-thread strong {
    color: #f7e9c7;
    font-size: 0.72rem;
  }

  .story-thread span {
    color: #7f6a65;
    font-size: 0.72rem;
  }

  .story-thread span.heard {
    color: #9ed0bc;
  }

  .pack-progress-copy {
    margin-bottom: 0;
  }

  @keyframes light-breathe {
    0%, 100% { opacity: 0.78; }
    50% { opacity: 1; }
  }

  @keyframes host-idle {
    0%, 100% { transform: translateY(0); }
    50% { transform: translateY(2px); }
  }

  @keyframes signal-flicker {
    0%, 100% { opacity: 0.2; }
    50% { opacity: var(--signal-pulse); }
  }

  @keyframes wave-skip {
    0%, 100% { transform: translateY(0); opacity: 0.32; }
    50% { transform: translateY(-4px); opacity: var(--signal-pulse); }
  }

  @keyframes mail-blink {
    0%, 100% { transform: translateY(0); }
    50% { transform: translateY(-3px); }
  }

  @keyframes on-air-pulse {
    0%, 100% { opacity: 0.45; }
    50% { opacity: 1; }
  }

  @media (prefers-reduced-motion: reduce) {
    .wall-light,
    .signal-rings span,
    .host,
    .radio-wave,
    .letter-flag,
    .on-air span {
      animation: none;
    }
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

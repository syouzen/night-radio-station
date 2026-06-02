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

  type OfflineBroadcastReport = {
    durationLabel: string;
    elapsedMinutes: number;
    listeners: number;
    signal: number;
    reputation: number;
    stories: number;
    letters: Letter[];
  };

  type CollectionReward = {
    id: string;
    packId: StoryPack["id"];
    title: string;
    description: string;
    preview: string;
    souvenirClass: string;
  };

  type FrequencyBand = {
    id: string;
    label: string;
    frequency: number;
    subtitle: string;
    description: string;
    sceneClass: string;
    accent: string;
    preferredPackIds: StoryPack["id"][];
    preferredCharacterIds: Letter["characterId"][];
  };

  type CollectionSet = {
    id: string;
    title: string;
    description: string;
    rewardIds: CollectionReward["id"][];
    bonusTitle: string;
    bonusDescription: string;
    sceneClass: string;
  };

  type RoomPlacementSlot = {
    id: string;
    title: string;
    description: string;
    emptyLabel: string;
    sceneClass: string;
  };

  type RoomPlacement = {
    slotId: RoomPlacementSlot["id"];
    rewardId: CollectionReward["id"] | null;
  };

  type RoomAmbience = {
    id: string;
    label: string;
    description: string;
    sceneClass: string;
  };

  type ListenerVisitTrace = {
    characterId: Letter["characterId"];
    title: string;
    description: string;
    sceneClass: string;
  };

  type KeepsakeSynergy = {
    id: string;
    title: string;
    description: string;
    rewardIds: CollectionReward["id"][];
    sceneClass: string;
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

  const collectionRewards: CollectionReward[] = [
    {
      id: "midnight-ticket",
      packId: "first-night",
      title: "심야 극장 티켓",
      description: "첫 번째 밤의 모든 사연을 모으면 선반에 낡은 극장 티켓이 놓입니다.",
      preview: "첫 번째 밤 사연 묶음 완성",
      souvenirClass: "ticket"
    },
    {
      id: "moonflower-pot",
      packId: "rooftop-garden",
      title: "달맞이꽃 표본",
      description: "옥상 정원의 모든 사연을 모으면 DJ 책상 옆에 노란 꽃 표본이 켜집니다.",
      preview: "옥상 정원 사연 묶음 완성",
      souvenirClass: "flower"
    }
  ];

  const frequencyBands: FrequencyBand[] = [
    {
      id: "alley",
      label: "따뜻한 골목",
      frequency: 91.7,
      subtitle: "귀가와 가로등",
      description: "택시, 편의점, 다리 위 귀가길처럼 도시의 가장 낮은 불빛을 잡습니다.",
      sceneClass: "band-alley",
      accent: "#ffcf91",
      preferredPackIds: ["first-night"],
      preferredCharacterIds: ["taxi-minu", "night-store-jun", "bridge-sora"]
    },
    {
      id: "rooftop",
      label: "옥상 정원",
      frequency: 95.3,
      subtitle: "비와 식물",
      description: "물탱크, 난간 리본, 달맞이꽃처럼 조용히 자라는 밤의 사연을 잡습니다.",
      sceneClass: "band-rooftop",
      accent: "#8bd7a4",
      preferredPackIds: ["rooftop-garden"],
      preferredCharacterIds: ["gardener-haerin", "rooftop-dalsoo", "delivery-mira", "sleepless-yeon"]
    },
    {
      id: "hidden-city",
      label: "숨은 도시",
      frequency: 97.3,
      subtitle: "잡음과 정류장",
      description: "지도에 없는 정류장, 꺼진 라디오, 흙냄새 나는 스피커의 기묘한 주파수입니다.",
      sceneClass: "band-hidden-city",
      accent: "#b99cff",
      preferredPackIds: ["first-night", "rooftop-garden"],
      preferredCharacterIds: ["hidden-city-listener", "night-store-jun", "repair-seoho"]
    }
  ];

  const collectionSets: CollectionSet[] = [
    {
      id: "dawn-keepsake-shelf",
      title: "새벽 보관함 세트",
      description: "첫 번째 밤과 옥상 정원의 소장품을 함께 모아 방송국 선반을 하나의 기억 보관함으로 엮습니다.",
      rewardIds: ["midnight-ticket", "moonflower-pot"],
      bonusTitle: "기억 보관함 조명",
      bonusDescription: "세트 완성 시 선반 아래 작은 호박색 조명이 켜지고 다음 보상 목표가 더 선명해집니다.",
      sceneClass: "archive-lamp"
    }
  ];

  const roomPlacementSlots: RoomPlacementSlot[] = [
    {
      id: "dj-desk",
      title: "DJ 책상",
      description: "방송 중 가장 자주 보이는 자리입니다. 배치한 소장품은 새 사연 신호를 조금 더 선명하게 만듭니다.",
      emptyLabel: "책상 위가 비어 있습니다",
      sceneClass: "desk-slot"
    },
    {
      id: "memory-shelf",
      title: "기억 선반",
      description: "세트 보관함과 맞닿은 자리입니다. 배치한 소장품은 청취자가 머무는 불빛을 늘립니다.",
      emptyLabel: "선반에 남길 물건을 기다립니다",
      sceneClass: "shelf-slot"
    },
    {
      id: "window-nook",
      title: "창가 틈새",
      description: "도시 야경 옆 작은 전시 공간입니다. 배치한 소장품은 장면 분위기를 가장 먼저 바꿉니다.",
      emptyLabel: "창가에 아직 전시품이 없습니다",
      sceneClass: "window-slot"
    }
  ];

  const roomAmbiences: RoomAmbience[] = [
    {
      id: "quiet-room",
      label: "고요한 방송실",
      description: "소장품이 아직 적어 벽 조명과 라디오 파동만 조용히 움직입니다.",
      sceneClass: "ambience-quiet"
    },
    {
      id: "warm-room",
      label: "따뜻한 방송실",
      description: "배치된 소장품이나 청취자 흔적이 생기면 방 전체가 부드럽게 살아납니다.",
      sceneClass: "ambience-warm"
    },
    {
      id: "alive-room",
      label: "깨어 있는 방송실",
      description: "소장품 조합과 완성된 청취자 흔적이 밤마다 작은 반응을 일으킵니다.",
      sceneClass: "ambience-alive"
    }
  ];

  const listenerVisitTraces: ListenerVisitTrace[] = [
    { characterId: "taxi-minu", title: "극장행 영수증", description: "민우가 남긴 낡은 요금 영수증이 DJ 책상 옆에 꽂혔습니다.", sceneClass: "trace-taxi" },
    { characterId: "gardener-haerin", title: "작은 잎 그림", description: "해린이 보낸 잎 그림이 창가 조명 아래 붙었습니다.", sceneClass: "trace-leaf" },
    { characterId: "hidden-city-listener", title: "없는 정류장 표식", description: "익명 청취자가 남긴 정류장 기호가 벽 앨범 구석에서 깜빡입니다.", sceneClass: "trace-hidden" },
    { characterId: "night-store-jun", title: "새벽 계산표", description: "준이 접어 둔 계산표가 라디오 잡음에 맞춰 흔들립니다.", sceneClass: "trace-store" },
    { characterId: "bridge-sora", title: "가로등 스티커", description: "소라가 보낸 작은 가로등 스티커가 도시 창문 옆에 붙었습니다.", sceneClass: "trace-bridge" },
    { characterId: "rooftop-dalsoo", title: "물뿌리개 메모", description: "달수가 남긴 물주기 메모가 옥상 화분 옆에 놓였습니다.", sceneClass: "trace-water" },
    { characterId: "delivery-mira", title: "노란 배송 리본", description: "미라의 리본이 창가 틈새에서 새벽빛을 붙잡습니다.", sceneClass: "trace-ribbon" },
    { characterId: "repair-seoho", title: "수리 나사", description: "서호가 남긴 나사가 스피커 진동에 맞춰 작게 빛납니다.", sceneClass: "trace-screw" },
    { characterId: "sleepless-yeon", title: "안테나 그림자 쪽지", description: "연이 접어 둔 쪽지가 안테나 그림자 아래 머뭅니다.", sceneClass: "trace-note" }
  ];

  const keepsakeSynergies: KeepsakeSynergy[] = [
    {
      id: "dawn-memory-glow",
      title: "새벽 기억 조명",
      description: "심야 극장 티켓과 달맞이꽃 표본이 함께 배치되면 선반 조명이 더 넓게 퍼집니다.",
      rewardIds: ["midnight-ticket", "moonflower-pot"],
      sceneClass: "synergy-dawn"
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
  let unlockedRewardIds = $state<CollectionReward["id"][]>([]);
  let currentBandId = $state<FrequencyBand["id"]>(frequencyBands[0].id);
  let unlockedCollectionSetIds = $state<CollectionSet["id"][]>([]);
  let roomPlacements = $state<RoomPlacement[]>(createDefaultRoomPlacements());
  let unlockedPackNotice = $state<StoryPack | null>(null);
  let completedPackNotice = $state<StoryPack | null>(null);
  let offlineReport = $state<OfflineBroadcastReport | null>(null);
  let rewardNotice = $state<CollectionReward | null>(null);
  let collectionSetNotice = $state<CollectionSet | null>(null);
  let hasLoadedState = $state(false);
  let letters = $state<Letter[]>([incomingLetters[0]]);
  let selectedLetter = $state<Letter>(incomingLetters[0]);
  let stationLog = $state("첫 사연이 접수되었습니다. 주파수는 아직 좁지만 방송은 살아 있습니다.");

  const saveKey = "night-radio-station-state";
  const offlineCapMs = 8 * 60 * 60 * 1000;
  const offlineMinimumMs = 60 * 1000;

  const currentBand = $derived(frequencyBands.find((band) => band.id === currentBandId) ?? frequencyBands[0]);
  const currentFrequency = $derived(currentBand.frequency.toFixed(1));
  const unlockedStoryPacks = $derived(storyPacks.filter((pack) => isStoryPackUnlocked(pack)));
  const nextStoryPack = $derived(storyPacks.find((pack) => !isStoryPackUnlocked(pack)));
  const nextStoryPackProgress = $derived(nextStoryPack ? storyPackProgress(nextStoryPack) : 100);
  const availableLetters = $derived(incomingLetters.filter((letter) => unlockedStoryPackIds.includes(letter.packId)));
  const selectedStoryPack = $derived(storyPacks.find((pack) => pack.id === selectedLetter.packId) ?? storyPacks[0]);
  const completedStoryPacks = $derived(storyPacks.filter((pack) => isStoryPackComplete(pack)));
  const unlockedRewards = $derived(collectionRewards.filter((reward) => isRewardUnlocked(reward)));
  const unlockedCollectionSets = $derived(collectionSets.filter((set) => isCollectionSetUnlocked(set)));
  const nextCollectionReward = $derived(collectionRewards.find((reward) => !isRewardUnlocked(reward)));
  const nextCollectionSet = $derived(collectionSets.find((set) => !isCollectionSetUnlocked(set)));
  const placedRoomRewards = $derived(roomPlacements.map((placement) => collectionRewards.find((reward) => reward.id === placement.rewardId)).filter((reward): reward is CollectionReward => Boolean(reward)));
  const unplacedUnlockedRewards = $derived(unlockedRewards.filter((reward) => !isRewardPlaced(reward)));
  const occupiedRoomPlacementCount = $derived(placedRoomRewards.length);
  const activeKeepsakeSynergies = $derived(keepsakeSynergies.filter((synergy) => synergy.rewardIds.every((rewardId) => roomPlacements.some((placement) => placement.rewardId === rewardId))));
  const roomPlacementSignalBonus = $derived(Math.min(3, occupiedRoomPlacementCount));
  const roomSynergySignalBonus = $derived(activeKeepsakeSynergies.length);
  const nextUnplacedReward = $derived(unplacedUnlockedRewards[0]);
  const characterIds = $derived(Array.from(new Set(incomingLetters.map((letter) => letter.characterId))));
  const completedCharacterIds = $derived(characterIds.filter((characterId) => characterReceivedCount(characterId) === characterLetters(characterId).length));
  const activeVisitorTraces = $derived(listenerVisitTraces.filter((trace) => completedCharacterIds.includes(trace.characterId)).slice(0, 5));
  const roomVisitorListenerBonus = $derived(Math.min(3, activeVisitorTraces.length));
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
  const isArchiveLampUnlocked = $derived(unlockedCollectionSetIds.includes("dawn-keepsake-shelf"));
  const cityActivityLevel = $derived(listeners >= 48 ? 3 : listeners >= 24 ? 2 : listeners >= 10 ? 1 : 0);
  const hasOfflineMail = $derived(Boolean(offlineReport && offlineReport.letters.length > 0));
  const activeCharacterEcho = $derived(selectedLetter.characterId);
  const sceneMilestoneText = $derived(
    completedStoryPacks.length > 0
      ? `${completedStoryPacks[completedStoryPacks.length - 1].title} 완성`
      : unlockedStoryPacks.length > 1
        ? `${unlockedStoryPacks[unlockedStoryPacks.length - 1].title} 수신 중`
        : `${currentBand.label} 대역 수신 중`
  );
  const roomPlacementListenerBonus = $derived(occupiedRoomPlacementCount === 0 ? 0 : occupiedRoomPlacementCount + (isArchiveLampUnlocked ? 1 : 0));
  const totalRoomSignalBonus = $derived(roomPlacementSignalBonus + roomSynergySignalBonus);
  const totalRoomListenerBonus = $derived(roomPlacementListenerBonus + roomVisitorListenerBonus);
  const currentRoomAmbience = $derived(roomAmbiences[activeKeepsakeSynergies.length > 0 && activeVisitorTraces.length > 0 ? 2 : occupiedRoomPlacementCount > 0 || activeVisitorTraces.length > 0 ? 1 : 0]);
  const sceneStatus = $derived(
    `현재 방송국은 FM ${currentFrequency} ${currentBand.label} 대역에서 ${signalMood === "clear" ? "선명한" : signalMood === "warm" ? "따뜻한" : "희미한"} 신호로 송출 중입니다. 도시 창문 ${listenerLightCount}개가 켜져 있고 도시 활동 단계는 ${cityActivityLevel}입니다. 안테나는 Lv.${antennaLevel}, 송신기는 Lv.${transmitterLevel}입니다.${hasOfflineMail ? " 책상 위에는 밤샘 방송 리포트 사연 더미가 쌓여 있습니다." : ""}${isRooftopGardenUnlocked ? " 창가에는 옥상 정원 화분이 놓여 있습니다." : ""}${isRooftopGardenComplete ? " 화분에는 완결된 사연을 닮은 노란 꽃이 피었습니다." : ""}${unlockedRewards.length > 0 ? ` 선반에는 소장품 ${unlockedRewards.length}개가 놓여 있습니다.` : ""}${currentRoomAmbience ? ` 방 분위기는 ${currentRoomAmbience.label}입니다.` : ""}${occupiedRoomPlacementCount > 0 ? ` 방송국 구역 ${occupiedRoomPlacementCount}곳에 소장품이 배치되어 새 사연 신호 +${totalRoomSignalBonus}, 청취자 +${totalRoomListenerBonus} 보너스를 줍니다.` : ""}${activeKeepsakeSynergies.length > 0 ? ` 소장품 동조 효과 ${activeKeepsakeSynergies.length}개가 켜져 있습니다.` : ""}${activeVisitorTraces.length > 0 ? ` 청취자 방문 흔적 ${activeVisitorTraces.length}개가 남아 있습니다.` : ""}${completedCharacterIds.length > 0 ? ` 벽 앨범에는 완성된 청취자 기록 ${completedCharacterIds.length}개가 꽂혀 있습니다.` : ""}${isArchiveLampUnlocked ? " 선반 아래 기억 보관함 조명이 켜져 있습니다." : ""}`
  );

  function savedNumber(value: unknown, fallback: number) {
    return typeof value === "number" && Number.isFinite(value) ? value : fallback;
  }

  function savedNonNegativeInteger(value: unknown, fallback: number) {
    return Math.max(0, Math.floor(savedNumber(value, fallback)));
  }

  function normalizeBandId(value: unknown) {
    return typeof value === "string" && frequencyBands.some((band) => band.id === value) ? value : frequencyBands[0].id;
  }

  function createDefaultRoomPlacements() {
    return roomPlacementSlots.map((slot) => ({ slotId: slot.id, rewardId: null }));
  }

  function normalizeRoomPlacements(value: unknown) {
    const savedPlacements = Array.isArray(value) ? value : [];
    const usedRewardIds = new Set<CollectionReward["id"]>();

    return roomPlacementSlots.map((slot) => {
      const savedPlacement = savedPlacements.find((placement) => placement?.slotId === slot.id);
      const rewardId = savedPlacement?.rewardId;
      const isValidReward = typeof rewardId === "string" && unlockedRewardIds.includes(rewardId) && !usedRewardIds.has(rewardId);
      if (!isValidReward) return { slotId: slot.id, rewardId: null };

      usedRewardIds.add(rewardId);
      return { slotId: slot.id, rewardId };
    });
  }

  function frequencyBandScore(letter: Letter, band = currentBand) {
    let score = 0;
    if (band.preferredPackIds.includes(letter.packId)) score += 2;
    if (band.preferredCharacterIds.includes(letter.characterId)) score += 3;
    return score;
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

    const reward = collectionRewards.find((collectionReward) => collectionReward.packId === completedPack.id);
    completedStoryPackIds = [...completedStoryPackIds, completedPack.id];
    completedPackNotice = completedPack;
    if (reward && !unlockedRewardIds.includes(reward.id)) {
      unlockedRewardIds = [...unlockedRewardIds, reward.id];
      rewardNotice = reward;
    }
    stationLog = reward
      ? `${completedPack.title} 사연 묶음이 완성되어 ${reward.title} 소장품이 선반에 놓였습니다.`
      : `${completedPack.title} 사연 묶음이 완성되었습니다. DJ가 마지막 코멘트를 편성표에 남겼습니다.`;
    saveStationState();
  });

  $effect(() => {
    if (!hasLoadedState) return;

    const completedSet = collectionSets.find((set) => isCollectionSetComplete(set) && !unlockedCollectionSetIds.includes(set.id));
    if (!completedSet) return;

    unlockedCollectionSetIds = [...unlockedCollectionSetIds, completedSet.id];
    collectionSetNotice = completedSet;
    stationLog = `${completedSet.title} 완성. ${completedSet.bonusTitle}이 방송국 장면에 켜졌습니다.`;
    saveStationState();
  });

  function normalizeLetter(letter: Partial<Letter> | null | undefined) {
    return incomingLetters.find((incomingLetter) => incomingLetter.id === letter?.id || incomingLetter.subject === letter?.subject) ?? incomingLetters[0];
  }

  function normalizeOfflineReport(report: Partial<OfflineBroadcastReport> | null) {
    if (!report) return null;
    const elapsedMinutes = savedNonNegativeInteger(report.elapsedMinutes, 0);
    if (elapsedMinutes < 1) return null;

    return {
      durationLabel: typeof report.durationLabel === "string" ? report.durationLabel : formatOfflineDuration(elapsedMinutes),
      elapsedMinutes,
      listeners: savedNonNegativeInteger(report.listeners, 0),
      signal: savedNonNegativeInteger(report.signal, 0),
      reputation: savedNonNegativeInteger(report.reputation, 0),
      stories: savedNonNegativeInteger(report.stories, 0),
      letters: Array.isArray(report.letters) ? report.letters.map(normalizeLetter) : []
    };
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

  function orderedLetters(availablePackIds = unlockedStoryPackIds, band = currentBand) {
    return incomingLetters
      .filter((letter) => availablePackIds.includes(letter.packId))
      .sort(
        (a, b) =>
          frequencyBandScore(b, band) - frequencyBandScore(a, band) ||
          storyPacks.findIndex((pack) => pack.id === a.packId) - storyPacks.findIndex((pack) => pack.id === b.packId) ||
          a.order - b.order
      );
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

  function formatOfflineDuration(minutes: number) {
    const hours = Math.floor(minutes / 60);
    const restMinutes = minutes % 60;
    if (hours === 0) return `${restMinutes}분`;
    if (restMinutes === 0) return `${hours}시간`;
    return `${hours}시간 ${restMinutes}분`;
  }

  function createOfflineReport(elapsedMs: number) {
    if (elapsedMs < offlineMinimumMs) return null;

    const cappedMs = Math.min(elapsedMs, offlineCapMs);
    const elapsedMinutes = Math.max(1, Math.floor(cappedMs / 60000));
    const sourceLetters = orderedLetters().filter((letter) => !receivedLetterIds.includes(letter.id));
    const fallbackLetters = orderedLetters().length > 0 ? orderedLetters() : incomingLetters;
    const letterPool = sourceLetters.length > 0 ? sourceLetters : fallbackLetters;
    const letterCount = Math.min(3, Math.max(1, Math.floor(elapsedMinutes / 90) + 1), letterPool.length);
    const queuedLetters = letterPool.slice(0, letterCount);

    return {
      durationLabel: formatOfflineDuration(elapsedMinutes),
      elapsedMinutes,
      listeners: Math.max(1, Math.floor(elapsedMinutes / 4) + transmitterLevel + totalRoomListenerBonus),
      signal: Math.min(18, Math.max(1, Math.floor(elapsedMinutes / 12) + antennaLevel + totalRoomSignalBonus)),
      reputation: Math.max(1, Math.floor(elapsedMinutes / 45)),
      stories: Math.max(1, Math.floor(elapsedMinutes / 60)),
      letters: queuedLetters
    };
  }

  function claimOfflineReport() {
    if (!offlineReport) return;

    const offlineLetters = offlineReport.letters.filter((letter) => !receivedLetterIds.includes(letter.id));
    secondsOnline += offlineReport.elapsedMinutes * 60;
    listeners += offlineReport.listeners;
    signal = Math.min(100, signal + offlineReport.signal);
    reputation += offlineReport.reputation;
    stories += offlineReport.stories;
    receivedLetterCount += offlineLetters.length;
    receivedLetterIds = knownLetterIds([...receivedLetterIds, ...offlineLetters.map((letter) => letter.id)]);
    letters = [...offlineLetters, ...letters].slice(0, 6);
    selectedLetter = offlineLetters[0] ?? selectedLetter;
    stationLog = `밤새 방송 리포트 확인 완료. ${offlineReport.durationLabel} 동안 새 청취자와 사연이 쌓였습니다.`;
    offlineReport = null;
    saveStationState();
  }

  function dismissOfflineReport() {
    offlineReport = null;
    stationLog = "밤샘 방송 리포트를 닫았습니다. 다음 방송 기록부터 다시 모읍니다.";
    saveStationState();
  }

  function selectFrequencyBand(band: FrequencyBand) {
    if (currentBandId === band.id) return;
    currentBandId = band.id;
    stationLog = `FM ${band.frequency.toFixed(1)} ${band.label} 대역으로 조율했습니다. ${band.subtitle} 사연이 더 선명하게 잡힙니다.`;
    saveStationState();
  }

  function isRewardUnlocked(reward: CollectionReward) {
    return unlockedRewardIds.includes(reward.id);
  }

  function roomPlacementSlotReward(slot: RoomPlacementSlot) {
    const placement = roomPlacements.find((roomPlacement) => roomPlacement.slotId === slot.id);
    return collectionRewards.find((reward) => reward.id === placement?.rewardId) ?? null;
  }

  function isRewardPlaced(reward: CollectionReward) {
    return roomPlacements.some((placement) => placement.rewardId === reward.id);
  }

  function placeReward(slot: RoomPlacementSlot, reward: CollectionReward) {
    if (!isRewardUnlocked(reward)) return;

    roomPlacements = roomPlacements.map((placement) => ({
      slotId: placement.slotId,
      rewardId: placement.slotId === slot.id ? reward.id : placement.rewardId === reward.id ? null : placement.rewardId
    }));
    stationLog = `${reward.title} 소장품을 ${slot.title}에 배치했습니다. 방송국 장면이 조금 더 채워졌습니다.`;
    saveStationState();
  }

  function clearRoomPlacement(slot: RoomPlacementSlot) {
    const reward = roomPlacementSlotReward(slot);
    if (!reward) return;

    roomPlacements = roomPlacements.map((placement) => (placement.slotId === slot.id ? { slotId: placement.slotId, rewardId: null } : placement));
    stationLog = `${slot.title}에서 ${reward.title} 소장품을 잠시 치웠습니다.`;
    saveStationState();
  }

  function rewardProgress(reward: CollectionReward) {
    const pack = storyPacks.find((storyPack) => storyPack.id === reward.packId) ?? storyPacks[0];
    return isStoryPackUnlocked(pack) ? Math.round((storyPackReceivedCount(pack) / storyPackLetters(pack).length) * 100) : storyPackProgress(pack);
  }

  function rewardPackTitle(reward: CollectionReward) {
    return storyPacks.find((pack) => pack.id === reward.packId)?.title ?? "알 수 없는 사연 묶음";
  }

  function collectionSetRewards(set: CollectionSet) {
    return collectionRewards.filter((reward) => set.rewardIds.includes(reward.id));
  }

  function collectionSetUnlockedRewards(set: CollectionSet) {
    return collectionSetRewards(set).filter((reward) => isRewardUnlocked(reward));
  }

  function isCollectionSetComplete(set: CollectionSet) {
    const rewards = collectionSetRewards(set);
    return rewards.length > 0 && rewards.every((reward) => isRewardUnlocked(reward));
  }

  function isCollectionSetUnlocked(set: CollectionSet) {
    return unlockedCollectionSetIds.includes(set.id);
  }

  function collectionSetProgress(set: CollectionSet) {
    const rewards = collectionSetRewards(set);
    if (rewards.length === 0) return 0;
    return Math.round((collectionSetUnlockedRewards(set).length / rewards.length) * 100);
  }

  function nextCollectionSetMissingReward(set: CollectionSet) {
    return collectionSetRewards(set).find((reward) => !isRewardUnlocked(reward));
  }

  function characterLetters(characterId: string) {
    return incomingLetters.filter((letter) => letter.characterId === characterId);
  }

  function characterReceivedCount(characterId: string) {
    return characterLetters(characterId).filter((letter) => receivedLetterIds.includes(letter.id)).length;
  }

  function characterProgress(characterId: string) {
    return Math.round((characterReceivedCount(characterId) / characterLetters(characterId).length) * 100);
  }

  function characterNextLetter(characterId: string) {
    return characterLetters(characterId).find((letter) => !receivedLetterIds.includes(letter.id));
  }

  function characterRelationStage(characterId: string) {
    const progress = characterProgress(characterId);
    if (progress === 100) return "고정 청취자";
    if (progress >= 50) return "이어지는 목소리";
    return "낯선 주파수";
  }

  function characterAlbumHint(characterId: string) {
    const nextLetter = characterNextLetter(characterId);
    if (!nextLetter) return "관계 기록 완성";

    const pack = storyPacks.find((storyPack) => storyPack.id === nextLetter.packId) ?? storyPacks[0];
    return isStoryPackUnlocked(pack) ? `다음 사연: ${nextLetter.sequence}` : `${pack.title} 해금 후 다음 사연 도착`;
  }

  function characterName(characterId: string) {
    return characterLetters(characterId)[0]?.author ?? "익명 청취자";
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
        unlockedRewardIds,
        currentBandId,
        unlockedCollectionSetIds,
        roomPlacements,
        letters,
        offlineReport,
        lastSavedAt: Date.now()
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
    signal = Math.min(100, signal + 4 + antennaLevel + totalRoomSignalBonus);
    listeners += 2 + transmitterLevel + totalRoomListenerBonus;
    stationLog = `FM ${currentFrequency} ${currentBand.label} 대역에서 ${next.author}의 사연이 도착했습니다. DJ 코멘트: ${next.djComment}`;
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
    unlockedRewardIds = [];
    currentBandId = frequencyBands[0].id;
    unlockedCollectionSetIds = [];
    roomPlacements = createDefaultRoomPlacements();
    unlockedPackNotice = null;
    completedPackNotice = null;
    offlineReport = null;
    rewardNotice = null;
    collectionSetNotice = null;
    letters = [incomingLetters[0]];
    selectedLetter = incomingLetters[0];
    stationLog = "방송국 기록을 지우고 첫 사연부터 다시 송출합니다.";
    if (browser) localStorage.removeItem(saveKey);
  }

  onMount(() => {
    if (browser) {
      const saved = localStorage.getItem(saveKey);
      if (saved) {
        try {
          const state = JSON.parse(saved);
          secondsOnline = savedNonNegativeInteger(state.secondsOnline, secondsOnline);
          signal = Math.max(18, Math.min(100, savedNonNegativeInteger(state.signal, signal)));
          listeners = Math.max(1, savedNonNegativeInteger(state.listeners, listeners));
          reputation = savedNonNegativeInteger(state.reputation, reputation);
          stories = savedNonNegativeInteger(state.stories, stories);
          antennaLevel = Math.max(1, savedNonNegativeInteger(state.antennaLevel, antennaLevel));
          transmitterLevel = Math.max(1, savedNonNegativeInteger(state.transmitterLevel, transmitterLevel));
          letters = Array.isArray(state.letters) ? state.letters.map(normalizeLetter) : letters;
          receivedLetterCount = savedNonNegativeInteger(state.receivedLetterCount, Math.max(letters.length, 1));
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
          const earnedRewardIds = collectionRewards.filter((reward) => completedStoryPackIds.includes(reward.packId)).map((reward) => reward.id);
          unlockedRewardIds = Array.isArray(state.unlockedRewardIds)
            ? Array.from(new Set([...state.unlockedRewardIds.filter((id: string) => collectionRewards.some((reward) => reward.id === id)), ...earnedRewardIds]))
            : earnedRewardIds;
          currentBandId = normalizeBandId(state.currentBandId);
          const earnedCollectionSetIds = collectionSets.filter(isCollectionSetComplete).map((set) => set.id);
          unlockedCollectionSetIds = Array.isArray(state.unlockedCollectionSetIds)
            ? Array.from(new Set([...state.unlockedCollectionSetIds.filter((id: string) => collectionSets.some((set) => set.id === id)), ...earnedCollectionSetIds]))
            : earnedCollectionSetIds;
          roomPlacements = normalizeRoomPlacements(state.roomPlacements);
          selectedLetter = letters[0] ?? incomingLetters[0];

          const lastSavedAt = savedNumber(state.lastSavedAt, Date.now());
          offlineReport = normalizeOfflineReport(state.offlineReport) ?? createOfflineReport(Date.now() - lastSavedAt);
          if (offlineReport) saveStationState();
        } catch {
          localStorage.removeItem(saveKey);
          stationLog = "손상된 저장 기록을 지우고 첫 방송 상태로 복구했습니다.";
        }
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

<main class="station-shell" style={`--band-accent: ${currentBand.accent};`} aria-label="Night Radio Station">
  <section
    class={`pixel-scene signal-${signalMood} ${currentBand.sceneClass} ${currentRoomAmbience.sceneClass} city-level-${cityActivityLevel} echo-${activeCharacterEcho}${hasOfflineMail ? " has-offline-mail" : ""}${isRooftopGardenUnlocked ? " has-rooftop" : ""}${isRooftopGardenComplete ? " rooftop-complete" : ""}${isArchiveLampUnlocked ? " has-archive-lamp" : ""}${occupiedRoomPlacementCount > 0 ? " has-placements" : ""}${activeKeepsakeSynergies.length > 0 ? " has-synergy" : ""}${activeVisitorTraces.length > 0 ? " has-visitor-traces" : ""}`}
    style={`--signal-pulse: ${scenePulse}; --scene-glow: ${sceneGlow}; --light-opacity: ${lightOpacity}; --antenna-reach: ${antennaReach}px; --listener-lights: ${listenerLightCount}; --band-accent: ${currentBand.accent};`}
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
    <div class="city-activity" aria-hidden="true"><span></span><span></span><span></span><span></span></div>
    <div class="ambient-trail" aria-hidden="true"><span></span><span></span><span></span></div>
    <div class={`character-echo ${activeCharacterEcho}`} aria-hidden="true"><span></span><span></span><span></span></div>
    <div class="scene-milestone" aria-hidden="true">{sceneMilestoneText}</div>
    <div class="signal-rings" aria-hidden="true">
      <span></span><span></span><span></span>
    </div>
    <div class={`band-prop ${currentBand.id}`} aria-hidden="true"><span></span><span></span><span></span></div>

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
      <div class="album-board" aria-hidden="true">
        {#each characterIds as characterId (characterId)}
          <span class:active={characterReceivedCount(characterId) > 0} class:complete={completedCharacterIds.includes(characterId)}></span>
        {/each}
      </div>
      {#if activeVisitorTraces.length > 0}
        <div class="visitor-notes" aria-hidden="true">
          {#each activeVisitorTraces as trace (trace.characterId)}
            <span class={trace.sceneClass}></span>
          {/each}
        </div>
      {/if}
      <div class="shelf" aria-hidden="true">
        {#if unlockedRewards.length === 0}
          <span class="souvenir placeholder"></span><span class="souvenir placeholder"></span><span class="souvenir placeholder"></span>
        {:else}
          {#each unlockedRewards as reward (reward.id)}
            <span class={`souvenir ${reward.souvenirClass}`}></span>
          {/each}
        {/if}
      </div>
      <div class="placement-stage" aria-hidden="true">
        {#each roomPlacementSlots as slot (slot.id)}
          <div class={`room-keepsake ${slot.sceneClass}`} class:filled={Boolean(roomPlacementSlotReward(slot))}>
            {#if roomPlacementSlotReward(slot)}
              <span class={`souvenir ${roomPlacementSlotReward(slot)?.souvenirClass}`}></span>
            {/if}
          </div>
        {/each}
        {#each activeKeepsakeSynergies as synergy (synergy.id)}
          <span class={`synergy-glow ${synergy.sceneClass}`}></span>
        {/each}
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
      {#if hasOfflineMail}
        <div class="offline-mail-stack" aria-hidden="true"><span></span><span></span><span></span></div>
      {/if}
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

    <section class="frequency-panel" aria-labelledby="frequency-title">
      <div>
        <p class="eyebrow">frequency tuning</p>
        <h2 id="frequency-title">FM {currentFrequency} · {currentBand.label}</h2>
        <p>{currentBand.description}</p>
      </div>
      <div class="frequency-options" aria-label="주파수 대역 선택">
        {#each frequencyBands as band (band.id)}
          <button
            type="button"
            class:active={currentBandId === band.id}
            aria-pressed={currentBandId === band.id}
            onclick={() => selectFrequencyBand(band)}
          >
            <span>{band.frequency.toFixed(1)}</span>
            {band.label}
            <small>{band.subtitle}</small>
          </button>
        {/each}
      </div>
    </section>

    <p class="station-log" aria-live="polite">{stationLog}</p>

    {#if offlineReport}
      <section class="offline-report" aria-labelledby="offline-report-title" aria-live="polite">
        <div>
          <span>밤샘 방송 리포트</span>
          <h2 id="offline-report-title">{offlineReport.durationLabel} 동안 방송국이 깨어 있었습니다</h2>
        </div>
        <dl>
          <div><dt>청취자</dt><dd>+{offlineReport.listeners}</dd></div>
          <div><dt>신호</dt><dd>+{offlineReport.signal}%</dd></div>
          <div><dt>평판</dt><dd>+{offlineReport.reputation}</dd></div>
          <div><dt>이야기</dt><dd>+{offlineReport.stories}</dd></div>
        </dl>
        <p>새 사연 {offlineReport.letters.length}통이 책상 위에 쌓였습니다.</p>
        <div class="offline-actions">
          <button type="button" onclick={claimOfflineReport}>방송 기록 확인</button>
          <button type="button" onclick={dismissOfflineReport}>이번 리포트 닫기</button>
        </div>
      </section>
    {/if}

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

    {#if rewardNotice}
      <div class="reward-notice" role="status" aria-live="polite">
        <span>소장품 해금</span>
        <strong>{rewardNotice.title}</strong>
        <small>{rewardNotice.description}</small>
      </div>
    {/if}

    {#if collectionSetNotice}
      <div class="set-notice" role="status" aria-live="polite">
        <span>세트 완성</span>
        <strong>{collectionSetNotice.bonusTitle}</strong>
        <small>{collectionSetNotice.bonusDescription}</small>
      </div>
    {/if}

    <div class="collection-summary" aria-label="도감 보상 진행도">
      <div>
        <span>도감 보상</span>
        <strong>{unlockedRewards.length}/{collectionRewards.length}</strong>
      </div>
      <p>완성된 사연 묶음 {completedStoryPacks.length}/{storyPacks.length}개 · 세트 {unlockedCollectionSets.length}/{collectionSets.length}개</p>
      {#if nextCollectionReward}
        <small>다음 보상: {nextCollectionReward.title} · {nextCollectionReward.preview}</small>
        <span class="progress-track" aria-hidden="true"><span style={`width: ${rewardProgress(nextCollectionReward)}%`}></span></span>
        {#if nextCollectionSet}
          <small>세트 목표: {nextCollectionSet.title} · {collectionSetProgress(nextCollectionSet)}%</small>
        {/if}
      {:else if nextCollectionSet}
        <small>다음 세트: {nextCollectionSet.title} · {nextCollectionSet.bonusTitle}</small>
        <span class="progress-track" aria-hidden="true"><span style={`width: ${collectionSetProgress(nextCollectionSet)}%`}></span></span>
      {:else}
        <small>현재 준비된 모든 소장품과 세트 보상이 방송국에 놓였습니다.</small>
      {/if}
    </div>

    <section class="ambience-panel" aria-labelledby="ambience-title">
      <div class="ambience-header">
        <div>
          <p class="eyebrow">alive broadcast room</p>
          <h2 id="ambience-title">{currentRoomAmbience.label}</h2>
        </div>
        <strong>신호 +{totalRoomSignalBonus} · 청취자 +{totalRoomListenerBonus}</strong>
      </div>
      <p>{currentRoomAmbience.description}</p>
      <div class="ambience-list" role="list" aria-label="방송국 생동감 효과">
        {#each activeKeepsakeSynergies as synergy (synergy.id)}
          <div class="ambience-chip" role="listitem">
            <span>소장품 동조</span>
            <strong>{synergy.title}</strong>
            <small>{synergy.description}</small>
          </div>
        {/each}
        {#each activeVisitorTraces as trace (trace.characterId)}
          <div class="ambience-chip" role="listitem">
            <span>방문 흔적</span>
            <strong>{trace.title}</strong>
            <small>{trace.description}</small>
          </div>
        {/each}
        {#if activeKeepsakeSynergies.length === 0 && activeVisitorTraces.length === 0}
          <div class="ambience-chip" role="listitem">
            <span>다음 변화</span>
            <strong>소장품 배치와 청취자 관계 완성</strong>
            <small>방송국에 놓인 물건과 완성된 청취자 기록이 늘면 방 분위기가 살아납니다.</small>
          </div>
        {/if}
      </div>
    </section>

    <section class="placement-panel" aria-labelledby="placement-title">
      <div class="placement-header">
        <div>
          <p class="eyebrow">broadcast room layout</p>
          <h2 id="placement-title">방송국 소장품 배치</h2>
        </div>
        <strong>{occupiedRoomPlacementCount}/{roomPlacementSlots.length}</strong>
      </div>
      <p>
        배치 보너스: 새 사연 신호 +{roomPlacementSignalBonus}, 청취자 +{roomPlacementListenerBonus}
        {#if nextUnplacedReward}
          · 다음 배치 후보: {nextUnplacedReward.title}
        {:else if unlockedRewards.length === 0}
          · 사연 묶음 완성 후 소장품을 배치할 수 있습니다.
        {:else}
          · 해금된 소장품이 모두 방송국에 놓였습니다.
        {/if}
      </p>
      <div class="placement-grid" role="list" aria-label="소장품 배치 슬롯">
        {#each roomPlacementSlots as slot (slot.id)}
          <div class:filled={Boolean(roomPlacementSlotReward(slot))} class="placement-card" role="listitem">
            <div>
              <span>{slot.title}</span>
              <strong>{roomPlacementSlotReward(slot)?.title ?? slot.emptyLabel}</strong>
            </div>
            <p>{roomPlacementSlotReward(slot)?.description ?? slot.description}</p>
            <div class="placement-actions" aria-label={`${slot.title} 소장품 선택`}>
              {#if unlockedRewards.length === 0}
                <small>아직 배치 가능한 소장품이 없습니다.</small>
              {:else}
                {#each unlockedRewards as reward (reward.id)}
                  <button
                    type="button"
                    class:active={roomPlacementSlotReward(slot)?.id === reward.id}
                    aria-pressed={roomPlacementSlotReward(slot)?.id === reward.id}
                    disabled={roomPlacementSlotReward(slot)?.id === reward.id}
                    onclick={() => placeReward(slot, reward)}
                  >
                    <span>{isRewardPlaced(reward) ? "이동" : "배치"}</span>
                    {reward.title}
                  </button>
                {/each}
                {#if roomPlacementSlotReward(slot)}
                  <button type="button" class="clear-placement" onclick={() => clearRoomPlacement(slot)}>비우기</button>
                {/if}
              {/if}
            </div>
          </div>
        {/each}
      </div>
    </section>

    <div class="set-gallery" role="list" aria-label="소장품 세트 도감">
      {#each collectionSets as set (set.id)}
        <div class:complete={isCollectionSetUnlocked(set)} class="set-card" role="listitem">
          <div>
            <span>{isCollectionSetUnlocked(set) ? "완성" : "수집 중"}</span>
            <strong>{set.title}</strong>
          </div>
          <p>{set.description}</p>
          <small>{isCollectionSetUnlocked(set) ? set.bonusDescription : `${collectionSetUnlockedRewards(set).length}/${collectionSetRewards(set).length}개 소장품 수집 · 다음: ${nextCollectionSetMissingReward(set)?.title ?? set.bonusTitle}`}</small>
          <span class="progress-track" aria-hidden="true"><span style={`width: ${collectionSetProgress(set)}%`}></span></span>
        </div>
      {/each}
    </div>

    <div class="reward-gallery" role="list" aria-label="소장품 도감">
      {#each collectionRewards as reward (reward.id)}
        <div class:unlocked={isRewardUnlocked(reward)} class="reward-card" role="listitem">
          <div>
            <span>{isRewardUnlocked(reward) ? "해금" : "예고"}</span>
            <strong>{reward.title}</strong>
          </div>
          <p>{isRewardUnlocked(reward) ? reward.description : `${rewardPackTitle(reward)} 완성 시 해금`}</p>
          <small>{reward.preview}</small>
          <span class="progress-track" aria-hidden="true"><span style={`width: ${rewardProgress(reward)}%`}></span></span>
        </div>
      {/each}
    </div>

    <div class="character-gallery" role="list" aria-label="청취자 도감">
      {#each characterIds as characterId (characterId)}
        <div class:complete={characterReceivedCount(characterId) === characterLetters(characterId).length} class="character-card" role="listitem">
          <div>
            <span>{characterReceivedCount(characterId) === characterLetters(characterId).length ? "완성" : "수집 중"}</span>
            <strong>{characterName(characterId)}</strong>
          </div>
          <p>{characterRelationStage(characterId)}</p>
          <small>연결 사연 {characterReceivedCount(characterId)}/{characterLetters(characterId).length} · {characterAlbumHint(characterId)}</small>
          <span class="progress-track" aria-hidden="true"><span style={`width: ${characterProgress(characterId)}%`}></span></span>
        </div>
      {/each}
    </div>

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
        <button type="button" class:active={selectedLetter.id === letter.id} aria-pressed={selectedLetter.id === letter.id} onclick={() => (selectedLetter = letter)}>
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
      <div class="listener-album-panel" role="group" aria-label={`${selectedLetter.author} 청취자 앨범`}>
        <div>
          <span>{characterRelationStage(selectedLetter.characterId)}</span>
          <strong>{characterProgress(selectedLetter.characterId)}%</strong>
        </div>
        <p>{characterAlbumHint(selectedLetter.characterId)}</p>
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
    --band-accent: #ffcf91;
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

  .pixel-scene.ambience-warm .studio-room {
    background:
      linear-gradient(90deg, rgba(249, 223, 143, 0.08) 0 4px, transparent 4px 100%),
      linear-gradient(#422c45 0 66%, #2f2135 66% 100%);
  }

  .pixel-scene.ambience-alive .studio-room {
    background:
      linear-gradient(90deg, rgba(249, 223, 143, 0.1) 0 4px, transparent 4px 100%),
      radial-gradient(circle at 76% 44%, rgba(249, 223, 143, 0.2), transparent 58px),
      linear-gradient(#48304c 0 66%, #32233a 66% 100%);
  }

  .pixel-scene.has-synergy .wall-light {
    box-shadow: 0 0 0 4px #4b3149, 0 0 calc(var(--scene-glow) + 12px) #f9df8f;
  }

  .pixel-scene.has-visitor-traces .album-board {
    box-shadow: 0 0 12px rgba(158, 208, 188, 0.42);
  }

  .pixel-scene.band-rooftop .scene-sky {
    background: linear-gradient(#102637 0 42%, #2a3f37 42% 100%);
  }

  .pixel-scene.band-hidden-city .scene-sky {
    background: linear-gradient(#120f2d 0 42%, #35204d 42% 100%);
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
    background: var(--band-accent);
    box-shadow: 0 0 var(--scene-glow) var(--band-accent);
    opacity: var(--light-opacity);
  }

  .city-activity {
    position: absolute;
    right: 16px;
    bottom: 198px;
    left: 20px;
    height: 18px;
    pointer-events: none;
  }

  .city-activity span {
    position: absolute;
    bottom: 0;
    width: 16px;
    height: 4px;
    background: #27365b;
    opacity: 0.28;
  }

  .city-activity span:nth-child(1) { left: 18px; }
  .city-activity span:nth-child(2) { left: 92px; }
  .city-activity span:nth-child(3) { right: 96px; }
  .city-activity span:nth-child(4) { right: 20px; }

  .pixel-scene.city-level-1 .city-activity span:nth-child(-n + 1),
  .pixel-scene.city-level-2 .city-activity span:nth-child(-n + 3),
  .pixel-scene.city-level-3 .city-activity span {
    height: 10px;
    background: var(--band-accent);
    box-shadow: 0 0 12px var(--band-accent);
    opacity: 0.86;
  }

  .ambient-trail {
    position: absolute;
    right: 18px;
    bottom: 184px;
    left: 18px;
    height: 34px;
    pointer-events: none;
  }

  .ambient-trail span {
    position: absolute;
    height: 4px;
    background: var(--band-accent);
    opacity: 0.42;
    animation: ambient-sweep 4.4s steps(4, end) infinite;
  }

  .ambient-trail span:nth-child(1) { top: 4px; left: 8px; width: 34px; }
  .ambient-trail span:nth-child(2) { top: 15px; right: 46px; width: 46px; animation-delay: 0.8s; }
  .ambient-trail span:nth-child(3) { top: 26px; left: 120px; width: 28px; animation-delay: 1.6s; }

  .pixel-scene.band-rooftop .ambient-trail span {
    width: 4px;
    height: 14px;
    animation-name: rain-fall;
  }

  .pixel-scene.band-hidden-city .ambient-trail span {
    height: 8px;
    animation-name: hidden-flicker;
  }

  .character-echo {
    position: absolute;
    left: 26px;
    bottom: 196px;
    width: 54px;
    height: 30px;
    pointer-events: none;
  }

  .character-echo span {
    position: absolute;
    bottom: 0;
    background: var(--band-accent);
    opacity: 0.36;
  }

  .character-echo span:nth-child(1) { left: 4px; width: 28px; height: 8px; }
  .character-echo span:nth-child(2) { left: 28px; width: 12px; height: 14px; }
  .character-echo span:nth-child(3) { right: 0; width: 8px; height: 8px; }

  .character-echo.gardener-haerin span:nth-child(1),
  .character-echo.rooftop-dalsoo span:nth-child(1),
  .character-echo.sleepless-yeon span:nth-child(1) {
    width: 18px;
    height: 14px;
    background: #4f8f80;
  }

  .character-echo.hidden-city-listener span,
  .character-echo.repair-seoho span {
    background: #b99cff;
    opacity: 0.55;
  }

  .character-echo.bridge-sora span:nth-child(3),
  .character-echo.taxi-minu span:nth-child(3) {
    background: #ff6b4a;
    opacity: 0.8;
  }

  .scene-milestone {
    position: absolute;
    top: 86px;
    left: 18px;
    max-width: 148px;
    border: 3px solid #442638;
    background: #15111d;
    color: var(--band-accent);
    padding: 0.25rem;
    font-size: 0.62rem;
    line-height: 1.2;
    pointer-events: none;
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
    border: 3px solid var(--band-accent);
    animation: signal-flicker 2.4s steps(2, end) infinite;
  }

  .signal-rings span:nth-child(1) { inset: 0; }
  .signal-rings span:nth-child(2) { inset: 11px; animation-delay: 0.2s; }
  .signal-rings span:nth-child(3) { inset: 22px; animation-delay: 0.4s; }

  .pixel-scene.signal-clear .signal-rings span {
    border-color: var(--band-accent);
  }

  .band-prop {
    position: absolute;
    right: 22px;
    bottom: 194px;
    width: 68px;
    height: 28px;
    pointer-events: none;
  }

  .band-prop span {
    position: absolute;
    background: var(--band-accent);
  }

  .band-prop.alley span:nth-child(1) { right: 6px; bottom: 3px; width: 30px; height: 10px; }
  .band-prop.alley span:nth-child(2) { right: 32px; bottom: 9px; width: 14px; height: 8px; }
  .band-prop.alley span:nth-child(3) { right: 0; bottom: 0; width: 8px; height: 8px; background: #ff6b4a; }

  .band-prop.rooftop span:nth-child(1) { right: 12px; bottom: 0; width: 34px; height: 8px; background: #4f8f80; }
  .band-prop.rooftop span:nth-child(2) { right: 26px; bottom: 8px; width: 8px; height: 18px; }
  .band-prop.rooftop span:nth-child(3) { right: 18px; bottom: 18px; width: 22px; height: 4px; }

  .band-prop.hidden-city span:nth-child(1) { right: 8px; bottom: 6px; width: 42px; height: 18px; border: 3px solid #442638; }
  .band-prop.hidden-city span:nth-child(2) { right: 16px; bottom: 12px; width: 8px; height: 4px; background: #f7e9c7; }
  .band-prop.hidden-city span:nth-child(3) { right: 30px; bottom: 12px; width: 14px; height: 4px; background: #f7e9c7; }

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
    background: var(--band-accent);
    box-shadow: 0 0 0 4px #4b3149, 0 0 var(--scene-glow) var(--band-accent);
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

  .album-board {
    display: grid;
    position: absolute;
    top: 116px;
    left: 24px;
    grid-template-columns: repeat(5, 6px);
    gap: 6px;
    width: 76px;
    height: 34px;
    border: 3px solid #4b3149;
    padding: 5px 8px;
    background: #20172a;
  }

  .album-board::before {
    position: absolute;
    top: 6px;
    left: 8px;
    width: 60px;
    height: 4px;
    content: "";
    background: #6b3f55;
    box-shadow: 0 12px 0 #6b3f55;
  }

  .album-board span {
    z-index: 1;
    width: 6px;
    height: 6px;
    background: #442638;
  }

  .album-board span.active {
    background: #f1a45f;
  }

  .album-board span.complete {
    background: #f9df8f;
    box-shadow: 0 0 10px rgba(249, 223, 143, 0.8);
  }

  .visitor-notes {
    position: absolute;
    inset: 0;
    pointer-events: none;
  }

  .visitor-notes span {
    position: absolute;
    width: 12px;
    height: 14px;
    border: 2px solid #442638;
    background: #f7e9c7;
    box-shadow: 0 3px 0 #6b3f55;
  }

  .visitor-notes .trace-taxi { top: 154px; left: 28px; background: #f9df8f; }
  .visitor-notes .trace-leaf { top: 70px; right: 78px; background: #9ed0bc; }
  .visitor-notes .trace-hidden { top: 126px; left: 82px; background: #b99cff; }
  .visitor-notes .trace-store { right: 58px; bottom: 48px; background: #f1a45f; }
  .visitor-notes .trace-bridge { top: 40px; right: 34px; background: #ffcf91; }
  .visitor-notes .trace-water { top: 92px; right: 44px; background: #8bd7a4; }
  .visitor-notes .trace-ribbon { top: 78px; right: 116px; background: #f9df8f; }
  .visitor-notes .trace-screw { right: 128px; bottom: 66px; background: #c7a77b; }
  .visitor-notes .trace-note { top: 150px; left: 66px; background: #ead7ad; }

  .shelf {
    position: absolute;
    top: 84px;
    right: 26px;
    width: 92px;
    height: 10px;
    background: #7a4b4f;
  }

  .pixel-scene.has-archive-lamp .shelf::after {
    position: absolute;
    right: 8px;
    bottom: -16px;
    width: 36px;
    height: 8px;
    content: "";
    background: #f9df8f;
    box-shadow: 0 0 var(--scene-glow) rgba(249, 223, 143, 0.82);
  }

  .souvenir {
    display: inline-block;
    position: relative;
    width: 14px;
    height: 22px;
    margin-left: 8px;
    transform: translateY(-20px);
    background: #b8675c;
  }

  .souvenir.placeholder {
    height: 14px;
    background: #442638;
    opacity: 0.42;
  }

  .souvenir.ticket {
    width: 20px;
    height: 12px;
    border: 3px solid #442638;
    background: #f9df8f;
  }

  .souvenir.ticket::after {
    position: absolute;
    top: 2px;
    left: 7px;
    width: 3px;
    height: 4px;
    content: "";
    background: #7f383e;
  }

  .souvenir.flower {
    width: 14px;
    height: 24px;
    background: #4f8f80;
  }

  .souvenir.flower::after {
    position: absolute;
    top: -7px;
    left: 3px;
    width: 8px;
    height: 8px;
    content: "";
    background: #f9df8f;
    box-shadow: 0 0 var(--scene-glow) #f1a45f;
  }

  .placement-stage {
    position: absolute;
    inset: 0;
    pointer-events: none;
  }

  .room-keepsake {
    position: absolute;
    width: 28px;
    height: 28px;
    border: 3px solid rgba(75, 49, 73, 0.72);
    background: rgba(21, 17, 29, 0.48);
  }

  .room-keepsake.filled {
    border-color: #f9df8f;
    background: rgba(249, 223, 143, 0.1);
    box-shadow: 0 0 12px rgba(249, 223, 143, 0.36);
  }

  .room-keepsake .souvenir {
    position: absolute;
    bottom: 2px;
    left: 4px;
    margin: 0;
    transform: none;
  }

  .room-keepsake .souvenir.ticket {
    bottom: 8px;
    left: 2px;
  }

  .room-keepsake .souvenir.flower {
    bottom: 2px;
    left: 7px;
  }

  .room-keepsake.desk-slot {
    right: 162px;
    bottom: 50px;
  }

  .room-keepsake.shelf-slot {
    top: 102px;
    right: 72px;
  }

  .room-keepsake.window-slot {
    top: 72px;
    right: 126px;
  }

  .synergy-glow {
    position: absolute;
    width: 58px;
    height: 18px;
    background: #f9df8f;
    opacity: 0.72;
    box-shadow: 0 0 calc(var(--scene-glow) + 16px) rgba(249, 223, 143, 0.86);
    animation: light-breathe 2.4s steps(3, end) infinite;
  }

  .synergy-glow.synergy-dawn {
    top: 104px;
    right: 54px;
  }

  .pixel-scene.has-placements .studio-room {
    box-shadow: inset 0 0 0 3px rgba(249, 223, 143, 0.08);
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

  .offline-mail-stack {
    position: absolute;
    right: 74px;
    bottom: 54px;
    width: 42px;
    height: 34px;
    animation: mail-stack-glow 2.2s steps(2, end) infinite;
  }

  .offline-mail-stack span {
    position: absolute;
    right: 0;
    width: 30px;
    height: 10px;
    border: 3px solid #442638;
    background: #f7e9c7;
  }

  .offline-mail-stack span:nth-child(1) { bottom: 0; }
  .offline-mail-stack span:nth-child(2) { right: 6px; bottom: 9px; background: #ffcf91; }
  .offline-mail-stack span:nth-child(3) { right: 12px; bottom: 18px; background: var(--band-accent); }

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
  .offline-report > div > span,
  .offline-report p,
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

  .frequency-panel {
    margin: 0 0 0.55rem;
    border: 3px solid #4b3149;
    background: linear-gradient(90deg, rgba(249, 223, 143, 0.05), transparent 48%), #15111d;
    padding: 0.55rem;
  }

  .frequency-panel h2 {
    margin-bottom: 0.25rem;
  }

  .frequency-panel p {
    margin-bottom: 0.45rem;
    color: #ead7ad;
    font-size: 0.74rem;
    line-height: 1.45;
  }

  .frequency-options {
    display: grid;
    grid-template-columns: repeat(3, minmax(0, 1fr));
    gap: 0.35rem;
  }

  .frequency-options button {
    border: 3px solid #6b3f55;
    color: #f7e9c7;
    background: #2a1c2f;
    cursor: pointer;
    padding: 0.4rem;
    text-align: left;
  }

  .frequency-options button.active {
    border-color: var(--band-accent);
    box-shadow: inset 0 0 0 2px var(--band-accent);
    background: #3a263f;
  }

  .frequency-options span,
  .frequency-options small {
    display: block;
  }

  .frequency-options span {
    color: var(--band-accent);
    font-weight: 700;
  }

  .frequency-options small {
    margin-top: 0.2rem;
    color: #c7a77b;
    font-size: 0.65rem;
    line-height: 1.25;
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
  .offline-actions,
  .letter-list,
  .pack-status,
  .pack-collection,
  .ambience-list,
  .placement-grid,
  .placement-actions,
  .set-gallery,
  .reward-gallery,
  .character-gallery {
    display: grid;
    gap: 0.45rem;
  }

  .actions button,
  .offline-actions button,
  .placement-actions button,
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
  .offline-actions button,
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
  .offline-report,
  .unlock-notice,
  .completion-notice,
  .reward-notice,
  .set-notice,
  .collection-summary,
  .ambience-panel,
  .placement-panel,
  .set-gallery,
  .reward-gallery,
  .character-gallery,
  .pack-collection {
    margin-bottom: 0.45rem;
  }

  .pack-status,
  .offline-report,
  .unlock-notice,
  .completion-notice,
  .reward-notice,
  .set-notice,
  .collection-summary,
  .ambience-panel,
  .ambience-chip,
  .placement-panel,
  .placement-card,
  .pack-card,
  .set-card,
  .reward-card,
  .character-card {
    border: 3px solid #4b3149;
    background: #15111d;
    color: #c7a77b;
    padding: 0.45rem;
    font-size: 0.72rem;
    line-height: 1.45;
  }

  .offline-report {
    border-color: #f1a45f;
    color: #ffcf91;
    background:
      repeating-linear-gradient(90deg, rgba(249, 223, 143, 0.08) 0 4px, transparent 4px 12px),
      #2a1c2f;
  }

  .offline-report h2 {
    margin: 0.2rem 0 0.45rem;
  }

  .offline-report dl {
    display: grid;
    grid-template-columns: repeat(4, minmax(0, 1fr));
    gap: 0.35rem;
    margin: 0 0 0.45rem;
  }

  .offline-report dl div {
    border: 2px solid #6b3f55;
    background: #15111d;
    padding: 0.35rem;
  }

  .offline-report p {
    margin-bottom: 0.45rem;
  }

  .offline-actions {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .unlock-notice,
  .completion-notice,
  .reward-notice,
  .set-notice {
    border-color: #f1a45f;
    color: #ffcf91;
    background: #2a1c2f;
  }

  .completion-notice,
  .reward-notice,
  .set-notice {
    border-color: #f9df8f;
    box-shadow: inset 0 0 0 2px #4f8f80;
  }

  .set-notice {
    background:
      repeating-linear-gradient(90deg, rgba(79, 143, 128, 0.16) 0 4px, transparent 4px 12px),
      #2a1c2f;
  }

  .unlock-notice span,
  .unlock-notice strong,
  .unlock-notice small,
  .completion-notice span,
  .completion-notice strong,
  .completion-notice small,
  .reward-notice span,
  .reward-notice strong,
  .reward-notice small,
  .set-notice span,
  .set-notice strong,
  .set-notice small,
  .collection-summary span,
  .collection-summary strong,
  .collection-summary small,
  .pack-card span,
  .pack-card strong,
  .pack-card small,
  .set-card span,
  .set-card strong,
  .set-card small,
  .reward-card span,
  .reward-card strong,
  .reward-card small,
  .character-card span,
  .character-card strong,
  .character-card small {
    display: block;
  }

  .collection-summary div,
  .ambience-header,
  .placement-header,
  .placement-card > div,
  .pack-card div,
  .set-card div,
  .reward-card div,
  .character-card div {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 0.45rem;
  }

  .ambience-panel {
    border-color: #f9df8f;
    background:
      radial-gradient(circle at 86% 18%, rgba(249, 223, 143, 0.16), transparent 52px),
      linear-gradient(90deg, rgba(185, 156, 255, 0.1), transparent 62%),
      #15111d;
  }

  .ambience-header h2,
  .placement-header h2 {
    margin-bottom: 0;
  }

  .ambience-header strong {
    color: #ffcf91;
    font-size: 0.72rem;
    text-align: right;
  }

  .ambience-panel > p {
    margin: 0.3rem 0 0.45rem;
  }

  .ambience-chip {
    background:
      linear-gradient(90deg, rgba(249, 223, 143, 0.08), transparent 58%),
      #20172a;
  }

  .ambience-chip span,
  .ambience-chip strong,
  .ambience-chip small {
    display: block;
  }

  .ambience-chip span {
    color: #9ed0bc;
  }

  .ambience-chip strong {
    color: #ffcf91;
  }

  .placement-panel {
    background:
      linear-gradient(90deg, rgba(79, 143, 128, 0.12), transparent 62%),
      #15111d;
  }

  .placement-header h2 {
    margin-bottom: 0;
  }

  .placement-header strong,
  .placement-card.filled strong {
    color: #ffcf91;
  }

  .placement-panel > p,
  .placement-card p {
    margin: 0.3rem 0 0.45rem;
  }

  .placement-card.filled {
    border-color: #f9df8f;
    box-shadow: inset 0 0 0 2px rgba(79, 143, 128, 0.45);
  }

  .placement-actions {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .placement-actions button {
    padding: 0.4rem;
    font-size: 0.68rem;
  }

  .placement-actions button.active {
    border-color: #f9df8f;
    color: #ffcf91;
    background: #3a263f;
  }

  .placement-actions button:disabled {
    cursor: not-allowed;
    opacity: 0.68;
  }

  .placement-actions span,
  .placement-actions small {
    display: block;
  }

  .placement-actions span {
    color: #9ed0bc;
    font-size: 0.62rem;
  }

  .clear-placement {
    color: #c7a77b;
  }

  .collection-summary strong {
    color: #ffcf91;
    font-size: 1rem;
  }

  .pack-card.unlocked,
  .reward-card.unlocked {
    border-color: #4f8f80;
  }

  .pack-card.complete,
  .set-card.complete,
  .character-card.complete {
    border-color: #f9df8f;
    color: #ffcf91;
  }

  .set-card.complete {
    background:
      linear-gradient(90deg, rgba(249, 223, 143, 0.12), transparent 56%),
      #15111d;
  }

  .reward-card.unlocked {
    color: #9ed0bc;
  }

  .collection-summary p,
  .pack-card p,
  .set-card p,
  .reward-card p,
  .character-card p {
    margin: 0.3rem 0;
  }

  .character-card p {
    color: #ead7ad;
    font-weight: 700;
  }

  .actions button:hover:not(:disabled),
  .offline-actions button:hover,
  .placement-actions button:hover:not(:disabled),
  .frequency-options button:hover,
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

  .listener-album-panel {
    margin: 0.55rem 0;
    border: 3px solid #4b3149;
    padding: 0.45rem;
    background:
      linear-gradient(90deg, rgba(79, 143, 128, 0.12), transparent 60%),
      #15111d;
  }

  .listener-album-panel div {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 0.45rem;
    color: #ffcf91;
    font-weight: 700;
  }

  .listener-album-panel p {
    margin: 0.3rem 0 0;
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

  @keyframes ambient-sweep {
    0%, 100% { transform: translateX(0); opacity: 0.18; }
    50% { transform: translateX(10px); opacity: 0.68; }
  }

  @keyframes rain-fall {
    0%, 100% { transform: translateY(-4px); opacity: 0.22; }
    50% { transform: translateY(8px); opacity: 0.72; }
  }

  @keyframes hidden-flicker {
    0%, 100% { opacity: 0.18; }
    25% { opacity: 0.74; }
    50% { opacity: 0.28; }
    75% { opacity: 0.92; }
  }

  @keyframes mail-stack-glow {
    0%, 100% { filter: brightness(0.8); }
    50% { filter: brightness(1.25); }
  }

  @media (prefers-reduced-motion: reduce) {
    .wall-light,
    .signal-rings span,
    .host,
    .ambient-trail span,
    .radio-wave,
    .letter-flag,
    .offline-mail-stack,
    .synergy-glow,
    .on-air span {
      animation: none;
    }
  }

  @media (max-width: 380px) {
    .station-shell {
      padding: 0.5rem;
    }

    .metrics-grid,
    .frequency-options {
      grid-template-columns: repeat(2, minmax(0, 1fr));
    }
  }
</style>

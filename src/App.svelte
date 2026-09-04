<script>
  import { onMount } from "svelte";

  const DEFAULT_CARDS = [
    { id: "swsh4-25", name: "Pikachu VMAX", frontImg: "https://images.pokemontcg.io/swsh4/25_hires.png" },
    { id: "swsh9-TG05", name: "Articuno", frontImg: "https://images.pokemontcg.io/swsh9/TG05_hires.png" },
    { id: "swsh9-TG06", name: "Zapdos", frontImg: "https://images.pokemontcg.io/swsh9/TG06_hires.png" },
    { id: "swsh9-TG08", name: "Morpeko", frontImg: "https://images.pokemontcg.io/swsh9/TG08_hires.png" },
    { id: "swsh10-027", name: "Radiant Charizard", frontImg: "https://images.pokemontcg.io/swsh10/27_hires.png" },
    { id: "swsh9-TG03", name: "Charizard", frontImg: "https://images.pokemontcg.io/swsh9/TG03_hires.png" },
    { id: "swsh7-111", name: "Rayquaza V", frontImg: "https://images.pokemontcg.io/swsh7/111_hires.png" },
    { id: "swsh8-069", name: "Mew V", frontImg: "https://images.pokemontcg.io/swsh8/69_hires.png" },
    { id: "swsh7-TG01", name: "Flareon V", frontImg: "https://images.pokemontcg.io/swsh7/TG01_hires.png" },
    { id: "swsh7-029", name: "Gyarados VMAX", frontImg: "https://images.pokemontcg.io/swsh7/29_hires.png" },
    { id: "swsh7-065", name: "Espeon VMAX", frontImg: "https://images.pokemontcg.io/swsh7/65_hires.png" },
    { id: "swsh9-TG14", name: "Mewtwo VSTAR", frontImg: "https://images.pokemontcg.io/swsh9/TG14_hires.png" },
    { id: "swsh6-196", name: "Peonia", frontImg: "https://images.pokemontcg.io/swsh6/196_hires.png" },
    { id: "swsh4-188", name: "Pikachu VMAX (Rainbow)", frontImg: "https://images.pokemontcg.io/swsh4/188_hires.png" },
    { id: "swsh2-174", name: "Twin Energy", frontImg: "https://images.pokemontcg.io/swsh2/174_hires.png" },
    { id: "swsh9-TG16", name: "Mimikyu V", frontImg: "https://images.pokemontcg.io/swsh9/TG16_hires.png" },
    { id: "swsh8-TG11", name: "Celebi", frontImg: "https://images.pokemontcg.io/swsh8/TG11_hires.png" },
    { id: "swsh45sv-SV090", name: "Minccino", frontImg: "https://images.pokemontcg.io/swsh45sv/SV090_hires.png" },
    { id: "swsh3-023", name: "Charmander", frontImg: "https://images.pokemontcg.io/swsh3/23_hires.png" },
    { id: "swsh4-116", name: "Dedenne", frontImg: "https://images.pokemontcg.io/swsh4/116_hires.png" },
    { id: "swsh9-014", name: "Squirtle", frontImg: "https://images.pokemontcg.io/swsh9/14_hires.png" }
  ];

  let cards = [];
  let isLoading = true;

  let isMenuOpen = true;

  // 3D 회전 물리 엔진 변수
  let currentRotation = 0;
  let targetRotation = 0;
  let rotationVelocity = 0;

  let isAutoRotating = false;

  // 강조 모드: 'off' | 'smooth' | 'snap'
  let highlightMode = "snap";

  // 정면 강조 세부 옵션 상태
  let showHighlightSettings = false;
  let snapEntryAngle = 7.5;
  let snapExitAngle = 5.0;
  let snapScale = 1.09;
  let snapPopZ = 70;
  let snapPopY = 32;

  // 카드 두께감 및 엣지 곡률 / 입체 단면 음영 세부 설정
  let showThicknessSettings = false;
  let cardThickness = 2.4;
  let cardRadius = 14.4;
  let edgeDepthShade = 0.65;

  // 바닥 거울 반사 세부 설정
  let showFloorSettings = false;
  let enableReflection = true;
  let reflectionOpacity = 0.28;
  let reflectionBlur = 3.5;
  let reflectionOffsetY = 0;

  // 바닥 그림자 세부 설정
  let enableShadow = true;
  let shadowBlur = 16;
  let shadowOpacity = 0.6;
  let shadowOffsetX = 0;
  let shadowOffsetY = 0;

  // 3D 조명 제어 설정
  let showLightSettings = false;
  let lightAngle = 45;
  let lightElevation = 50;
  let lightIntensity = 1.0;

  // 스냅 햅틱 사운드
  let soundEnabled = true;
  let soundVolume = 0.55;
  let audioCtx = null;
  let lastSnappedIndex = -1;

  // 인플레이스 카드 확대/플립 상태
  let flippedCardId = null;

  // 카메라 시선
  const DEFAULT_PITCH = 0;
  let currentCameraPitch = DEFAULT_PITCH;
  let targetCameraPitch = DEFAULT_PITCH;
  let currentCameraHeight = 0;
  let targetCameraHeight = 0;

  // 화각 (Perspective)
  const DEFAULT_PERSPECTIVE = 2100;
  let currentPerspective = DEFAULT_PERSPECTIVE;
  let targetPerspective = DEFAULT_PERSPECTIVE;

  // 3D 링 반지름 (카드 간격)
  const DEFAULT_RADIUS = 1060;
  let currentRadius = DEFAULT_RADIUS;
  let targetRadius = DEFAULT_RADIUS;

  // 바닥 그림자 기본 깊이
  const DEFAULT_SHADOW_DEPTH = 240;
  let currentShadowDepth = DEFAULT_SHADOW_DEPTH;
  let targetShadowDepth = DEFAULT_SHADOW_DEPTH;

  // 마우스 드래그 상태
  let isDragging = false;
  let isMiddleDragging = false;
  let startX = 0;
  let startY = 0;
  let previousX = 0;
  let previousY = 0;
  let dragVelocity = 0;
  let totalDragDistance = 0;

  // Zoom
  let currentZoom = 1.0;
  let targetZoom = 1.0;

  // 키보드 상태 및 회전 속도
  let isLeftPressed = false;
  let isRightPressed = false;
  let isUpPressed = false;
  let isDownPressed = false;

  const DEFAULT_SPEED = 35.0;
  let rotationSpeed = DEFAULT_SPEED;

  // UI 상태
  let showCardManager = false;
  const DEFAULT_BACK = "https://poke-holo.simey.me/img/back.png";
  let customBackImage = DEFAULT_BACK;

  let draggedCardIndex = null;
  let dragOverCardIndex = null;

  // 배경 상태
  let siteBgImage = null;
  let siteBgSize = "cover";
  let bgBaseX = 50;
  let bgBaseY = 50;
  let bgOffsetX = 0;
  let bgOffsetY = 0;
  let bgScale = 1.0; // 배경 이미지 배율 (0.2 ~ 3.0)

  // 녹화 상태
  let isRecording = false;
  let mediaRecorder = null;
  let recordedChunks = [];
  let recordTimer = 0;
  let recordTimerInterval = null;

  const CARD_COUNT = 21;
  const STEP_ANGLE = 360 / CARD_COUNT;

  let cardInteractions = new Map();

  const DB_NAME = "PokeCarouselDB";
  const STORE_NAME = "customData";

  function openDB() {
    return new Promise((resolve, reject) => {
      const req = indexedDB.open(DB_NAME, 1);
      req.onupgradeneeded = (e) => {
        const db = e.target.result;
        if (!db.objectStoreNames.contains(STORE_NAME)) {
          db.createObjectStore(STORE_NAME);
        }
      };
      req.onsuccess = () => resolve(req.result);
      req.onerror = () => reject(req.error);
    });
  }

  async function saveToDB(key, val) {
    try {
      const db = await openDB();
      const tx = db.transaction(STORE_NAME, "readwrite");
      tx.objectStore(STORE_NAME).put(val, key);
    } catch (err) {
      console.error("DB Save failed:", err);
    }
  }

  async function getFromDB(key) {
    try {
      const db = await openDB();
      return new Promise((resolve) => {
        const tx = db.transaction(STORE_NAME, "readonly");
        const req = tx.objectStore(STORE_NAME).get(key);
        req.onsuccess = () => resolve(req.result);
        req.onerror = () => resolve(null);
      });
    } catch (err) {
      return null;
    }
  }

  async function clearDB() {
    try {
      const db = await openDB();
      const tx = db.transaction(STORE_NAME, "readwrite");
      tx.objectStore(STORE_NAME).clear();
    } catch (err) {
      console.error("DB Clear failed:", err);
    }
  }

  function playCardSwooshSound() {
    if (!soundEnabled || soundVolume <= 0) return;
    try {
      if (!audioCtx) {
        const AudioContextClass = window.AudioContext || window.webkitAudioContext;
        if (!AudioContextClass) return;
        audioCtx = new AudioContextClass();
      }
      if (audioCtx.state === "suspended") {
        audioCtx.resume();
      }

      const now = audioCtx.currentTime;
      const duration = 0.16;

      const bufferSize = audioCtx.sampleRate * duration;
      const noiseBuffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate);
      const output = noiseBuffer.getChannelData(0);
      let lastOut = 0.0;
      for (let i = 0; i < bufferSize; i++) {
        const white = Math.random() * 2 - 1;
        output[i] = (lastOut + 0.02 * white) / 1.02;
        lastOut = output[i];
        output[i] *= 3.5;
      }

      const noiseSource = audioCtx.createBufferSource();
      noiseSource.buffer = noiseBuffer;

      const sweepFilter = audioCtx.createBiquadFilter();
      sweepFilter.type = "bandpass";
      sweepFilter.Q.setValueAtTime(2.2, now);
      sweepFilter.frequency.setValueAtTime(800, now);
      sweepFilter.frequency.exponentialRampToValueAtTime(2600, now + 0.06);
      sweepFilter.frequency.exponentialRampToValueAtTime(450, now + duration);

      const gainNode = audioCtx.createGain();
      gainNode.gain.setValueAtTime(0.001, now);
      gainNode.gain.linearRampToValueAtTime(soundVolume * 0.7, now + 0.04);
      gainNode.gain.exponentialRampToValueAtTime(0.001, now + duration);

      noiseSource.connect(sweepFilter);
      sweepFilter.connect(gainNode);
      gainNode.connect(audioCtx.destination);

      noiseSource.start(now);
      noiseSource.stop(now + duration);
    } catch (e) {
      // Audio safe
    }
  }

  function getCardState(id) {
    if (!cardInteractions.has(id)) {
      cardInteractions.set(id, {
        currentX: 50, currentY: 50,
        targetX: 50, targetY: 50,
        currentRx: 0, currentRy: 0,
        targetRx: 0, targetRy: 0,
        currentOp: 0, targetOp: 0,
        el: null
      });
    }
    return cardInteractions.get(id);
  }

  function registerCard(node, cardId) {
    const state = getCardState(cardId);
    state.el = node;
    return {
      update(newId) {
        const newState = getCardState(newId);
        newState.el = node;
      },
      destroy() {
        if (cardInteractions.has(cardId)) {
          cardInteractions.get(cardId).el = null;
        }
      }
    };
  }

  function getFrontAlignment(index, rotation) {
    if (highlightMode === "off") return 0;
    const cardAngle = (index * STEP_ANGLE + rotation) % 360;
    let normalized = ((cardAngle % 360) + 360) % 360;
    if (normalized > 180) normalized = 360 - normalized;

    if (highlightMode === "snap") {
      if (normalized <= snapExitAngle) return 1.0;
      if (normalized <= snapEntryAngle) {
        const span = snapEntryAngle - snapExitAngle;
        if (span <= 0) return 1.0;
        return 1.0 - (normalized - snapExitAngle) / span;
      }
      return 0;
    } else {
      const threshold = 18;
      if (normalized < threshold) {
        return Math.cos((normalized / threshold) * (Math.PI / 2));
      }
      return 0;
    }
  }

  function optimizeImage(file) {
    return new Promise((resolve) => {
      const reader = new FileReader();
      reader.onload = (e) => {
        resolve(e.target.result);
      };
      reader.readAsDataURL(file);
    });
  }

  function handleCardClick(cardId) {
    if (totalDragDistance > 6) return;
    flippedCardId = flippedCardId === cardId ? null : cardId;
  }

  function nudgeCardHeight(deltaY) {
    targetCameraHeight = Math.max(-800, Math.min(800, targetCameraHeight + deltaY));
    persistAll();
  }
  function resetCardHeight() {
    targetCameraHeight = 0;
    persistAll();
  }

  function adjustBgScale(delta) {
    bgScale = Math.min(3.0, Math.max(0.2, Number((bgScale + delta).toFixed(2))));
    persistAll();
  }
  function resetBgScale() {
    bgScale = 1.0;
    persistAll();
  }

  const loadCards = async () => {
    try {
      const savedState = await getFromDB("full_app_state");

      if (savedState) {
        if (savedState.cards && savedState.cards.length === CARD_COUNT) {
          cards = savedState.cards;
        } else {
          cards = JSON.parse(JSON.stringify(DEFAULT_CARDS));
        }

        if (savedState.customBackImage) customBackImage = savedState.customBackImage;
        if (savedState.siteBgImage !== undefined) siteBgImage = savedState.siteBgImage;
        if (savedState.siteBgSize) siteBgSize = savedState.siteBgSize;
        if (savedState.bgBaseX !== undefined) bgBaseX = savedState.bgBaseX;
        if (savedState.bgBaseY !== undefined) bgBaseY = savedState.bgBaseY;
        if (savedState.bgOffsetX !== undefined) bgOffsetX = savedState.bgOffsetX;
        if (savedState.bgOffsetY !== undefined) bgOffsetY = savedState.bgOffsetY;
        if (savedState.bgScale !== undefined) bgScale = savedState.bgScale;

        if (savedState.targetPerspective) {
          targetPerspective = savedState.targetPerspective;
          currentPerspective = savedState.targetPerspective;
        }
        if (savedState.targetRadius) {
          targetRadius = savedState.targetRadius;
          currentRadius = savedState.targetRadius;
        }
        if (savedState.targetShadowDepth) {
          targetShadowDepth = savedState.targetShadowDepth;
          currentShadowDepth = savedState.targetShadowDepth;
        }
        if (savedState.targetCameraPitch !== undefined) {
          targetCameraPitch = savedState.targetCameraPitch;
          currentCameraPitch = savedState.targetCameraPitch;
        }
        if (savedState.targetCameraHeight !== undefined) {
          targetCameraHeight = savedState.targetCameraHeight;
          currentCameraHeight = savedState.targetCameraHeight;
        }
        if (savedState.targetZoom) {
          targetZoom = savedState.targetZoom;
          currentZoom = savedState.targetZoom;
        }
        if (savedState.rotationSpeed) rotationSpeed = savedState.rotationSpeed;
        if (savedState.targetRotation !== undefined) {
          targetRotation = savedState.targetRotation;
          currentRotation = savedState.targetRotation;
        }
        if (savedState.isAutoRotating !== undefined) {
          isAutoRotating = savedState.isAutoRotating;
        }
        if (savedState.highlightMode) {
          highlightMode = savedState.highlightMode;
        }
        if (savedState.snapEntryAngle !== undefined) snapEntryAngle = savedState.snapEntryAngle;
        if (savedState.snapExitAngle !== undefined) snapExitAngle = savedState.snapExitAngle;
        if (savedState.snapScale !== undefined) snapScale = savedState.snapScale;
        if (savedState.snapPopZ !== undefined) snapPopZ = savedState.snapPopZ;
        if (savedState.snapPopY !== undefined) snapPopY = savedState.snapPopY;

        if (savedState.cardThickness !== undefined) cardThickness = savedState.cardThickness;
        if (savedState.cardRadius !== undefined) cardRadius = savedState.cardRadius;
        if (savedState.edgeDepthShade !== undefined) edgeDepthShade = savedState.edgeDepthShade;

        if (savedState.enableReflection !== undefined) enableReflection = savedState.enableReflection;
        if (savedState.reflectionOpacity !== undefined) reflectionOpacity = savedState.reflectionOpacity;
        if (savedState.reflectionBlur !== undefined) reflectionBlur = savedState.reflectionBlur;
        if (savedState.reflectionOffsetY !== undefined) reflectionOffsetY = savedState.reflectionOffsetY;

        if (savedState.enableShadow !== undefined) enableShadow = savedState.enableShadow;
        if (savedState.shadowBlur !== undefined) shadowBlur = savedState.shadowBlur;
        if (savedState.shadowOpacity !== undefined) shadowOpacity = savedState.shadowOpacity;
        if (savedState.shadowOffsetX !== undefined) shadowOffsetX = savedState.shadowOffsetX;
        if (savedState.shadowOffsetY !== undefined) shadowOffsetY = savedState.shadowOffsetY;

        if (savedState.lightAngle !== undefined) lightAngle = savedState.lightAngle;
        if (savedState.lightElevation !== undefined) lightElevation = savedState.lightElevation;
        if (savedState.lightIntensity !== undefined) lightIntensity = savedState.lightIntensity;

        if (savedState.soundEnabled !== undefined) soundEnabled = savedState.soundEnabled;
        if (savedState.soundVolume !== undefined) soundVolume = savedState.soundVolume;
      } else {
        cards = JSON.parse(JSON.stringify(DEFAULT_CARDS));
      }

      isLoading = false;
    } catch (err) {
      cards = JSON.parse(JSON.stringify(DEFAULT_CARDS));
      isLoading = false;
    }
  };

  async function persistAll() {
    const fullState = {
      cards,
      customBackImage,
      siteBgImage,
      siteBgSize,
      bgBaseX,
      bgBaseY,
      bgOffsetX,
      bgOffsetY,
      bgScale,
      targetPerspective,
      targetRadius,
      targetShadowDepth,
      targetCameraPitch,
      targetCameraHeight,
      targetZoom,
      rotationSpeed,
      targetRotation,
      isAutoRotating,
      highlightMode,
      snapEntryAngle,
      snapExitAngle,
      snapScale,
      snapPopZ,
      snapPopY,
      cardThickness,
      cardRadius,
      edgeDepthShade,
      enableReflection,
      reflectionOpacity,
      reflectionBlur,
      reflectionOffsetY,
      enableShadow,
      shadowBlur,
      shadowOpacity,
      shadowOffsetX,
      shadowOffsetY,
      lightAngle,
      lightElevation,
      lightIntensity,
      soundEnabled,
      soundVolume
    };
    await saveToDB("full_app_state", fullState);
  }

  async function handleManualSave() {
    await persistAll();
    alert("모든 설정이 안전하게 저장되었습니다!");
  }

  async function handleResetToDefault() {
    if (!confirm("모든 설정을 초기화하고 기본 원본 상태로 복구하시겠습니까?")) return;
    isLoading = true;
    await clearDB();
    customBackImage = DEFAULT_BACK;
    siteBgImage = null;
    siteBgSize = "cover";
    bgBaseX = 50;
    bgBaseY = 50;
    bgOffsetX = 0;
    bgOffsetY = 0;
    bgScale = 1.0;
    targetPerspective = DEFAULT_PERSPECTIVE;
    targetRadius = DEFAULT_RADIUS;
    targetShadowDepth = DEFAULT_SHADOW_DEPTH;
    targetCameraPitch = DEFAULT_PITCH;
    targetCameraHeight = 0;
    targetZoom = 1.0;
    rotationSpeed = DEFAULT_SPEED;
    targetRotation = 0;
    isAutoRotating = false;
    highlightMode = "snap";
    snapEntryAngle = 7.5;
    snapExitAngle = 5.0;
    snapScale = 1.09;
    snapPopZ = 70;
    snapPopY = 32;
    cardThickness = 2.4;
    cardRadius = 14.4;
    edgeDepthShade = 0.65;
    enableReflection = true;
    reflectionOpacity = 0.28;
    reflectionBlur = 3.5;
    reflectionOffsetY = 0;
    enableShadow = true;
    shadowBlur = 16;
    shadowOpacity = 0.6;
    shadowOffsetX = 0;
    shadowOffsetY = 0;
    lightAngle = 45;
    lightElevation = 50;
    lightIntensity = 1.0;
    soundEnabled = true;
    soundVolume = 0.55;
    flippedCardId = null;
    cards = JSON.parse(JSON.stringify(DEFAULT_CARDS));
    isLoading = false;
    alert("기본 원본 설정으로 복구되었습니다.");
  }

  function toggleAutoRotate() {
    isAutoRotating = !isAutoRotating;
    persistAll();
  }

  function selectHighlightMode(mode) {
    highlightMode = highlightMode === mode ? "off" : mode;
    persistAll();
  }

  async function startRecording() {
    if (!navigator.mediaDevices || !navigator.mediaDevices.getDisplayMedia) {
      alert("현재 브라우저 환경에서는 화면 캡처 API를 지원하지 않습니다. 로컬 서버(localhost) 또는 HTTPS 환경에서 실행해 주세요.");
      return;
    }

    try {
      const stream = await navigator.mediaDevices.getDisplayMedia({
        video: true,
        audio: false
      });

      recordedChunks = [];

      let options = {};
      if (MediaRecorder.isTypeSupported("video/webm;codecs=vp9")) {
        options = { mimeType: "video/webm;codecs=vp9" };
      } else if (MediaRecorder.isTypeSupported("video/webm;codecs=vp8")) {
        options = { mimeType: "video/webm;codecs=vp8" };
      } else if (MediaRecorder.isTypeSupported("video/webm")) {
        options = { mimeType: "video/webm" };
      }

      mediaRecorder = new MediaRecorder(stream, options);

      mediaRecorder.ondataavailable = (e) => {
        if (e.data && e.data.size > 0) {
          recordedChunks.push(e.data);
        }
      };

      mediaRecorder.onstop = () => {
        if (recordedChunks.length > 0) {
          const blob = new Blob(recordedChunks, { type: mediaRecorder.mimeType || "video/webm" });
          const url = URL.createObjectURL(blob);
          const a = document.createElement("a");
          a.href = url;
          a.download = `3d-card-carousel-${Date.now()}.webm`;
          a.click();
          URL.revokeObjectURL(url);
        }

        stream.getTracks().forEach((track) => track.stop());
        isRecording = false;
        clearInterval(recordTimerInterval);
        recordTimer = 0;
      };

      stream.getVideoTracks()[0].onended = () => {
        if (isRecording) stopRecording();
      };

      mediaRecorder.start(250);
      isRecording = true;
      recordTimer = 0;
      recordTimerInterval = setInterval(() => {
        recordTimer += 1;
      }, 1000);
    } catch (err) {
      console.warn("녹화 취소 또는 오류:", err);
      isRecording = false;
    }
  }

  function stopRecording() {
    if (mediaRecorder && mediaRecorder.state !== "inactive") {
      mediaRecorder.stop();
    }
  }

  function formatTime(sec) {
    const m = Math.floor(sec / 60).toString().padStart(2, "0");
    const s = (sec % 60).toString().padStart(2, "0");
    return `${m}:${s}`;
  }

  function setFov(val) {
    targetPerspective = val;
    persistAll();
  }
  function adjustFov(delta) {
    targetPerspective = Math.max(900, Math.min(5000, targetPerspective + delta));
    persistAll();
  }

  function adjustRadius(delta) {
    targetRadius = Math.max(600, Math.min(2200, targetRadius + delta));
    persistAll();
  }
  function resetRadius() {
    targetRadius = DEFAULT_RADIUS;
    persistAll();
  }

  function adjustShadowDepth(delta) {
    targetShadowDepth = Math.max(100, Math.min(600, targetShadowDepth + delta));
    persistAll();
  }
  function resetShadowDepth() {
    targetShadowDepth = DEFAULT_SHADOW_DEPTH;
    persistAll();
  }

  function handleDragStart(e, index) {
    draggedCardIndex = index;
    e.dataTransfer.effectAllowed = "move";
  }

  function handleDragOver(e, index) {
    e.preventDefault();
    e.dataTransfer.dropEffect = "move";
    dragOverCardIndex = index;
  }

  function handleDragLeave(index) {
    if (dragOverCardIndex === index) dragOverCardIndex = null;
  }

  function handleDrop(e, targetIndex) {
    e.preventDefault();
    if (draggedCardIndex === null || draggedCardIndex === targetIndex) {
      draggedCardIndex = null;
      dragOverCardIndex = null;
      return;
    }

    const updatedCards = [...cards];
    const [movedCard] = updatedCards.splice(draggedCardIndex, 1);
    updatedCards.splice(targetIndex, 0, movedCard);

    cards = updatedCards;
    draggedCardIndex = null;
    dragOverCardIndex = null;
    persistAll();
  }

  function handleDragEnd() {
    draggedCardIndex = null;
    dragOverCardIndex = null;
  }

  async function handleSingleCardUpload(e, index) {
    const file = e.target.files[0];
    if (!file) return;

    const dataUrl = await optimizeImage(file);
    cards[index].frontImg = dataUrl;
    cards = [...cards];
    persistAll();
  }

  async function handleBatchFrontUpload(e) {
    const files = Array.from(e.target.files);
    if (!files.length) return;

    for (let index = 0; index < files.length; index++) {
      if (index >= CARD_COUNT) break;
      const dataUrl = await optimizeImage(files[index]);
      if (files.length === 1) {
        cards = cards.map((c) => ({ ...c, frontImg: dataUrl }));
        break;
      } else {
        cards[index].frontImg = dataUrl;
        cards = [...cards];
      }
    }
    persistAll();
  }

  async function handleBackUpload(e) {
    const file = e.target.files[0];
    if (!file) return;

    const dataUrl = await optimizeImage(file);
    customBackImage = dataUrl;
    persistAll();
  }

  async function handleSiteBgUpload(e) {
    const file = e.target.files[0];
    if (!file) return;

    const dataUrl = await optimizeImage(file);
    siteBgImage = dataUrl;
    persistAll();
  }

  function resetSiteBg() {
    siteBgImage = null;
    bgBaseX = 50;
    bgBaseY = 50;
    bgOffsetX = 0;
    bgOffsetY = 0;
    bgScale = 1.0;
    persistAll();
  }

  function handleBgPresetChange(e) {
    const [x, y] = e.target.value.split(" ").map(Number);
    bgBaseX = x;
    bgBaseY = y;
    persistAll();
  }

  function nudgeBg(dx, dy) {
    bgOffsetX += dx;
    bgOffsetY += dy;
    persistAll();
  }

  function resetBgOffset() {
    bgOffsetX = 0;
    bgOffsetY = 0;
    persistAll();
  }

  function setCameraPreset(pitch, height) {
    targetCameraPitch = pitch;
    targetCameraHeight = height;
    persistAll();
  }

  function resetCamera() {
    targetCameraPitch = DEFAULT_PITCH;
    targetCameraHeight = 0;
    persistAll();
  }

  function zoomIn() { 
    targetZoom = Math.min(targetZoom + 0.15, 2.0); 
    persistAll();
  }
  function zoomOut() { 
    targetZoom = Math.max(targetZoom - 0.15, 0.5); 
    persistAll();
  }
  function resetZoom() { 
    targetZoom = 1.0; 
    persistAll();
  }

  function speedDown() { 
    rotationSpeed = Math.max(5.0, rotationSpeed - 5.0); 
    persistAll();
  }
  function speedUp() { 
    rotationSpeed = Math.min(80.0, rotationSpeed + 5.0); 
    persistAll();
  }
  function resetSpeed() { 
    rotationSpeed = DEFAULT_SPEED; 
    persistAll();
  }

  function handleWheel(e) {
    if (
      e.target.closest(".manager-modal") || 
      e.target.closest(".highlight-settings-panel") ||
      e.target.closest(".floating-sub-panel")
    ) return;
    if (e.deltaY < 0) {
      targetZoom = Math.min(targetZoom + 0.08, 2.0);
    } else {
      targetZoom = Math.max(targetZoom - 0.08, 0.5);
    }
  }

  function handleKeyDown(e) {
    if (e.key === " " || e.key === "Spacebar") {
      e.preventDefault();
      toggleAutoRotate();
      return;
    }

    if (e.key === "Escape") {
      if (flippedCardId) flippedCardId = null;
      return;
    }

    if (e.key === "ArrowLeft" || e.key === "a" || e.key === "A") isLeftPressed = true;
    if (e.key === "ArrowRight" || e.key === "d" || e.key === "D") isRightPressed = true;
    if (e.key === "ArrowUp" || e.key === "w" || e.key === "W") isUpPressed = true;
    if (e.key === "ArrowDown" || e.key === "s" || e.key === "S") isDownPressed = true;

    if (e.key === "+" || e.key === "=") zoomIn();
    if (e.key === "-" || e.key === "_") zoomOut();
  }

  function handleKeyUp(e) {
    if (e.key === " " || e.key === "Spacebar") return;
    if (e.key === "ArrowLeft" || e.key === "a" || e.key === "A") isLeftPressed = false;
    if (e.key === "ArrowRight" || e.key === "d" || e.key === "D") isRightPressed = false;
    if (e.key === "ArrowUp" || e.key === "w" || e.key === "W") isUpPressed = false;
    if (e.key === "ArrowDown" || e.key === "s" || e.key === "S") isDownPressed = false;
  }

  function handleMouseDown(e) {
    if (
      e.target.closest(".controls-panel") || 
      e.target.closest(".manager-modal") || 
      e.target.closest(".menu-collapsed-wrapper") ||
      e.target.closest(".highlight-settings-panel") ||
      e.target.closest(".floating-sub-panel")
    ) return;

    totalDragDistance = 0;
    if (e.button === 1) {
      e.preventDefault();
      isMiddleDragging = true;
    } else if (e.button === 0) {
      isDragging = true;
    }

    startX = e.clientX;
    startY = e.clientY;
    previousX = e.clientX;
    previousY = e.clientY;
    dragVelocity = 0;
  }

  function handleMouseMove(e) {
    const deltaX = e.clientX - previousX;
    const deltaY = e.clientY - previousY;
    previousX = e.clientX;
    previousY = e.clientY;

    if (isDragging) {
      totalDragDistance += Math.abs(deltaX) + Math.abs(deltaY);
      targetRotation += deltaX * 0.15;
      dragVelocity = deltaX * 8.0;
    } else if (isMiddleDragging) {
      targetCameraPitch = Math.max(-25, Math.min(35, targetCameraPitch - deltaY * 0.1));
      targetCameraHeight = Math.max(-800, Math.min(800, targetCameraHeight + deltaY * 0.8));
    }
  }

  function handleMouseUp(e) {
    if (e.button === 0 && isDragging) {
      isDragging = false;
      rotationVelocity = dragVelocity;
      persistAll();
    } else if (e.button === 1) {
      isMiddleDragging = false;
      persistAll();
    }
  }

  function handleSlotMouseMove(e, cardId) {
    if (isDragging || isMiddleDragging) return;
    const state = getCardState(cardId);
    const slotEl = e.currentTarget;
    const rect = slotEl.getBoundingClientRect();

    const x = e.clientX - rect.left;
    const y = e.clientY - rect.top;

    state.targetX = Math.min(Math.max((x / rect.width) * 100, 0), 100);
    state.targetY = Math.min(Math.max((y / rect.height) * 100, 0), 100);

    const isFlipped = flippedCardId === cardId;
    const tiltMultiplier = isFlipped ? -1 : 1;
    state.targetRx = ((y / rect.height) - 0.5) * -18;
    state.targetRy = ((x / rect.width) - 0.5) * 18 * tiltMultiplier;
    state.targetOp = 0.65;
  }

  function handleSlotMouseLeave(cardId) {
    const state = getCardState(cardId);
    state.targetRx = 0;
    state.targetRy = 0;
    state.targetOp = 0;
  }

  onMount(() => {
    loadCards();

    let animId;
    let lastTime = performance.now();

    const loop = (currentTime) => {
      const delta = Math.min((currentTime - lastTime) / 1000, 0.1);
      lastTime = currentTime;

      if (isLeftPressed) {
        rotationVelocity += (rotationSpeed - rotationVelocity) * (1 - Math.exp(-10 * delta));
      } else if (isRightPressed) {
        rotationVelocity += (-rotationSpeed - rotationVelocity) * (1 - Math.exp(-10 * delta));
      } else if (isAutoRotating) {
        rotationVelocity += (-rotationSpeed - rotationVelocity) * (1 - Math.exp(-10 * delta));
      } else if (!isDragging) {
        rotationVelocity *= Math.exp(-4 * delta);
      }

      if (isUpPressed) {
        targetCameraPitch = Math.min(30, targetCameraPitch + 25 * delta);
        targetCameraHeight = Math.min(800, targetCameraHeight + 200 * delta);
      } else if (isDownPressed) {
        targetCameraPitch = Math.max(-20, targetCameraPitch - 25 * delta);
        targetCameraHeight = Math.max(-800, targetCameraHeight - 200 * delta);
      }

      targetRotation += rotationVelocity * delta;

      const rotSmoothing = 1 - Math.exp(-12 * delta);
      currentRotation += (targetRotation - currentRotation) * rotSmoothing;

      if (highlightMode === "snap" && cards.length > 0) {
        let currentSnappedIdx = -1;
        for (let i = 0; i < cards.length; i++) {
          const cardAngle = (i * STEP_ANGLE + currentRotation) % 360;
          let norm = ((cardAngle % 360) + 360) % 360;
          if (norm > 180) norm = 360 - norm;
          if (norm <= snapExitAngle) {
            currentSnappedIdx = i;
            break;
          }
        }
        if (currentSnappedIdx !== -1 && currentSnappedIdx !== lastSnappedIndex) {
          lastSnappedIndex = currentSnappedIdx;
          playCardSwooshSound();
        } else if (currentSnappedIdx === -1) {
          lastSnappedIndex = -1;
        }
      }

      const pitchSmoothing = 1 - Math.exp(-10 * delta);
      currentCameraPitch += (targetCameraPitch - currentCameraPitch) * pitchSmoothing;
      currentCameraHeight += (targetCameraHeight - currentCameraHeight) * pitchSmoothing;

      const fovSmoothing = 1 - Math.exp(-10 * delta);
      currentPerspective += (targetPerspective - currentPerspective) * fovSmoothing;

      const radiusSmoothing = 1 - Math.exp(-10 * delta);
      currentRadius += (targetRadius - currentRadius) * radiusSmoothing;

      const shadowSmoothing = 1 - Math.exp(-10 * delta);
      currentShadowDepth += (targetShadowDepth - currentShadowDepth) * shadowSmoothing;

      const zoomSmoothing = 1 - Math.exp(-10 * delta);
      currentZoom += (targetZoom - currentZoom) * zoomSmoothing;

      const lightRad = (lightAngle * Math.PI) / 180;
      const lightElevFactor = Math.cos((lightElevation * Math.PI) / 180);
      const lightElevSin = Math.sin((lightElevation * Math.PI) / 180);

      const lightXOffset = Math.sin(lightRad) * lightElevFactor * 45;
      const lightYOffset = -lightElevSin * 45;

      const cardBrightness = 0.85 + lightIntensity * 0.15 + (lightElevSin * 0.1);

      cardInteractions.forEach((state) => {
        if (!state.el) return;

        const lerpFactor = 1 - Math.exp(-14 * delta);
        state.currentX += (state.targetX - state.currentX) * lerpFactor;
        state.currentY += (state.targetY - state.currentY) * lerpFactor;
        state.currentRx += (state.targetRx - state.currentRx) * lerpFactor;
        state.currentRy += (state.targetRy - state.currentRy) * lerpFactor;
        state.currentOp += (state.targetOp - state.currentOp) * (1 - Math.exp(-10 * delta));

        state.el.style.setProperty("--tilt-rx", `${state.currentRx.toFixed(2)}deg`);
        state.el.style.setProperty("--tilt-ry", `${state.currentRy.toFixed(2)}deg`);
        
        const finalPx = Math.min(100, Math.max(0, 50 + lightXOffset + (state.currentX - 50) * 0.6));
        const finalPy = Math.min(100, Math.max(0, 50 + lightYOffset + (state.currentY - 50) * 0.6));
        state.el.style.setProperty("--pointer-x", `${finalPx.toFixed(1)}%`);
        state.el.style.setProperty("--pointer-y", `${finalPy.toFixed(1)}%`);

        state.el.style.setProperty("--card-opacity", (state.currentOp * lightIntensity).toFixed(3));
        state.el.style.setProperty("--card-brightness", cardBrightness.toFixed(2));
      });

      animId = requestAnimationFrame(loop);
    };
    animId = requestAnimationFrame(loop);

    return () => {
      cancelAnimationFrame(animId);
      if (recordTimerInterval) clearInterval(recordTimerInterval);
      if (audioCtx) audioCtx.close();
    };
  });
</script>

<svelte:window 
  on:keydown={handleKeyDown} 
  on:keyup={handleKeyUp} 
  on:wheel|passive={handleWheel}
  on:mousedown={handleMouseDown}
  on:mousemove={handleMouseMove}
  on:mouseup={handleMouseUp}
/>

<!-- 배경 레이어: 위치 및 배율(scale) 적용 -->
<div 
  class="app-background"
  style="
    {siteBgImage ? `background-image: url('${siteBgImage}');` : ''}
    background-size: {siteBgSize};
    background-position: calc({bgBaseX}% + {bgOffsetX}px) calc({bgBaseY}% + {bgOffsetY}px);
    transform: scale({bgScale});
    transform-origin: {bgBaseX}% {bgBaseY}%;
  "
></div>

{#if isMenuOpen}
  <header class="controls-panel">
    <!-- 1. 카드 이미지 관리 그룹 -->
    <div class="control-group">
      <button class="nav-btn primary-btn" on:click={() => showCardManager = !showCardManager}>
        ⚙️ 21장 개별 앞면
      </button>
      <label class="nav-btn">
        📁 앞면 일괄
        <input type="file" accept="image/*" multiple on:change={handleBatchFrontUpload} />
      </label>
      <label class="nav-btn">
        🔄 뒷면 변경
        <input type="file" accept="image/*" on:change={handleBackUpload} />
      </label>
    </div>

    <!-- 2. 배경 설정 그룹 (위치 이동 및 확대/축소) -->
    <div class="control-group">
      <label class="nav-btn bg-label">
        🖼️ 배경
        <input type="file" accept="image/*" on:change={handleSiteBgUpload} />
      </label>
      <select class="nav-select" bind:value={siteBgSize} on:change={persistAll} title="배경 크기 비율">
        <option value="cover">화면 꽉 채움 (Cover)</option>
        <option value="contain">전체 보기 (Contain)</option>
        <option value="100% 100%">화면 맞춤 (Stretch)</option>
        <option value="auto">반복 패턴 (Repeat)</option>
      </select>
      <select class="nav-select" on:change={handleBgPresetChange} title="배경 기준 위치">
        <option value="50 50">🎯 중앙</option>
        <option value="50 0">⬆️ 상단</option>
        <option value="50 100">⬇️ 하단</option>
        <option value="0 50">⬅️ 좌측</option>
        <option value="100 50">➡️ 우측</option>
        <option value="0 0">↖️ 좌상단</option>
        <option value="100 0">↗️ 우상단</option>
        <option value="0 100">↙️ 좌하단</option>
        <option value="100 100">↘️ 우하단</option>
      </select>

      {#if siteBgImage}
        <div class="nudge-box">
          <button class="nudge-btn" on:click={() => nudgeBg(-30, 0)} title="배경 좌로 30px">◀</button>
          <button class="nudge-btn" on:click={() => nudgeBg(30, 0)} title="배경 우로 30px">▶</button>
          <button class="nudge-btn" on:click={() => nudgeBg(0, -30)} title="배경 위로 30px">▲</button>
          <button class="nudge-btn" on:click={() => nudgeBg(0, 30)} title="배경 아래로 30px">▼</button>
          <button class="nudge-btn reset-nudge" on:click={resetBgOffset} title="배경 위치 리셋">↺0</button>
        </div>
        <!-- 배경 배율(Zoom) 조절 컨트롤 -->
        <div class="nudge-box">
          <span class="control-label" style="padding-left: 2px;">배경줌</span>
          <button class="nudge-btn" on:click={() => adjustBgScale(-0.1)} title="배경 축소">-</button>
          <button class="nudge-btn" on:click={() => adjustBgScale(0.1)} title="배경 확대">+</button>
          <button class="nudge-btn reset-nudge" on:click={resetBgScale} title="배경 배율 리셋">
            ↺{Math.round(bgScale * 100)}%
          </button>
        </div>
        <button class="nav-btn text-blue" on:click={resetSiteBg} title="배경 제거">↺ 배경</button>
      {/if}
    </div>

    <!-- 3. 카드 자체의 화면 상하 위치(높낮이) 조절 그룹 -->
    <div class="control-group">
      <span class="control-label" title="배경과 맞추기 위해 3D 카드 씬 전체를 위아래로 이동합니다">↕️ 카드 높이</span>
      <div class="nudge-box">
        <button class="nudge-btn" on:click={() => nudgeCardHeight(-30)} title="카드 전체를 화면 위로 이동">▲</button>
        <button class="nudge-btn" on:click={() => nudgeCardHeight(30)} title="카드 전체를 화면 아래로 이동">▼</button>
        <button class="nudge-btn reset-nudge" on:click={resetCardHeight} title="카드 위치 중앙 리셋">
          ↺{Math.round(-targetCameraHeight)}
        </button>
      </div>
    </div>

    <!-- 4. 화면 녹화 제어 그룹 -->
    <div class="control-group">
      {#if !isRecording}
        <button class="nav-btn record-start-btn" on:click={startRecording} title="화면 녹화 시작">
          ⏺️ 녹화
        </button>
      {:else}
        <button class="nav-btn record-stop-btn" on:click={stopRecording} title="녹화 중지 및 파일 다운로드">
          ⏹️ 중지 ({formatTime(recordTimer)})
        </button>
      {/if}
    </div>

    <!-- 5. 저장 및 원본 복구 그룹 -->
    <div class="control-group">
      <button class="nav-btn save-btn" on:click={handleManualSave} title="현재 설정 저장">
        💾 저장
      </button>
      <button class="nav-btn danger-btn" on:click={handleResetToDefault} title="기본 포켓몬 카드로 전체 복구">
        🔄 원본 복구
      </button>
    </div>

    <!-- 6. 카메라 시점 그룹 -->
    <div class="control-group">
      <button class="nav-btn" on:click={() => setCameraPreset(0, 0)}>👁️ 정면</button>
      <button class="nav-btn" on:click={() => setCameraPreset(18, 140)}>🦅 탑뷰</button>
      <button class="nav-btn" on:click={() => setCameraPreset(-12, -90)}>📐 로우뷰</button>
      <button class="nav-btn text-blue" on:click={resetCamera}>↺ 앵글</button>
    </div>

    <!-- 7. 정면 강조 및 세부 조절 그룹 -->
    <div class="control-group">
      <button 
        class="nav-btn mode-btn" 
        class:is-active={highlightMode === 'smooth'}
        on:click={() => selectHighlightMode('smooth')}
        title="정면 카드를 부드럽게 확대 및 서서히 돌출"
      >
        ✨ 부드러운 강조
      </button>
      <button 
        class="nav-btn mode-btn snap-btn" 
        class:is-active={highlightMode === 'snap'}
        on:click={() => selectHighlightMode('snap')}
        title="정면 도달 시 1장씩 교차되어 딱 걸려 올라옴"
      >
        ⚡ 스냅 팝
      </button>
      <button 
        class="nav-btn settings-gear-btn"
        class:is-open={showHighlightSettings}
        on:click={() => {
          showHighlightSettings = !showHighlightSettings;
          if (showHighlightSettings) { showFloorSettings = false; showLightSettings = false; showThicknessSettings = false; }
        }}
        title="강조 타이밍/크기 슬라이더 열기"
      >
        🎛️ 강조 설정 {showHighlightSettings ? '▲' : '▼'}
      </button>
    </div>

    <!-- 8. 카드 두께감 & 엣지 곡률 & 단면 입체 음영 조절 그룹 -->
    <div class="control-group">
      <button 
        class="nav-btn sub-toggle-btn"
        class:is-active={showThicknessSettings}
        on:click={() => {
          showThicknessSettings = !showThicknessSettings;
          if (showThicknessSettings) { showHighlightSettings = false; showFloorSettings = false; showLightSettings = false; }
        }}
        title="카드의 3D 두께감(소수점), 엣지 곡률, 단면 입체 음영 조절"
      >
        🃏 두께/엣지 {showThicknessSettings ? '▲' : '▼'}
      </button>
    </div>

    <!-- 9. 바닥 거울 반사 & 그림자 세부 조절 그룹 -->
    <div class="control-group">
      <button 
        class="nav-btn sub-toggle-btn"
        class:is-active={enableReflection || showFloorSettings}
        on:click={() => {
          showFloorSettings = !showFloorSettings;
          if (showFloorSettings) { showHighlightSettings = false; showLightSettings = false; showThicknessSettings = false; }
        }}
        title="바닥 거울 반사 및 그림자 세부 조절 슬라이더 패널 열기"
      >
        🪞 거울 & 그림자 {showFloorSettings ? '▲' : '▼'}
      </button>

      <!-- 3D 조명 제어 버튼 -->
      <button 
        class="nav-btn sub-toggle-btn"
        class:is-active={showLightSettings}
        on:click={() => {
          showLightSettings = !showLightSettings;
          if (showLightSettings) { showHighlightSettings = false; showFloorSettings = false; showThicknessSettings = false; }
        }}
        title="가상 3D 조명 방향/고도/밝기 조절"
      >
        💡 3D 조명 {showLightSettings ? '▲' : '▼'}
      </button>

      <!-- 스냅 넘김 바람소리 사운드 토글 버튼 -->
      <button 
        class="nav-btn icon-only" 
        class:is-active={soundEnabled}
        on:click={() => {
          soundEnabled = !soundEnabled;
          if (soundEnabled) playCardSwooshSound();
          persistAll();
        }}
        title="카드 넘김 바람 소리(슉~) 켜기/끄기"
      >
        {soundEnabled ? '🍃 슉~ 소리' : '🔇 묵음'}
      </button>
    </div>

    <!-- 10. 화각(FOV) 조절 그룹 -->
    <div class="control-group">
      <button class="nav-btn" on:click={() => setFov(1200)} title="광각">📷 광각</button>
      <button class="nav-btn" on:click={() => setFov(2100)} title="표준 화각">📷 표준</button>
      <button class="nav-btn" on:click={() => setFov(3800)} title="망원">📷 망원</button>
      <div class="nudge-box">
        <button class="nudge-btn" on:click={() => adjustFov(-300)} title="광각 쪽으로">-</button>
        <button class="nudge-btn" on:click={() => adjustFov(300)} title="망원 쪽으로">+</button>
      </div>
    </div>

    <!-- 11. 카드 간격(반지름) 조절 그룹 -->
    <div class="control-group">
      <span class="control-label">간격</span>
      <div class="nudge-box">
        <button class="nudge-btn" on:click={() => adjustRadius(-60)} title="간격 좁히기">-</button>
        <button class="nudge-btn" on:click={() => adjustRadius(60)} title="간격 넓히기">+</button>
        <button class="nudge-btn reset-nudge" on:click={resetRadius} title="기본 간격 리셋">↺</button>
      </div>
    </div>

    <!-- 12. 줌 컨트롤 그룹 -->
    <div class="control-group">
      <button class="nav-btn icon-only" on:click={zoomIn} title="확대 (+)">🔍 +</button>
      <button class="nav-btn icon-only" on:click={zoomOut} title="축소 (-)">🔍 -</button>
      <button class="nav-btn text-blue text-sm" on:click={resetZoom} title="크기 초기화">↺ {Math.round(currentZoom * 100)}%</button>
    </div>

    <!-- 13. 속도 컨트롤 및 자동 회전 그룹 -->
    <div class="control-group">
      <button 
        class="nav-btn auto-rotate-btn" 
        class:is-active={isAutoRotating}
        on:click={toggleAutoRotate} 
        title="카드 자동 회전 시작/정지 (단축키: Space)"
      >
        {isAutoRotating ? '⏸️ 정지' : '▶ 자동 회전'}
      </button>
      <button class="nav-btn icon-only" on:click={speedDown} title="속도 -">⚡ -</button>
      <button class="nav-btn icon-only" on:click={speedUp} title="속도 +">⚡ +</button>
      <button class="nav-btn text-blue text-sm" on:click={resetSpeed} title="속도 초기화">{rotationSpeed}</button>
    </div>

    <!-- 일체형 메뉴 접기 버튼 -->
    <button 
      class="inline-menu-toggle"
      on:click={() => isMenuOpen = false}
      title="상단 메뉴 접기"
    >
      ▲ 접기
    </button>
  </header>

  <!-- 팝업 패널 1: 정면 강조 슬라이더 -->
  {#if showHighlightSettings}
    <div class="highlight-settings-panel floating-sub-panel">
      <div class="settings-title-row">
        <span class="settings-title">⚙️ 정면 강조 세부 옵션</span>
        <button class="mini-close" on:click={() => showHighlightSettings = false}>✕</button>
      </div>
      <div class="slider-grid">
        <div class="slider-item">
          <div class="slider-label-row">
            <span>시작 각도 (진입)</span>
            <span class="slider-val">{snapEntryAngle.toFixed(1)}°</span>
          </div>
          <input type="range" min="2.0" max="18.0" step="0.5" bind:value={snapEntryAngle} on:input={persistAll} />
        </div>
        <div class="slider-item">
          <div class="slider-label-row">
            <span>종료 각도 (이탈)</span>
            <span class="slider-val">{snapExitAngle.toFixed(1)}°</span>
          </div>
          <input type="range" min="1.0" max="12.0" step="0.5" bind:value={snapExitAngle} on:input={persistAll} />
        </div>
        <div class="slider-item">
          <div class="slider-label-row">
            <span>강조 크기 (Scale)</span>
            <span class="slider-val">{snapScale.toFixed(2)}x</span>
          </div>
          <input type="range" min="1.00" max="1.30" step="0.01" bind:value={snapScale} on:input={persistAll} />
        </div>
        <div class="slider-item">
          <div class="slider-label-row">
            <span>돌출 깊이 (Z-Pop)</span>
            <span class="slider-val">+{snapPopZ}px</span>
          </div>
          <input type="range" min="0" max="180" step="5" bind:value={snapPopZ} on:input={persistAll} />
        </div>
        <div class="slider-item">
          <div class="slider-label-row">
            <span>상승 높이 (Y-Lift)</span>
            <span class="slider-val">-{snapPopY}px</span>
          </div>
          <input type="range" min="0" max="80" step="2" bind:value={snapPopY} on:input={persistAll} />
        </div>
      </div>
    </div>
  {/if}

  <!-- 팝업 패널 2: 카드 두께감 & 엣지 곡률 & 단면 입체 음영 조절 슬라이더 -->
  {#if showThicknessSettings}
    <div class="highlight-settings-panel floating-sub-panel">
      <div class="settings-title-row">
        <span class="settings-title">🃏 카드 실물 두께 & 단면 입체감 조절</span>
        <button class="mini-close" on:click={() => showThicknessSettings = false}>✕</button>
      </div>
      <div class="slider-grid">
        <div class="slider-item">
          <div class="slider-label-row">
            <span>카드 두께 (Thickness)</span>
            <span class="slider-val">{cardThickness.toFixed(1)}mm</span>
          </div>
          <input 
            type="range" 
            min="0.0" 
            max="8.0" 
            step="0.1" 
            bind:value={cardThickness} 
            on:input={persistAll} 
          />
        </div>
        <div class="slider-item">
          <div class="slider-label-row">
            <span>단면 입체 음영 (Depth Shade)</span>
            <span class="slider-val">{Math.round(edgeDepthShade * 100)}%</span>
          </div>
          <input 
            type="range" 
            min="0.0" 
            max="1.0" 
            step="0.05" 
            bind:value={edgeDepthShade} 
            on:input={persistAll} 
          />
        </div>
        <div class="slider-item">
          <div class="slider-label-row">
            <span>엣지 곡률 (날카로움~부드러움)</span>
            <span class="slider-val">{cardRadius.toFixed(1)}px</span>
          </div>
          <input 
            type="range" 
            min="0.0" 
            max="30.0" 
            step="0.5" 
            bind:value={cardRadius} 
            on:input={persistAll} 
          />
        </div>
      </div>
    </div>
  {/if}

  <!-- 팝업 패널 3: 거울 반사 & 그림자 세부 조절 슬라이더 -->
  {#if showFloorSettings}
    <div class="highlight-settings-panel floating-sub-panel">
      <div class="settings-title-row">
        <span class="settings-title">🪞 바닥 거울 반사</span>
        <label style="font-size: 11px; color: #cbd5e1; cursor: pointer; display: flex; align-items: center; gap: 4px;">
          <input type="checkbox" bind:checked={enableReflection} on:change={persistAll} /> 반사 켜기
        </label>
      </div>
      {#if enableReflection}
        <div class="slider-grid">
          <div class="slider-item">
            <div class="slider-label-row">
              <span>반사 선명도 (투명도)</span>
              <span class="slider-val">{Math.round(reflectionOpacity * 100)}%</span>
            </div>
            <input type="range" min="0.0" max="0.7" step="0.02" bind:value={reflectionOpacity} on:input={persistAll} />
          </div>
          <div class="slider-item">
            <div class="slider-label-row">
              <span>반사 흐림도 (Blur)</span>
              <span class="slider-val">{reflectionBlur.toFixed(1)}px</span>
            </div>
            <input type="range" min="0" max="15" step="0.5" bind:value={reflectionBlur} on:input={persistAll} />
          </div>
          <div class="slider-item">
            <div class="slider-label-row">
              <span>반사 높낮이 (Y-Offset)</span>
              <span class="slider-val">{reflectionOffsetY > 0 ? `+${reflectionOffsetY}` : reflectionOffsetY}px</span>
            </div>
            <input type="range" min="-100" max="100" step="2" bind:value={reflectionOffsetY} on:input={persistAll} />
          </div>
        </div>
      {/if}

      <div style="height: 1px; background: rgba(255,255,255,0.1); margin: 6px 0;"></div>

      <div class="settings-title-row">
        <span class="settings-title">🌑 바닥 그림자 (Shadow)</span>
        <div style="display: flex; gap: 8px; align-items: center;">
          <label style="font-size: 11px; color: #cbd5e1; cursor: pointer; display: flex; align-items: center; gap: 4px;">
            <input type="checkbox" bind:checked={enableShadow} on:change={persistAll} /> 그림자 켜기
          </label>
          <button class="mini-close" on:click={() => showFloorSettings = false}>✕</button>
        </div>
      </div>
      {#if enableShadow}
        <div class="slider-grid">
          <div class="slider-item">
            <div class="slider-label-row">
              <span>그림자 깊이 (높낮이)</span>
              <span class="slider-val">{Math.round(targetShadowDepth)}px</span>
            </div>
            <input type="range" min="100" max="500" step="5" bind:value={targetShadowDepth} on:input={persistAll} />
          </div>
          <div class="slider-item">
            <div class="slider-label-row">
              <span>그림자 흐림도 (Blur)</span>
              <span class="slider-val">{shadowBlur}px</span>
            </div>
            <input type="range" min="4" max="40" step="2" bind:value={shadowBlur} on:input={persistAll} />
          </div>
          <div class="slider-item">
            <div class="slider-label-row">
              <span>그림자 진하기 (Opacity)</span>
              <span class="slider-val">{Math.round(shadowOpacity * 100)}%</span>
            </div>
            <input type="range" min="0.1" max="1.0" step="0.05" bind:value={shadowOpacity} on:input={persistAll} />
          </div>
          <div class="slider-item">
            <div class="slider-label-row">
              <span>가로 위치 (Offset X)</span>
              <span class="slider-val">{shadowOffsetX > 0 ? `+${shadowOffsetX}` : shadowOffsetX}px</span>
            </div>
            <input type="range" min="-80" max="80" step="2" bind:value={shadowOffsetX} on:input={persistAll} />
          </div>
          <div class="slider-item">
            <div class="slider-label-row">
              <span>세로 위치 (Offset Y)</span>
              <span class="slider-val">{shadowOffsetY > 0 ? `+${shadowOffsetY}` : shadowOffsetY}px</span>
            </div>
            <input type="range" min="-80" max="80" step="2" bind:value={shadowOffsetY} on:input={persistAll} />
          </div>
        </div>
      {/if}
    </div>
  {/if}

  <!-- 팝업 패널 4: 3D 가상 조명 & 바람소리 볼륨 제어 슬라이더 -->
  {#if showLightSettings}
    <div class="highlight-settings-panel floating-sub-panel">
      <div class="settings-title-row">
        <span class="settings-title">💡 3D 가상 조명 위치 & 바람소리 음량</span>
        <button class="mini-close" on:click={() => showLightSettings = false}>✕</button>
      </div>
      <div class="slider-grid">
        <div class="slider-item">
          <div class="slider-label-row">
            <span>조명 수평 각도</span>
            <span class="slider-val">{lightAngle}°</span>
          </div>
          <input type="range" min="0" max="360" step="5" bind:value={lightAngle} on:input={persistAll} />
        </div>
        <div class="slider-item">
          <div class="slider-label-row">
            <span>조명 수직 고도</span>
            <span class="slider-val">{lightElevation}°</span>
          </div>
          <input type="range" min="10" max="85" step="5" bind:value={lightElevation} on:input={persistAll} />
        </div>
        <div class="slider-item">
          <div class="slider-label-row">
            <span>조명 강도 (전체 밝기)</span>
            <span class="slider-val">{Math.round(lightIntensity * 100)}%</span>
          </div>
          <input type="range" min="0.2" max="2.0" step="0.05" bind:value={lightIntensity} on:input={persistAll} />
        </div>
        <div class="slider-item">
          <div class="slider-label-row">
            <span>카드 넘김 바람음(슉~) 볼륨</span>
            <span class="slider-val">{Math.round(soundVolume * 100)}%</span>
          </div>
          <input type="range" min="0.0" max="1.0" step="0.05" bind:value={soundVolume} on:input={persistAll} />
        </div>
      </div>
    </div>
  {/if}
{:else}
  <div class="menu-collapsed-wrapper">
    <button 
      class="menu-open-pill-btn"
      on:click={() => isMenuOpen = true}
      title="상단 메뉴 열기"
    >
      ⚙️ 메뉴 열기 ▼
    </button>
  </div>
{/if}

<!-- 21장 개별 앞면 이미지 설정 모달 -->
{#if showCardManager}
  <div class="manager-modal-backdrop" on:click={() => showCardManager = false}></div>
  <div class="manager-modal" class:menu-opened={isMenuOpen}>
    <div class="modal-header">
      <div>
        <h3>21장 개별 앞면 이미지 설정</h3>
        <p class="modal-subtitle">💡 카드를 드래그하여 원하는 슬롯으로 놓으면 자리가 바뀝니다.</p>
      </div>
      <button class="close-btn" on:click={() => showCardManager = false}>✕</button>
    </div>
    <div class="card-grid-manager">
      {#each cards as card, idx (card.id || idx)}
        <!-- svelte-ignore a11y-no-static-element-interactions -->
        <div 
          class="grid-card-item"
          class:is-dragging-item={draggedCardIndex === idx}
          class:is-drag-over={dragOverCardIndex === idx && draggedCardIndex !== idx}
          draggable="true"
          on:dragstart={(e) => handleDragStart(e, idx)}
          on:dragover={(e) => handleDragOver(e, idx)}
          on:dragleave={() => handleDragLeave(idx)}
          on:drop={(e) => handleDrop(e, idx)}
          on:dragend={handleDragEnd}
        >
          <span class="card-num">#{idx + 1}</span>
          <div class="thumb-preview" style="background-image: url('{card.frontImg}');">
            <div class="drag-handle-hint">⠿ 드래그 이동</div>
          </div>
          <span class="card-name">{card.name}</span>
          <label class="mini-btn">
            변경
            <input type="file" accept="image/*" on:change={(e) => handleSingleCardUpload(e, idx)} />
          </label>
        </div>
      {/each}
    </div>
  </div>
{/if}

<!-- 3D 씬 메인 뷰포트 -->
<main 
  class="carousel-scene" 
  class:is-dragging={isDragging || isMiddleDragging}
  style="perspective: {currentPerspective.toFixed(0)}px;"
>
  {#if isLoading}
    <div class="loading">Loading 21 Cards...</div>
  {:else}
    <div 
      class="camera-rig"
      style="
        transform: 
          scale({currentZoom}) 
          translateY({currentCameraHeight.toFixed(2)}px) 
          rotateX({currentCameraPitch.toFixed(2)}deg) 
          translate3d(0, 0, 0);
      "
    >
      <div 
        class="carousel-ring" 
        style="transform: translate3d(0, 0, 0) rotateY({currentRotation.toFixed(3)}deg);"
      >
        {#each cards as card, index (card.id || index)}
          {@const angle = index * STEP_ANGLE}
          {@const frontFactor = getFrontAlignment(index, currentRotation)}
          
          {@const isFlipped = flippedCardId === card.id}
          {@const isSnap = highlightMode === 'snap'}
          
          {@const basePopY = isSnap ? frontFactor * -snapPopY : 0}
          {@const basePopZ = isSnap ? frontFactor * snapPopZ : frontFactor * 55}
          {@const baseScale = isSnap ? (1 + frontFactor * (snapScale - 1.0)) : (1 + frontFactor * 0.08)}

          {@const finalPopY = isFlipped ? -20 : basePopY}
          {@const finalPopZ = isFlipped ? basePopZ + 180 : basePopZ}
          {@const finalScale = isFlipped ? 1.45 : baseScale}

          {@const lightRad = (lightAngle * Math.PI) / 180}
          {@const dynamicShadowX = shadowOffsetX - Math.sin(lightRad) * 45}
          {@const dynamicShadowY = shadowOffsetY + Math.cos(lightRad) * 45}

          <!-- svelte-ignore a11y-no-static-element-interactions -->
          <div 
            class="card-slot" 
            class:transition-snap={isSnap}
            class:is-card-active={isFlipped}
            style="
              transform: 
                rotateY({angle}deg) 
                translateZ({(currentRadius + finalPopZ).toFixed(2)}px) 
                translateY({finalPopY.toFixed(2)}px)
                scale({finalScale.toFixed(3)}) 
                translate3d(0, 0, 0);
              z-index: {isFlipped ? 9999 : Math.round(frontFactor * 100)};
            "
            on:mousemove={(e) => handleSlotMouseMove(e, card.id)}
            on:mouseleave={() => handleSlotMouseLeave(card.id)}
            on:click={() => handleCardClick(card.id)}
            title="클릭 시 그 자리에서 확대 및 뒷면 회전"
          >
            <!-- 3D 카드 본체 -->
            <div 
              class="custom-holo-card"
              class:is-focused={highlightMode === 'smooth' && frontFactor > 0.35}
              class:is-snap-focused={isSnap && frontFactor > 0.5}
              class:is-flipped={isFlipped}
              style="
                border-radius: {cardRadius}px;
                --thickness-half: {(cardThickness * 0.5).toFixed(2)}px;
                --thickness-neg: {(-cardThickness * 0.5).toFixed(2)}px;
                --card-radius: {cardRadius}px;
                --edge-shade: {edgeDepthShade.toFixed(2)};
              "
              use:registerCard={card.id}
            >
              <!-- 3D 입체 단면 코어 레이어 -->
              {#if cardThickness > 0.2}
                <div class="card-pulp-core" style="border-radius: {cardRadius}px;"></div>
              {/if}

              <!-- 카드 앞면 -->
              <div class="card-face card-front" style="border-radius: {cardRadius}px; transform: translateZ({(cardThickness * 0.5).toFixed(2)}px);">
                <img src={card.frontImg} alt={card.name} class="card-img" style="border-radius: {cardRadius}px;" decoding="async" loading="eager" />
                <div class="holo-shine" style="border-radius: {cardRadius}px;"></div>
                <div class="holo-glare" style="border-radius: {cardRadius}px;"></div>
              </div>

              <!-- 카드 뒷면 -->
              <div class="card-face card-back" style="border-radius: {cardRadius}px; transform: rotateY(180deg) translateZ({(cardThickness * 0.5).toFixed(2)}px);">
                <img src={customBackImage} alt="Card Back" class="card-img" style="border-radius: {cardRadius}px;" decoding="async" loading="eager" />
                <div class="holo-shine" style="border-radius: {cardRadius}px;"></div>
                <div class="holo-glare" style="border-radius: {cardRadius}px;"></div>
              </div>
            </div>

            <!-- 바닥 평면 투영 그림자 -->
            {#if enableShadow}
              <div 
                class="ground-shadow-anchor"
                style="
                  transform: 
                    translate(calc(-50% + {dynamicShadowX.toFixed(1)}px), calc(-50% + {dynamicShadowY.toFixed(1)}px)) 
                    rotateX(90deg) 
                    translateY({(-finalPopY).toFixed(2)}px) 
                    translateZ(-{currentShadowDepth.toFixed(2)}px) 
                    scale({(1 + (isFlipped ? 0.35 : frontFactor * (isSnap ? (snapScale - 1.0) * 1.4 : 0.10))).toFixed(2)});
                "
              >
                <div 
                  class="floor-shadow-soft" 
                  style="
                    filter: blur({shadowBlur}px);
                    opacity: {(shadowOpacity * (0.9 + frontFactor * 0.3)).toFixed(3)};
                  "
                ></div>
                <div 
                  class="floor-shadow-core" 
                  style="
                    filter: blur({Math.max(2, shadowBlur * 0.5)}px);
                    opacity: {(shadowOpacity * 1.25 * (0.9 + frontFactor * 0.3)).toFixed(3)};
                  "
                ></div>
              </div>
            {/if}

            <!-- 바닥 거울 반사 -->
            {#if enableReflection}
              <div 
                class="floor-mirror-reflection"
                style="
                  border-radius: {cardRadius}px;
                  transform: 
                    translateY({(373.5 + (currentShadowDepth - 240) * 0.5 - finalPopY * 2 + reflectionOffsetY).toFixed(2)}px) 
                    scaleY(-1);
                  opacity: {(reflectionOpacity * (1 + frontFactor * 0.4)).toFixed(3)};
                  filter: blur({reflectionBlur}px);
                "
              >
                <img src={isFlipped ? customBackImage : card.frontImg} alt="Reflection" class="mirror-card-img" style="border-radius: {cardRadius}px;" />
                <div class="mirror-fade-mask" style="border-radius: {cardRadius}px;"></div>
              </div>
            {/if}
          </div>
        {/each}
      </div>
    </div>
  {/if}
</main>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    background-color: #0c0e12;
    overflow: hidden;
    user-select: none;
    font-family: Roboto, -apple-system, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }

  .app-background {
    position: fixed;
    inset: 0;
    z-index: 0;
    background-color: #0c0e12;
    background-repeat: no-repeat;
    pointer-events: none;
    transition: background-image 0.3s ease, background-position 0.15s ease-out, transform 0.15s ease-out;
  }

  .controls-panel {
    position: fixed;
    top: 14px;
    left: 50%;
    transform: translateX(-50%);
    z-index: 10000;
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    align-items: center;
    gap: 6px 8px;
    background: rgba(20, 24, 33, 0.94);
    padding: 8px 12px;
    border-radius: 28px;
    border: 1px solid rgba(255, 255, 255, 0.15);
    backdrop-filter: blur(14px);
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.6);
    max-width: 96vw;
    box-sizing: border-box;
    transition: all 0.2s ease-out;
  }

  .floating-sub-panel {
    position: fixed;
    top: 96px;
    left: 50%;
    transform: translateX(-50%);
    z-index: 10001;
    background: rgba(18, 22, 32, 0.97);
    border: 1px solid rgba(96, 165, 250, 0.35);
    padding: 12px 18px;
    border-radius: 18px;
    backdrop-filter: blur(20px);
    box-shadow: 0 14px 40px rgba(0, 0, 0, 0.8);
    display: flex;
    flex-direction: column;
    gap: 10px;
    min-width: 330px;
    max-width: 92vw;
    box-sizing: border-box;
  }

  .settings-title-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    border-bottom: 1px solid rgba(255, 255, 255, 0.1);
    padding-bottom: 6px;
  }

  .settings-title {
    font-size: 11.5px;
    font-weight: 600;
    color: #93c5fd;
  }

  .mini-close {
    background: none;
    border: none;
    color: #94a3b8;
    cursor: pointer;
    font-size: 13px;
    padding: 0 4px;
  }
  .mini-close:hover { color: #fff; }

  .slider-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(130px, 1fr));
    gap: 10px 14px;
  }

  .slider-item {
    display: flex;
    flex-direction: column;
    gap: 3px;
  }

  .slider-label-row {
    display: flex;
    justify-content: space-between;
    font-size: 10.5px;
    color: #cbd5e1;
  }

  .slider-val {
    color: #60a5fa;
    font-weight: 600;
  }

  .slider-item input[type="range"] {
    width: 100%;
    accent-color: #3b82f6;
    height: 4px;
    cursor: pointer;
  }

  .settings-gear-btn {
    background: #1e2538;
    color: #93c5fd;
    border-color: rgba(96, 165, 250, 0.3);
  }

  .settings-gear-btn.is-open, .sub-toggle-btn.is-active {
    background: #2563eb;
    color: #fff;
    border-color: #60a5fa;
  }

  .sub-toggle-btn {
    background: #1e2538;
    color: #cbd5e1;
  }

  .inline-menu-toggle {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    height: 28px;
    padding: 0 10px;
    background: #1e2538;
    color: #94a3b8;
    font-size: 11px;
    font-weight: 600;
    border-radius: 14px;
    cursor: pointer;
    border: 1px solid rgba(255, 255, 255, 0.15);
    transition: all 0.15s ease;
    white-space: nowrap;
    flex-shrink: 0;
    margin-left: 2px;
  }

  .inline-menu-toggle:hover {
    background: #334155;
    color: #fff;
    border-color: rgba(255, 255, 255, 0.3);
  }

  .menu-collapsed-wrapper {
    position: fixed;
    top: 14px;
    left: 50%;
    transform: translateX(-50%);
    z-index: 10000;
  }

  .menu-open-pill-btn {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    height: 34px;
    padding: 0 16px;
    background: rgba(20, 24, 33, 0.94);
    color: #93c5fd;
    font-size: 12px;
    font-weight: 600;
    border-radius: 20px;
    border: 1px solid rgba(96, 165, 250, 0.4);
    backdrop-filter: blur(14px);
    box-shadow: 0 8px 24px rgba(0, 0, 0, 0.7);
    cursor: pointer;
    transition: all 0.2s ease;
  }

  .menu-open-pill-btn:hover {
    background: #2563eb;
    color: #ffffff;
    border-color: #60a5fa;
    transform: scale(1.04);
  }

  .control-group {
    display: inline-flex;
    align-items: center;
    gap: 4px;
    flex-shrink: 0;
    background: rgba(255, 255, 255, 0.03);
    padding: 3px 6px;
    border-radius: 18px;
    border: 1px solid rgba(255, 255, 255, 0.05);
  }

  .control-label {
    font-size: 11px;
    color: #94a3b8;
    margin-right: 2px;
    padding-left: 2px;
  }

  .nav-btn {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    height: 28px;
    padding: 0 9px;
    background: #232a3b;
    color: #e2e8f0;
    font-size: 11px;
    font-weight: 500;
    border-radius: 14px;
    cursor: pointer;
    border: 1px solid rgba(255, 255, 255, 0.1);
    transition: all 0.15s ease;
    white-space: nowrap;
    flex-shrink: 0;
    box-sizing: border-box;
  }

  .nav-btn:hover {
    background: #333f57;
    border-color: rgba(255, 255, 255, 0.25);
    transform: translateY(-1px);
  }

  .primary-btn {
    background: #2b3954;
    border-color: rgba(96, 165, 250, 0.3);
    color: #93c5fd;
  }

  .mode-btn {
    background: #1e293b;
    border-color: rgba(255, 255, 255, 0.15);
    color: #94a3b8;
  }

  .mode-btn.is-active {
    background: #4338ca;
    color: #e0e7ff;
    border-color: #818cf8;
    font-weight: 600;
    box-shadow: 0 0 10px rgba(99, 102, 241, 0.4);
  }

  .snap-btn.is-active {
    background: #0f766e;
    color: #ccfbf1;
    border-color: #2dd4bf;
    box-shadow: 0 0 10px rgba(45, 212, 191, 0.3);
  }

  .auto-rotate-btn {
    background: #1e3a5f;
    border-color: #3b82f6;
    color: #93c5fd;
    font-weight: 600;
  }

  .auto-rotate-btn.is-active {
    background: #2563eb;
    color: #ffffff;
    border-color: #60a5fa;
    box-shadow: 0 0 10px rgba(59, 130, 246, 0.5);
  }

  .record-start-btn {
    background: #4a1d24;
    border-color: #f43f5e;
    color: #fda4af;
    font-weight: 600;
  }

  .record-start-btn:hover {
    background: #62232c;
  }

  .record-stop-btn {
    background: #e11d48;
    border-color: #ff859b;
    color: #ffffff;
    font-weight: bold;
    animation: recordPulse 1.4s infinite;
  }

  @keyframes recordPulse {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.75; }
  }

  .save-btn {
    background: #1e3a5f;
    border-color: #3b82f6;
    color: #93c5fd;
    font-weight: 600;
  }

  .danger-btn {
    background: #451a1a;
    border-color: #ef4444;
    color: #fca5a5;
  }

  .danger-btn:hover {
    background: #5c2222;
  }

  .bg-label {
    background: #2a3447;
  }

  .text-blue {
    color: #60a5fa;
    font-weight: 600;
  }

  .text-sm {
    font-size: 10.5px;
    min-width: 44px;
  }

  .icon-only {
    padding: 0 7px;
    font-size: 11px;
  }

  .nav-btn input[type="file"], .mini-btn input[type="file"] {
    display: none;
  }

  .nav-select {
    height: 28px;
    background: #19202f;
    color: #e2e8f0;
    border: 1px solid rgba(255, 255, 255, 0.15);
    border-radius: 14px;
    font-size: 11px;
    padding: 0 8px;
    cursor: pointer;
    outline: none;
    white-space: nowrap;
    flex-shrink: 0;
    box-sizing: border-box;
  }

  .nav-select:hover {
    border-color: #3b82f6;
  }

  .nudge-box {
    display: inline-flex;
    align-items: center;
    gap: 2px;
    background: rgba(0, 0, 0, 0.35);
    padding: 2px 4px;
    border-radius: 14px;
    border: 1px solid rgba(255, 255, 255, 0.08);
    height: 28px;
    box-sizing: border-box;
    flex-shrink: 0;
  }

  .nudge-btn {
    background: #232a3b;
    border: 1px solid rgba(255, 255, 255, 0.1);
    color: #cbd5e1;
    height: 20px;
    min-width: 18px;
    padding: 0 4px;
    border-radius: 5px;
    font-size: 9px;
    cursor: pointer;
    display: inline-flex;
    align-items: center;
    justify-content: center;
  }

  .nudge-btn:hover {
    background: #3b82f6;
    color: #fff;
  }

  .nudge-btn.reset-nudge {
    color: #60a5fa;
    font-weight: bold;
    font-size: 9px;
  }

  .manager-modal-backdrop {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.65);
    backdrop-filter: blur(4px);
    z-index: 10002;
  }

  .manager-modal {
    position: fixed;
    top: 60px;
    left: 50%;
    transform: translateX(-50%);
    width: 92vw;
    max-width: 1020px;
    max-height: 82vh;
    background: rgba(18, 22, 30, 0.98);
    border: 1px solid rgba(255, 255, 255, 0.2);
    border-radius: 18px;
    backdrop-filter: blur(24px);
    z-index: 10003;
    padding: 22px;
    box-sizing: border-box;
    display: flex;
    flex-direction: column;
    box-shadow: 0 25px 60px rgba(0, 0, 0, 0.9);
    transition: top 0.2s ease;
  }

  .manager-modal.menu-opened {
    top: 130px;
    max-height: 74vh;
  }

  .modal-header {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    border-bottom: 1px solid rgba(255, 255, 255, 0.12);
    padding-bottom: 12px;
    margin-bottom: 16px;
  }

  .modal-header h3 {
    margin: 0 0 4px 0;
    color: #fff;
    font-size: 18px;
  }

  .modal-subtitle {
    margin: 0;
    font-size: 11.5px;
    color: #94a3b8;
  }

  .close-btn {
    background: none;
    border: none;
    color: #aaa;
    font-size: 22px;
    cursor: pointer;
    line-height: 1;
    padding: 2px 6px;
  }

  .close-btn:hover {
    color: #fff;
  }

  .card-grid-manager {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(110px, 1fr));
    gap: 14px;
    overflow-y: auto;
    padding-right: 6px;
  }

  .grid-card-item {
    background: rgba(255, 255, 255, 0.04);
    border: 1px solid rgba(255, 255, 255, 0.08);
    border-radius: 10px;
    padding: 8px;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 6px;
    cursor: grab;
    transition: transform 0.15s ease, border-color 0.15s ease, opacity 0.15s ease;
  }

  .grid-card-item:hover {
    border-color: rgba(96, 165, 250, 0.4);
    transform: translateY(-2px);
  }

  .grid-card-item:active {
    cursor: grabbing;
  }

  .grid-card-item.is-dragging-item {
    opacity: 0.35;
    border: 2px dashed #60a5fa;
    transform: scale(0.95);
  }

  .grid-card-item.is-drag-over {
    border: 2px solid #3b82f6;
    background: rgba(59, 130, 246, 0.15);
    transform: scale(1.05);
  }

  .card-num {
    font-size: 11px;
    font-weight: bold;
    color: #60a5fa;
  }

  .thumb-preview {
    position: relative;
    width: 60px;
    height: 80.3px;
    border-radius: 3.2px;
    background-size: cover;
    background-position: center;
    background-color: #222;
    border: 1px solid rgba(255, 255, 255, 0.2);
    overflow: hidden;
  }

  .drag-handle-hint {
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    background: rgba(0, 0, 0, 0.7);
    color: #cbd5e1;
    font-size: 8.5px;
    text-align: center;
    padding: 2px 0;
    opacity: 0;
    transition: opacity 0.15s ease;
    pointer-events: none;
  }

  .grid-card-item:hover .drag-handle-hint {
    opacity: 1;
  }

  .card-name {
    font-size: 11px;
    color: #ddd;
    text-align: center;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    width: 100%;
  }

  .mini-btn {
    background: #3b82f6;
    color: white;
    font-size: 11px;
    padding: 3px 10px;
    border-radius: 12px;
    cursor: pointer;
  }

  .mini-btn:hover {
    background: #2563eb;
  }

  /* 3D 씬 */
  .carousel-scene {
    position: relative;
    z-index: 1;
    width: 100vw;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    perspective-origin: 50% 50%;
    contain: layout size style;
    cursor: grab;
  }

  .carousel-scene.is-dragging {
    cursor: grabbing;
  }

  .loading {
    color: #fff;
    font-size: 1.5rem;
  }

  .camera-rig {
    transform-style: preserve-3d;
    will-change: transform;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .carousel-ring {
    position: relative;
    width: 279px;
    height: 373.5px;
    transform-style: preserve-3d;
    will-change: transform;
  }

  .card-slot {
    position: absolute;
    inset: 0;
    transform-style: preserve-3d;
    display: flex;
    justify-content: center;
    align-items: center;
    transition: transform 0.28s cubic-bezier(0.2, 0.8, 0.2, 1);
    cursor: pointer;
  }

  .card-slot.transition-snap {
    transition: transform 0.24s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  }

  .card-slot.is-card-active {
    transition: transform 0.5s cubic-bezier(0.19, 1, 0.22, 1);
  }

  /* 3D 카드 본체 */
  .custom-holo-card {
    position: relative;
    width: 279px;
    height: 373.5px;
    transform-style: preserve-3d;
    transform: rotateX(var(--tilt-rx, 0deg)) rotateY(var(--tilt-ry, 0deg));
    box-shadow: 0 6px 18px rgba(0, 0, 0, 0.65);
    outline: none;
    border: none;
    pointer-events: none;
    transition: transform 0.6s cubic-bezier(0.25, 1, 0.3, 1), box-shadow 0.25s ease;
  }

  /* 3D 실물 카드 압축 펄프 코어 단면 */
  .card-pulp-core {
    position: absolute;
    inset: 0;
    transform-style: preserve-3d;
    background: #d8d4cb;
    box-shadow: 
      inset 0 0 0 1px rgba(0, 0, 0, calc(0.35 * var(--edge-shade, 0.65))),
      0 0 0 1px rgba(40, 36, 30, calc(0.85 * var(--edge-shade, 0.65))),
      0 0 6px rgba(0, 0, 0, calc(0.6 * var(--edge-shade, 0.65)));
    transform: translateZ(0);
    pointer-events: none;
  }

  .custom-holo-card.is-flipped {
    transform: rotateX(var(--tilt-rx, 0deg)) rotateY(calc(180deg + var(--tilt-ry, 0deg)));
    box-shadow: 0 24px 60px rgba(0, 0, 0, 0.95);
  }

  .custom-holo-card.is-focused {
    box-shadow: 0 12px 36px rgba(0, 0, 0, 0.8);
  }

  .custom-holo-card.is-snap-focused {
    box-shadow: 0 16px 44px rgba(0, 0, 0, 0.9);
  }

  .card-face {
    position: absolute;
    inset: 0;
    overflow: hidden;
    backface-visibility: hidden;
    -webkit-backface-visibility: hidden;
    background-color: #1a1a1a;
  }

  .card-front {
    transform: translateZ(var(--thickness-half, 1.2px));
  }

  .card-back {
    transform: rotateY(180deg) translateZ(var(--thickness-half, 1.2px));
  }

  /* 텍스처 선명도 극대화 */
  .card-img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    display: block;
    pointer-events: none;
    transform: translateZ(0);
    image-rendering: auto;
    filter: brightness(var(--card-brightness, 1.0));
  }

  /* 바닥 투영 그림자 */
  .ground-shadow-anchor {
    position: absolute;
    top: 50%;
    left: 50%;
    width: 320px;
    height: 140px;
    transform-style: preserve-3d;
    pointer-events: none;
    transition: transform 0.24s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  }

  .floor-shadow-soft {
    position: absolute;
    inset: 0;
    border-radius: 50%;
    background: radial-gradient(
      ellipse at center,
      rgba(0, 0, 0, 0.55) 0%,
      rgba(0, 0, 0, 0.25) 45%,
      transparent 70%
    );
    transition: opacity 0.18s ease, filter 0.18s ease;
  }

  .floor-shadow-core {
    position: absolute;
    top: 15%;
    left: 10%;
    width: 80%;
    height: 70%;
    border-radius: 50%;
    background: radial-gradient(
      ellipse at center,
      rgba(0, 0, 0, 0.75) 0%,
      rgba(0, 0, 0, 0.4) 50%,
      transparent 80%
    );
    transition: opacity 0.18s ease, filter 0.18s ease;
  }

  /* 바닥 거울 반사 */
  .floor-mirror-reflection {
    position: absolute;
    top: 0;
    left: 0;
    width: 279px;
    height: 373.5px;
    border-radius: 14.4px;
    transform-style: preserve-3d;
    pointer-events: none;
    overflow: hidden;
    mask-image: linear-gradient(to top, rgba(0, 0, 1) 0%, rgba(0, 0, 0) 75%);
    -webkit-mask-image: linear-gradient(to top, rgba(0, 0, 1) 0%, rgba(0, 0, 0) 75%);
    transition: opacity 0.2s ease, transform 0.24s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  }

  .mirror-card-img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  .mirror-fade-mask {
    position: absolute;
    inset: 0;
    background: linear-gradient(to bottom, rgba(12, 14, 18, 0.3) 0%, rgba(12, 14, 18, 0.85) 100%);
  }

  .holo-shine {
    position: absolute;
    inset: 0;
    background: linear-gradient(
      115deg,
      transparent 0%,
      rgba(255, 255, 255, 0.18) 30%,
      rgba(255, 255, 255, 0.35) 45%,
      rgba(255, 255, 255, 0) 60%
    );
    background-size: 250% 250%;
    background-position: var(--pointer-x, 50%) var(--pointer-y, 50%);
    mix-blend-mode: overlay;
    opacity: var(--card-opacity, 0);
    pointer-events: none;
    transition: opacity 0.2s ease;
  }

  .holo-glare {
    position: absolute;
    inset: 0;
    background: radial-gradient(
      circle at var(--pointer-x, 50%) var(--pointer-y, 50%),
      rgba(255, 255, 255, 0.42) 10%,
      rgba(255, 255, 255, 0.15) 30%,
      transparent 70%
    );
    mix-blend-mode: color-dodge;
    opacity: var(--card-opacity, 0);
    pointer-events: none;
    transition: opacity 0.2s ease;
  }
</style>
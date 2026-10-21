# Giftoo
Interactive Gift Message Builder - Create and share beautiful personalized gift messages with photos and music
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <meta name="theme-color" content="#11121d">
  <meta name="description" content="Create and share a personalized digital gift card with photos, music, and a message.">
  <meta property="og:title" content="Giftoo — A little gift from the heart">
  <meta property="og:description" content="Create and share a personalized digital gift card.">
  <title>Giftoo — A little gift from the heart</title>
  <link rel="icon" href="data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 64 64'%3E%3Crect width='64' height='64' rx='16' fill='%23ff5577'/%3E%3Cpath d='M12 28h40v28H12zM8 20h48v12H8zM28 20c-13-1-13-14-5-14 7 0 9 14 9 14m4 0c0-9 3-14 9-14 9 0 8 13-9 14' fill='none' stroke='white' stroke-width='4' stroke-linejoin='round'/%3E%3C/svg%3E">
  <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js" defer></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js" defer></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/lz-string/1.4.4/lz-string.min.js" defer></script>
  <style>
    :root {
      color-scheme: dark;
      --primary: #ff5577;
      --secondary: #a9baff;
      --bg: #11121d;
      --card-bg: #1d2030;
      --text: #fff;
      --muted: #c3c5d0;
      font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
    }

    * { box-sizing: border-box; }
    body { min-width: 320px; margin: 0; padding: 24px; background: var(--bg); color: var(--text); }
    button, input, select, textarea { font: inherit; }
    button { cursor: pointer; }
    button:focus-visible, input:focus-visible, select:focus-visible, textarea:focus-visible {
      outline: 3px solid var(--secondary);
      outline-offset: 2px;
    }

    .site-header { display: flex; justify-content: center; margin: 0 auto 24px; }
    .brand { display: inline-flex; align-items: center; gap: 12px; color: var(--primary); text-decoration: none; }
    .brand svg { width: 46px; height: 46px; filter: drop-shadow(0 4px 12px #ff557755); }
    .brand-name { margin: 0; font-size: clamp(2rem, 6vw, 2.7rem); letter-spacing: .02em; }
    .tagline { margin: 4px 0 0; color: var(--muted); text-align: center; }

    .container { display: grid; grid-template-columns: minmax(280px, 1fr) minmax(280px, 1fr); gap: 20px; max-width: 1200px; margin: 0 auto; }
    .container.gift-view { grid-template-columns: minmax(280px, 840px); justify-content: center; }
    .panel { min-width: 0; padding: 24px; border: 1px solid #303448; border-radius: 16px; background: var(--card-bg); box-shadow: 0 10px 30px #0004; }
    .panel h2 { margin: 0 0 16px; padding-bottom: 9px; border-bottom: 2px solid #34384c; color: var(--secondary); font-size: 1.25rem; }
    .form-group { margin-bottom: 16px; }
    label { display: block; margin-bottom: 6px; color: var(--muted); font-size: .95rem; }
    input, select, textarea, .action-button {
      width: 100%; padding: 12px; border: 1px solid #3a3e54; border-radius: 8px; background: #292d40; color: var(--text);
    }
    textarea { min-height: 100px; resize: vertical; }
    input[type="file"] { padding: 9px; }
    .assistant-box { padding: 14px; border: 1px solid #8057b8; border-radius: 10px; background: #a55eea18; }
    .assistant-box label { color: var(--text); font-weight: 700; }
    .action-button { border: 0; font-weight: 700; }
    .btn-ai { margin-top: 8px; background: linear-gradient(135deg, #a55eea, #45aaf2); }
    .btn-share { margin-top: 8px; background: linear-gradient(135deg, #2ed573, #1e90ff); font-size: 1.05rem; }

    .gift-card {
      position: relative; display: flex; min-height: 520px; overflow: hidden; align-items: center; justify-content: center;
      padding: 30px; border-radius: 12px; background: linear-gradient(135deg, #ff9a9e, #fecfef); background-position: center; background-size: cover;
      color: white; text-align: center; box-shadow: 0 15px 35px #0003; animation: float 4s ease-in-out infinite;
    }
    .gift-card-overlay { position: absolute; inset: 0; z-index: 1; border-radius: inherit; background: #0007; }
    .gift-card-content { position: relative; z-index: 2; display: flex; width: 100%; flex-direction: column; align-items: center; overflow-wrap: anywhere; }
    #previewTitle { margin: 0; color: white; font-size: 1.6rem; }
    .gift-message { margin: 10px 0 0; font-size: 1.2rem; font-weight: 600; line-height: 1.5; white-space: pre-line; text-shadow: 0 2px 4px #0009; }
    #audioPlayer { width: min(100%, 360px); margin-top: 15px; }
    @keyframes float { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-8px); } }

    .slider-wrapper { position: relative; display: none; width: 100%; max-width: 360px; height: 220px; margin: 16px 0; overflow: hidden; border-radius: 12px; box-shadow: 0 8px 20px #0007; touch-action: pan-y; }
    .slider-track { display: flex; height: 100%; transition: transform .3s ease-out; }
    .slider-slide { min-width: 100%; height: 100%; object-fit: cover; user-select: none; }
    .slider-nav { position: absolute; z-index: 5; top: 50%; width: 36px; height: 36px; padding: 0; transform: translateY(-50%); border: 0; border-radius: 50%; background: #0009; color: white; font-size: 1.5rem; }
    .slider-nav.prev { left: 8px; }
    .slider-nav.next { right: 8px; }
    .slider-dots { position: absolute; z-index: 5; right: 0; bottom: 8px; left: 0; display: flex; justify-content: center; gap: 7px; }
    .dot { width: 8px; height: 8px; border-radius: 50%; background: #ffffff88; }
    .dot.active { transform: scale(1.2); background: white; }

    .modal-overlay { position: fixed; z-index: 10; inset: 0; display: none; align-items: center; justify-content: center; padding: 16px; background: #000d; backdrop-filter: blur(5px); }
    .modal-card { width: min(100%, 440px); max-height: 95vh; overflow: auto; padding: 24px; border: 1px solid #3a3e54; border-radius: 16px; background: var(--card-bg); text-align: center; box-shadow: 0 10px 30px #0008; }
    .modal-card h2 { margin-top: 0; }
    #modalQrCode { display: inline-block; min-height: 10px; margin: 12px 0; padding: 10px; border-radius: 10px; background: white; }
    #modalShareUrl { margin: 5px 0 0; background: #090a0f; color: #8affb4; font-family: monospace; font-size: .8rem; text-align: center; }
    .modal-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-top: 14px; }
    .btn-copy-link { background: #2ed573; }
    .btn-share-app { background: #1e90ff; }
    .btn-download-qr { background: #a55eea; }
    .btn-open-preview { background: #287ce0; }
    .btn-close { grid-column: span 2; background: var(--primary); }
    .status { min-height: 1.4em; margin: 10px 0 0; color: var(--muted); font-size: .9rem; }
    .status.error { color: #ff9ba7; }
    .status.success { color: #8affb4; }
    .visually-hidden { position: absolute; width: 1px; height: 1px; overflow: hidden; clip: rect(0, 0, 0, 0); white-space: nowrap; }

    @media (max-width: 800px) {
      body { padding: 16px; }
      .container { grid-template-columns: 1fr; }
      .panel { padding: 18px; }
      .gift-card { min-height: 440px; }
    }
    @media (prefers-reduced-motion: reduce) {
      *, *::before, *::after { scroll-behavior: auto !important; animation-duration: .01ms !important; animation-iteration-count: 1 !important; transition-duration: .01ms !important; }
    }
  </style>
</head>
<body>
  <header class="site-header">
    <div>
      <a class="brand" href="./" aria-label="Giftoo home">
        <svg viewBox="0 0 64 64" aria-hidden="true">
          <path d="M12 28h40v28H12zM8 20h48v12H8zM28 20c-13-1-13-14-5-14 7 0 9 14 9 14m4 0c0-9 3-14 9-14 9 0 8 13-9 14" fill="none" stroke="currentColor" stroke-linejoin="round" stroke-width="4"/>
        </svg>
        <h1 class="brand-name">Giftoo</h1>
      </a>
      <p class="tagline">A little gift from the heart</p>
    </div>
  </header>

  <main class="container">
    <section class="panel editor-panel" aria-labelledby="customizeHeading">
      <h2 id="customizeHeading">1. Customize your gift</h2>

      <div class="form-group assistant-box">
        <label for="aiOccasion">✨ Message ideas</label>
        <select id="aiOccasion">
          <option value="Love">Love &amp; romance</option>
          <option value="Birthday">Birthday</option>
          <option value="Anniversary">Anniversary</option>
        </select>
        <button type="button" class="action-button btn-ai" id="generateButton">Suggest a message</button>
      </div>

      <div class="form-group">
        <label for="giftTitle">Recipient name or title</label>
        <input type="text" id="giftTitle" placeholder="e.g. To my love..." autocomplete="off">
      </div>
      <div class="form-group">
        <label for="giftMessage">Your message (Enter creates a share link; Shift+Enter adds a line)</label>
        <textarea id="giftMessage" placeholder="Write something from the heart..."></textarea>
      </div>
      <div class="form-group">
        <label for="bgSelect">Background theme</label>
        <select id="bgSelect">
          <option value="default">Pink gradient</option>
          <option value="hearts-gif">Animated hearts</option>
          <option value="romantic-stars">Romantic stars</option>
        </select>
      </div>
      <div class="form-group">
        <label for="photoInput">Add photos (they are embedded in the share link)</label>
        <input type="file" id="photoInput" accept="image/*" multiple>
      </div>
      <div class="form-group">
        <label for="presetMusicSelect">🎵 Background music</label>
        <select id="presetMusicSelect">
          <option value="">No music</option>
          <option value="https://cdn.pixabay.com/download/audio/2022/05/27/audio_1808fbf07a.mp3">Romantic piano melody</option>
          <option value="https://cdn.pixabay.com/download/audio/2022/01/18/audio_d0a13f69d2.mp3">Acoustic love guitar</option>
          <option value="https://cdn.pixabay.com/download/audio/2022/10/14/audio_9939f792cb.mp3">Gentle cinematic strings</option>
        </select>
        <input type="file" id="musicFileInput" accept="audio/*" class="visually-hidden" aria-label="Choose an audio file">
        <button type="button" class="action-button" id="chooseMusicButton">Choose music file</button>
      </div>
      <div class="form-group">
        <label for="effectSelect">Celebration effect</label>
        <select id="effectSelect">
          <option value="hearts">Floating hearts</option>
          <option value="confetti">Confetti</option>
        </select>
      </div>

      <h2>2. Share your gift</h2>
      <button type="button" class="action-button btn-share" id="createShareButton">🚀 Create link and QR code</button>
      <p id="pageNotice" class="status" role="status"></p>
    </section>

    <section class="panel preview-panel" aria-labelledby="previewHeading">
      <h2 id="previewHeading">Live gift preview</h2>
      <article id="giftCard" class="gift-card" aria-label="Gift card preview">
        <div class="gift-card-overlay" aria-hidden="true"></div>
        <div class="gift-card-content">
          <h2 id="previewTitle">To my love ❤️</h2>
          <div id="sliderWrapper" class="slider-wrapper" aria-label="Gift photos">
            <button type="button" class="slider-nav prev" id="prevButton" aria-label="Previous photo">‹</button>
            <div id="sliderTrack" class="slider-track"></div>
            <button type="button" class="slider-nav next" id="nextButton" aria-label="Next photo">›</button>
            <div id="sliderDots" class="slider-dots" aria-hidden="true"></div>
          </div>
          <p id="previewMessage" class="gift-message">Every moment spent with you is like a beautiful dream come true.</p>
          <audio id="audioPlayer" controls loop preload="none" hidden></audio>
        </div>
      </article>
    </section>
  </main>

  <div class="modal-overlay" id="shareModal" role="dialog" aria-modal="true" aria-labelledby="shareHeading" aria-hidden="true">
    <section class="modal-card">
      <h2 id="shareHeading">Your gift link and QR code</h2>
      <p id="shareNotice" class="status" role="status"></p>
      <div id="modalQrCode" aria-label="QR code for your gift"></div>
      <label for="modalShareUrl" class="visually-hidden">Share link</label>
      <input type="text" id="modalShareUrl" readonly>
      <p id="shareStatus" class="status" role="status" aria-live="polite"></p>
      <div class="modal-grid">
        <button type="button" class="action-button btn-copy-link" id="copyButton">📋 Copy link</button>
        <button type="button" class="action-button btn-share-app" id="nativeShareButton">📲 Share</button>
        <button type="button" class="action-button btn-download-qr" id="downloadQrButton">🖼️ Save QR</button>
        <button type="button" class="action-button btn-open-preview" id="openPreviewButton">🌐 Open preview</button>
        <button type="button" class="action-button btn-close" id="closeModalButton">✏️ Edit gift / close</button>
      </div>
    </section>
  </div>

  <script>
    "use strict";

    const photoList = [];
    let currentIndex = 0;
    let currentAudioUrl = "";
    let lastGeneratedUrl = "";
    let pointerStartX = null;

    const byId = (id) => document.getElementById(id);
    const defaultTitle = "To my love ❤️";
    const defaultMessage = "Every moment spent with you is like a beautiful dream come true.";

    function setStatus(element, message, kind) {
      element.textContent = message;
      element.className = `status${kind ? ` ${kind}` : ""}`;
    }

    function updatePreview() {
      byId("previewTitle").textContent = byId("giftTitle").value.trim() || defaultTitle;
      byId("previewMessage").textContent = byId("giftMessage").value || defaultMessage;
    }

    function updateBackground() {
      const backgrounds = {
        default: "linear-gradient(135deg, #ff9a9e 0%, #fecfef 99%)",
        "hearts-gif": "url('https://media.giphy.com/media/26hpK8Aa3MyUvhC00/giphy.gif')",
        "romantic-stars": "url('https://media.giphy.com/media/l0E01D9qU6a11eP4I/giphy.gif')"
      };
      byId("giftCard").style.backgroundImage = backgrounds[byId("bgSelect").value] || backgrounds.default;
    }

    function compressImage(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onerror = () => reject(new Error(`Could not read ${file.name}.`));
        reader.onload = () => {
          const image = new Image();
          image.onerror = () => reject(new Error(`${file.name} is not a readable image.`));
          image.onload = () => {
            const maxDimension = 480;
            const scale = Math.min(1, maxDimension / Math.max(image.width, image.height));
            const canvas = document.createElement("canvas");
            canvas.width = Math.max(1, Math.round(image.width * scale));
            canvas.height = Math.max(1, Math.round(image.height * scale));
            const context = canvas.getContext("2d");
            if (!context) {
              reject(new Error("This browser could not process an image."));
              return;
            }
            context.drawImage(image, 0, 0, canvas.width, canvas.height);
            resolve(canvas.toDataURL("image/jpeg", 0.55));
          };
          image.src = reader.result;
        };
        reader.readAsDataURL(file);
      });
    }

    async function handlePhotosUpload(event) {
      const files = Array.from(event.target.files || []);
      if (files.length === 0) return;
      if (files.some((file) => !file.type.startsWith("image/"))) {
        setStatus(byId("pageNotice"), "Please choose image files only.", "error");
        event.target.value = "";
        return;
      }

      setStatus(byId("pageNotice"), "Preparing your photos...");
      try {
        const compressedPhotos = await Promise.all(files.map(compressImage));
        photoList.splice(0, photoList.length, ...compressedPhotos);
        renderSlider();
        setStatus(byId("pageNotice"), `${photoList.length} photo${photoList.length === 1 ? "" : "s"} added.`, "success");
      } catch (error) {
        console.error("Photo upload failed", error);
        photoList.splice(0);
        renderSlider();
        setStatus(byId("pageNotice"), error.message || "Could not process the selected photos.", "error");
      }
    }

    function renderSlider() {
      const wrapper = byId("sliderWrapper");
      const track = byId("sliderTrack");
      const dots = byId("sliderDots");
      track.replaceChildren();
      dots.replaceChildren();
      wrapper.style.display = photoList.length ? "block" : "none";
      photoList.forEach((source, index) => {
        const image = document.createElement("img");
        image.src = source;
        image.className = "slider-slide";
        image.alt = `Gift photo ${index + 1}`;
        track.appendChild(image);

        const dot = document.createElement("span");
        dot.className = `dot${index === 0 ? " active" : ""}`;
        dots.appendChild(dot);
      });
      currentIndex = 0;
      updateSliderPosition();
    }

    function updateSliderPosition() {
      byId("sliderTrack").style.transform = `translateX(-${currentIndex * 100}%)`;
      byId("sliderDots").querySelectorAll(".dot").forEach((dot, index) => {
        dot.classList.toggle("active", index === currentIndex);
      });
    }

    function moveSlide(direction) {
      if (!photoList.length) return;
      currentIndex = (currentIndex + direction + photoList.length) % photoList.length;
      updateSliderPosition();
    }

    function playAudio(url) {
      const player = byId("audioPlayer");
      currentAudioUrl = url;
      player.src = url;
      player.hidden = !url;
      if (url) {
        player.play().catch(() => {
          setStatus(byId("pageNotice"), "Tap play on the audio controls to start the music.", "success");
        });
      } else {
        player.pause();
        player.removeAttribute("src");
        player.load();
      }
    }

    function readAudioFile(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onerror = () => reject(new Error(`Could not read ${file.name}.`));
        reader.onload = () => {
          if (typeof reader.result !== "string") {
            reject(new Error(`Could not read ${file.name}.`));
            return;
          }
          resolve(reader.result);
        };
        reader.readAsDataURL(file);
      });
    }

    async function handleMusicUpload(event) {
      const input = event.target;
      const file = input.files && input.files[0];
      if (!file) return;
      input.value = "";
      if (!file.type.startsWith("audio/")) {
        setStatus(byId("pageNotice"), "Please choose an audio file.", "error");
        return;
      }

      try {
        const audioUrl = await readAudioFile(file);
        byId("presetMusicSelect").value = "";
        playAudio(audioUrl);
        setStatus(byId("pageNotice"), `${file.name} added as background music.`, "success");
      } catch (error) {
        console.error("Music upload failed", error);
        setStatus(byId("pageNotice"), error.message || "Could not read the selected music file.", "error");
      }
    }

    function triggerEffect() {
      if (typeof window.confetti !== "function") {
        setStatus(byId("pageNotice"), "The visual effect could not load. You can still create and share your gift.", "error");
        return;
      }
      if (byId("effectSelect").value === "confetti") {
        window.confetti({ particleCount: 100, spread: 70, origin: { y: 0.6 } });
      } else {
        window.confetti({ particleCount: 60, spread: 100, scalar: 1.2, shapes: ["star"] });
      }
    }

    function generateMessage() {
      const suggestions = {
        Love: "You are my heart, my soul, and my best friend. Life with you is my favorite adventure!",
        Birthday: "Wishing you a year filled with laughter, joy, and unforgettable moments!",
        Anniversary: "Cheers to another year of beautiful memories, shared smiles, and timeless love!"
      };
      byId("giftMessage").value = suggestions[byId("aiOccasion").value] || suggestions.Love;
      updatePreview();
      triggerEffect();
    }

    function makeShareUrl() {
      if (!window.LZString) throw new Error("The link-sharing library did not load. Check your internet connection and try again.");
      const payload = {
        t: byId("giftTitle").value,
        m: byId("giftMessage").value,
        b: byId("bgSelect").value,
        p: photoList,
        a: currentAudioUrl,
        e: byId("effectSelect").value
      };
      const compressed = window.LZString.compressToEncodedURIComponent(JSON.stringify(payload));
      const url = new URL(window.location.href);
      url.search = "";
      url.hash = "";
      url.searchParams.set("g", compressed);
      return url.href;
    }

    function openShareModal() {
      try {
        lastGeneratedUrl = makeShareUrl();
      } catch (error) {
        console.error("Could not create gift link", error);
        setStatus(byId("pageNotice"), error.message || "Could not create the share link.", "error");
        return;
      }

      byId("modalShareUrl").value = lastGeneratedUrl;
      byId("modalQrCode").replaceChildren();
      setStatus(byId("shareStatus"), "");
      if (window.location.protocol === "file:") {
        setStatus(byId("shareNotice"), "This file is being opened locally. Publish it on GitHub Pages first so other people can open the link.", "error");
      } else {
        setStatus(byId("shareNotice"), "Anyone with this link can view the gift. Photos are included in the link.");
      }

      if (typeof window.QRCode === "function") {
        try {
          new window.QRCode(byId("modalQrCode"), { text: lastGeneratedUrl, width: 160, height: 160 });
        } catch (error) {
          console.error("Could not create QR code", error);
          setStatus(byId("shareStatus"), "The link is ready, but it is too long for a QR code. Copy or share the link instead.", "error");
        }
      } else {
        setStatus(byId("shareStatus"), "The QR-code library did not load. You can still copy or share the link.", "error");
      }

      const modal = byId("shareModal");
      modal.style.display = "flex";
      modal.setAttribute("aria-hidden", "false");
      byId("closeModalButton").focus();
    }

    function closeShareModal() {
      const modal = byId("shareModal");
      modal.style.display = "none";
      modal.setAttribute("aria-hidden", "true");
      byId("createShareButton").focus();
    }

    async function copyGeneratedLink() {
      try {
        await navigator.clipboard.writeText(lastGeneratedUrl);
        setStatus(byId("shareStatus"), "Link copied to clipboard.", "success");
      } catch (error) {
        console.error("Clipboard copy failed", error);
        byId("modalShareUrl").focus();
        byId("modalShareUrl").select();
        setStatus(byId("shareStatus"), "Clipboard access is unavailable. Select the link above and copy it.", "error");
      }
    }

    async function shareNative() {
      if (typeof navigator.share !== "function") {
        await copyGeneratedLink();
        return;
      }
      try {
        await navigator.share({ title: "A special gift for you", text: "A little gift from the heart", url: lastGeneratedUrl });
      } catch (error) {
        if (error.name !== "AbortError") {
          console.error("Native sharing failed", error);
          setStatus(byId("shareStatus"), "Could not open the share menu. Copy the link instead.", "error");
        }
      }
    }

    function downloadQr() {
      const image = byId("modalQrCode").querySelector("img");
      if (!image) {
        setStatus(byId("shareStatus"), "There is no QR code to download. Copy the share link instead.", "error");
        return;
      }
      const downloadLink = document.createElement("a");
      downloadLink.href = image.src;
      downloadLink.download = "Giftoo-QR.png";
      downloadLink.click();
    }

    function openDirectPage() {
      if (lastGeneratedUrl) window.open(lastGeneratedUrl, "_blank", "noopener,noreferrer");
    }

    function decodeLegacyPayload(value) {
      const binary = window.atob(decodeURIComponent(value));
      const bytes = Uint8Array.from(binary, (character) => character.charCodeAt(0));
      return new TextDecoder().decode(bytes);
    }

    function loadGiftFromUrl() {
      const params = new URLSearchParams(window.location.search);
      const encodedGift = params.get("g") || params.get("gift");
      if (!encodedGift) return;

      try {
        if (!window.LZString) throw new Error("The link-decoding library did not load.");
        const json = window.LZString.decompressFromEncodedURIComponent(encodedGift) || decodeLegacyPayload(encodedGift);
        const payload = JSON.parse(json);
        if (!payload || typeof payload !== "object" || Array.isArray(payload)) throw new Error("Gift data is invalid.");

        byId("giftTitle").value = typeof payload.t === "string" ? payload.t : "";
        byId("giftMessage").value = typeof payload.m === "string" ? payload.m : "";
        byId("bgSelect").value = ["default", "hearts-gif", "romantic-stars"].includes(payload.b) ? payload.b : "default";
        byId("effectSelect").value = ["hearts", "confetti"].includes(payload.e) ? payload.e : "hearts";
        const photos = Array.isArray(payload.p) ? payload.p.filter((photo) => typeof photo === "string" && photo.startsWith("data:image/")) : [];
        photoList.splice(0, photoList.length, ...photos);
        renderSlider();
        if (typeof payload.a === "string" && (payload.a.startsWith("https://") || payload.a.startsWith("data:audio/"))) playAudio(payload.a);

        document.querySelector(".editor-panel").hidden = true;
        document.querySelector(".container").classList.add("gift-view");
        updateBackground();
        updatePreview();
        window.setTimeout(triggerEffect, 500);
      } catch (error) {
        console.error("Invalid gift link", error);
        setStatus(byId("pageNotice"), "This gift link is invalid or incomplete.", "error");
      }
    }

    byId("giftTitle").addEventListener("input", updatePreview);
    byId("giftMessage").addEventListener("input", updatePreview);
    byId("giftMessage").addEventListener("keydown", (event) => {
      if (event.key === "Enter" && !event.shiftKey) {
        event.preventDefault();
        openShareModal();
      }
    });
    byId("bgSelect").addEventListener("change", updateBackground);
    byId("photoInput").addEventListener("change", handlePhotosUpload);
    byId("chooseMusicButton").addEventListener("click", () => byId("musicFileInput").click());
    byId("musicFileInput").addEventListener("change", handleMusicUpload);
    byId("presetMusicSelect").addEventListener("change", (event) => playAudio(event.target.value));
    byId("effectSelect").addEventListener("change", triggerEffect);
    byId("generateButton").addEventListener("click", generateMessage);
    byId("createShareButton").addEventListener("click", openShareModal);
    byId("prevButton").addEventListener("click", () => moveSlide(-1));
    byId("nextButton").addEventListener("click", () => moveSlide(1));
    byId("copyButton").addEventListener("click", copyGeneratedLink);
    byId("nativeShareButton").addEventListener("click", shareNative);
    byId("downloadQrButton").addEventListener("click", downloadQr);
    byId("openPreviewButton").addEventListener("click", openDirectPage);
    byId("closeModalButton").addEventListener("click", closeShareModal);
    byId("shareModal").addEventListener("click", (event) => {
      if (event.target === byId("shareModal")) closeShareModal();
    });
    document.addEventListener("keydown", (event) => {
      if (event.key === "Escape" && byId("shareModal").getAttribute("aria-hidden") === "false") closeShareModal();
    });
    byId("sliderWrapper").addEventListener("pointerdown", (event) => { pointerStartX = event.clientX; });
    byId("sliderWrapper").addEventListener("pointerup", (event) => {
      if (pointerStartX === null) return;
      const difference = pointerStartX - event.clientX;
      pointerStartX = null;
      if (difference > 40) moveSlide(1);
      else if (difference < -40) moveSlide(-1);
    });

    updatePreview();
    updateBackground();
    window.addEventListener("DOMContentLoaded", loadGiftFromUrl, { once: true });
  </script>
</body>
</html>

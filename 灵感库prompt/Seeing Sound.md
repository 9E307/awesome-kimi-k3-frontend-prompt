# 标题
Seeing Sound

## 演示链接
https://visionsound.ok.kimi.link/

## 提示词
## Sound Made Visible — Production Prompt

Create **“Sound Made Visible”**, a polished, public-facing interactive science experience built with **Three.js**. Turn sound into beautiful, responsive 3D forms that anyone can explore without musical or scientific knowledge. Deliver a **deployed website** and a **cinematic MP4 showcase video**.

The experience should feel like a premium interactive science museum: visually striking, immediately understandable, playful, and scientifically honest. Design for a broad audience across desktop and mobile.

### FIRST EXPERIENCE

Open with a luminous, slowly moving sound sculpture against a deep midnight background. Use warm ivory typography, electric cyan, violet, and restrained coral accents. Keep the composition elegant and the interface minimal.

Display the headline **“See what sound can do.”** Offer three clear entry points:

- **Play a Sound** — immediately start a curated audio example and its visualization.
- **Use My Microphone** — visualize the user’s voice or surrounding sounds.
- **Upload Audio** — explore a local audio file.

Let visitors experience something compelling with one click. Provide built-in examples so microphone access and file uploads are optional. Before playback, run a clearly identified silent preview.

### THREE WAYS TO EXPLORE

**1. Sound Sculpture — artistic audio visualization**

Create a sculptural field of particles and flowing ribbons that responds smoothly to live audio analysis. Map low frequencies to broad movement, mid frequencies to structural detail, and high frequencies to fine shimmering accents. Let loudness influence scale and energy without abrupt jumps.

Give speech, clapping, humming, and music visibly different responses. Label this mode as an artistic interpretation of sound.

**2. Wave Explorer — understand the signal**

Display an animated waveform alongside a frequency spectrum, with an optional spatial presentation. Explain plainly: the waveform shows how the signal changes over time; the spectrum shows its frequency content.

Include an interactive tone generator with clearly labeled frequency and volume controls. Let users compare a pure tone with a richer tone and see the difference. Use gentle default volume, smooth audio transitions, and an obvious stop control. Explain that visual amplitude represents the input signal level, not calibrated physical loudness.

**3. Resonance Patterns — explore vibration**

Show a virtual vibrating plate with sand-like particles forming nodal patterns. Provide several carefully implemented vibration modes that users can explore by changing the excitation frequency.

Base the patterns on a documented plate model with defined geometry and boundary conditions. Explain that particles collect near regions of minimal vibration. Label the experience as an educational simulation and describe its simplifications. Keep decorative effects visually distinct from the scientific model.

### MAKE DISCOVERY EASY

Offer short, contextual invitations such as **“Try humming a steady note,” “Clap once,”** and **“Compare a low tone with a high tone.”** Pair each with an immediately visible response.

Keep advanced controls collapsed. Provide pause, reset, full-screen, sensitivity, and motion-intensity controls. Preserve clear audio-source and playback states when switching modes. Use a brief, accessible explanation for each mode without covering the scene.

### AUDIO AND PRIVACY

Use the **Web Audio API** for playback and analysis, with stable smoothing and sensible normalization. Request microphone access only after the user selects it. Process microphone input and uploaded files locally; do not upload, store, or transmit their audio.

Show a persistent microphone indicator and a clear **“Stop microphone”** control that releases the microphone. Disable microphone monitoring through speakers to prevent feedback. Handle permission denial, unavailable devices, unsupported files, silence, and interrupted audio gracefully.

Use verified CC0 or appropriately licensed built-in audio, and include an asset-license manifest.

### VISUAL AND TECHNICAL QUALITY

Use refined particle rendering, convincing depth, controlled bloom, smooth animation, and restrained camera movement. Keep text crisp and controls readable. Avoid aggressive flashing and excessive visual noise.

Build a maintainable Three.js application with GPU-efficient geometry and shaders, reusable buffers, capped pixel density, adaptive particle counts, and quality presets. Analyze audio locally so simultaneous visitors do not require server-side audio processing. Serve optimized static assets through CDN-compatible hosting.

Target smooth desktop performance and usable performance on mid-range phones. Measure actual frame rate, memory use, and loading behavior on the tested devices. Pause unnecessary rendering when the page is hidden and properly release audio and GPU resources.

Support touch, keyboard navigation, visible focus states, screen-reader-accessible controls, and reduced motion. Provide a lightweight 2D fallback when WebGL is unavailable. Ensure essential information is understandable without color or audio alone.

### DELIVERABLES

Deploy the finished website to a working public **HTTPS URL**.

Produce a downloadable **30–45-second, 1920 × 1080 H.264 MP4** with synchronized licensed audio, rendered from the actual application. Show a pure tone becoming a waveform, a richer sound transforming the sculpture, and a sequence of resonance patterns. Use deterministic playback and stable frame capture for clean results.

Deliver the source project, local assets, license manifest, documented scientific models and visualization mappings, deployment instructions, and a concise verification report. Test the deployed experience across desktop and mobile layouts, microphone permission flows, audio playback, uploads, mode switching, accessibility, and video playback. Report the tested environments and remaining limitations accurately.

**DO NOT INCLUDE:** decorative effects presented as physical simulations, microphone permission requests on page load, audio uploads to a server, autoplaying sound, speaker feedback, overwhelming controls, rapid flashing, unreadable labels, placeholder functionality, broken mobile layouts, watermarks, or unsupported claims of production readiness.

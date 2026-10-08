# 标题
Hanging Gardens of Babylon

## 演示链接
https://babylongardens.ok.kimi.link/

## 提示词
Create **“Hanging Gardens of Babylon”**, a production-quality interactive 3D website built with **Three.js**, accompanied by a polished cinematic **MP4 video** rendered from the same scene.

**VISUAL DIRECTION**  
An artistic reconstruction of the legendary Hanging Gardens: monumental terraced architecture rising from a desert riverbank, warm sandstone and sun-dried brick, cobalt-blue glazed tile bands, restrained golden reliefs, vaulted galleries, broad staircases, shaded courtyards, and cascading water channels. Every terrace supports lush palms, cypress trees, flowering shrubs, and trailing vines. Combine a clearly readable stepped silhouette with rich architectural detail at close range. Warm late-afternoon sunlight, soft atmospheric depth, realistic material roughness, subtle weathering, and convincing scale. Aim for the finish of a premium architectural visualization and museum exhibition website.

**RESEARCH AND TEXTURES**  
Search the web for visual references of Babylonian architecture, glazed bricks, sandstone, carved reliefs, ancient irrigation, and regional vegetation. Find high-resolution public-domain, CC0, or appropriately licensed images for texture creation; document their sources and licenses. Download permitted assets and serve them locally. Use suitable photographs as source material for seamless PBR textures and UV-projected details, correcting perspective, removing baked directional lighting, and matching color and scale. Build base-color, roughness, and normal maps where appropriate. Bake ambient occlusion and indirect lighting from the actual scene into optimized texture atlases, keeping animated foliage and water compatible with the lighting setup. Preserve sharp architectural details without visible seams, stretched UVs, or repetitive texture patterns.

**3D SCENE AND INTERACTION**  
Create coherent, explorable architecture with believable structural supports, connected stairs, accessible terraces, and carefully composed vegetation. Animate flowing water, gentle foliage movement, and subtle airborne particles. Begin with a cinematic approach from the river toward the gardens, then transition smoothly into user control. Support orbit, zoom, and constrained exploration, with curated viewpoints for the river gate, vaulted gallery, irrigation terrace, and summit garden. Include a guided journey following water from the upper reservoir through the terraces. Provide daylight and sunset modes with consistent lighting. Prevent camera clipping, movement through walls, and disorienting transitions.

**WEBSITE EXPERIENCE**  
Use elegant, restrained typography and minimal controls that let the scene dominate. Include a clear loading indicator, an introductory title, an “Explore the Gardens” action, viewpoint navigation, a guided-tour control, and a video download. Identify the reconstruction briefly as an artistic interpretation. Design responsive layouts for desktop and mobile, support keyboard navigation and reduced motion, and provide a polished fallback when WebGL is unavailable. Keep all controls functional and all interface text finished.

**PRODUCTION ENGINEERING**  
Use modular Three.js code and an asset pipeline designed for reliable deployment. Optimize geometry, draw calls, shadows, vegetation, and texture memory with instancing, level of detail, compressed GLB assets, KTX2/Basis textures, and progressive loading. Match texture resolution to visible detail. Use physically based materials, correct color management, controlled tone mapping, and restrained post-processing. Target smooth performance on modern desktops and usable performance on mid-range mobile devices, with adaptive quality settings and capped pixel density. Measure and report actual performance on the tested devices. Handle loading failures gracefully and release unused GPU resources.

**VIDEO DELIVERABLE**  
Produce an actual downloadable **30–45-second, 1920 × 1080 MP4**, encoded as H.264 with broad playback compatibility. Render from the finished Three.js scene using deterministic animation timing and stable frame capture. Sequence a river-level establishing shot, a passage through the arches, an ascent beside cascading water, and a final aerial reveal. Maintain smooth camera movement, consistent lighting, and clean frames without website controls. Include subtle water and garden ambience using appropriately licensed audio.

**DELIVERY AND ACCEPTANCE**  
Build and deploy the complete website to a working public HTTPS URL. Deliver the MP4 file, complete source project, locally hosted assets, asset-license manifest, and concise build and deployment instructions. Verify the deployed website, asset loading, every interaction, responsive layouts, browser compatibility, and video playback. Check for console errors, missing textures, camera collisions, animation defects, and performance bottlenecks. Report any remaining limitations honestly. The final result must be a finished, deployable experience with the visual consistency, reliability, and polish expected of a professional production.

**DO NOT INCLUDE:** placeholder geometry, flat image backdrops replacing the main architecture, unlicensed textures, hotlinked assets, broken controls, fake loading or export buttons, stretched textures, excessive bloom, plastic-looking stone, floating vegetation, flickering shadows, camera clipping, unfinished mobile layouts, watermarks, or unsupported claims of historical accuracy.

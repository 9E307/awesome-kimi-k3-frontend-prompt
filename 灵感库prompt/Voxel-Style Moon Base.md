# 标题
Voxel-Style Moon Base

## 演示链接
https://mmqlumufq4eqo.ok.kimi.link/

## 提示词
Build an interactive Three.js scene: a futuristic moon base rendered as voxel art, presented as a tabletop diorama. The base sits on a wooden table inside a room whose walls are a pixel-art starfield; a big pixel Earth hangs on the back wall and rises as the lunar day passes. The default view is a person standing at the front edge of the table, looking down at the model.
Everything on the table is built from unit cubes, with face culling, per-vertex ambient occlusion, per-voxel color dither, and chunked meshing so the scene stays smooth at millions of voxels. The diorama must contain at least 5,000,000 individual voxel blocks.
The base should feel dense and lived-in: rolling regolith with craters, boulders, and rover tracks; glowing habitat domes linked by tunnels; a command tower; greenhouses; a launch pad with a rocket and a small hopper pad; a solar farm; a reactor with radiators; propellant tanks; a regolith processing plant; radio dishes; a rover garage; container yards with a crane; floodlights and light strips everywhere.
An elevated engineering rail loop circles the whole base, and a lunar transport train (engine plus ore cars) runs on it continuously, passing a loading gantry at the plant and a small transit station at the front.
The scene is alive: astronauts walk between modules, rovers drive the roads with dust puffs, a hopper flies between pads, dishes rotate, the crane works, a drone circles the tower, a mining robot digs in a crater. A long day/night cycle drives hard shadows and the Earthrise; optional meteor showers and dust storms can be triggered.
Clicking things does something and the camera flies there: the rocket launches, the tower toggles all lights, the garage sends rovers on patrol, a pad calls the hopper, the greenhouse flips its grow-lights, astronauts wave and walk off, the train can be followed, clicking the ground makes a dust ripple.
Camera: damped orbit, right-drag pan, wheel zoom from a single astronaut out to the whole room, double-click reset, number keys for preset views.
UI: a single sci-fi bottom toolbar (pause, speed, time slider, day cycle, launch, hopper, patrol, meteors, storm, lights, labels, shadows, sound, reset, help) and a loading overlay while the world builds. Nothing floats or clips. Colorful and detailed. Use Three.js r128; textures, posters, and sound effects can be separate asset files.

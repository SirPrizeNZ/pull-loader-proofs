# Proof 1: hand pull loader in a financial app

[Watch the Note 10+ recording](media/financial-app-hand-proof.mp4)

![Hand holding the financial interface](media/financial-app-hand-proof.jpg)

The hand enters as the user pulls down, feels for the surface, closes around it and pulls it back up. The accounts view appears to crease with the hand. The bottom navigation stays in place.

The balance stays the same. Only the interface suffers.

## What happens, in order

1. **Pull.** The list receives the scroll first. When it is already at the top, unused downward movement opens a gap. The code adds resistance as the gap grows and moves the arm into view.
2. **Release.** Releasing below 100 dp cancels the gesture. Releasing at or above that distance starts the sequence and settles the gap toward 140 dp.
3. **Feel for the surface.** The arm swings to +28°, back to centre, to −20°, then back. A short contact sequence lowers the fingertips onto the surface.
4. **Close and gather.** Transparent hand frames close the fingers over 210 ms. Cloth shading gathers over 260 ms at the same contact point. Both start together. A soft haptic tick marks the grip.
5. **Pull upward.** The refresh callback fires as the yank begins. The hand moves offscreen over 220 ms. A snapshot of the accounts content bends toward the fist while travelling upward.
6. **Settle.** The content targets a small −9 dp overshoot over 450 ms, then springs to rest. Cloth shading fades, the snapshot is discarded and the current interface is drawn again.

The durations above are animation settings in the implementation. They are not measured performance results.

## How the code creates the illusion

**The hand is a sequence of images.** The hand and sleeve were prepared as transparent rendered frames. The app selects a frame from a grip value between 0 and 1, then positions and rotates it. Lighting and depth are already in those images.

**The folds are a separate layer.** Matching grey cloth frames are placed over the interface using `BlendMode.Hardlight`. Mid-grey leaves the interface unchanged; darker regions shade the fold valleys and brighter regions lift the ridges. The hand and cloth use measured offsets so their contact points line up.

**The interface bends too.** The code captures the content into an image and draws it through a 24 × 44 mesh. Vertices near the fist move inward and upward. Vertices lower down move less. That makes the middle lead the upward pull. The bottom navigation sits outside this deforming content.

## More detail

<details>
<summary>Gesture, frame selection and movement</summary>

Android uses Jetpack Compose, a nested-scroll connection, coroutine animations and `drawBitmapMesh`. There is no live 3D scene or cloth simulation running on the phone.

The gesture resistance is:

```text
resistance = 1 / (1 + offset / (1.8 × threshold))
offset += 0.6 × unusedDragDistance × resistance
```

The search rotations last 560, 380, 260 and 300 ms respectively. The contact frames play to frame 5 at 30 × 1.4 frames per second. A further 140 ms reach transition precedes closing. The grip tick occurs 90 ms into closing; the sequence holds another 230 ms before the yank.

```text
handFrame = round(grip × (handFrameCount − 1))
clothFrame = grip × (clothGripFrameCount − 1)
```

There are 12 search, 17 contact and 15 grip images. The grip/exit cloth sequence contains 47 images at a nominal 30 fps. After grip, elapsed time advances the cloth sequence through the yank and settling frames. Two neighbouring cloth frames are drawn with complementary alpha weights to soften frame changes.

An arm root above the top edge rotates the sleeve and hand together. A damped wrist lag follows angular velocity; the wrist does not move as a rigid extension of the arm. A short velocity-based stretch adds follow-through during exit.

</details>

<details>
<summary>How the snapshot is deformed</summary>

The mesh has 1,125 vertices. This simplified pseudocode shows the main haul; the implementation also adds a local pinch around the fingertips:

```text
verticalInfluence = (1 − y / screenHeight)^1.5
distance = (x − pinchX) / (0.75 × screenWidth)
horizontalInfluence = 0.7 + 0.3 × exp(−distance²)

newX = x + (pinchX − x) × 0.07 × (haul / 110dp) × verticalInfluence
newY = y + contentOffset − haul × horizontalInfluence × verticalInfluence
```

The local pinch attracts nearby vertices toward the contact point with a Gaussian falloff over a 120 dp radius. Its strength rises with contact/gather and falls as the sheet becomes taut. The haul control targets 56 dp before springing back.

The snapshot is captured when search begins. It is used during grip and yank; if capture fails, the content continues as a translated live layer. UI updates arriving while the snapshot is visible appear when the sequence resets.

</details>

<details>
<summary>How the image layers were prepared</summary>

A posed hand and sleeve were rendered with consistent lighting and a fixed camera. The light cloth folds were shaped as animated geometry: a mound beneath the fist, wrinkles toward the corners and longer drapes during the yank. Each corresponding cloth pass hides the hand while retaining its shadow. The export turns a flat sheet into neutral mid-grey for Hardlight compositing.

The renderer produces the appearance; Android supplies the interaction, frame playback, layer registration and interface distortion. The public proof includes only the finished recording and still, not the frame library.

</details>

## What this recording proves

The supplied clip shows the pull, search, closing grip, upward exit and return to the original layout on the Note 10+. The displayed accounts are prototype artwork, and the refresh uses a 700 ms delay. It does not connect to a financial service. The animation finishes on its own schedule; a host refresh flag blocks another pull while the simulated operation is active.

[Recording details](verification.md)

---
id: blender-solid-objects
title: Building recognisable objects from primitives
triggers: [blender, 3d model, render a model, car, vehicle, chair, table, building]
tools: [blender]
status: approved
---

Every rule here was paid for by a render that came out wrong.

## Scale is a HALF-extent

Blender's add-operators make a **2-unit** cube and a **radius-1** cylinder, so
`scale` is the distance from the centre to the edge, not the full size:

    scale = [2.0, 1.0, 0.5]   ->  4.0 long, 2.0 wide, 1.0 tall

`Sb[2]` is therefore HALF the body height. Every formula below is written
against `Sb`, the half-extent. Getting this backwards is the single most
expensive mistake available here: it builds toys.

## Parts must not be inside each other

The commonest failure, and it raises no error: a part placed where another
part already is renders invisibly. A car came back as a plain red box because
all four wheels sat at `z=0.2, y=±0.7` while the body spanned `z 0..1, y ±1` -
correctly created, correctly coloured, completely swallowed.

Before placing a part, work out the span of the part it attaches to. A
primitive at `location=L` with `scale=S` occupies `L[i] ± S[i]`. A part is
visible only if it sticks OUT of that span.

## Wheels

Given a body at location `Lb` with scale (half-extent) `Sb`:

- wheel radius `r = 0.75 × Sb[2]`, so the wheel's diameter is about one and a
  half times the body's height. That looks far too big written down and is
  right on the render: the eye reads anything smaller as a trolley.
- the wheel's centre sits at `z = r`, so it touches the ground it rests on
- the body's underside belongs at about `1.2 × r` - above the axle, below the
  top of the wheel, so the body's lower edge crosses it the way a wheel arch
  does. That makes the body's centre `Lb[2] = 1.2 × r + Sb[2]`.
  Check it as a SPAN, not as a formula: wheel z 0.00..0.90, body underside
  0.54, so the body covers the top 40% of each wheel.
- wheels sit at `y = ±(Sb[1] + r×0.3)` - just OUTSIDE the body's width
- front and rear at `x = ±Sb[0] × 0.65`, inside the ends rather than at the corners
- a cylinder's axis is Z, so a wheel needs `rotation_deg = [90, 0, 0]`
- scale a wheel `[r, r, width]` with width about `r × 0.4`, and give ALL FOUR
  the same width. One wheel at half the width of the other three is a defect
  no position check looks for, and it happened.
- wheels need their own dark material. Left alone they render default white,
  and four white discs read as castors rather than tyres.

After rotating a wheel 90° about X its vertical reach is `±r`, not `±width`.
Check ground contact against `r`.

## Proportion, before any of the detail

- a car is roughly **4 long : 1.7 wide : 1.2 high**. Keep the whole silhouette
  longer than 2.5× its height or it reads as a van.
- the cabin sits **middle-to-rear**, not centred - offset it back by about
  `0.1 × Sb[0]`. That offset is the difference between a car and a bus.
- the cabin's height is about `0.6 × Sb[2]`, never more than the body's.
- one box at one height is a van whatever else is done to it. Three sections
  at three heights is a car. That is the recipe below, and it costs two extra
  calls.

## A car, as actual numbers

Not a template to paste - the worked example showing what the rules above
produce together. Take a car 4.0 long, 1.7 wide, on wheels of radius `r = 0.45`.

    Wheels    scale [0.45, 0.45, 0.18], rotation [90, 0, 0]
              at x = 1.4 and -1.75, y = ±0.985, z = 0.45
              -> each spans z 0.00..0.90. On the ground, under bonnet and boot.

    Body      the MIDDLE section only, not the whole car.
              scale [1.0, 0.85, 0.355] at x = -0.2, z = 0.895
              -> spans x -1.2..0.8, z 0.54..1.25.

    Cabin     the passenger box, the tallest part, set back.
              scale [0.7, 0.72, 0.225] at x = -0.2, z = 1.475
              -> spans z 1.25..1.70, meeting the body's top exactly.

    Bonnet    scale [0.65, 0.8, 0.28] at x = 1.45, z = 0.82
              -> spans x 0.8..2.1, z 0.54..1.10. Lower than the body.

    Boot      scale [0.55, 0.8, 0.305] at x = -1.75, z = 0.845
              -> spans x -2.3..-1.2, z 0.54..1.15. Same idea, shorter.

The three lower sections share an underside at `z = 0.54` and differ in
HEIGHT; that step down from cabin to body to bonnet is the whole silhouette.
The result spans 4.40 long and 1.16 tall above the wheels - keep that ratio
near 3.8. Anything much squarer is a van.

Give the bonnet and boot the SAME material as the body: different colours make
three objects, one colour makes one car.

## Silhouette first: draw it, do not stack it

Boxes give a toy. When the object has a recognisable outline, DRAW it:

- `add_profile` - the outline in metres, extruded. A car body is ONE side
  profile (plane "side", x = length, z = height): front bumper, bonnet rising
  to a raked windscreen, roof, rear window, boot, rear bumper - 8 to 14
  points - with `thickness` = the car's width (about 1.75), then a bevel.
  This replaces body + cabin + bonnet + boot.
- `add_lathe` - a profile turned round z: a tyre is [[0,-0.12],[0.22,-0.12],
  [0.31,-0.11],[0.34,-0.06],[0.34,0.06],[0.31,0.11],[0.22,0.12],[0,0.12]],
  then rotation [90,0,0]. Motors, hubs, bottles and lamps the same way.
- Leave room for the wheels: start the profile's underside ABOVE the wheel
  centres, or the wheels end up inside the body (it is checked).

Before building, write down the parts and their real sizes in metres. A thing
looks right when its proportions are right, not when it has more parts.

## Bevel everything manufactured

The cheapest large improvement available here, and the one most often skipped.

A primitive has mathematically perfect edges. No manufactured object does - a
car panel, a table top, a phone case all have a small radius where two faces
meet, and that radius is what catches a highlight. Without it the eye reads
"primitive" before it reads what the thing is supposed to be, however correct
the proportions are.

So after the parts are placed and before the render:

    set_modifier(target="Body",   kind="bevel")
    set_modifier(target="Cabin",  kind="bevel")
    set_modifier(target="Bonnet", kind="bevel")

Leave `amount` alone. It is derived from each object's own size, which is the
only way it can be right: 0.03 is a crisp edge on a car body and swallows a
wheel. Three calls, no arithmetic, and it is the difference between a render
that looks built and one that looks like a diagram.

`kind="smooth"` is subdivision, and it is the wrong tool for anything with
panels - it rounds a car body into a bar of soap. Use it for organic shapes.

## Let the derivations do their work

Omit what is computed for you. `key_energy`, backdrop dimensions and camera
distance are all derived from the scene: passing a guess replaces a correct
value with a worse one. A key light of 1 W rendered a completely black frame
because a number was supplied where silence would have worked.

**Set up once.** `add_camera`, `add_backdrop` and `add_three_point_lighting`
are called **once each**, after the objects exist. Moving something afterwards
does NOT invalidate them: at render time the camera is slid back over the
current scene until everything is in shot, the backdrop is re-fitted to what
the camera sees, and the exposure is recomputed for the distance.

The order is: build, `check_scene`, fix, set up, render.

## What will be checked before you render

Not advice. Every rule below is evaluated on the scene you built, by
arithmetic, and each violation comes back naming the object and the move that
fixes it. Call `check_scene` yourself and fix what it reports - getting them
right first costs nothing, getting them wrong costs a rebuild round.

```checks
- rule: outside
  parts: wheel
  of: body
  axis: y
- rule: on_ground
  parts: wheel
  tolerance: 0.06
- rule: not_swallowed
  parts: wheel
  by: body
  min_covered: 0.15
  max_covered: 0.45
- rule: same_size
  parts: wheel
- rule: mirrored
  parts: wheel
  axis: y
- rule: has_material
  parts: wheel,body,bonnet,boot,cabin
- rule: same_material
  parts: bonnet,boot
  of: body
- rule: has_modifier
  parts: body,bonnet,boot,cabin
  kind: BEVEL
- rule: level
  parts: body,bonnet,boot
  axis: z
  edge: low
- rule: spread
  parts: wheel
  of: body,bonnet,boot
  axis: x
  min: 0.55
- rule: sits_on
  part: cabin
  on: body
- rule: longer_than_tall
  parts: body,bonnet,boot,cabin
  ratio: 2.5
- rule: relative_size
  parts: wheel
  of: body
  axis: z
  min: 0.6
  max: 1.9
- rule: aspect
  parts: wheel
  long: z
  short: y
  max_ratio: 0.6
```

## Directions from an earlier Blender training

(Learned in the Blender window; with the typed tools the same intent is set_modifier, set_material and the sizes passed to add_object.)

### Round 1
- Align the object's local up axis with Blender's world Z axis before finalizing; verify by selecting the object and checking that its main height dimension runs along the Z axis.
- Create hollow parts using the Solidify modifier with a specified wall thickness (e.g., 0.2 × the object's smallest dimension) rather than modeling as a solid mesh.
- Validate proportions by comparing key dimension ratios to real‑world references (e.g., mug height ≈ 3.5 × inner radius, chair seat height ≈ 45 cm, leg length ≈ seat height − 10 cm) and adjust until they fall within a ±10% tolerance.

### Round 1
- After adding any primitive, press **Ctrl+F → Shade Smooth** (or use Object → Shade Smooth) and then enable **Auto Smooth** with a 30° angle in the object's Normals panel to eliminate visible faceting.
- Immediately after modeling, clear all rotations with **Alt+R**, set the object's rotation values to (0, 0, 0) and confirm its local Z axis aligns with world Z by checking the Transform panel’s orientation matrix.
- Create a new material, disable **Use Nodes**, set the Base Color to the required RGB (e.g., (0, 0, 1) for plain blue), set Specular to 0 and Roughness to 1, then assign this material to the object via the Material slot.

### Round 2
- When adding any primitive (e.g., Torus, Sphere), pass explicit parameters for location = (0,0,0) and rotation = (0,0,0) in the Add‑Mesh operator, then immediately apply Scale and Rotation (Ctrl+A → Apply Scale & Apply Rotation) to lock the geometry and guarantee world Z‑up alignment.
- After modeling, read the object's Dimensions property (object.dimensions) and compare each axis to the target size (e.g., major radius = 1.5, minor radius = 0.5). If any value falls outside a ±2% tolerance, adjust the object's Scale or Dimensions accordingly before applying transforms and rendering.

### Round 1
- After creating a material for the lamp head, open the Properties editor, go to the Material tab, enable 'Emission' and set its color to a warm yellow (e.g., RGB 1.0, 0.8, 0.4) with a strength of at least 5 so the sphere acts as a light source.
- For each of the three-point lights, select the light object, press Shift‑A → Light → [type], then add a 'Track To' constraint via Object > Constraints > Add Object Constraint > Track To; set Target to the main object (e.g., the lamp or bookshelf), set 'To' axis to '-Z' and 'Up' axis to 'Y' so the lights are aimed at the subject.
- Place the camera for a low-angle shot by selecting the camera, pressing N to open the sidebar, and under the Transform panel set the Z location lower than the object's base (e.g., Z = 0.3 m) and the X rotation to about 45°, then press Ctrl‑Alt‑Numpad0 to align the view to the camera.

### Round 2
- After adding the camera (Shift+A → Camera), select it and in the Item → Transform panel set Rotation to X = 90° (or π/2 rad), Y = 0°, Z = 0° for a straight‑on view; alternatively press Numpad 0, then press Shift+~ to enter Fly/Walk mode and use the mouse to align the view exactly to the object’s front, confirming the camera’s orientation is orthogonal to the model’s main plane.

### Round 3
- Before rendering, enable **Viewport Overlays → Bounding Box** (click the arrow on the overlay button in the top‑right of the 3D Viewport and check "Bounding Box"), then inspect that each new part’s Bounding Box extends beyond the parent object's Bounding Box to guarantee no invisible intersections.

### Round 4
- Before rendering, open the Properties editor, go to the Output tab (shortcut: Ctrl+F12 opens render, then click the tab with the printer icon), expand the 'Output' panel and type the full filepath (e.g., //output/compact_car.png) into the 'File Path' field; then press Enter to confirm – this guarantees the image is saved to the required location on the first render.
- To guarantee all four wheels are visible in a side‑on view, select the camera, press N to open the sidebar, go to the 'View' panel, set Location to X=0.0, Y=-3.0, Z=body_center_z (≈0.75 m for a 1.5 m tall car) and set Rotation (Euler XYZ) to X=90°, Y=0°, Z=0°; also increase 'Clip End' in the camera settings (e.g., 100 m) so the whole car fits within the frame.
- When adding a bevel to any manufactured part, use the menu: select the object, choose 'Add Modifier' → 'Bevel', then set 'Width' to 0.02 (≈2 % of the object's largest dimension) and 'Segments' to 3 (leaving 'Limit Method' at 'None'); this produces a visibly rounded edge on the first attempt.

<!-- reference -->

# Why these numbers

Everything here is for whoever maintains this skill. It is kept in the file
and never sent to an agent - see `skills.REFERENCE_MARKER`.

## Cabin and ground, in full

Sits ON the body, not floating above and not sunk in: `z = Lb[2] + Sb[2] +
Sc[2]`, so its underside meets the body's top. Make it shorter than the body
in both horizontal axes - roughly `0.55 ×` the length and `0.9 ×` the width.
A cabin the same width as the body and centred on it is the commonest "it is
technically correct and still looks wrong" outcome.

## Ground

A subject touches the floor. The backdrop's floor is `z = 0`, so the lowest
point of the lowest part belongs at `z = 0` - for a wheeled thing, the wheels,
with the body riding above them.

## Where the primitives run out

(Written before add_profile and add_lathe existed - those now give the raked
windscreen, the roof line and real tyres. What follows still holds for boxes.)

Worth knowing so effort goes where it can pay. With boxes and cylinders you
reach a recognisable car silhouette and no further: no wheel arches, no raked
windscreen, no rounded shoulders, no panel gaps. A render correct by every
rule above still reads as a toy, and that is the material rather than the
arithmetic. Do not spend rebuild rounds chasing it, and do not let a vision
critic spend them either.

## The half-extent error, made twice

The wheel rule originally read "a bit over a third of the body height" and was
coded `0.35 × Sb[2]`. `Sb` is a half-extent, so that is a third of HALF the
height - out by two. An agent followed it exactly and rendered a brick on
castors.

Its replacement, `z = r + Sb[2] × 0.45`, put the body underside at 0.12
against a wheel reaching 0.90, and a vision model reported the wheels as
half-submerged. The same class of error, made while fixing the first one, by
someone who had just written a paragraph about it.

Both were found by looking at a render. Neither was found by reading the file.
That is the argument for the `checks` block: prose cannot check itself.

## The setup thrash

One run moved the subject, correctly concluded its camera framing was now
stale, and re-ran the setup: sixteen `add_camera` calls, six backdrops and six
lighting rigs. The reasoning was sound and the premise was false - the server
re-derives framing, backdrop fit and exposure at render time. Nothing in the
tools' replies said so, so the model paid for the same correction sixteen
times and hit the call ceiling with work still to do. The tools now say so.

## Why `longer_than_tall` names four parts

Because `Body` stopped being the car. Once the recipe split the shape into
bonnet, body and boot, the body alone measured 2.0 long by 1.2 tall - a ratio
of 1.67, which the rule failed, correctly, for a car that was actually 4.1
long. A rule naming one object cannot measure a silhouette made of several.

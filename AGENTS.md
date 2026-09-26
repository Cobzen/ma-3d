# MA 3D — Codex-instruktioner

Du arbejder på **MA 3D**, et letvægts 3D-sketching/blockout-værktøj til concept artists.

Programmet skal føles mere som **at tegne i 3D** end som traditionel CAD-software.

Det primære workflow er:

**tegn simple masser → form dem → find kamera/komposition → vurder value/depth → eksportér til paintover**

Appen skal fungere godt på:
- Desktop
- iPad
- iPhone

Appens entry point er:

`index.html`

## Kerneprincip

**Den faktiske geometri er altid source of truth.**

Faces, outlines, selection, snapping, Push/Pull og rendering skal altid udledes af den faktiske geometri.

Vedligehold ikke separat visuel geometri, som kan komme ud af sync med den rigtige geometri.

Hvis et eksisterende subsystem gentagne gange kræver patches, så foretræk en enklere refaktorering frem for flere special-cases.

## Produktfilosofi

MA 3D skal være:

- hurtigt
- intuitivt
- taktilt
- minimalt
- stabilt
- nyttigt til komposition og paintover

Det skal IKKE udvikle sig til:

- CAD-software
- Blender
- en parametrisk modeller
- et præcisionsværktøj til engineering
- et feature-tungt program

Foretræk få værktøjer, der fungerer ekstremt godt.

## Primære modelleringsværktøjer

Kerneværktøjerne er:

- Select / Move
- Brush
- 2D Outline
- Push / Pull
- Modify

Tegnefamilier:

**Brush**
- Curve
- Box Brush

**2D Outline**
- Polyline
- Box
- Circle
- Sphere

Sphere hører under Outline-familien, fordi brugeren tegner et 2D-aftryk/gesture, som skaber en 3D-form.

## Navigation

Navigationens kvalitet er ekstremt vigtig.

Desktop:
- Orbit
- Pan
- Zoom

iPad/iPhone:
- finger-navigation
- Apple Pencil til modellering

Ændr ikke navigationens sensitivity uden en konkret grund.

Pan, orbit og zoom skal være uafhængige systemer, så ændringer i én ikke påvirker de andre.

Vær særligt opmærksom på iOS Home Screen / standalone web-app-adfærd.

## Geometri

Foretræk én samlet geometri-model.

Creation tools må gerne starte forskelligt, men når geometrien er skabt, skal værktøjerne arbejde på faktisk mesh-geometri frem for mange inkompatible object types.

Tænk konceptuelt:

**Creation → Geometry → Editing → Rendering**

Push/Pull ændrer geometri.

Modify ændrer geometri.

Cut ændrer geometri.

Outlines genereres fra geometri.

Face picking genereres fra geometri.

Undo gemmer/gendanner geometri og state.

Undgå special-adfærd baseret på, hvordan et objekt oprindeligt blev skabt, medmindre det er absolut nødvendigt.

## Outlines

Outlines skal:

- repræsentere faktiske model-/synlige kanter
- opdatere efter hver geometrisk ændring
- aldrig vise triangulering
- aldrig vise interne hjælpelinjer
- være tydelige og mørke

Patch ikke manglende outlines manuelt.

Ret i stedet geometry-to-outline-logikken.

## Push / Pull

Push/Pull er en kernefunktion og skal være stabil.

Det skal:

- virke på plane faces
- tydeligt forstå push versus pull-retning
- være forudsigeligt fra forskellige kameravinkler
- opdatere geometrien korrekt
- opdatere outlines med det samme
- virke på geometri, der allerede er ændret af Push/Pull eller boolean-operationer

Ofre ikke Push/Pull-stabilitet, når geometri-systemet ændres.

## Tegning på faces

Brugeren skal kunne tegne direkte på eksisterende geometri.

En klikket face skal naturligt kunne blive tegneplan.

Workplanes skal være så usynlige/simple som muligt.

Default:
- ground plane når intet andet er relevant
- klikket face ved tegning på geometri
- eksplicit View Workplane kun når det er nødvendigt

## Add / Cut

Normal tegning skal skabe/tilføje geometri.

Cut skal føles mere som en modifier end som en kompliceret modeling mode.

Undgå separate geometri-systemer for Add og Cut.

## Undo / Redo

Undo/Redo skal forblive hurtigt og pålideligt.

Reducer aldrig bevidst Undo-funktionalitet som performance-workaround.

Ændr ikke history-arkitekturen uden at teste:

- create object → undo → redo
- move → undo
- Push/Pull → undo
- Modify → undo
- Cut → undo
- Delete All → undo

## Views

MA har kompositions-/render-views som:

- Workspace
- Masse
- Notan / Shadow
- Depth

Hvert view skal huske sine egne indstillinger.

Skift mellem views må ikke nulstille Depth range, shadow direction eller andre view-specifikke værdier.

Alle views skal bruge samme kamera og geometri.

## Export

Exports er lavet til concept-art paintover.

Supportér:

- current transparent PNG
- alle views som separate transparente PNG-filer
- layered PSD til Procreate

Batch-export skal bruge præcis samme kamera-framing og safe frame.

Export må ikke permanent ændre det aktive view eller scene-state.

## UI

Hold den primære toolbar ekstremt lille.

Den ønskede retning er cirka:

**DRAW | OUTLINE | SELECT | PUSH/PULL | MODIFY | VIEW | MENU**

Sekundære funktioner hører hjemme i menuer.

Eksempler:

- filer
- shortcuts
- toolbar settings
- group/ungroup
- scenes
- export-varianter
- avancerede view settings

Brugeren kan selv vælge, hvilke toolbar-sektioner der er synlige.

Tilføj ikke permanente toolbar-knapper, medmindre funktionen bruges konstant.

## Shortcuts

Shortcuts skal forblive bruger-konfigurerbare.

Vigtige defaults:

- Escape → Select / Move
- Space → unassigned som default

View Workplane-shortcut skal være en rigtig toggle:

første tryk → åbn
andet tryk → luk

Hold-modifiers skal kun være aktive, mens knappen holdes nede.

## Eksisterende funktionalitet

Når kode ændres, skal eksisterende fungerende adfærd bevares, medmindre opgaven specifikt siger noget andet.

Efter ændringer i geometri eller arkitektur skal mindst dette kontrolleres:

1. Drawing
2. Drawing on faces
3. Push/Pull
4. Modify
5. Move
6. Outlines
7. Add/Cut
8. Undo/Redo
9. Orbit
10. Pan
11. Zoom
12. iPad touch navigation
13. View switching
14. PNG export

## Refactoring-regel

Lad være med at stable fixes oven på et subsystem, der grundlæggende er forkert.

Hvis flere bugs kommer fra samme arkitektur, så stop og forenkl/refaktorér arkitekturen.

Foretræk:

**ét klart system**

over:

**mange special-cases**

Refaktorér ikke unrelated fungerende systemer uden en konkret grund.

## Coding approach

Før du ændrer kode:

1. Find den faktiske implementation, der er involveret.
2. Forstå dataflowet.
3. Afgør om problemet er arkitektonisk eller lokalt.
4. Lav den mindste rene ændring, der løser det egentlige problem.

Efter kodeændringer:

1. Tjek JavaScript-syntaks.
2. Tjek for duplicate DOM IDs.
3. Tjek references til omdøbte/fjernede funktioner.
4. Bekræft at initialization stadig gennemføres.
5. Gennemgå diff'en for utilsigtede ændringer.

Undgå store blinde search/replace-operationer.

## Git-workflow

Repositoryet bruger Git.

Før større ændringer:

- tjek `git status`
- forstå hvilken branch du står på
- ødelæg aldrig brugerens uncommitted arbejde

Lav fokuserede commits med beskrivende commit-messages.

Når en opgave er færdig og valideret:

- commit ændringen
- push den aktuelle working branch til GitHub

Brug aldrig force-push.

Omskriv ikke Git history, medmindre brugeren specifikt beder om det.

Push ikke direkte til `main`, medmindre brugeren specifikt beder om det.

Foretræk en beskrivende working branch som fx:

`codex/pushpull-direction`

eller

`codex/simplify-toolbar`

## Kommunikation

Når en opgave afsluttes, rapportér kort:

- hvad der blev ændret
- den faktiske root cause ved bugfixes
- hvad der bevidst ikke blev rørt
- hvad der blev testet
- hvilken commit/branch der blev brugt

Påstå aldrig at noget er testet, hvis det kun er blevet inspiceret statisk.

## Nuværende prioritet

Prioriteten er nu **forenkling og stabilitet**, ikke flere features.

Se aktivt efter muligheder for at reducere:

- duplicate state
- overlappende geometri-repræsentationer
- special-case-logik
- toolbar-kompleksitet
- gentagne rendering/update paths

Slutmålet er, at en concept artist kan åbne MA 3D og på få minutter:

**sketche masser → etablere perspektiv/komposition → eksportere → paintover**

# 05 — Narrative and presentation

How story content becomes staged, voiced, localized scenes. The storylet and battle systems are
named here but designed elsewhere.

## Storylets

- Narrative content is authored as **storylets**: units with preconditions over world state and
  effects on it.
- The defining requirement is **cross-storylet narrative coherence**. Storylets are validated
  against each other and against the shared world state, so that combinations of storylets
  produce a story that is consistent, not just individually valid ones.
- The system is being developed in `/c/dev/mass-effect-storylet-study` and will be specified in
  its own documents.
- **Outputs this pipeline consumes:**
  - scene requests (who is present, where, what is said and decided)
  - line IDs for every line of dialogue
  - the world-state schema, which also drives the save system
  - references to locations and characters, by ID

## Battle system

- Implemented per game, in whatever form the game needs.
- It reads and writes the same world state as storylets, and uses the shared character,
  animation and level pipelines.

## Dialogue cinematics director

The system that turns a conversation into a staged scene. With combinatorial storylets, staging
every scene by hand isn't possible, so this is the second most important presentation system
after the narrative engine.

- **Inputs:** participants, location anchor, dialogue lines with emotion tags, and choice points.
- **Outputs:** staging marks, camera shots and cuts, framing, look-at targets, gesture choices,
  and timing aligned to the VO.
- **Templates and rules,** following known practice in the genre (shot/reverse-shot, eyelines,
  the 180-degree rule, cutaways on reactions).
- **Human overrides** for important scenes, stored as edits on top of the generated result, so a
  change to the underlying lines or the location regenerates the scene and keeps the overrides
  where possible.
- **Escalation to Sequencer** for hand-built cinematics, which reuse the same actors and anchors.

## Voice-over

- **Line IDs from storylet content** are the key into everything: VO scripts, recording
  tracking, facial animation, subtitles and localization.
- **TTS temp VO** for every line, regenerated when text changes, so the whole game is playable
  and reviewable before recording.
- **Recording management:** script export per actor and session, take selection, and status
  tracking per line.
- **Processing:** loudness normalization, trimming, and feeding audio-driven facial animation.

## Localization

- String tables generated from storylet content and UI, keyed by line ID.
- Localized VO, or subtitles only, per language.
- Facial animation regenerated per language when the language is voiced.
- Pseudo-localization in CI to catch truncation and hard-coded strings.

## UI

- Dialogue wheel or choice UI, codex and journal, map, inventory, and per-game battle UI.
- The journal and codex are driven by world state and storylet metadata rather than written
  separately.

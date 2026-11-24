---
layout: default
title: MumbleGen Documentation
---

[Home](./) · [API reference](api.html) · Support: [Discord](https://discord.gg/VNmNpzVAEB)

Version 1.0 · Unreal Engine 5.8 · Windows (Win64)

Free Windows demo: [MumbleGen_Demo_Win64_v1.0.zip](https://github.com/enquel/MumbleGen-Demo/releases/tag/v1.0)

---

MumbleGen gives your characters a gibberish voice. Each letter of a line becomes a short syllable, played through a MetaSound with pitch, speed, variance, moods, punctuation pauses and letter skipping. While a character talks, its voice drives visuals: a glowing material, Niagara parameters, a mouth that opens and closes, a nodding head and typewriter subtitles that stay in sync with the syllables. Every speaker is its own component, so any number of characters can talk at the same time.

Platform: Windows (Win64), Unreal Engine 5.8. Full C++ source is included; everything is exposed to Blueprint.

## Two-minute demo

1. Copy the `MumbleGen` folder into your project's `Plugins` folder and open the project. Enable the plugin if the editor asks.
2. In the Content Browser open the settings (the eye icon) and turn on **Show Plugin Content**.
3. Open `MumbleGen Content > Samples > Maps > L_MumbleGenDemo` and press Play.

Four characters hold a conversation, sometimes two at once. Keys:

| Key | Action |
|---|---|
| 1, 2, 3, 4 | Pick a speaker (it says a line) |
| Left / Right (or A / D) | Switch the picked speaker's voice preset |
| Space | The picked speaker says a line |
| T | Two speakers talk at the same time |
| C | Conversation on or off |
| H | Hide or show the panels |
| Mouse | Click a speaker, a category tab or a preset |

## Quick start: Blueprint

1. Add a **Mumble Speaker** component to any actor (an NPC, a pawn, a prop).
2. Set its **Voice** to a preset from `MumbleGen Content > Voices > Presets`, for example `MV_Friendly`.
3. Call **Speak** with a line of text. It returns the line length in seconds.
4. Listen to **On Speech Started**, **On Syllable** and **On Speech Finished** for your own logic.

To see the character talk, add one or more visual drivers to the same actor (they find the speaker on their own):

- **Mumble Material Driver**: writes `MumbleEnvelope`, `MumbleFrequency`, `MumbleProgress` (scalars) and `MumbleColor` (vector) into dynamic material instances of the actor's meshes. Use **Component Tag** to limit it to some meshes (for example a mouth or a face screen).
- **Mumble Transform Driver**: moves, turns or scales a tagged component by the envelope (a jaw that drops, a mouth that stretches).
- **Mumble Morph Driver**: drives a morph target (default `MouthOpen`) on a skeletal mesh.
- **Mumble Niagara Driver**: sets the user parameters `Envelope`, `Frequency`, `Progress` and `Color` on a Niagara component.
- **Mumble Audio Bars**: an actor that shows a row or ring of bars that move with a speaker's voice.

For subtitles, create a **Mumble Subtitle Widget**, call **Add Speaker** for each speaker and add it to the viewport. It shows "Name: text" and types the text out in sync with the syllables. **Get Revealed Text** on the speaker gives the same text for your own widgets.

The demo characters (`Mumble Demo Character`) show the usual setup: a speaker, a material driver on the parts tagged `MumbleGlow`, a transform driver on the mouth (`MumbleMouth`) and one on the head (`MumbleHead`).

## The speaker

| Call | What it does |
|---|---|
| Speak (Text) | Says a line with the speaker's voice and Default Mood |
| Speak With Voice (Text, Voice) | Uses another voice for this line only |
| Speak With Mood (Text, Mood) | Says a line in a mood |
| Speak Line (Text, Voice, Mood) | Both |
| Speak External (Text, Envelope, Rate, Frequency) | Runs events, subtitles and visuals from audio you play yourself |
| Stop Speaking, Set Paused | Stop or pause the current line |
| Get Envelope, Get Frequency, Get Normalized Frequency, Get Speech Progress | Live values for your own visuals |
| Get Revealed Text, Get Revealed Character Count | Typewriter text |

Useful settings: **Pitch Offset** and **Speed Multiplier** (give two characters with the same voice their own sound), **Seed** (0 = new randomness each line; any other value = the same text always sounds the same), **Attenuation** (3D sound), **Play Audio** (off = silent captions), **Envelope Attack / Release / Gain** (how the visuals react).

The envelope is computed from the scheduled syllables and their measured loudness, not by listening to the audio, so it is frame-rate independent and also works with audio off (dedicated servers, tests, silent captions).

**Global Parameters** (optional): set a Material Parameter Collection (the plugin ships `MPC_MumbleGlobal`) and the speaker also writes `Envelope`, `Frequency`, `Progress` and `Color` into it. That is handy for a single narrator; with several speakers the last writer wins, so prefer the per-speaker drivers.

## Voices

A **Mumble Voice Preset** (data asset) holds a sample set, a MetaSound voice source, an effect amount, a color for the visuals, a category, and the speaking settings: base pitch, pitch range, sentence fall, question rise, speed, syllable interval, timing variance, skip chance, word / comma / sentence pauses, letter mapping, reading rules, digit and symbol sounds, volume, emphasis and mood strength.

35 presets ship, in five categories:

| Category | Presets |
|---|---|
| Humans | Friendly, Elder, Narrator, Merchant, Child, Grumpy, Murmur, Murmur Low, Murmur High |
| Creatures | Giant, Ogre, Beast, Alien, Alien Drone, Alien Organic, Spirit |
| Robots | Robot, Servo Glitch, Servo Buzz, Servo Smooth, Android, Tin Toy |
| Critters | Chirpy, Tiny, Squeaker, Chipmunk, Buzzy Bug |
| Effects | Radio Operator, Pilot, Whisper, Cyber Whisper, Soft Whisper, Vocoder Static, Vocoder Warm, Hologram |

Sounds come in two modes:

- **Generated** sample sets (`Voices/Generated`): syllables rendered by the plugin's own formant synthesiser (vowels and consonant-vowel pairs). Eight ship: Soft, Bright, Deep, Robot, Alien, Breath, Squeak, Growl.
- **Sampled** packs (`Voices/Packs`): recordings sliced into syllable-sized pieces. Nine ship.

Five MetaSound voice sources ship in `Voices/Sources`: Clean, Robotic (ring modulator and bitcrusher), Whisper (noise-excited), Radio (band-limited, saturated, with static that follows the voice) and Vocoder (a buzz carrier shaped by the voice). The preset's **Effect Amount** mixes the dry voice and the effect.

### Moods

Moods change pitch, pitch range, speed, loudness and the pitch contour: **Neutral, Happy, Angry, Sad, Scared, Question, Exclaim, Soft**. Use them per line (Speak With Mood, Default Mood) or inline in the text: `[happy]What a day! [sad]But it might rain... [pause]Oh well.` Tags are removed from subtitles. Unknown bracketed words stay in the text. Punctuation works on its own too: `?` rises at the end, `!` is louder, upper-case letters are stressed. A voice's **Mood Strength** scales every mood (0 turns them off). `Get Mood Profile` returns the built-in values.

### Text rules

- Any script works. Letters of every alphabet and ideographs (Latin with accents, Cyrillic, Greek, Arabic, Hebrew, Devanagari, Thai, kana, hangul, CJK and so on) map to syllables by a stable code-point rule: the same character always gives the same syllable with the **Per Letter** mapping. Accents and vowel signs written as separate code points belong to the letter before them.
- **Letter Mapping**: Per Letter (stable), Per Letter Class (vowels pick vowel syllables, consonants consonant syllables) or Random.
- **Read Letter Pairs**: `th sh ch ph wh ck ng qu gh oo ee ea ou ai oi au ie` count as one sound.
- **Reading Rules**: your own table of letter groups, each read as one chosen syllable, as one picked syllable, or as silence (for example `ough` silent, `x` always syllable 3). Rules are checked first, longest first.
- **Digit Sound** and **Symbol Sound**: Speak, Tick (a short, quiet, high tick) or Skip. Apostrophes, quotes and brackets are always silent.
- Spaces, commas and sentence ends pause; `...` and `?!` pause once plus a little.

## MumbleGen Studio (editor window)

**Tools > MumbleGen Studio** (or the console command `MumbleGen.Studio`):

1. Type a line, pick a voice, a mood and a seed, press **Play** to hear it.
2. **Export Line** writes a 16-bit 48 kHz mono **WAV file** and a **SoundWave asset** (choose the folders; leave one empty to skip it).
3. **Export Data Table** batch-exports every row of a data table with the row struct **Mumble Line Row** (Text, Voice, Mood, Seed; the row name becomes the file name). Rows without a voice use the window's voice.
4. **Make Set From Recordings** slices the WAV files of a folder into a sampled sample set (syllable length, loudness gate and levelling are adjustable).
5. **Make Generated Set** renders a new sample set with the synthesiser (source, pitch, glide, formant scale, length, vibrato, breathiness).

Point a voice preset at your new sample set and it is ready to use. The export mixes the syllables exactly as scheduled and applies a C++ version of the voice source's effect, so it sounds very close to, but not bit-identical with, what the MetaSound plays at runtime.

C++ users can call the same code: `FMumbleOfflineRenderer` (render, export, batch) and `FMumblePackBuilder` (slice recordings, generated sets, sets from clips).

## Make your own voice

- **New preset**: duplicate a preset, or create a **Mumble Voice Preset** data asset and set its sample set and voice source.
- **New sample set from your recordings**: record a few seconds of mumbling or nonsense words per file, then use **Make Set From Recordings** in the Studio. Or build a **Mumble Sample Set** by hand: add short waves and press **Analyze** so each syllable gets its envelope, length and pitch.
- **Your own MetaSound voice source** needs these inputs: `WaveA`, `WaveB` (Wave Asset), `PlayA`, `PlayB` (Trigger), `PitchA`, `PitchB` (Float, semitones), `GainA`, `GainB` (Float) and `Effect` (Float, 0 to 1). The speaker alternates between the A and B players so a syllable can ring while the next starts. Open one of the shipped sources to see the pattern.

## StoryCore bridge (Extras)

`Extras/MumbleGenStoryCore` connects MumbleGen to the StoryCore dialogue plugin. Copy the folder into your project's `Plugins` folder (next to `MumbleGen` and `StoryCore`) and enable it.

- When a dialogue line starts, the speaking actor's Mumble Speaker mumbles the line text. Actors without one get a speaker added (can be turned off).
- The line ends when the mumble ends, plus StoryCore's **Voice Line Extra Seconds**, the way voice-over lines work. To do this the bridge switches StoryCore's **Auto Advance Npc Lines** off while it is linked and ends lines itself; the setting is restored when the link ends. Lines with **Manual** duration still wait for the player. If Auto Advance Npc Lines was off already, lines wait for the player as before.
- Skipping a line stops its mumble; ending the dialogue stops all mumbling. Lines that have their own voice-over audio are left to StoryCore.
- **Project Settings > Plugins > MumbleGen StoryCore**: auto link, voices per speaker id, moods per line style, player lines.

## SynthVox bridge (Extras)

`Extras/MumbleGenSynthVox` lets a speaker use real SynthVox speech. Copy the folder into `Plugins` (next to `MumbleGen` and `SynthVox`) and enable it.

- Add **SynthVox Speech (MumbleGen)** next to a Mumble Speaker, pick a SynthVox voice and call **Say**. In **SynthVox** mode the line is synthesised off the game thread, played, and its audio is analysed into an envelope and a pitch track that drive the speaker: events, subtitles and every visual driver work as with a mumbled line. In **Mumble** mode Say mumbles. Set the mode per speaker, or per line with **Say With Mode**.
- Speech starts once synthesis has finished (SynthVox needs roughly 0.7 seconds per second of speech on a desktop CPU); **On Speech Ready** fires when it starts.
- The console command `MumbleGen.SynthVoxSampleSet <voice> <folder> <name>` renders a syllable sample set from a SynthVox voice (for example `/SynthVox/Voices/af_heart.af_heart /Game/Voices SS_Heart`), so a character can mumble in a TTS voice's timbre.

## Performance

A talking speaker uses one audio component and one MetaSound instance; it ticks only while speaking or while its envelope fades out. Drivers do no work of their own: they are called by their speaker when its values change. Syllable sounds are short and load with their sample set.

## Limits

- Win64 and Unreal Engine 5.8 only.
- No network replication: speech, events and visuals run where Speak is called. Call it on each client that should hear the line.
- The mouth follows loudness (mouth flap); there is no phoneme or viseme lip sync.
- Effects are fixed chains inside the voice sources; Effect Amount sets the mix. For other effects, make your own voice source with the same inputs.
- The Studio export is close to the runtime sound but not identical (C++ effects instead of the MetaSound).
- No Niagara system and no skeletal mesh ship; the Niagara and Morph drivers work with your own assets. The demo uses the material, transform and audio-bar drivers.
- Reading rules work on letters, not pronunciation; there is no dictionary.
- StoryCore bridge: it takes over NPC line timing while linked (see above).
- SynthVox bridge: speech starts after synthesis finishes, the voice's language must match the text, and the sample-set helper needs the SynthVox model loaded (about 400 MB).

## Support

Discord: https://discord.gg/VNmNpzVAEB

The full list of classes, functions and properties is in the [API reference](api.html).

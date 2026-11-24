---
layout: default
title: MumbleGen API reference
---

[Home](./) · [API reference](api.html) · Support: [Discord](https://discord.gg/VNmNpzVAEB)

---

Generated from the doc comments of the public headers. The same text is the tooltip of the Blueprint node or property.

# MumbleGen (runtime module)

## MumbleAudioBars.h

### EMumbleBarLayout

### AMumbleAudioBars

Audio-bar visualiser: a row or ring of cubes that rise with a speaker's envelope and scroll its recent history. Place it in a level and pick the speaker actor. One instanced mesh, per-instance color and height. Per-instance custom data 0 is the bar's level (0..1), for materials that glow with it.

- **Property** `TObjectPtr<AActor> SpeakerActor`: Actor whose Mumble Speaker drives the bars.
- **Property** `float Spacing`: Distance between bars (Line) or ring radius (Ring), in cm.
- **Property** `float BarWidth`: Bar width and depth in cm.
- **Property** `float MinHeight`: Height in cm at silence and at full envelope.
- **Property** `float HistorySeconds`: Seconds of history across half the bars.
- **Property** `float PitchShape`: How much the syllable pitch shapes the bars (0 flat, 1 strong).
- **Property** `TObjectPtr<UMaterialInterface> BarMaterial`: Material of the bars (gets MumbleColor from the speaker's voice).
- **Function** `void SetSpeaker(UMumbleSpeakerComponent* InSpeaker)`: Follows another speaker.
- **Function** `float GetBarLevel(int32 Index) const`: Current level (0..1) of a bar.

## MumbleDrivers.h

### UMumbleDriverComponent

Base of the visual drivers: follows one Mumble Speaker (the first one on the same actor unless set) and applies its envelope every frame it changes. Drivers do not tick on their own.

- **Property** `float EnvelopeScale`: Multiplier on the speaker's envelope before it is applied.
- **Function** `void SetSpeaker(UMumbleSpeakerComponent* InSpeaker)`: Follows another speaker (for example one on a different actor). Null: the first speaker on this actor.
- **Property** `TObjectPtr<UMumbleSpeakerComponent> Speaker`: Called whenever the speaker's visuals change.
### UMumbleMaterialDriver

Drives material parameters per instance: creates one dynamic material instance per target slot (once, cached) and writes scalar parameters MumbleEnvelope, MumbleFrequency (0..1), MumbleProgress and the vector MumbleColor. Two speakers never share these values.

- **Property** `FName ComponentTag`: Only meshes with this component tag. None: every mesh on the actor (unless targets are set with Set Targets).
- **Property** `TArray<int32> MaterialSlots`: Material slots to drive. Empty: all slots.
- **Property** `FName EnvelopeParameter`: Parameter names (None skips one).
- **Function** `void SetTargets(const TArray<UMeshComponent*>& InTargets)`: Drives these meshes instead of searching the actor.
- **Function** `TArray<UMaterialInstanceDynamic*> GetMaterialInstances() const`: The dynamic material instances this driver writes to (created on first use).
### UMumbleNiagaraDriver

Drives Niagara user parameters per instance: Envelope, Frequency (0..1), Progress (floats) and Color.

- **Property** `FName EnvelopeParameter`: User parameter names, without the User. prefix (None skips one).
- **Property** `FName ComponentTag`: Only a Niagara component with this tag. None: the first Niagara component on the actor.
### UMumbleMorphDriver

Opens a mouth (or any shape) through a morph target on a skeletal mesh, by envelope.

- **Property** `FName MorphTarget`: Morph target to drive.
- **Property** `float MaxWeight`: Weight at full envelope.
- **Property** `FName ComponentTag`: Only a skeletal mesh with this tag. None: the first skeletal mesh on the actor.
- **Function** `float GetLastWeight() const`: Weight written last.
### UMumbleTransformDriver

Moves, turns or scales a component by envelope: a jaw that drops, a mouth that stretches, an antenna that bobs. The offsets are added to the component's starting transform at full envelope.

- **Property** `FName ComponentTag`: Only the component with this tag. Required unless Set Target is used.
- **Property** `FVector ScaleOffset`: Added to the starting scale at full envelope.
- **Function** `void SetTarget(USceneComponent* InTarget)`: Drives this component and records its current transform as the rest pose.

## MumbleFormant.h

### EMumbleFormantSource

Sound source of a synthesised syllable.

### EMumbleConsonant

Consonant at the start of a synthesised syllable.

### FMumbleFormantSyllable

Everything needed to render one syllable.

- **Property** `FVector Formants`: Vowel formants in Hz (F1, F2, F3).
- **Property** `float FormantScale`: Formant frequency multiplier (above 1 sounds smaller, below 1 bigger).
- **Property** `float PitchStart`: Pitch at the start and end of the syllable in Hz.
- **Property** `float VibratoRate`: Vibrato rate in Hz and depth in semitones.
- **Property** `float Breathiness`: Noise mixed into the voiced source, 0 to 1.
- **Property** `float ConsonantColor`: Centre frequency of the consonant noise in Hz.
- **Property** `float Duration`: Total length in seconds.
- **Property** `int32 Seed`: Seed for noise and jitter.
### UMumbleFormantLibrary

Small source-filter syllable synthesiser (glottal-like source through three formant resonators, with consonant onsets) and the audio analysis used for envelopes. Every sample set that ships with MumbleGen is rendered by it.

- **Function** `TArray<float> AnalyzeEnvelope(const TArray<float>& Samples, int32 SampleRate, float Rate = 100.0f)`: Envelope analysis for Blueprint: RMS per 1/Rate seconds, scaled so the loudest value is 1.

## MumbleGenSettings.h

### UMumbleGenSettings

Project Settings > Plugins > MumbleGen.

- **Property** `TSoftObjectPtr<UMumbleVoicePreset> DefaultVoice`: Voice used by speakers that have none.

## MumblePlanner.h

### UMumblePlanner

Turns a line of text into timed syllables. Pure and deterministic for a given seed.

- **Function** `FMumblePlan PlanLine(const FString& Text, const UMumbleSampleSet* SampleSet, const FMumbleVoiceSettings& Settings, int32 Seed = 0, EMumbleMood Mood = EMumbleMood::Neutral)`: Plans a line: one syllable per letter (minus skipped ones), pauses for spaces and punctuation, pitch per syllable. Seed 0 picks a random seed.
- **Function** `FMumblePlan PlanLineWithVoice(const FString& Text, const UMumbleVoicePreset* Voice, int32 Seed = 0, EMumbleMood Mood = EMumbleMood::Neutral)`: Plans a line with a voice preset's sample set and settings.
- **Function** `FMumbleMoodProfile GetMoodProfile(EMumbleMood Mood)`: How a mood changes a voice (the built-in table).
- **Function** `FString StripTags(const FString& Text)`: The text without inline tags ([happy], [pause] ...), as subtitles show it.
- **Function** `float EstimateDuration(const FString& Text, const UMumbleVoicePreset* Voice)`: Estimated length in seconds of a line spoken with a voice (uses a fixed seed, so it is close but not exact for other seeds).

## MumbleSampleSet.h

### UMumbleSampleSet

A set of short syllable sounds that letters are mapped to. Make one with Create > Miscellaneous > Data Asset > Mumble Sample Set, add waves, then run Analyze (editor) so each syllable gets its envelope, length and pitch.

- **Property** `EMumbleSampleMode Mode`: Generated by the synthesiser or sliced from recordings (shown in pickers and docs).
- **Property** `FString SourceNote`: Where the sounds came from (for your own records).
- **Property** `TArray<FMumbleSyllable> Syllables`: The syllable sounds.
- **Function** `int32 GetNumPlayableSyllables() const`: Number of syllables with a wave.
- **Function** `void Analyze()`: Fills Duration, Base Frequency and Envelope of every syllable from its wave's audio data (editor only).

## MumbleSpeakerComponent.h

### UMumbleSpeakerComponent

Gives an actor a gibberish voice. Call Speak(Text): each letter becomes a short syllable played through the voice's MetaSound, with pitch, speed, variance, punctuation pauses and letter skipping from the voice preset. Every speaker keeps its own timeline, envelope and audio, so any number of characters can talk at once. The envelope (0 to 1) and frequency (Hz) are computed from the scheduled syllables, so they work on any frame rate and without an audio device. Visual drivers on the same actor (material, Niagara, morph target, transform) follow it automatically.

- **Property** `TObjectPtr<UMumbleVoicePreset> Voice`: The voice this speaker uses. Empty: the project default voice (Project Settings > Plugins > MumbleGen).
- **Property** `EMumbleMood DefaultMood`: Mood used by Speak and Speak With Voice. Inline tags such as [sad] still change it within a line.
- **Property** `FText SpeakerName`: Name shown by subtitles.
- **Property** `bool bPlayAudio`: Play sound. Off: the timeline, events and visuals still run (useful for silent captions or tests).
- **Property** `float VolumeMultiplier`: Volume multiplier on top of the voice's volume.
- **Property** `float PitchOffset`: Pitch shift in semitones on top of the voice (give two characters with the same voice a different pitch).
- **Property** `float SpeedMultiplier`: Speed multiplier on top of the voice.
- **Property** `TObjectPtr<USoundAttenuation> Attenuation`: 3D attenuation of the voice. Empty: the sound plays without attenuation (2D-like at the speaker).
- **Property** `TObjectPtr<USoundConcurrency> Concurrency`: Optional concurrency for the voice sound.
- **Property** `int32 Seed`: Random seed for lines. 0: a new random seed every line. Any other value: the same text always sounds the same.
- **Property** `float EnvelopeAttack`: Seconds for the envelope to rise toward a louder value.
- **Property** `float EnvelopeRelease`: Seconds for the envelope to fall toward a quieter value (frame-rate independent).
- **Property** `float EnvelopeGain`: Envelope multiplier before clamping to 0..1.
- **Property** `TObjectPtr<UMaterialParameterCollection> GlobalParameters`: Also write Envelope, Frequency (normalised), Progress and Color into this Material Parameter Collection. It is global: with two speakers at once the last writer wins. Use the per-speaker drivers for several speakers.
- **Property** `FMumbleSpeechStarted OnSpeechStarted`: A line started. Duration is its length in seconds.
- **Property** `FMumbleSyllablePlayed OnSyllable`: A syllable started playing.
- **Property** `FMumbleSpeechFinished OnSpeechFinished`: A line ended. Interrupted is true when Stop Speaking or a new line cut it short.
- **Property** `FMumbleEnvelopeUpdated OnEnvelopeUpdated`: Every frame while the envelope moves: envelope (0 to 1) and frequency (Hz). For custom visuals.
- **Function** `float Speak(const FString& Text)`: Speaks a line with this speaker's voice. Cuts off a line in progress. Returns the line length in seconds.
- **Function** `float SpeakWithVoice(const FString& Text, UMumbleVoicePreset* InVoice)`: Speaks a line with another voice (this line only). Returns the line length in seconds.
- **Function** `float SpeakWithMood(const FString& Text, EMumbleMood Mood)`: Speaks a line in a mood (happy, angry, sad, scared, question, exclaim, soft). Returns the line length in seconds.
- **Function** `float SpeakLine(const FString& Text, UMumbleVoicePreset* InVoice, EMumbleMood Mood)`: Speaks a line with a voice (null: this speaker's voice) and a mood. Returns the line length in seconds.
- **Function** `float SpeakExternal(const FString& Text, const TArray<float>& Envelope, float EnvelopeRate, const TArray<float>& Frequency)`: Starts a line whose audio someone else plays (for example a text-to-speech voice). The speaker runs events, subtitles and visuals from the given envelope (0 to 1, Envelope Rate samples per second) and optional frequency track (Hz, same rate). Returns the line length in seconds.
- **Function** `void StopSpeaking()`: Stops the current line (On Speech Finished fires with Interrupted).
- **Function** `void SetPaused(bool bInPaused)`: Pauses or resumes the current line.
- **Function** `void SetVoice(UMumbleVoicePreset* InVoice)`: Changes the voice for the next lines.
- **Function** `UMumbleVoicePreset* GetActiveVoice() const`: The voice in use: the line's voice while speaking, otherwise Voice or the project default.
- **Function** `float GetEnvelope() const`: Smoothed loudness 0 to 1. Drives mouths, glows and particles.
- **Function** `float GetFrequency() const`: Pitch of the current syllable in Hz (0 when silent).
- **Function** `float GetNormalizedFrequency() const`: Frequency mapped to 0..1 on a log scale from 60 Hz to 1000 Hz.
- **Function** `float GetSpeechProgress() const`: Progress of the current line, 0 to 1.
- **Function** `float GetSpeechTime() const`: Seconds since the line started.
- **Function** `FString GetCurrentText() const`: The current (or last) line's text.
- **Function** `const FMumblePlan& GetPlan() const`: The current (or last) line's plan.
- **Function** `int32 GetRevealedCharacterCount() const`: Characters of the current line revealed so far (typewriter subtitles).
- **Function** `FString GetRevealedText() const`: The part of the current line revealed so far.
- **Function** `FLinearColor GetVoiceColor() const`: The color of the active voice.
- **Function** `UAudioComponent* GetAudioComponent() const`: The audio component playing the voice (created on the first line; null while audio is off).
- **Property** `TObjectPtr<UAudioComponent> AudioComponent`: Name of the parameters written to Global Parameters.

## MumbleSubtitleWidget.h

### UMumbleSubtitleWidget

Typewriter subtitles synced to the syllables: one row per speaker ("Name: text so far"), the name in the voice color. Add speakers with Add Speaker; a row appears while its speaker talks and fades after Hold Seconds. Works as is (layout built in code); a Blueprint subclass can change fonts and colors through the properties.

- **Property** `int32 FontSize`: Text size.
- **Property** `FLinearColor TextColor`: Line text color.
- **Property** `FLinearColor BackgroundColor`: Row background.
- **Property** `float HoldSeconds`: Seconds a row stays after its line ends.
- **Property** `bool bShowWholeLine`: Show the whole line at once instead of typing it out.
- **Function** `void AddSpeaker(UMumbleSpeakerComponent* Speaker)`: Shows this speaker's lines.
- **Function** `FString GetShownText(UMumbleSpeakerComponent* Speaker) const`: Text a speaker's row shows right now (empty when hidden).
- **Property** `TObjectPtr<UVerticalBox> Rows`: Updates the rows now (called every frame by the widget; public for tests).

## MumbleTypes.h

### EMumbleLetterClass

Which letters a syllable sound stands for when a voice maps letters by class.

### EMumbleLetterMapping

How a voice picks the syllable sound for each letter.

### EMumbleVoiceEffect

The effect a voice source applies. Informational: the effect itself lives in the voice's MetaSound source.

### EMumbleMood

Emotion of a line. Changes pitch, pitch range, speed, loudness and the pitch contour. Also usable inline: "[happy]Hi! [sad]Bye."

### EMumbleCharacterSound

What a digit or symbol sounds like.

### EMumbleVoiceCategory

Grouping of voice presets in pickers.

### EMumbleSampleMode

How a pack of syllables was made.

### FMumbleMoodProfile

How a mood changes a voice. Values are applied at full strength; a voice's Mood Strength scales them.

- **Property** `float PitchOffset`: Semitones added to every syllable.
- **Property** `float PitchRangeScale`: Multiplier on the voice's pitch range.
- **Property** `float SpeedScale`: Multiplier on speaking speed.
- **Property** `float VolumeScale`: Multiplier on volume.
- **Property** `float EndContour`: Semitones of rise (positive) or fall (negative) over the last word of each sentence.
- **Property** `float Tremble`: Extra random pitch per syllable in semitones (a trembling voice).
### FMumbleReadingRule

A reading rule: letters that are read as one sound, or not at all. Checked before single letters, longest first.

- **Property** `FString Letters`: The letters, any case, for example "th", "ough" or "x".
- **Property** `bool bSilent`: Read as silence (the letters still show in subtitles).
- **Property** `int32 SyllableIndex`: Syllable to play. -1: picked from the letters like any letter.
### FMumbleSyllable

One syllable sound of a sample set, with the analysis the speaker uses to drive visuals without listening to the audio.

- **Property** `TObjectPtr<USoundWave> Wave`: The sound played for this syllable (mono, short).
- **Property** `FString Label`: A readable name such as "ba" (shown in tools and logs only).
- **Property** `EMumbleLetterClass LetterClass`: Which letters this syllable stands for when the voice maps letters by class.
- **Property** `FString PreferredLetters`: Letters (lower case) that always use this syllable when the voice maps per letter, for example "bp".
- **Property** `float BaseFrequency`: Pitch of the sound in Hz at its original speed. Drives the Frequency output.
- **Property** `float Duration`: Length of the sound in seconds at its original speed.
- **Property** `TArray<float> Envelope`: Amplitude envelope, 0 to 1, sampled at Envelope Rate. Filled by Analyze Sample Set or by the sample builder.
- **Property** `float EnvelopeRate`: Samples per second of Envelope.
### FMumbleVoiceSettings

Voice settings used to turn text into syllables. Held by a voice preset; also usable on their own from C++.

- **Property** `float BasePitch`: Base pitch shift in semitones (negative is deeper).
- **Property** `float PitchRange`: Random pitch spread per syllable in semitones (plus or minus).
- **Property** `float SentenceFall`: Pitch drop in semitones from the start to the end of a sentence, like natural speech.
- **Property** `float QuestionRise`: Pitch rise in semitones over the last word of a question.
- **Property** `float Speed`: Speaking speed multiplier. 2 is twice as fast.
- **Property** `float SyllableInterval`: Seconds per letter at speed 1.
- **Property** `float Variance`: Random timing variance per letter, as a fraction of the interval (0.2 is plus or minus 20 percent).
- **Property** `float SkipChance`: Chance (0 to 1) to skip a letter. Skipped letters take a short slot of silence; one-letter words are never skipped.
- **Property** `float WordPause`: Pause in seconds for a space between words (at speed 1).
- **Property** `float CommaPause`: Pause in seconds after , ; : and dashes (at speed 1).
- **Property** `float SentencePause`: Pause in seconds after . ! ? and line breaks (at speed 1).
- **Property** `EMumbleLetterMapping LetterMapping`: How letters choose their syllable.
- **Property** `bool bReadLetterPairs`: Read common letter pairs (th, sh, ch, ph, ck, ng, qu, oo, ee, ou, ai...) as one sound.
- **Property** `TArray<FMumbleReadingRule> ReadingRules`: Your own reading rules (letter groups read as one sound or as silence). Checked before the built-in pairs.
- **Property** `EMumbleCharacterSound DigitSound`: What digits sound like.
- **Property** `EMumbleCharacterSound SymbolSound`: What symbols such as # % & @ + = sound like.
- **Property** `float MoodStrength`: How strongly moods change this voice (0 none, 1 normal, 2 double).
- **Property** `float Volume`: Syllable volume, 0 to 2.
- **Property** `float EmphasisVolume`: Extra volume for upper-case letters and lines ending in an exclamation mark.
- **Property** `float VolumeVariance`: Random volume spread per syllable as a fraction (0.1 is plus or minus 10 percent).
### FMumbleSyllableEvent

One scheduled syllable of a line.

- **Property** `float Time`: Seconds from the start of the line.
- **Property** `float Duration`: Seconds the sound lasts at its pitch (pitch up plays faster).
- **Property** `int32 SyllableIndex`: Index into the sample set's syllables.
- **Property** `float PitchSemitones`: Pitch shift in semitones for this syllable.
- **Property** `float Volume`: Volume of this syllable.
- **Property** `int32 CharIndex`: Index of the letter in the line text.
- **Property** `FString Character`: The letter as text.
- **Property** `EMumbleMood Mood`: Mood in effect for this syllable.
- **Property** `bool bTick`: True for a digit or symbol tick rather than a syllable.
- **Property** `int32 CharCount`: Number of characters this syllable reads (2 for a letter pair such as "th").
- **Property** `float Frequency`: Frequency in Hz of the sound at this pitch (the syllable's base frequency times the pitch ratio).
### FMumblePlan

A whole line turned into timed syllables.

- **Property** `FString Text`: The line text as shown (inline tags such as [happy] removed).
- **Property** `EMumbleMood Mood`: Mood at the start of the line.
- **Property** `TArray<FMumbleSyllableEvent> Events`: Syllables in time order.
- **Property** `TArray<float> RevealTimes`: Time in seconds at which each character of Text is revealed (same length as Text). Drives typewriter subtitles.
- **Property** `int32 SkippedLetters`: Letters skipped by Skip Chance.
- **Property** `float Duration`: Seconds from the start to the end of the last sound.
### FMumbleLineRow

A line for data tables (batch export in the MumbleGen Studio window, or your own dialogue tables).

- **Property** `FString Text`: The line (inline tags such as [happy] work).
- **Property** `TSoftObjectPtr<class UMumbleVoicePreset> Voice`: Voice for this line. Empty: the default voice of the export.
- **Property** `int32 Seed`: Seed for the line's randomness (the same seed always sounds the same).

## MumbleVoicePreset.h

### UMumbleVoicePreset

A character voice: which syllables, which MetaSound voice source (clean or with an effect) and how to speak. Assign it to a Mumble Speaker component, or pass it to Speak With Voice.

- **Property** `FText DisplayName`: Name shown in pickers and the demo.
- **Property** `FText Description`: One line about how it sounds.
- **Property** `EMumbleVoiceCategory Category`: Group in pickers.
- **Property** `TObjectPtr<UMumbleSampleSet> SampleSet`: Syllable sounds.
- **Property** `TObjectPtr<USoundBase> VoiceSource`: MetaSound source that plays the syllables. It needs the inputs WaveA, PlayA, PitchA, GainA, WaveB, PlayB, PitchB, GainB and Effect (see the Docs). The plugin ships MSS_MumbleVoice_Clean, _Robotic, _Whisper, _Radio and _Vocoder. Empty: the speaker plays nothing but still runs its timeline, events and visuals.
- **Property** `EMumbleVoiceEffect Effect`: Which effect the voice source applies (label only).
- **Property** `float EffectAmount`: Strength of the voice source's effect, 0 to 1 (sent as the Effect input).
- **Property** `FMumbleVoiceSettings Settings`: Pitch, timing, pauses, letter mapping and volume.
- **Property** `FLinearColor Color`: Color handed to visual drivers (material Color parameter, Niagara Color user parameter).

# MumbleGenEditor (editor tools)

## MumbleOfflineRenderer.h

### FMumbleExportLine

One line to export.

### FMumbleExportResult

Result of one exported line.

### FMumbleOfflineRenderer

Renders lines to audio without playing them (editor only): mixes the voice's syllable sounds exactly as scheduled (pitch, volume, two overlapping players) and applies a C++ version of the voice source's effect. The export is close to what the MetaSound plays at runtime, not bit-identical.


## MumblePackBuilder.h

### FMumbleSliceOptions

How recordings are cut into syllables.

- **Property** `float TargetLength`: Preferred syllable length in seconds; cuts land on the quietest point near it.
- **Property** `int32 MaxSlicesPerFile`: Most syllables taken from one recording.
- **Property** `float MinLoudness`: Slices quieter than this fraction of the recording's loudest moment are dropped.
- **Property** `float TargetRms`: Every slice is levelled to this RMS (peaks limited to 0.95), so the pack plays evenly.
### FMumbleGeneratedSetSpec

Timbre of a generated (synthesised) sample set.

- **Property** `float Pitch`: Pitch in Hz.
- **Property** `float Glide`: End pitch divided by start pitch (below 1 falls, above 1 rises).
- **Property** `float FormantScale`: Above 1 sounds smaller, below 1 bigger.
- **Property** `float Duration`: Syllable length in seconds.
- **Property** `float VisualFrequency`: Frequency the visuals use when the sound has no pitch (breath). 0: the synth pitch.
### FMumbleWavData

Mono PCM read from a WAV file.

### FMumblePackBuilder

Builds sample sets: generated by the synthesiser, or sliced from recordings. Editor only.


## MumbleStudio.h

### UMumbleStudioSettings

Everything the MumbleGen Studio window edits (Tools > MumbleGen Studio).

- **Property** `FString Text`: The line to hear or export. Inline tags work: [happy] [sad] [angry] [scared] [question] [exclaim] [soft] [neutral] [pause].
- **Property** `int32 Seed`: The same seed always sounds the same.
- **Property** `FString ExportName`: Name of the exported WAV file and asset (SW_ is added to the asset).
- **Property** `FString AssetFolder`: Content folder for SoundWave assets, for example /Game/Audio/Mumble. Empty: no asset.
- **Property** `FDirectoryPath WavFolder`: Folder on disk for WAV files. Empty: no WAV file.
- **Property** `TObjectPtr<UDataTable> Lines`: Lines to batch export: a data table with the row struct Mumble Line Row. Rows without a voice use Voice above.
- **Property** `FDirectoryPath RecordingsFolder`: Folder of WAV recordings to slice into a new sampled pack.
- **Property** `FMumbleGeneratedSetSpec Generated`: Settings of a new generated set.
- **Property** `FString SetFolder`: Content folder and name of the new sample set.
### SMumbleStudio

The MumbleGen Studio window: type a line, pick a voice and mood, play it, export it, batch export, make sample sets.


# MumbleGenStoryCore (Extras bridge)

## MumbleStoryCoreLink.h

### UMumbleStoryCoreSettings

Project Settings > Plugins > MumbleGen StoryCore.

- **Property** `bool bAutoLink`: Link to the world's StoryCore component automatically (it registers itself in BeginPlay).
- **Property** `bool bLinesFollowMumble`: Lines last as long as their mumble, like voice-over lines: while linked, StoryCore's Auto Advance Npc Lines is switched off and the link ends each line itself (mumble length plus StoryCore's Voice Line Extra Seconds). The setting is restored when the link ends. If Auto Advance Npc Lines was already off, lines wait for the player.
- **Property** `bool bSkipLinesWithVoiceOver`: Lines that have their own voice-over (Audio) are left to StoryCore.
- **Property** `bool bMumblePlayerLines`: Also mumble the player's own lines (needs a Mumble Speaker on the player pawn, or Add Speaker When Missing).
- **Property** `bool bAddSpeakerWhenMissing`: Give a speaking actor a Mumble Speaker if it has none (StoryCore stand-ins, quick tests).
- **Property** `TMap<FName, TSoftObjectPtr<UMumbleVoicePreset>> SpeakerVoices`: Voice per StoryCore speaker id. Used when the speaker's own component has no voice, or always with Override.
- **Property** `bool bSpeakerVoicesOverride`: Speaker Voices win over the voice set on the actor's Mumble Speaker.
- **Property** `TMap<FName, EMumbleMood> StyleMoods`: Mood per StoryCore line style (the node's Style name), for example Angry -> Angry.
### UMumbleStoryCoreLink

Connects StoryCore dialogue to MumbleGen. When a dialogue line starts, its speaker actor's Mumble Speaker speaks the line text (with the speaker's voice, or the voice mapped to its speaker id); when the mumble ends the line ends, the way a voice-over line does. Skipping a line stops its mumble. Links itself to the world's StoryCore component.

- **Function** `void Link(UStoryComponent* Story)`: Links to a StoryCore component (done automatically when Auto Link is on).
- **Function** `UMumbleSpeakerComponent* GetCurrentSpeaker() const`: The speaker mumbling the current line (null between lines).
- **Function** `float GetSecondsUntilLineEnd() const`: Seconds until the link ends the current line (negative when it will not).
- **Property** `FMumbleStoryLineMumbled OnLineMumbled`: A line started mumbling.
- **Function** `void HandleLineStarted(UStoryDialogueInstance* Dialogue, const FStoryDialogueLine& Line)`: Ticks the link (auto link, line end countdown). Public for tests and custom loops.

# MumbleGenSynthVox (Extras bridge)

## SynthVoxSpeechComponent.h

### EMumbleSpeechMode

How a speaker says its lines.

- **Property** `EMumbleSpeechMode Mode`: Mumble or speak.
- **Property** `TObjectPtr<UTTSVoiceAsset> Voice`: SynthVox voice, for example /SynthVox/Voices/af_heart.
- **Property** `float Speed`: SynthVox speaking speed.
- **Property** `bool bPlayAudio`: Play the speech audio. Off: only the envelope, events and visuals run.
- **Property** `TObjectPtr<USoundAttenuation> Attenuation`: 3D attenuation of the speech. Empty: the Mumble Speaker's attenuation.
- **Property** `TObjectPtr<UMumbleSpeakerComponent> Speaker`: Mumble Speaker to drive. Empty: the first one on this actor.
- **Property** `FSynthVoxSpeechReady OnSpeechReady`: The speech is synthesised and playing (Duration in seconds).
- **Property** `FSynthVoxSpeechFailed OnSpeechFailed`: SynthVox could not speak the line (not ready, no voice, or no phonemes for the text).
- **Function** `float Say(const FString& Text)`: Says a line in this component's mode. Mumble mode returns the line length; SynthVox mode returns 0 and fires On Speech Ready when it starts.
- **Function** `float SayWithMode(const FString& Text, EMumbleSpeechMode InMode)`: Says a line in a given mode (this line only).
- **Function** `void Stop()`: Stops speech and any synthesis still running for this component.
- **Property** `TObjectPtr<UAudioComponent> Audio`: The SynthVox subsystem for a world (in the editor without a game running, a private game instance is made for it).


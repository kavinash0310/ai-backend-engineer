# Multimodal AI foundations

## What multimodal means

A multimodal system accepts or produces more than one data type, such as text, images, audio, video or structured sensor inputs. It may use separate encoders, map modalities into a shared representation, cross-attend between them, or feed modality-specific token sequences into a common Transformer. The phrase does not specify one architecture.

## 1. Image-language systems

An image encoder converts patches or regions into feature vectors/tokens. A language model may attend to these features or use a projection into its representation space. High-resolution images can be tiled or encoded at multiple scales; this changes context and compute.

Image-language tasks include captioning, visual question answering, OCR/document understanding, image search and grounding. A text answer about an image may sound certain while missing small text or spatial relationships. Evaluate perception separately from fluent wording.

## 2. Audio systems

Audio may be represented as waveform samples, spectrograms, learned acoustic tokens or discrete codec tokens. Core tasks:
- automatic speech recognition (audio → text);
- speech synthesis/text-to-speech (text → audio);
- speech translation;
- speaker diarization (who spoke when);
- audio event classification;
- real-time voice conversation.

A speech system may use cascaded components (ASR → language model → TTS) or an integrated audio-text model. Cascades are modular and debuggable but add latency and error propagation; integrated models may reduce handoffs but can be harder to inspect.

Audio sampling rate, channel count, duration, segmentation and noise handling are part of the API contract. Speaker identity and voice data can be sensitive biometric information.

## 3. Video and temporal inputs

Video is a sequence of frames plus timestamps and possibly audio. Processing every frame can be expensive; frame sampling may miss short events. Temporal ordering and synchronization matter. A model's apparent “video understanding” can be driven by selected still images, audio transcript or metadata, so evaluation should isolate these signals.

Store frame timestamps and source offsets so evidence can be reviewed. Consider face/location privacy, retention, upload size and processing latency.

## 4. Cross-modal alignment and fusion

- **Early fusion** combines modality representations near the input.
- **Late fusion** combines predictions/features after separate processing.
- **Cross-attention** lets one modality's representations query another.
- **Shared embedding space** trains related modalities to have comparable vectors.

Fusion choice controls what information can interact and how missing modalities are handled. A model trained on paired image-text data may align broad concepts but still fail exact counting, small text, temporal reasoning or domain-specific scenes.

## 5. Evaluation

Evaluate by modality and task:
- image: classification/detection/segmentation metrics and corruption slices;
- speech recognition: word/character error rates plus accents, noise and speaker slices;
- generation: intelligibility, factual content, safety and human listening;
- video: event detection, temporal localization and timestamp accuracy;
- cross-modal retrieval: recall/ranking and false match analysis.

Do not use one overall score to hide performance across languages, devices, disabilities or demographic groups. Human review may be needed; design it for accessibility and privacy.

## 6. Backend architecture considerations

Multimodal payloads are large. Use object storage and short-lived references instead of embedding large binary data in JSON or message queues. Validate media type, byte size, duration, resolution and codec; inspect actual content, not only extension. Use signed URLs carefully and bind access to authenticated tenant. Apply malware scanning/sandboxing to uploaded files.

Long-running processing should usually be asynchronous:
1. authenticate and validate metadata;
2. store encrypted object with limited retention;
3. create idempotent processing job;
4. process in isolated workers with quotas;
5. store result and source offsets;
6. notify or expose job status;
7. delete source/derived data per policy.

Streaming audio/video needs explicit chunk ordering, timestamps, backpressure, reconnection and cancellation behavior. Track model version and preprocessing to reproduce outputs.

## 7. Failure and safety cases

- OCR misses a negation, decimal or small-font exception.
- ASR transcribes a name/number incorrectly.
- Image model overlooks rare small objects.
- Audio leaks speaker identity.
- Video sampling misses a short event.
- Different modalities conflict (spoken words versus visible text).
- User uploads malformed/oversized or adversarial content.
- Model outputs a description not present in media.

Design confidence/abstention and human review for high-impact cases. Do not convert uncertain perception into an irreversible action without authoritative validation.

## Questions and answers

**Q: Does multimodal mean one model accepts everything natively?**  
A: No. The system may be a pipeline of specialized models or an integrated model with multiple encoders/decoders.

**Q: Why can a video answer be wrong even if its summary is fluent?**  
A: The model may miss frames, timestamps, small details, audio, or modality conflicts. Fluency measures language behavior, not perception correctness.

**Q: Is an audio transcript equivalent to audio understanding?**  
A: No. A transcript can omit tone, speaker identity, overlapping speech, sound events and uncertainty.

**Q: How should a backend handle large media uploads?**  
A: Validate and store binary data separately, process asynchronously with quotas, preserve offsets/provenance, and enforce access/deletion policies.

## Practice

1. Choose cascaded versus integrated architecture for voice customer support; list latency and debugging trade-offs.
2. Design an upload API that enforces bytes, duration, media type and tenant scope.
3. Create evaluation slices for accented speech and quiet audio.
4. Define how an answer cites an image page, video timestamp or audio interval.
5. Specify deletion behavior for original media, transcripts, embeddings and cached outputs.

## Completion check

You can explain image/audio/video representation patterns, common task types, evaluation risks and backend implications of large, sensitive multimodal data.
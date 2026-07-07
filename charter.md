# Project Charter: Military Aircraft Identifier

## What we're building (one sentence)
A model that looks at a photo of a military aircraft and identifies which of 8 U.S. aircraft it is (F-4, F-14, F-15, F-16, F/A-18, F-22, F-35, A-10).

## Cohort
Image (image classification).

## The data or tools we'll use
- **Primary data source:** Hugging Face — searching for existing labeled military-aircraft image datasets before building one from scratch, checking per-class coverage and class balance early (e.g., uneven counts like lots of F-16 images and few F-4 images would bias the model).
- **Supplemental data collection (Module 7 phase):** a search agent that pulls a user-specified number of images per aircraft (or spread evenly across all 8), used to fill in thin classes. Every collected image passes a human approval step before entering the dataset — nothing scraped is trusted on the agent's word alone.
- **Modeling tools:** Keras 3 on the PyTorch backend, Python virtual environment. Transfer learning with a pretrained CNN backbone + a small trainable classification head, softmax over the 8 classes. Images resized to a fixed input size (e.g., 224×224) with light data augmentation (flips, small rotations/zoom) on the training set only. Trained locally on a GPU.
- **Backbone plan (a deliberate two-model comparison, not just an upgrade):** MobileNetV2 (frozen) as the Module 6 walking-skeleton backbone — fast to iterate with, gets the pipeline running end-to-end quickly. EfficientNetV2B0 added in Module 7 specifically to test the hypothesis that MobileNetV2's lighter feature extraction won't resolve fine details well enough to separate visually similar aircraft (e.g., F-14 vs. F-15, both twin-tail; F-15 vs. F/A-18 in silhouette), while EfficientNetV2B0's deeper features should. Both backbones are trained and compared using confusion matrices, characterizing each one's fine-detail limitations rather than just picking whichever scores higher.
- **Data storage (Module 7 phase):** approved images and their verified labels are written to a SQLAlchemy database, supporting batched retraining.

## Definition of "good enough"
Before we build, we agree this project is good enough when:

**Metric:** classification accuracy, read alongside a confusion matrix — because a model could score decently by nailing the easy classes (A-10 is visually distinct) while never telling F-14 from F-15 apart. Per-class performance matters, not just the overall number.

**Evaluation protocol:** k-fold cross-validation, chosen over a single train/validation split because the dataset is likely modest in size and several classes are visually close, so a more stable accuracy estimate is worth the extra compute.

**Baseline to beat:** random chance for 8 classes = 12.5%; also beat a naive "always guess the most common class" baseline.

**Two-tier threshold:**
- **Floor (Module 6 walking skeleton):** the classifier runs end-to-end on a fixed Hugging Face dataset and clearly beats both baselines (above 12.5% chance and above the most-common-class guess).
- **Stretch (Module 7 "make it good"):** validation accuracy in the 80-85% range, with no single class collapsing to chance in the confusion matrix — so the score isn't carried by easy classes like the A-10 alone. This target is a judgment call based on how visually separable these classes are likely to be after fine-tuning, not a number pulled from a published benchmark; it will be revisited if early results suggest it's off.

## What we are NOT doing (scope guard)
- **Object detection / bounding boxes / real-time video** — this is single-image classification, not detection.
- **Variant/paint-scheme/sub-model recognition** — just the 8 top-level aircraft types.
- **Deployment / web app / phone app** — runs locally.
- **Out-of-distribution handling** — assumes every input image contains exactly one of the 8 listed aircraft; the model is not expected to handle images with no aircraft, a civilian plane, or a military aircraft outside the 8 classes.
- **The full active-learning pipeline is not part of the Module 6 floor.** Module 6 is one clean path: fixed dataset → train once → evaluate. The search agent, human-approval step, SQLAlchemy database, and batched retraining loop are Module 7 scope, added only after the walking skeleton works.

## Team & roles
Solo. Self-review documented in pull requests per the Sacred Flow.

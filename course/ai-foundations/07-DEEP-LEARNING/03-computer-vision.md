# Computer vision foundations

## 1. What computer vision systems do

Computer vision (CV) processes images or video. Common tasks:
- **classification**: one or more labels for an image;
- **localization/detection**: class plus bounding box for each object;
- **segmentation**: class label per pixel (semantic) or separated object masks (instance);
- **keypoint estimation**: positions of landmarks/joints;
- **OCR/document understanding**: text and layout extraction;
- **tracking**: associate objects across video frames.

A model choice depends on output structure, image resolution, latency, hardware, annotation cost and consequence of misses.

## 2. Image representation and preprocessing

An RGB image is typically a tensor with channels, height and width. Pixel scale may be 0–255 or normalized; channel order and layout vary by library. Resize/crop policy affects what is visible. Store preprocessing version with the model.

Augmentation creates transformed training examples. A horizontal flip may be valid for some natural images but invalid for text, directional signs, anatomy or labels that depend on orientation. Augmentation must preserve label semantics. Evaluate robustness on realistic corruptions (lighting, blur, compression, camera shift) rather than only training transformations.

## 3. CNNs and transfer learning

Convolutional networks use local filters and shared weights; see [CNN and sequence models](02-cnn-and-sequence-models.md) for kernels, shapes and receptive fields. In transfer learning, start from a model pretrained on a broad or related dataset, replace or adapt task-specific heads, and fine-tune some or all layers.

A common progression:
1. freeze pretrained feature extractor and train a new head;
2. measure validation behavior;
3. unfreeze selected layers at a smaller learning rate if data/compute allow;
4. compare against a frozen baseline;
5. evaluate domain shift and class-specific errors.

A pretrained model's data, license, domain and preprocessing are part of the decision. A model trained on natural images may not transfer directly to medical, satellite or industrial images.

## 4. Detection and segmentation metrics

For object detection, a predicted box is matched to a ground-truth box using intersection-over-union (IoU):
IoU = area(intersection) / area(union).
Average precision summarizes precision/recall over confidence thresholds; mAP averages over classes and often IoU thresholds. State the exact convention because benchmark definitions differ.

For segmentation, pixel accuracy can be misleading when background dominates. Intersection-over-union (IoU/Jaccard) and Dice/F1 are common overlap metrics. Review small objects and boundary quality separately.

Detection thresholds influence false alarms and missed objects. In safety contexts, report per-class recall, false-positive workload and confidence intervals; a single mAP may hide harmful failures.

## 5. Data splitting and annotation quality

Images from the same video, patient, site, camera, scene or near-duplicate crop should not be randomly split across train and test if deployment requires new entities. Use group, site or temporal splits. Annotation policies need consistent box boundaries, occlusion handling, small-object rules and class definitions. Track annotator agreement and adjudication.

Privacy is central for face, health, workplace and location images. Minimize raw data, control access, define retention and use approved storage. Consider whether the system should collect/retain images at all.

## 6. Deployment considerations

Image size and batch size influence memory and latency. Video multiplies frame processing and may require tracking or frame sampling. Edge deployment adds quantization, device and power constraints. Measure preprocessing, model and post-processing together. Define what happens to low-confidence predictions and whether a human must review.

## Questions and answers

**Q: What is the difference between classification and detection?**  
A: Classification labels an image; detection identifies object locations and labels, commonly with boxes.

**Q: Why use transfer learning?**  
A: Pretrained features can reduce data and compute needs for a related task. Domain mismatch, license and preprocessing still need review.

**Q: Is image augmentation always safe?**  
A: No. A transform is valid only if it preserves the label and intended semantics.

**Q: Why can high pixel accuracy hide poor segmentation?**  
A: Background may occupy most pixels, so a model can perform well overall while missing a small but important object.

## Practice

1. Select task formulation for identifying defects, counting people and reading invoice fields.
2. Compute IoU for two simple boxes and interpret the result.
3. Design a split that keeps video frames from one recording in one partition.
4. List which augmentations preserve labels in a traffic-sign dataset.
5. Create evaluation metrics for rare safety-critical objects and human review capacity.

## Completion check

You can distinguish CV task types, describe image tensor/preprocessing choices, explain transfer learning and metrics, and design leakage-aware, privacy-conscious evaluation.
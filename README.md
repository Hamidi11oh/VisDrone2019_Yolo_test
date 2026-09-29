# VisDrone2019 — YOLOv8n Baseline Experiment (10 + 20 Epochs)

## 1. Experiment purpose

This experiment establishes a baseline for **small-object detection in aerial/UAV images** using **YOLOv8n** on the VisDrone2019 dataset.

The main goal is not yet model improvement. The current goal is to:

1. build a reproducible YOLOv8n baseline,
2. observe class-level performance,
3. identify failure cases,
4. define the next research problem from the evidence.

This experiment is part of the research direction:

**2D Small-Object Detection → Model Improvement → Lightweight / Real-time → Edge Deployment → Autonomous / Robotic Vision**

Long-term 3D perception is considered a future extension, not an immediate target.

---

## 2. Environment

- OS: Windows 10
- Python: 3.10.21
- Ultralytics: 8.4.144
- PyTorch: 2.5.1+cu121
- GPU: NVIDIA GeForce RTX 3060 12 GB
- CPU: 24 logical CPUs
- RAM: 31.8 GB

Project directory:

```text
D:\Hamin\Yolo\VisDrone2019_Yolo_test
```

Built-in Ultralytics dataset YAML used in this experiment:

```text
C:\Users\JOOYOUNGBOK\anaconda3\envs\yolo_env\lib\site-packages\ultralytics\cfg\datasets\VisDrone.yaml
```

---

## 3. Dataset

Dataset: **VisDrone2019**

- Training images: 6,471
- Validation images: 548
- Classes: 10

Classes:

```text
pedestrian
people
bicycle
car
van
truck
tricycle
awning-tricycle
bus
motor
```

The dataset contains very dense images. During training, some images contained a very large number of objects, and Ultralytics automatically increased `max_det` because the maximum object count was high.

---

## 4. Baseline model

Model:

```text
YOLOv8n
```

Model size:

- Parameters: ~3.01 M
- GFLOPs: 8.1

No architectural modification was made in this experiment.

---

## 5. Experiment 1 — Initial 10 epochs

Command:

```bat
yolo detect train model=yolov8n.pt data=VisDrone.yaml epochs=10 imgsz=640 batch=8 device=0 workers=0 name=visdrone_baseline_10epoch
```

Output directory:

```text
D:\Hamin\Yolo\VisDrone2019_Yolo_test\runs\detect\visdrone_baseline_10epoch
```

### 10-epoch result

| Metric | Result |
|---|---:|
| Precision | ~34.5% |
| Recall | ~27.8% |
| mAP@0.5 | 24.8% |
| mAP@0.5:0.95 | 13.5% |

The best 10-epoch model showed a large class-level difference.

| Class | mAP@0.5 |
|---|---:|
| pedestrian | 25.5% |
| people | 19.4% |
| bicycle | 3.45% |
| car | 67.5% |
| van | 27.4% |
| truck | 22.5% |
| tricycle | 15.3% |
| awning-tricycle | 7.41% |
| bus | 33.0% |
| motor | 26.0% |

Initial observation:

- `car` is detected substantially better than the other classes.
- `bicycle` and `awning-tricycle` are especially difficult.
- Small, dense, and partially occluded objects remain problematic.

---

## 6. Experiment 2 — Additional 20 epochs

The 10-epoch checkpoint was used as the starting model for another 20-epoch training run.

Important: this was **not** a true `resume=True` continuation. It loaded the 10-epoch weights and started a new 20-epoch training run with the same VisDrone configuration.

Command:

```bat
yolo detect train model="D:\Hamin\Yolo\VisDrone2019_Yolo_test\runs\detect\visdrone_baseline_10epoch\weights\last.pt" data="C:\Users\JOOYOUNGBOK\anaconda3\envs\yolo_env\lib\site-packages\ultralytics\cfg\datasets\VisDrone.yaml" epochs=20 imgsz=640 batch=8 device=0 workers=0 name=visdrone_baseline_30epoch
```

Output directory:

```text
D:\Hamin\Yolo\VisDrone2019_Yolo_test\runs\detect\visdrone_baseline_30epoch
```

Training time for the additional 20 epochs was about **1 hour 48 minutes**.

---

## 7. Final 10 + 20 result

The best checkpoint from the second run (`best.pt`) reported:

| Metric | Result |
|---|---:|
| Precision | 40.9% |
| Recall | 30.1% |
| mAP@0.5 | 28.2% |
| mAP@0.5:0.95 | 15.6% |

The final epoch was approximately 28.3% mAP@0.5, so the best and final values are very close.

### Class-level mAP@0.5

| Class | mAP@0.5 |
|---|---:|
| pedestrian | 29.7% |
| people | 22.2% |
| bicycle | 5.21% |
| car | 70.4% |
| van | 32.4% |
| truck | 27.0% |
| tricycle | 19.2% |
| awning-tricycle | 9.01% |
| bus | 37.5% |
| motor | 29.8% |

### 10 epoch → additional 20 epoch change

- mAP@0.5: **24.8% → ~28.2%**
- mAP@0.5:0.95: **13.5% → 15.6%**
- Precision: improved to **40.9%**
- Recall: improved to **30.1%**
- `car`: **67.5% → 70.4%**
- `bicycle`: **3.45% → 5.21%**
- `awning-tricycle`: **7.41% → 9.01%**

Therefore, the additional training improved the baseline, but the difficult small-object classes are still much weaker than `car`.

---

## 8. What does “30 epochs” mean here?

For reporting, this experiment can be described as:

> **A total of 30 epochs of training exposure: 10 initial epochs + an additional 20 epochs initialized from the 10-epoch weights.**

For technical precision, it was **two separate training runs**, not one continuous 30-epoch `resume=True` run.

Recommended wording for a seminar/professor:

> VisDrone에서 YOLOv8n baseline을 먼저 10 epoch 학습한 후, 해당 weight를 이용해서 추가로 20 epoch 학습했습니다. 따라서 총 30 epoch 수준으로 실험을 확장했습니다.

---

## 9. Inference speed

The best validation run reported approximately:

- Preprocess: ~0.3 ms/image
- Inference: ~3.0 ms/image
- Postprocess: ~2.0 ms/image

These values are useful as a baseline for later lightweight/real-time comparisons.

---

## 10. Current interpretation

The current baseline is working correctly on the intended VisDrone dataset.

The most important research signal is **not simply that mAP is 28.2%**. The more important observation is the **large class gap**:

```text
car                 70.4%
bicycle              5.21%
awning-tricycle       9.01%
```

This suggests that the model has difficulty with some small and crowded object categories.

Qualitative observations from prediction/confusion results also indicate issues related to:

- missed detections / background predictions,
- pedestrian ↔ people confusion,
- car ↔ van confusion,
- tricycle ↔ awning-tricycle confusion,
- bicycle ↔ motor confusion,
- dense scenes,
- small object scale,
- occlusion and complex backgrounds.

A prediction image alone is not enough to prove a miss; ground-truth comparison is needed for a reliable failure analysis.

---

## 11. Why we should not train more yet

The next research step should be **failure-case analysis**, not another arbitrary increase in epochs.

The current experiment already gives enough evidence to ask:

> Why does YOLOv8n detect large/easier objects such as cars relatively well, but struggle with small and visually similar objects such as bicycles and awning-tricycles?

This question is more useful for research than simply increasing the epoch count again.

---

## 12. Next research steps

### Step 1 — Freeze the baseline

Keep the current 10 + 20 experiment as the baseline reference.

Save/record:

```text
results.png
confusion_matrix.png
PR_curve.png
F1_curve.png
P_curve.png
R_curve.png
val_batch0_pred.jpg
val_batch0_labels.jpg
best.pt
```

Do not modify the baseline results.

### Step 2 — Failure-case analysis

Analyze the difficult classes first:

1. bicycle
2. awning-tricycle
3. people / pedestrian
4. tricycle
5. motor

For each class, check:

- object size,
- occlusion,
- object density,
- background complexity,
- false negatives,
- false positives,
- class confusion.

### Step 3 — Define one research problem

Based on the failure analysis, choose one focused problem.

Possible directions supported by the current evidence include:

- improving small-object feature extraction,
- improving multi-scale feature fusion,
- preserving higher-resolution features,
- reducing confusion between visually similar classes,
- improving efficiency while improving small-object detection.

The exact direction should be selected after the failure analysis rather than assumed in advance.

### Step 4 — Build one improvement experiment

Use YOLOv8n as the baseline and change **one major component at a time**.

For example:

```text
Baseline
   ↓
One modification
   ↓
Same dataset / same evaluation protocol
   ↓
Compare metrics
   ↓
Inspect failure cases again
```

### Step 5 — Efficiency / deployment

After achieving a meaningful detection improvement, evaluate:

- inference speed,
- model size,
- GFLOPs / parameters,
- ONNX export,
- TensorRT or edge deployment.

This connects the work to the longer-term goal of **efficient perception for autonomous driving and robotic vision**.

### Step 6 — Long-term extension

After a strong 2D foundation is established, the research can later expand toward:

```text
2D Small-Object Detection
        ↓
Efficient / Real-time Vision
        ↓
Edge Deployment
        ↓
Autonomous / Robotic Vision
        ↓
Long-term 3D Perception
```

---

## 13. Recommended experiment log structure

For every new experiment, record:

```text
Experiment ID:
Date:
Dataset:
Model:
Epochs:
Image size:
Batch size:
Workers:
GPU:
Modification:
mAP@0.5:
mAP@0.5:0.95:
Precision:
Recall:
Best class:
Worst class:
Main failure case:
Next hypothesis:
```

This makes the project easier to reproduce and later convert into a paper.

---

## 14. Current project status

```text
[Done]
VisDrone dataset setup
        ↓
[Done]
YOLOv8n 10-epoch baseline
        ↓
[Done]
Additional 20 epochs
        ↓
[Current]
Failure-case analysis
        ↓
[Next]
Research problem definition
        ↓
[Next]
One controlled model improvement
        ↓
[Future]
Efficiency / edge deployment
        ↓
[Long-term]
Autonomous / robotic / 3D perception
```

---

## 15. One-sentence seminar summary

> We established a YOLOv8n baseline on VisDrone2019, extended training from 10 to an additional 20 epochs, and improved mAP@0.5 from 24.8% to about 28.2%; however, small and crowded object classes such as bicycle and awning-tricycle remain difficult, so the next step is failure-case analysis and evidence-based model improvement.

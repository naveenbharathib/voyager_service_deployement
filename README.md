# Voyager SDK Model Deployment Guide

This document explains how to deploy both **default** and **custom** models using the Voyager SDK AI Server.

---

# Prerequisites

Ensure the following are available before deployment:

- Voyager SDK installed
- AI Server running on port **5000**
- `deploy.py` available inside the Voyager SDK
- Model YAML file
- Model weights
- Dataset configuration (`data.yaml` for custom models)

---

# Deployment API

The AI Server exposes the following API:

```
POST http://localhost:5000/deploy
```



After successful deployment:

- Deployment logs are generated.
- Model is registered in `deployed_models.json`.
- Model becomes available for inference. 

---

# Verify Environment

Check Voyager SDK environment:

```bash
echo $AXELERA_FRAMEWORK
```

Go to Voyager SDK directory:

```bash
cd /home/ubuntu/voyager-sdk
```

---

# Default Model Example

Example: **Person Detection**

```yaml
name: person_detection

models:
  person_detection:
    class: AxUltralyticsYOLO
    weight_path: weights/yolov8m.pt
    weight_url: https://github.com/ultralytics/assets/releases/download/v8.3.0/yolov8m.pt
    num_classes: 80
    dataset: CocoDataset-COCO2017
```

Characteristics:

- Uses COCO dataset
- Downloads pretrained YOLOv8m automatically
- Detects COCO classes
- Filters only Person (Class ID 0)

---

# Custom Model Example

Example: **PPE Detection**

```yaml
name: ppe_detection

models:
  ppe_detection:
    class: AxUltralyticsYOLO
    weight_path: $AXELERA_FRAMEWORK/customers/ppe_detection_v1.0.0/models/v5.pt
    num_classes: 10
    dataset: ppe_detection_v1.0.0Calibration
```

Dataset section:

```yaml
datasets:
  ppe_detection_v1.0.0Calibration:
    class: ObjDataAdapter
    class_path: $AXELERA_FRAMEWORK/ax_datasets/objdataadapter.py
    data_dir_name: ppe_detection_v1.0.0
    label_type: YOLOv8
    ultralytics_data_yaml: data.yaml
```

Characteristics:

- Uses custom YOLOv8 model
- Uses local trained weights
- Uses custom dataset
- Supports any number of classes

---

# Verify Required Files

## YAML

```bash
ls -lh /home/ubuntu/voyager-sdk/customers/ppe/ppe.yaml
```

## Model Weights

```bash
ls -lh $AXELERA_FRAMEWORK/customers/ppe_detection_v1.0.0/models/v5.pt
```

## Dataset YAML

```bash
ls -lh $AXELERA_FRAMEWORK/customers/ppe_detection_v1.0.0/data.yaml
```

---

# Deploy the Model

Example request:

```bash
curl -X POST http://localhost:5000/deploy \
  -H "Content-Type: application/json" \
  -d '{
    "deploy_id": "ppe-001",
    "model_name": "ppe_detection",
    "yaml_path": "/home/ubuntu/voyager-sdk/customers/ppe/ppe.yaml"
  }'
```

Example response:

```json
{
    "isOk": true,
    "deploy_id": "ppe-001",
    "logs": "/logs/ppe-001"
}
```

---

# Monitor Deployment

## Live Logs

```bash
curl http://localhost:5000/logs/ppe-001
```

---

## Deployment Status

```bash
curl "http://localhost:5000/deploy-log?deploy_id=ppe-001"
```

---

## Deployment Percentage

```bash
curl "http://localhost:5000/deploy-percentage?deploy_id=ppe-001"
```

The AI server exposes APIs for log streaming and deployment progress tracking. 

---

# View Deployed Models

```bash
curl http://localhost:5000/deployed-models
```

Example response:

```json
{
    "count": 1,
    "data": [
        {
            "model_name": "ppe_detection",
            "status": "deployed",
            "version": "v1"
        }
    ]
}
```

The deployed model registry is maintained by the AI server. 

---

# Common Errors

## YAML Not Found

```
YAML not found
```

Verify:

```bash
ls -lh /home/ubuntu/voyager-sdk/customers/ppe/ppe.yaml
```

---

## Weight File Missing

```
Weight file not found
```

Verify:

```bash
ls -lh $AXELERA_FRAMEWORK/customers/ppe_detection_v1.0.0/models/v5.pt
```

---

## data.yaml Missing

```
Dataset not found
```

Verify:

```bash
ls -lh $AXELERA_FRAMEWORK/customers/ppe_detection_v1.0.0/data.yaml
```

---

## Incorrect Number of Classes

Ensure the YAML matches the trained model:

```yaml
num_classes: 10
```

---

## Model Name Mismatch

The API request should match the YAML.

API:

```json
{
    "model_name": "ppe_detection"
}
```

YAML:

```yaml
name: ppe_detection
```

---

# Deployment Workflow

```text
                Train YOLO Model
                       │
                       ▼
            Export Trained Weights (.pt)
                       │
                       ▼
             Create Voyager YAML File
                       │
                       ▼
        Configure weight_path, dataset,
          num_classes and pipeline
                       │
                       ▼
             Call /deploy REST API
                       │
                       ▼
         AI Server executes deploy.py
                       │
                       ▼
         Model Compilation & Deployment
                       │
                       ▼
            Deployment Logs Generated
                       │
                       ▼
      Model Registered (deployed_models.json)
                       │
                       ▼
          Ready for Inference Requests
```

---

# Summary

The deployment process consists of the following steps:

1. Prepare the model YAML.
2. Verify the model weights and dataset files.
3. Call the `/deploy` API.
4. Monitor deployment logs and progress.
5. Verify the deployed model using `/deployed-models`.
6. Start inference using the deployed model.

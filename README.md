# 🫁 Lung Tumor Detection using NSCLC-Radiomics Dataset

A comprehensive approach for automated lung tumor detection in Non-Small Cell Lung Cancer (NSCLC) patients using deep learning and medical imaging.

## 📋 Table of Contents
- [Overview](#overview)
- [Dataset](#dataset)
- [Data Pipeline](#data-pipeline)
- [Preprocessing](#preprocessing)
- [Bounding Box Generation](#bounding-box-generation)
- [Model Training](#model-training)
- [Evaluation](#evaluation)
- [Results](#results)


## 🎯 Overview

This repository contains our implementation for automated lung tumor detection using the NSCLC-Radiomics dataset. Our approach leverages 3D deep learning models to identify and localize tumors in CT scans from Non-Small Cell Lung Cancer patients across different stages.

## 📊 Dataset

**Source**: [NSCLC-Radiomics Dataset](https://www.cancerimagingarchive.net/collection/nsclc-radiomics/) from The Cancer Imaging Archive

**Specifications**:
- 📁 **Total Patients**: 420 CT scans with segmentation maps
- 🏥 **Patient Population**: Non-Small Cell Lung Cancer patients (various stages)
- 💾 **Format**: DICOM files with corresponding segmentation masks

**Data Split**:
- 🧪 **Test Set**: 42 patients (10% held out for final evaluation)
- 🔄 **Training/Validation**: 378 patients (90% with 10-fold cross-validation)

## 🔄 Data Pipeline

### 1. Format Conversion (DICOM → NRRD)

We utilize **Plastimatch** for efficient DICOM to NRRD conversion:

#### CT Scan Conversion:
```bash
plastimatch convert --input DCM_FOLDER --output-img CT.nrrd
```

#### Mask Conversion:
```bash
plastimatch convert \
  --input 3.000000-78236/1-1.dcm \
  --interpolation=nn \
  --output-type=uchar \
  --referenced-ct 0.000000-82046 \
  --output-prefix segmentations \
  --prefix-format nrrd \
  --fixed
```

## 🔧 Preprocessing

### Image Standardization
- **📏 Resampling**: Standardized to 1.0 × 1.0 × 3.0 mm (x, y, z) resolution
- **🎚️ Intensity Clipping**: Values clipped between -1000 and 500 HU
- **📊 Normalization**: Mean = 0, Standard deviation = 1

### Training Configuration
- **📦 Patch Size**: [144, 144, 33] (x, y, z)
- **🔄 Inference Method**: Sliding window approach for full volume coverage
- **🏃 Training Duration**: 300 epochs with identical hyperparameters
- **🎯 Model Selection**: Based on lowest validation loss

## 📦 Bounding Box Generation

Since the dataset lacks bounding box annotations, we extract them from segmentation masks using **SimpleITK**:

### Process Workflow:
1. **📥 Load Images**: Load CT scans and corresponding mask images
2. **🔍 Component Analysis**: Identify connected components in segmentation mask
3. **🏷️ Label Assignment**: Assign unique labels to each connected region
4. **📊 Property Computation**: Calculate properties for each connected component
5. **📐 Coordinate Extraction**: Extract bounding box coordinates
6. **📋 Format Conversion**: Convert to standardized format
   
<img width="769" height="706" alt="BBox" src="https://github.com/user-attachments/assets/f50dda5b-dbd7-4d5b-b898-16f37fd2fc79" />

### Output Formats:
- **Voxel Coordinates**: `[x_min, y_min, z_min, x_size, y_size, z_size]`
- **Physical Coordinates**: `[x_center, y_center, z_center, x_size_physical, y_size_physical, z_size_physical]`

## 🤖 Model Training

### Framework
We leverage the **MONAI Detection** framework for robust medical image analysis:
- **Repository**: [MONAI Detection Tutorials](https://github.com/Project-MONAI/tutorials/tree/main/detection)
- **Architecture**: 3D detection model optimized for medical imaging
- **Training Strategy**: 10-fold cross-validation on 90% of data

## 📈 Evaluation

### Primary Metrics
- **📊 Mean Average Precision (mAP)**: Across different cross-validation folds
- **🎯 Mean Average Recall (mAR)**: Comprehensive detection performance assessment

### Detailed Analysis
- **📋 Case-by-case IoU**: Individual patient evaluation on 42-patient test set
- **📈 Correlation Analysis**: Pearson correlation between IoU and:
  - Tumor size
  - Tumor location

## 🎉 Results

### Key Findings
- **✅ Unbiased Detection**: No strong correlation found between model performance and tumor characteristics
- **📏 Size Independence**: Model performs consistently across different tumor sizes
- **📍 Location Independence**: Performance not biased by tumor location
- **🎯 Robust Performance**: Consistent detection across various patient cases

### Statistical Analysis
The Pearson correlation analysis revealed **no significant bias** in the model's identification capabilities, indicating robust generalization across diverse tumor presentations.



## 📚 References

- **Dataset**: Aerts, H. J. W. L., et al. "Data From NSCLC-Radiomics." The Cancer Imaging Archive (2019).
- **Framework**: MONAI Detection - PyTorch-based framework for medical imaging AI
- **Tools**: Plastimatch for medical image processing

**Note**: This project demonstrates the application of deep learning techniques for automated medical image analysis, specifically targeting lung tumor detection in clinical CT scans.

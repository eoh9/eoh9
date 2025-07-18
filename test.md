# QAT Food Classification Experiment Report

## Executive Summary
**Experiment**: Quantization-Aware Training (QAT) with/without Class Weight  
**Model**: MobileNetV2 + Attention Module  
**Dataset**: 86,481 training images, 14,690 validation images (14 classes)  
**Objective**: Compare performance between Class Weight applied vs. non-applied QAT

## Key Results

### Performance Comparison
| Configuration | Training Accuracy | Validation Accuracy | Best Epoch | Model Size |
|---------------|------------------|-------------------|------------|------------|
| **No Class Weight** | 85.66% | 68.28% | Epoch 3 | 13.07MB |
| **With Class Weight** | 85.14% | 68.50% | Epoch 28 | 13.07MB |

### QAT Effectiveness
- **Performance Improvement**: Both configurations showed ~2% accuracy improvement after QAT
- **Training Stability**: No Class Weight configuration showed more stable learning
- **Convergence Speed**: No Class Weight converged faster (3 epochs vs 28 epochs)

## Critical Findings

### 1. Class Weight Impact
- **Minimal Performance Gain**: Only 0.22% improvement in validation accuracy
- **Training Instability**: Higher loss values and less stable learning curve
- **Slower Convergence**: Required more epochs to reach optimal performance

### 2. QAT Benefits
- **Consistent Improvement**: Both configurations benefited from QAT
- **Model Optimization**: 13.07MB TFLite model with 74 operators
- **Mobile Deployment Ready**: Optimized for mobile inference

## Recommendations

### ✅ **Recommended Approach: No Class Weight**
**Rationale:**
- Similar final perfo

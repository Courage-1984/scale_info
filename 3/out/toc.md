# Table of Contents - Native Resolution Restoration and Descaling

## Chunk Overview

This document collection contains 16 semantically coherent chunks covering the comprehensive topic of native resolution restoration and descaling in digital media.

### Core Concepts and Theory

1. **chunk_01_executive_summary.md** - Executive Summary

   - Core benefits of native resolution restoration
   - Detection methodologies overview
   - Software ecosystem summary
   - Challenges and limitations
   - TL;DR summary

2. **chunk_02_introduction_digital_resizing.md** - Introduction to Digital Image Resizing and Native Resolution

   - Defining image resizing and scaling
   - Upscaling vs. downscaling concepts
   - Native resolution definition
   - Common native resolutions in anime
   - Mixed resolution scenarios

3. **chunk_03_imperative_restoration.md** - The Imperative for Native Resolution Restoration
   - Economic and storage benefits
   - Processing and workflow optimization
   - Quality enhancement through strategic re-upscaling
   - Archival and preservation benefits

### Mathematical and Technical Foundations

4. **chunk_04_theoretical_foundations_kernels.md** - Theoretical Foundations of Image Resizing Kernels

   - Principles of traditional resizers
   - Common resizing kernels (Point, Linear, Bilinear, Cubic, Bicubic, Lanczos, Spline)
   - Mathematical properties and determinism
   - Kernel reference table

5. **chunk_05_deterministic_vs_neural.md** - Deterministic vs. Neural Network Resizers

   - Characteristics of deterministic algorithms
   - Neural network-based upscaling overview
   - Challenges of inverting neural network upscales
   - Special cases: NNEDI3 inversion

6. **chunk_06_mathematical_basis_inverse.md** - Mathematical Basis of Inverse Resizing
   - Linear equation model: A×x = b
   - Solving the inverse problem
   - LDLT decomposition
   - Forward and back substitution for pixel restoration

### Detection Methodologies

7. **chunk_07_fourier_transform_detection.md** - Fourier Transform Analysis for Detection

   - Frequency domain signatures of upscaling
   - Implementation and tools (DCT, FFT)
   - Limitations and optimal conditions
   - resdet tool specifics

8. **chunk_08_error_measurement_detection.md** - Error Measurement Method for Detection

   - Trial-and-error process
   - Error minimization principle
   - Practical implementation (getnative, GetFnative)
   - Native 1080p detection limitation

9. **chunk_09_detection_methodology_comparison.md** - Comparison of Detection Methodologies
   - Method comparison table
   - Real-world examples and case studies
   - Studio-specific workflows
   - Evolution of techniques

### Practical Implementation

10. **chunk_10_vapoursynth_ecosystem.md** - Vapoursynth Ecosystem for Descaling

    - Core descaling plugin
    - Detection and analysis tools
    - Specialized detection tools summary
    - Tool integration

11. **chunk_11_descaling_workflow.md** - The Descaling Workflow

    - Step-by-step workflow process
    - Frame selection and preparation
    - Native resolution finding
    - Kernel determination and verification

12. **chunk_12_recommended_workflow.md** - Recommended, Repeatable Workflow
    - Six-phase workflow (Preparation through Final Output)
    - Example commands and references
    - Short checklist for implementation
    - Tool-specific guidance

### Advanced Topics and Applications

13. **chunk_13_advanced_techniques.md** - Advanced Techniques and Challenges

    - Mixed resolution content handling
    - Chroma subsampling considerations
    - Post-processing artifacts
    - Neural network limitations

14. **chunk_14_case_studies.md** - Case Studies and Real-World Applications

    - Anime content analysis
    - Success and failure case studies
    - Professional workflow examples
    - Studio-specific patterns

15. **chunk_15_future_directions.md** - Future Directions and Emerging Technologies

    - Tool development trends
    - Neural network evolution
    - Industry adoption
    - Standardization efforts

16. **chunk_16_best_practices.md** - Best Practices and Recommendations
    - Summary of key principles
    - Best practices for different scenarios
    - Future outlook
    - Final implementation checklist

## Document Statistics

- **Total Chunks**: 16
- **Average Chunk Size**: ~1,500-2,000 tokens
- **Coverage**: Complete coverage of native resolution restoration topic
- **Structure**: Logical progression from theory to practice to advanced applications
- **Format**: Markdown files with clear headings and code examples

## Usage Notes

- Chunks are designed to be read independently or in sequence
- Each chunk focuses on a specific aspect of the descaling process
- Code examples and practical implementations are included where relevant
- Cross-references between chunks are maintained for comprehensive understanding

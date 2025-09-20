# Comprehensive Guide to Native Resolution Restoration and Descaling in Digital Media

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Introduction to Digital Image Resizing and Native Resolution](#introduction)
3. [Theoretical Foundations of Image Resizing Kernels](#theoretical-foundations)
4. [Methodologies for Native Resolution Detection](#detection-methodologies)
5. [Mathematical Basis of Inverse Resizing](#mathematical-basis)
6. [Practical Implementation and Tools](#practical-implementation)
7. [Advanced Techniques and Challenges](#advanced-techniques)
8. [Case Studies and Real-World Applications](#case-studies)
9. [Future Directions and Emerging Technologies](#future-directions)
10. [Conclusion and Best Practices](#conclusion)

---

## 1. Executive Summary

Native resolution restoration, commonly known as descaling, represents a sophisticated technique for reversing upscaling operations in digital media. This process involves identifying the original resolution at which content was produced and applying inverse mathematical operations to restore the pristine, pre-upscale state.

### Key Benefits:

- **File Size Reduction**: Eliminates redundant interpolated data, reducing storage and bandwidth requirements
- **Processing Efficiency**: Lower computational demands for subsequent operations
- **Quality Enhancement**: Enables superior re-upscaling using advanced algorithms
- **Archival Integrity**: Preserves content in its original, unmodified state

### Core Principles:

- **Deterministic Reversibility**: Traditional resizing kernels can be mathematically inverted
- **Near-Lossless Operation**: Achieves high fidelity restoration despite real-world imperfections
- **Strategic Application**: Best suited for clean sources with minimal post-processing artifacts

### Primary Applications:

- Anime and animated content restoration
- Legacy video content optimization
- Digital media archival and preservation
- Professional video production workflows

---

## 2. Introduction to Digital Image Resizing and Native Resolution

### 2.1 Defining Image Resizing and Scaling

Image resizing constitutes a fundamental operation in digital image processing, systematically altering image dimensions while preserving content integrity. This process employs sophisticated algorithms that either interpolate new pixel data during upscaling or sample existing data during downscaling.

**Upscaling (Magnification):**

- Generates new pixels to fill expanded grid
- Uses interpolation based on existing pixel values
- Does not create genuine new information
- Results in perceived softening or artifacts if suboptimal

**Downscaling (Minification):**

- Reduces resolution through selective pixel discarding
- Involves averaging existing data to fit smaller grid
- Inherently loses fine information through consolidation

### 2.2 The Concept of "Native Resolution"

Native resolution represents the original resolution at which material was initially produced, encompassing:

- Filmed content captured at specific resolutions
- Scanned physical media at original dimensions
- Hand-drawn or animated content at production resolution
- Computer-generated imagery at render resolution

**Example Scenario:**
A video originally produced at 720p (1280×720) but upscaled to 1080p (1920×1080) for Blu-ray release maintains 720p as its native resolution. The goal of descaling is to revert to this original, pre-upscale state.

### 2.3 Imperative for Native Resolution Restoration

**Economic Benefits:**

- Significant file size reduction (upscaled 1080p from 720p source contains no additional information)
- Reduced bandwidth consumption for distribution
- Optimized storage infrastructure costs
- Faster download times for end-users

**Processing Advantages:**

- Lower computational demands for video filters and compression
- Faster encoding, post-production, and transcoding workflows
- Reduced resource requirements for complex operations

**Quality Enhancement:**

- Enables strategic re-upscaling using superior algorithms
- Eliminates artifacts from "cheap" initial upscales
- Provides cleaner base for subsequent processing
- Allows end-user playback systems to perform high-quality real-time upscaling

**Preservation Benefits:**

- Maintains archival integrity and fidelity to original source
- Preserves intrinsic quality for long-term storage
- Ensures content authenticity in preservation environments

---

## 3. Theoretical Foundations of Image Resizing Kernels

### 3.1 Principles of Traditional Resizers

Traditional resizers operate on the principle of calculating new pixel values as weighted averages of surrounding reference pixels. The number of reference pixels (n) varies by kernel type, directly influencing complexity and visual characteristics.

**Common Kernel Types:**

| Kernel                 | Reference Pixels (n) | Characteristics              | Use Cases                   |
| ---------------------- | -------------------- | ---------------------------- | --------------------------- |
| Point/Nearest Neighbor | 1                    | Fastest, blocky output       | Quick previews              |
| Linear                 | 2                    | Basic linear interpolation   | Simple scaling              |
| Bilinear               | 4                    | Linear in x and y directions | General purpose             |
| Cubic                  | 4                    | Uses cubic curves (Lagrange) | Sharper than linear         |
| Bicubic                | 16                   | Four separate cubic curves   | High quality, parameterized |
| Lanczos                | Variable (taps)      | Sinc function, anti-aliasing | Professional scaling        |
| Spline (Spline16/36)   | Variable             | Smooth transitions           | Anime upscales              |

**Mathematical Properties:**

- **Deterministic Nature**: Identical input and parameters produce identical output
- **Theoretical Reversibility**: Predictable behavior enables mathematical inversion
- **Artifact Characteristics**: Each kernel introduces distinct visual signatures
- **Parameter Sensitivity**: Bicubic kernels use b and c parameters (often b + 2c = 1)

### 3.2 Deterministic vs. Neural Network Resizers

**Deterministic Algorithms:**

- Fixed mathematical formulas with predictable behavior
- Linear equation model: A×x = b
- Theoretical reversibility through matrix inversion
- Practical constraints from real-world data imperfections
- Achieves "near-lossless" rather than perfect restoration

**Neural Network-Based Upscaling:**

- Examples: waifu2x, NNEDI3, Google's RAISR
- Initial resize followed by convolutional layers
- "Hallucination" of new details based on training data
- Generative rather than interpolative approach
- Generally non-deterministically reversible

**Special Case - NNEDI3:**

- Unique neural network with partial reversibility
- Interpolates only every other pixel
- Leaves original reference pixels untouched
- Reversible for powers of 2 magnification
- Provides bridge between deterministic and generative approaches

---

## 4. Methodologies for Native Resolution Detection

### 4.1 Fourier Transform Analysis

**Principle:**
Fourier Transform analysis examines the frequency domain representation of images, identifying characteristic patterns that indicate upscaling operations.

**Frequency Domain Characteristics:**

- **High Frequencies**: Represent fine details, sharp edges, intricate textures
- **Mid to Low Frequencies**: Represent basic image structure and smooth gradients
- **Upscaling Signatures**: Noticeable absence or attenuation of high-frequency information
- **Zero-Crossings**: Characteristic patterns in frequency domain indicating original resolution

**Implementation (resdet):**

- Analyzes frequency domain for unique upscaling signatures
- Identifies "zero-crossings" corresponding to original resolution
- Performs optimally on clear, highly detailed images
- Sensitive to compression artifacts and filtering
- Limited to traditional resampling methods

**Optimal Conditions:**

- Clean, uncompressed source material
- Same colorspace as original resize operation
- Keyframes with low quantizer for video content
- YUV4MPEG streams for superior chroma preservation

### 4.2 Error Measurement (Trial and Error) Method

**Principle:**
Systematic testing of candidate native resolutions through iterative descale-rescale cycles, measuring error between original and processed images.

**Process:**

1. Downscale source to candidate resolution using inverse resizer
2. Re-upscale to original dimensions using same kernel
3. Calculate absolute difference from original source
4. Identify resolution with minimum error (distinct "dip")

**Implementation (getnative, GetFnative):**

- Automated iterative testing across resolution ranges
- Generates error plots (X-axis: resolution, Y-axis: error)
- Supports fractional resolution detection
- Handles mixed-resolution scenarios
- Provides visual graphs for analysis

**Advantages:**

- Relative simplicity of implementation
- Capacity to identify multiple native resolutions
- Robust for precise resolution identification
- Handles fractional values and complex scenarios

**Limitations:**

- Slower than Fourier Transform methods
- Requires careful frame selection
- Cannot detect truly native 1080p content
- Sensitive to source quality and artifacts

### 4.3 Frame Selection Criteria

**Optimal Frame Characteristics:**

- Bright, well-lit scenes with minimal blur
- Clear lineart and sharp edges
- Minimal post-processed elements or effects
- Absence of dynamic grain or noise
- No letterboxing or pillarboxing

**Problematic Frame Types:**

- Dark or low-contrast scenes
- Heavy visual effects or compositing
- Dynamic grain or noise patterns
- Credits, overlays, or text elements
- Heavily compressed or artifacted content

---

## 5. Mathematical Basis of Inverse Resizing (Descaling)

### 5.1 The Linear Equation Model: A×x = b

**Fundamental Representation:**
The resampling operation can be precisely modeled as a linear algebraic equation where:

- **A (Resize Kernel Matrix)**: n×m matrix encapsulating kernel properties and parameters
- **x (Original Pixels Vector)**: Vector of m original pixel values (unknown in descaling)
- **b (Upscaled Pixels Vector)**: Vector of n upscaled pixel values (known input)

**Matrix Characteristics:**

- **Rectangular Nature**: n ≠ m for upscaling operations
- **Sparseness**: Many zero entries due to limited pixel dependencies
- **Banded Structure**: Non-zero elements concentrated around main diagonal
- **Ill-Conditioned**: Near-linear dependencies in real-world scenarios

### 5.2 Solving the Inverse Problem

**Normal Equations Transformation:**
To overcome direct inversion difficulties, the problem is transformed:

- Multiply both sides by A^T: A^T × A × x = A^T × b
- Results in square, symmetric m×m matrix A^T × A
- Positive-definite for well-behaved kernels
- Amenable to efficient numerical solution methods

**LDLT Decomposition:**

- Factors symmetric matrix A^T × A into L×D×L^T
- L: Lower triangular matrix
- D: Diagonal matrix
- Enables efficient forward and back substitution

**Solution Process:**

1. **Forward Substitution**: Solve L×D×y = A^T × b for intermediate vector y
2. **Back Substitution**: Solve L^T × x = y for original pixel values x
3. **Result**: Restored pixel values representing native resolution image

**Computational Efficiency:**

- Triangular structure simplifies calculations
- Banded nature reduces computational complexity
- Suitable for large, sparse symmetric matrices
- Robust numerical stability for real-world applications

---

## 6. Practical Implementation and Tools

### 6.1 The Vapoursynth Ecosystem

**Core Platform:**
Vapoursynth serves as the primary framework for descaling operations, providing:

- Flexible Python scripting interface
- Extensible plugin architecture
- High-performance video processing capabilities
- Comprehensive format support

**Descale Plugin:**

- Primary tool for inverse scaling operations
- Supports multiple kernel types (bicubic, bilinear, Lanczos, spline)
- Handles various input formats (GrayS, RGBS, YUV444PS)
- Provides precise control over kernel parameters
- Includes source cropping and border handling options

**Key Functions:**

```python
# Basic descaling operations
core.descale.Debilinear(clip, width, height)
core.descale.Debicubic(clip, width, height, b, c)
core.descale.Despline36(clip, width, height)
```

### 6.2 Detection and Analysis Tools

**resdet (Frequency Domain Analysis):**

- C utility implementing spectral detection
- Fast analysis using Discrete Cosine Transform
- Identifies zero-crossings in frequency domain
- Command-line interface for batch processing
- Optimal for clean, detailed images

**getnative (Error Measurement):**

- Python script for iterative resolution testing
- Generates error plots and statistical analysis
- Supports multiple kernel types and parameters
- Automated candidate resolution identification
- Visual output for result interpretation

**GetFnative (Fractional Resolution Support):**

- Extends getnative for fractional resolutions
- Handles non-integer scaling factors
- Supports crop offset detection
- Manages parity requirements
- Quick version (getfnativeq) for preset testing

**getfscaler (Advanced Detection):**

- Rewrite of original getscaler with fractional support
- Uses JET tooling for robust operation
- Includes post-filtering for edge and dither reduction
- Supports cross-converted video analysis
- Provides comprehensive error reporting

### 6.3 Workflow Integration

**Preparation Phase:**

1. Source quality assessment and frame selection
2. Tool installation and environment setup
3. Initial screening with fast detection methods
4. Frame extraction and preprocessing

**Detection Phase:**

1. Fourier Transform analysis for quick assessment
2. Error measurement testing for precise identification
3. Fractional resolution analysis if needed
4. Cross-validation between multiple tools

**Application Phase:**

1. Kernel parameter determination
2. Descaling operation execution
3. Masking for mixed-resolution content
4. Chroma handling and format conversion

**Verification Phase:**

1. Rescale testing for accuracy confirmation
2. Visual inspection for artifact detection
3. Error metric calculation
4. Quality assessment and iteration

---

## 7. Advanced Techniques and Challenges

### 7.1 Mixed Resolution Content

**Common Scenarios:**

- Background elements at different resolutions than foreground
- 1080p credits or overlays on lower-resolution content
- Multiple animation layers with varying native resolutions
- Post-production elements added after upscaling

**Masking Solutions:**

- **DescaleM Functions**: Automatic masking for 1080p overlays
- **MaskDetail**: Difference-based masking for artifacts
- **inverse_scale**: Comprehensive masking with kagefunc
- **Manual Masking**: Custom solutions for complex scenarios

**Implementation Example:**

```python
# Using inverse_scale for automatic masking
descaled = kgf.inverse_scale(src, height=878, kernel='bicubic',
                            b=0, c=1/2, mask_detail=True)

# Manual masking approach
mask = maskDetail(src, 1280, 720, kernel='bilinear')
masked = core.std.MaskedMerge(noalias, descaled,
                             core.std.Invert(mask, 0))
```

### 7.2 Chroma Subsampling Considerations

**Subsampling Issues:**

- Blu-ray sources use 4:2:0 subsampling
- Chroma planes at half resolution of luma
- Descaling luma creates chroma resolution mismatch
- Potential color bleeding and desaturation

**Solutions:**

- **Plane Separation**: Process luma and chroma independently
- **4:4:4 Conversion**: Eliminate subsampling for processing
- **Chroma Upscaling**: Scale chroma to match descaled luma
- **Format Preservation**: Maintain subsampling for compatibility

**Implementation:**

```python
# Separate plane processing
y = core.std.ShufflePlanes(src, 0, colorfamily=vs.GRAY)
u = core.std.ShufflePlanes(src, 1, colorfamily=vs.GRAY)
v = core.std.ShufflePlanes(src, 2, colorfamily=vs.GRAY)

# Descale luma, upscale chroma
y = core.descale.Debilinear(y, 1280, 720)
u = core.resize.Spline36(u, 1280, 720)
v = core.resize.Spline36(v, 1280, 720)

# Recombine planes
out = core.std.ShufflePlanes([y, u, v], [0,0,0], colorfamily=vs.YUV)
```

### 7.3 Post-Processing Artifacts

**Common Challenges:**

- Sharpening filters applied after upscaling
- Dynamic grain or noise injection
- Color grading and correction
- Compression artifacts from source material

**Impact on Descaling:**

- Reduces detection accuracy
- Introduces irreversible artifacts
- Complicates kernel identification
- May require alternative approaches

**Mitigation Strategies:**

- Source quality prioritization
- Multiple frame analysis
- Conservative parameter selection
- Fallback to standard resizing when necessary

### 7.4 Neural Network Limitations

**Current Constraints:**

- Most neural upscalers are non-deterministically reversible
- Generative nature creates "hallucinated" details
- No direct mathematical inverse for synthetic information
- Limited to traditional resampling methods

**Future Possibilities:**

- Structured neural architectures with reversibility
- Explainable AI approaches for inverse operations
- Hybrid deterministic-neural workflows
- Specialized training for inverse operations

---

## 8. Case Studies and Real-World Applications

### 8.1 Anime Content Analysis

**Studio-Specific Patterns:**

- **Kyoto Animation**: Historical use of 955.5p for features
- **CloverWorks**: Lanczos-878 upscaling with minimal taps
- **A-1 Pictures**: Post-production sharpening complicating detection
- **Various Studios**: Common bilinear/bicubic upscales from 720p-900p

**Common Native Resolutions:**

- 720p (1280×720): Traditional TV anime standard
- 810p (1440×810): Intermediate production resolution
- 846p (1504×846): Non Non Biyori Repeat example
- 864p (1536×864): Modern production standard
- 900p (1600×900): High-quality production target

**Fractional Resolutions:**

- 955.5p: Kyoto Animation feature films
- 878p: Various modern productions
- 873p: Some TV series
- Results from 1088p upscale followed by 1080p crop

### 8.2 Success Case Studies

**Manaria Friends (878p Detection):**

- Clear error graph with distinct spike at 878p
- Successful bicubic descaling with b=0, c=1/2
- Significant quality improvement over standard resizing
- Effective masking for credit sequences

**Nichijou (720p Bilinear):**

- Classic example of bilinear upscale from 720p
- Clear frequency domain signatures
- Successful restoration with minimal artifacts
- Demonstrated superiority over Spline36 resizing

**Aikatsu Friends! (810p):**

- Very clear error graph with distinct minimum
- Successful kernel identification and application
- Excellent example of clean source material
- Minimal post-processing artifacts

### 8.3 Failure Case Studies

**Miru Tights (Unclear Results):**

- Ambiguous error graph with multiple candidates
- No distinct spikes indicating native resolution
- Recommended against descaling due to uncertainty
- Example of when to avoid descaling

**Black Lagoon (Problematic Source):**

- Unnatural graph swerves and multiple candidates
- Heavy post-processing complicating detection
- Impossible to determine reliable native resolution
- Case study in source quality importance

**Kizumonogatari (Letterboxing Issues):**

- Distorted results due to letterboxing
- Required cropping before analysis
- Demonstrates importance of proper frame preparation
- Successful after letterboxing removal

### 8.4 Professional Workflow Examples

**Beatrice-Raws Case Study:**

- Cleaned Lanczos upscaling artifacts
- Used frequency filtering and detail preservation
- Reduced audio bit depth to accommodate video bandwidth
- Comprehensive quality optimization approach

**Fansubbing Community Practices:**

- Systematic approach to source analysis
- Multiple tool validation for accuracy
- Conservative application when uncertainty exists
- Emphasis on visual verification over algorithmic results

---

## 9. Future Directions and Emerging Technologies

### 9.1 Tool Development Trends

**Automation Improvements:**

- Enhanced fractional resolution detection
- Automated kernel parameter optimization
- Machine learning-assisted frame selection
- Real-time quality assessment

**Integration Enhancements:**

- Unified workflow platforms
- Cloud-based processing capabilities
- Batch processing optimization
- Cross-platform compatibility improvements

**Detection Methodologies:**

- Advanced frequency domain analysis
- Multi-frame statistical approaches
- Temporal consistency analysis
- Hybrid detection algorithms

### 9.2 Neural Network Evolution

**Reversible Architectures:**

- Structured neural networks with mathematical inverses
- Deterministic components in generative models
- Explainable AI approaches for inverse operations
- Hybrid deterministic-neural workflows

**Training Methodologies:**

- Inverse operation training datasets
- Loss functions optimized for reversibility
- Adversarial training for artifact reduction
- Multi-scale processing approaches

**Practical Applications:**

- Real-time descaling capabilities
- Adaptive parameter selection
- Quality-aware processing decisions
- Integration with traditional methods

### 9.3 Industry Adoption

**Professional Applications:**

- Broadcast and streaming optimization
- Archival restoration projects
- Content delivery network optimization
- Quality assurance workflows

**Consumer Applications:**

- Media player integration
- Automatic quality optimization
- User-friendly tools and interfaces
- Educational resources and tutorials

**Standardization Efforts:**

- Metadata standards for native resolution
- Quality assessment metrics
- Workflow documentation
- Community best practices

---

## 10. Conclusion and Best Practices

### 10.1 Summary of Key Principles

**Theoretical Foundation:**

- Native resolution restoration is mathematically sound for deterministic kernels
- Linear algebraic approach provides robust inversion methodology
- Near-lossless operation achievable despite real-world imperfections
- Strategic application yields significant benefits across multiple domains

**Practical Implementation:**

- Complementary detection methodologies provide optimal results
- Tool ecosystem supports comprehensive analysis and application
- Workflow integration enables efficient processing pipelines
- Quality assessment and verification essential for success

**Application Guidelines:**

- Source quality prioritization critical for success
- Conservative approach when uncertainty exists
- Visual verification complements algorithmic results
- Fallback strategies essential for problematic sources

### 10.2 Best Practices Summary

**Source Assessment:**

1. Prioritize highest quality available source material
2. Select appropriate frames for analysis (bright, detailed, minimal effects)
3. Avoid heavily compressed or artifacted content
4. Consider source history and production pipeline

**Detection Methodology:**

1. Use multiple complementary detection tools
2. Cross-validate results across different approaches
3. Analyze multiple frames for consistency
4. Interpret results conservatively when uncertainty exists

**Application Process:**

1. Verify detection results through rescale testing
2. Apply appropriate masking for mixed-resolution content
3. Handle chroma subsampling appropriately
4. Monitor for artifacts and quality degradation

**Quality Assurance:**

1. Perform visual inspection at high magnification
2. Calculate error metrics for quantitative assessment
3. Compare against standard resizing methods
4. Iterate parameters if necessary

### 10.3 Recommendations for Different Scenarios

**High-Quality Sources:**

- Aggressive descaling approach with multiple kernel testing
- Comprehensive masking for mixed-resolution elements
- 4:4:4 processing for optimal quality
- Extensive verification and quality assessment

**Problematic Sources:**

- Conservative detection with multiple validation
- Fallback to standard resizing when uncertainty exists
- Focus on artifact reduction rather than perfect restoration
- Consider alternative processing approaches

**Production Workflows:**

- Automated detection and application pipelines
- Quality metrics integration
- Batch processing optimization
- Comprehensive documentation and versioning

**Archival Applications:**

- Preservation-focused approach
- Multiple format output options
- Comprehensive metadata preservation
- Long-term storage optimization

### 10.4 Future Outlook

**Technology Evolution:**

- Continued development of detection and application tools
- Integration of machine learning approaches
- Standardization of workflows and methodologies
- Broader industry adoption and recognition

**Application Expansion:**

- Beyond anime to general video content
- Real-time processing capabilities
- Consumer-level tools and applications
- Professional workflow integration

**Research Directions:**

- Advanced mathematical modeling
- Neural network reversibility research
- Quality assessment methodology development
- Cross-platform compatibility improvements

---

## Appendices

### Appendix A: Tool Installation and Setup

**Vapoursynth Installation:**

```bash
# Windows
pip install vapoursynth

# Linux
sudo apt-get install vapoursynth

# macOS
brew install vapoursynth
```

**Plugin Installation:**

```bash
# Descaling plugin
pip install vs-descale

# Detection tools
pip install getnative
pip install getfnative
```

**Source Filters:**

```bash
# LSMASHSource
pip install vs-lsmashsource

# FFMS2
pip install vs-ffms2
```

### Appendix B: Common Kernel Parameters

**Bicubic Variants:**

- Mitchell-Netravali: b=1/3, c=1/3
- Catmull-Rom: b=0, c=0.5
- B-Spline: b=1, c=0
- Sharp Bicubic: b=0, c=1
- Hermite: b=0, c=0

**Lanczos Parameters:**

- Standard: taps=3
- High Quality: taps=5
- Custom: variable taps based on requirements

**Spline Variants:**

- Spline16: 16-tap spline
- Spline36: 36-tap spline
- Spline64: 64-tap spline

### Appendix C: Error Analysis and Interpretation

**Error Metrics:**

- **PlaneStatsAverage**: Average absolute difference
- **PlaneStatsDiff**: Difference between processed and original
- **Relative Error**: Normalized error for comparison
- **Peak Error**: Maximum deviation from original

**Graph Interpretation:**

- **Clear Spikes**: Strong indication of native resolution
- **Multiple Minima**: Potential mixed-resolution content
- **Flat Lines**: Likely native high-resolution content
- **Unnatural Swerves**: Problematic source material

**Quality Thresholds:**

- **Excellent**: Error < 0.01
- **Good**: Error < 0.05
- **Acceptable**: Error < 0.10
- **Poor**: Error > 0.10

### Appendix D: Troubleshooting Guide

**Common Issues:**

1. **No Clear Detection Results**: Try different frames or avoid descaling
2. **Artifacts After Descaling**: Verify kernel parameters or try different approach
3. **Chroma Issues**: Check subsampling handling and plane processing
4. **Performance Problems**: Optimize frame selection and processing parameters

**Debugging Steps:**

1. Verify source quality and frame selection
2. Test multiple detection tools and parameters
3. Perform rescale testing for accuracy verification
4. Inspect results at high magnification
5. Compare against standard resizing methods

**Fallback Strategies:**

1. Standard Spline36 resizing
2. Conservative descaling with masking
3. Alternative kernel testing
4. Source replacement if available

---

## References and Resources

### Primary Sources:

1. Kageru's Legacy Article: https://blog.kageru.moe/legacy/resolutions.html
2. KBZ12 Mathematical Foundation: https://ddl.kageru.moe/kbz12.pdf
3. Encode.moe Guide: https://guide.encode.moe/encoding/descaling.html
4. Silent Aperture Guide: https://silentaperture.gitlab.io/mdbook-guide/filtering/descaling.html

### Tools and Software:

1. Vapoursynth: https://www.vapoursynth.com/
2. Descaling Plugin: https://github.com/Irrational-Encoding-Wizardry/descale
3. getnative: https://github.com/Infiziert90/getnative
4. GetFnative: https://github.com/YomikoR/GetFnative
5. resdet: https://github.com/0x09/resdet
6. getfscaler: https://github.com/Jaded-Encoding-Thaumaturgy/getfscaler

### Community Resources:

1. Vapoursynth Database: http://vsdb.top/
2. Anibin Blog (Archived): http://anibin.blogspot.de/
3. Doom9 Forum: https://forum.doom9.org/
4. Reddit Communities: r/animepiracy, r/encoding

### Academic and Technical References:

1. Fourier Transform Analysis in Image Processing
2. Linear Algebra and Matrix Decomposition Methods
3. Digital Signal Processing Fundamentals
4. Video Compression and Quality Assessment Standards

---

_This comprehensive guide represents the current state of native resolution restoration technology and methodologies as of 2025. The field continues to evolve with new tools, techniques, and applications emerging regularly. Readers are encouraged to consult the latest resources and community discussions for the most current information and best practices._

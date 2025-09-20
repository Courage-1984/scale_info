Based on a thorough analysis of the provided search results and additional linked resources, here is a comprehensive technical report on anime resolution detection methodologies, incorporating both historical context and modern innovations:

### 1. **Fundamentals of Anime Resolution Analysis**
- **Native vs. Upscaled Content**: Anime production pipelines often involve rendering at lower resolutions (e.g., 900p) followed by upscaling to 1080p/4K. Detection requires identifying scaling artifacts like **haloing**, **moiré patterns**, and **frequency gaps** in spectral analysis .
- **Studio-Specific Workflows**: 
  - Kyoto Animation historically used **955.5p** for features with full HD reserved for OP/ED sequences .
  - CloverWorks employs **Lanczos-878 upscaling** with minimal taps, causing haloing and moiré that require post-processing cleanup .
- **Hybrid Workflows**: Modern productions mix 2D/CG elements rendered at different resolutions, complicating detection .

### 2. **Detection Methodologies & Tools**
#### **Spectral Analysis Techniques**
- **DCT (Discrete Cosine Transform)**:
  - **Implementation**: Python-based scripts process frames to reveal grid-like noise patterns indicating original resolution (e.g., 960×540 → 1920×1080 scale) .
  - **Limitations**: Inconsistent color depth replication between Python and VapourSynth implementations reduces accuracy .
- **FFT (Fast Fourier Transform)**:
  - **Tools**: `FFTSpectrum` (AviSynth/VapourSynth) visualizes frequency distribution. Native resolutions show as **distinct "gaps"** in the spectrum where high-frequency data is absent .
  - **Use Case**: Effective for fixed-resolution assets (e.g., pixel art games like *Shovel Knight*) but struggles with mixed-content anime .
  
#### **Specialized Detection Tools**
| **Tool**               | **Functionality**                                      | **GitHub Repo**                     |
|------------------------|-------------------------------------------------------|-------------------------------------|
| `getnative`/`getfnative` | Estimates native resolution via edge sharpness metrics | Infiziert90/getnative, YomikoR/GetFnative |
| `resdet`               | Statistical analysis of compression artifacts          | 0x09/resdet                         |
| `Jaded-Encoding`       | Descaling filters for reverse engineering              | Jaded-Encoding-Thaumaturgy/getfscaler |

### 3. **Real-World Encoding Practices**
- **Beatrice-Raws Case Study**: 
  - Cleaned Lanczos upscaling artifacts using **frequency filtering** and **detail-preservation masks**.
  - Reduced DTS-HD MA audio bit depth to 20-bit to accommodate video bandwidth constraints .
- **Descaling Workflows**: 
  - Reverse upscaling using **Bicubic/Bilinear** kernels to approximate source resolution.
  - Tools like `getfscaler` automate kernel selection for optimal descaling .

### 4. **Challenges & Limitations**
- **False Positives**: 
  - Post-production sharpening (e.g., A-1 Pictures) mimics upscaling artifacts .
  - Cel-shaded animation lacks high-frequency details, confusing spectral tools .
- **Tooling Gaps**: 
  - Machine translation (MTL) fails for manga due to cultural nuances/slang .
  - GPU buffer access limitations in frameworks like Flutter hinder efficient screen capture .

### 5. **Future Directions**
- **Hybrid Approaches**: Combining `resdet`'s statistical models with `getfnative`'s edge analysis.
- **Community Verification**: Crowdsourced databases (e.g., AniBin) require skepticism due to unverified methodologies .
- **AI-Assisted Workflows**: Training CNNs on studio-specific upscaling fingerprints.

### Key Technical References:
- **Encoding Guides**: [Silent Aperture](https://silentaperture.gitlab.io/mdbook-guide/filtering/descaling.html) for descaling workflows.
- **Algorithm Details**: [DCT Python Implementation](https://github.com/joaocarvalhoopen/Detecting_the_original_resolution_of_an_upscale_image_DCT) .

This synthesis reveals that resolution detection remains an art requiring cross-validation of tools, studio awareness, and artifact interpretation. Emerging solutions focus on open-source collaboration and adaptive filtering pipelines.
# Fourier Transform Analysis for Native Resolution Detection

Fourier Transform analysis, a powerful technique rooted in signal processing, involves transforming an image from its spatial domain (where pixels are represented by their location) into the frequency domain. This transformation, often achieved using the Discrete Cosine Transform (DCT) in image processing contexts, allows for the analysis of the image's constituent frequencies. In the frequency domain, different image components are represented by their corresponding frequencies: high frequencies correspond to very small image components and fine details, such as sharp edges, intricate textures, or subtle noise. Conversely, mid to low frequencies represent the basic image structure, broader features, and smoother gradients.

## Frequency Domain Signatures of Upscaling

A key indicator of an upscaled image, particularly one processed with traditional deterministic resampling methods, is a noticeable absence or significant attenuation of high-frequency information. This phenomenon occurs because upscaling, being an interpolation process, does not genuinely create new fine details; it merely spreads the existing information across a larger pixel grid. Consequently, the high-frequency content, which represents these fine details, is either smoothed out or simply not generated.

Tools like resdet leverage this principle by specifically analyzing the frequency domain for unique patterns indicative of upscaling. Traditional resampling methods, such as those used by tools like ImageMagick, introduce characteristic "zero-crossings" at specific offsets within the frequency domain. These zero-crossings act as a distinct signature, directly corresponding to the original resolution from which the image was upscaled.

resdet identifies these specific inversions in the frequency spectrum to make its "best guess" about the original resolution. This method has a well-established history within certain online communities, having been utilized for over a decade, particularly for identifying upscales in Japanese animation.

## Implementation and Tools

**DCT (Discrete Cosine Transform)**: Python-based scripts process frames to reveal grid-like noise patterns indicating original resolution (e.g., 960×540 → 1920×1080 scale). However, inconsistent color depth replication between Python and Vapoursynth implementations can reduce accuracy.

**FFT (Fast Fourier Transform)**: Tools like FFTSpectrum (AviSynth/Vapoursynth) visualize frequency distribution. Native resolutions show as distinct "gaps" in the spectrum where high-frequency data is absent. This approach is effective for fixed-resolution assets (e.g., pixel art games like Shovel Knight) but struggles with mixed-content anime.

## Limitations and Optimal Conditions

Despite its speed and ability to identify specific signatures, Fourier Transform analysis, particularly as implemented in resdet, has certain limitations and optimal conditions for performance:

- **Source Quality Requirements**: The tool performs optimally on clear, highly detailed images that are as close as possible to the original source
- **Compression Sensitivity**: Its accuracy can be significantly compromised by the presence of compression artifacts (e.g., from highly compressed JPEGs or video codecs) and subsequent filtering applied to the image
- **Colorspace Considerations**: For the most accurate results, detection should ideally be performed in the same colorspace in which the image was originally resized, such as linear RGB
- **Video Analysis**: When analyzing video content, resdet is most effective with keyframes that have a low quantizer, and yuv4mpeg streams are preferred due to their superior preservation of chroma planes
- **False Positives**: A notable practical limitation is the potential for false positives when analyzing moderate to heavily compressed JPEG files, although the application of a deblocking filter can help mitigate this issue
- **Neural Network Limitations**: Crucially, this method is fundamentally designed for traditional resampling techniques and will not work with newer neural network-based resizers due to their generative nature, which synthesizes new details rather than merely interpolating existing ones

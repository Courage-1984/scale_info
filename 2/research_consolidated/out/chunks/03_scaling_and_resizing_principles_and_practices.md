# Scaling and Resizing: Principles and Practices

## III. Scaling and Resizing: Principles and Practices

### A. Differentiating "Scaling" and "Resizing" in Digital Image and Video Processing

In the field of digital image and video processing, the terms "scaling" and "resizing" are frequently used interchangeably.[9] This common practice reflects the fundamental similarity of their underlying computational operations, both involving the re-sampling of pixels to fit new dimensions. However, a nuanced distinction can be drawn, particularly when considering specific application contexts or the precise implications for image properties.

In some non-image processing contexts, such as user interface (GUI) design tools, a conceptual difference is sometimes articulated. Here, **resizing** might refer to changing only an object's dimensions (width and height) while maintaining other properties, such as stroke thickness, at their original values.[9] For example, if a square with a 2-pixel border is resized to be larger, its border might still remain 2 pixels thick. In contrast, **scaling** in this context would imply applying a transform that resizes the object based on a multiple of its initial X, Y, and Z (for 3D objects) values. This operation would cause properties like stroke thickness to scale proportionally with the object's dimensions.[9] Using the previous example, scaling the square larger would also increase its border thickness proportionally.

Within the realm of image processing itself, a further nuance sometimes suggests that **rescaling** implies maintaining the original aspect ratio to avoid distortion, whereas **resizing** could involve independent changes to width and height, potentially leading to skewing or distortion.[9] However, this particular distinction is debated among experts in the field, with many asserting that the two terms are, in practical application, synonymous.[9]

The semantic ambiguity surrounding "scaling" and "resizing" highlights that from a computational perspective, both operations fundamentally involve altering the pixel grid of an image. The choice of *how* these pixels are re-calculated (the specific algorithm or kernel used) and *whether* the aspect ratio is preserved are parameters of the operation, rather than defining distinct operations themselves. This underscores the importance of specifying the precise algorithm and target dimensions when discussing these processes, rather than relying solely on the general terms. For effective communication in technical discussions, it is crucial to clarify the intended meaning to avoid potential misunderstandings.

### B. The Fundamental Operations: Upscaling (Enlargement) vs. Downscaling (Reduction)

At the core of resolution management are two fundamental operations: upscaling and downscaling. These processes involve altering the pixel dimensions of an image or video to either enlarge or reduce its size, respectively.

**Upscaling**, also known as enlargement, is the process of increasing the resolution of an image or video. This operation involves taking the existing pixels and spreading them over a larger number of new pixels. Since the original data does not contain information for these new pixels, the scaling algorithm must interpolate or estimate the values of the missing pixel data.[1, 5] The quality of upscaling heavily depends on the interpolation algorithm used; if not performed effectively, it can lead to undesirable visual artifacts such as blurring, pixelation, or a general loss of sharpness.[1] The goal of upscaling is to make an image appear larger while minimizing the visual degradation that naturally occurs when synthesizing new data.

Conversely, **downscaling**, or reduction, involves decreasing the resolution of an image or video. This process condenses the existing pixel information into a smaller number of pixels. When done correctly, downscaling can actually sharpen an image by bringing the pixels closer together and effectively increasing the pixel density of the displayed object.[1] For instance, if a $4\\text{K}$ image is downscaled to $1080\\text{p}$, the information from four original pixels is typically combined into one new pixel, leading to a more compact and potentially sharper representation.

The overarching purpose of **resolution scaling** is to transform how an image or video is displayed on a screen's pixels.[1] When content is downscaled, each displayed object effectively occupies more pixels relative to its original size, creating a zoomed-in effect. Conversely, when content is upscaled, its pixels expand, potentially blurring or pixelating the image as it increases in size.[1] For example, downscaling a game's render resolution from $2560 \\times 1440$ to $1920 \\times 1080$ can significantly boost performance, though it may require additional sharpening and anti-aliasing to maintain visual quality.[1] Both upscaling and downscaling are critical tools in adapting content for various display sizes, bandwidth constraints, and performance requirements, with the choice of operation and algorithm profoundly impacting the final visual outcome.

### C. In-Depth Analysis of Image and Video Scaling Algorithms (Interpolation Kernels)

The quality of any scaling operation, whether upscaling or downscaling, hinges critically on the underlying **scaling algorithm**, also known as the interpolation kernel. These algorithms determine how new pixel values are calculated when an image's resolution is changed, directly impacting the output's sharpness, smoothness, and the presence of artifacts.[10, 11] There is no single "best" algorithm; rather, a spectrum of trade-offs exists between speed, computational cost, and the type/severity of artifacts introduced or mitigated. The choice depends entirely on the specific application, source material characteristics, and the desired balance between output quality and performance.

#### 1\. Nearest-Neighbor Interpolation

  * **Method:** This is the simplest and fastest scaling method. It operates by replacing each original pixel with a block of new pixels, all having the exact same color as the original pixel.[10] Essentially, it duplicates pixels.
  * **Characteristics:** While it preserves all original detail and can be used for magnifying without smoothing [5], its primary drawback is the introduction of severe "jaggedness" or "stairway" shapes on diagonal lines and curves, a common form of aliasing.[10, 12] This gives the image a blocky, pixelated appearance.
  * **Use Case:** Primarily used when speed is paramount and visual smoothness is not a priority, or for magnifying pixel art where sharp, blocky edges are desired.

#### 2\. Bilinear Interpolation

  * **Method:** This algorithm considers a $2 \\times 2$ neighborhood of pixels surrounding the location where a new pixel is to be created. It then calculates the new pixel's value by taking a weighted average of these four neighboring pixels.[11]
  * **Characteristics:** Bilinear interpolation is relatively fast and generally good for changing image sizes.[10, 11] However, it tends to cause some undesirable softening or blurring of details and can still produce somewhat jagged edges, though less pronounced than Nearest-Neighbor.[10] It is a common choice for upscaling anime.[7]
  * **Use Case:** Suitable for real-time applications where a balance between speed and acceptable quality is needed, or as a default for general image resizing.

#### 3\. Bicubic Interpolation

  * **Method:** A more sophisticated algorithm, Bicubic interpolation considers a larger neighborhood of pixels, typically a $4 \\times 4$ grid, around the new pixel's location. It then applies a cubic polynomial function to weigh and average these 16 pixels to determine the new pixel's value.[11]
  * **Characteristics:** Bicubic interpolation generally produces smoother edges and can enhance perceived sharpness compared to bilinear.[11] It is widely considered a high-quality general-purpose scaling algorithm and can be effectively used for both upscaling and downscaling.[11]
  * **Variants:** There are several variants of bicubic interpolation, each defined by different parameters (b and c values) that influence the sharpness and ringing characteristics. Common variants include Mitchell-Netravali ($b=1/3, c=1/3$), B-Spline ($b=1, c=0$), Hermite ($b=0, c=0$), Catmull-Rom ($b=0, c=0.5$), and Sharp Bicubic ($b=0, c=1$).[7] Some modern Blu-ray releases utilize a bicubic kernel for their initial upscaling.[5, 7]
  * **Use Case:** A popular choice for high-quality image and video scaling where detail preservation and smooth transitions are prioritized, even at the cost of slightly higher computational demands.

#### 4\. Lanczos Resampling

  * **Method:** Lanczos resampling involves a sinc filter, a mathematical function that approximates the ideal sinc function for resampling. It considers a larger number of neighboring pixels (typically 3 or 4 in each dimension, referred to as `a=3` or `a=4` Lanczos) and applies a windowed sinc function for interpolation.[11]
  * **Characteristics:** This algorithm is described as very high quality for both upscaling and downscaling.[11] It is known for producing sharp results with minimal ringing, making it a preferred choice for many professionals. However, it is more computationally expensive than bilinear or bicubic interpolation.[11]
  * **Performance:** While slower, Lanczos generally offers superior quality compared to faster algorithms like fast bilinear, especially when evaluated with metrics like VMAF, PSNR, and SSIM.[13]
  * **Use Case:** Highly recommended for situations demanding the best possible image quality, particularly when downscaling, and when computational resources are not a limiting factor.

#### 5\. Spline Interpolation (Spline16, Spline36)

  * **Method:** Spline interpolation algorithms, such as Spline16 and Spline36, utilize higher-order polynomials to calculate new pixel values.[11] Spline36, for instance, uses a 6x6 pixel neighborhood.
  * **Characteristics:** Spline algorithms are generally considered a better choice for resizing than bilinear interpolation, producing smoother results.[5, 11] They offer a good balance between sharpness and smoothness, often preferred when a less aggressive, more natural look is desired.
  * **Use Case:** Often recommended as a safer alternative to descaling when dealing with complex mixed-resolution content within a frame, as it is less prone to introducing destructive artifacts.[6]

#### 6\. Specialized Algorithms

Beyond the general-purpose algorithms, several specialized methods cater to specific content types or desired visual effects:

  * **Fourier-based Interpolation:** This method operates in the frequency domain, padding it with zero components. It can offer good detail conservation but may introduce ringing artifacts and circular bleeding.[10]
  * **Edge-Directed Interpolation (NEDI, EGGI, ICBI, DCCI):** These algorithms are designed to preserve edges in an image after scaling, actively avoiding the staircase artifacts (jaggies) that can appear on diagonal lines or curves with other methods. DCCI, for example, has shown superior scores in PSNR and SSIM on test images.[10]
  * **Pixel Art Scaling Algorithms (hqx, xbr, GemCutter):** Tailored for magnifying low-resolution computer graphics with limited color palettes (e.g., retro video games). Algorithms like hqx produce sharp, crisp edges, while the xbr family creates smoother edges, though potentially deforming shapes or grouping local areas into single colors.[10] GemCutter aims to preserve the shape and coordinates of original details without blurring them.[10]
  * **Vectorization:** This technique first converts a raster image into a resolution-independent vector representation. The vector version is then rendered as a raster image at the desired resolution. It is highly effective for simple geometric images but generally unsuitable for complex photographs due to their intricate detail.[10]

#### 7\. Emerging Paradigms: Deep Convolutional Neural Networks (SRGAN, ESRGAN, Anime4K) for Super-Resolution

A significant advancement in scaling technology is the emergence of **Deep Convolutional Neural Networks (DNNs)** for super-resolution. These machine learning-based approaches represent a paradigm shift, moving beyond traditional interpolation to **generate** convincing details by learning common patterns from vast training datasets.[10] The upscaled results are sometimes described as "hallucinations" because the introduced information may not have been present in the original source, but rather inferred by the network.[10]

  * **SRGAN (Super-Resolution Generative Adversarial Network) and ESRGAN (Enhanced SRGAN):** These methods rely on a perceptual loss function during training, aiming to produce results that are perceptually pleasing to the human eye rather than just minimizing pixel-level differences.[10]
  * **Anime4K:** This is an open-source project offering high-quality, real-time anime upscaling and denoising algorithms.[14] Anime4K is optimized for native $1080\\text{p}$ anime encoded with H.264, H.265, or VC-1, and while it might work, it is not specifically optimized for lower-resolution or standard-definition anime.[14] It aims to provide effects similar to SRGANs but with real-time performance, significantly outperforming traditional methods like waifu2x in speed.[14] Ongoing research in Anime4K includes line reconstruction algorithms to handle "distribution shift" problems in varied anime sources, line art deblurring, denoising, blind resampling artifact reduction, and experimental line darkening/thinning for perceptual quality.[14]

The evolution of scaling algorithms reveals a continuous effort to balance computational efficiency with visual fidelity. Simpler algorithms like Nearest-Neighbor and Bilinear are fast but prone to artifacts. More complex algorithms such as Lanczos, Bicubic, and Spline offer superior quality but demand more processing power. The emergence of AI/DNN-based upscalers pushes the boundaries of what is possible, enabling the generation of missing detail rather than mere interpolation, albeit with high computational costs and the potential for "hallucinations" of non-existent detail. This continuous development informs encoder choices and hardware acceleration strategies, with specialized Video Processing Units (VPUs) now capable of accelerating complex scaling operations in real-time.[13]

Furthermore, the optimal algorithm choice often differs between upscaling and downscaling, as the problems they address are inverse. Upscaling requires algorithms that intelligently infer new pixel data without excessive blurring or ringing, while downscaling demands algorithms that preserve essential detail and prevent aliasing or moiré patterns. The principle of applying a "mild sharpening effect" when downscaling and a "mild blurring effect" when upscaling is a key consideration.[11] High-quality video processing software, such as MadVR, often implements different default algorithms for these two directions, reflecting this nuanced understanding.[11] This highlights that a "one-size-fits-all" approach to scaling is suboptimal, and a deep understanding of each algorithm's strengths and weaknesses for specific tasks is critical for achieving professional results.

### D. Comparative Performance and Quality Assessment of Scaling Methods

Beyond the theoretical principles of scaling algorithms, their real-world application necessitates a comparative assessment of their performance and output quality. This evaluation typically employs objective metrics such as **VMAF (Video Multi-method Assessment Fusion)**, **PSNR (Peak Signal-to-Noise Ratio)**, and **SSIM (Structural Similarity Index Measure)**, which quantify visual quality.[13]

One advanced scaling technique explored for optimizing both quality and throughput is **cascaded scaling**. This approach involves performing scaling operations sequentially, starting from the full resolution and progressively scaling down to the next lower resolution, then using that intermediate resolution as the source for the subsequent scale. For example, instead of scaling from $4\\text{K}$ directly to $2\\text{K}$, $1080\\text{p}$, $720\\text{p}$, and $360\\text{p}$ independently, cascaded scaling would involve a chain: $4\\text{K} \> 2\\text{K} \> 1080\\text{p} \> 720\\text{p} \> 360\\text{p}$.[13]

A study comparing various scaling methods revealed significant differences in quality and performance [13]:

  * **Default Scaling (`-s` function):** Using the default scaling function in encoding tools was found to leave a noticeable amount of quality on the table compared to other methods.[13]
  * **Cascade using fast bilinear:** This method emerged as a clear loser in terms of quality. Despite the "fast" designation, it did not provide a meaningful throughput increase and significantly degraded the output quality, making it generally not recommended.[13]
  * **Cascade using Lanczos:** This approach produced the **best results** in terms of quality across all evaluated metrics (VMAF, PSNR, SSIM).[13] It slightly outperformed even the direct video filter using Lanczos. This suggests that the cumulative effect of multiple high-quality downscales in a cascaded chain can be superior to performing multiple direct downscales from the original source.
  * **Video filter using Lanczos (`-vf scale`):** This method also delivered good quality, only marginally lower than cascaded Lanczos.[13]

From a throughput perspective, the relationship between scaling method, codec, and hardware is complex. Faster scaling methods, surprisingly, may not always deliver performance increases, especially when paired with computationally intensive codecs like HEVC, AV1, or VVC, and can lead to quality degradation.[13] However, with faster codecs like H.264, cascaded scaling might indeed improve throughput.[13] The analysis also highlighted the role of specialized hardware, such as **Video Processing Units (VPUs)** like the NETINT Quadra, which can accelerate scaling and overlay operations in hardware, fundamentally altering the performance-quality trade-offs that software-only solutions face.[13]

The findings from this comparative analysis challenge the intuitive assumption that simpler or faster algorithms, or direct scaling, are always superior for performance or sufficient for quality. The observation that chained, high-quality scaling (cascaded Lanczos) can outperform direct scaling in terms of quality, and that "fast" algorithms can be detrimental without significant speed gains, reveals a complex interaction between the algorithm, the video codec, and the overall processing pipeline. For multi-resolution encoding ladders, cascaded scaling with a high-quality algorithm like Lanczos appears to be a superior strategy for quality. This suggests that the *cumulative effect* of multiple high-quality downscales is better than multiple direct downscales from the original source. This understanding is crucial for optimizing video encoding workflows, particularly in scenarios requiring multiple output resolutions for adaptive streaming.

### Table 1: Comparative Analysis of Common Video Scaling Algorithms

| Algorithm | Method/Principle | Computational Cost (Relative) | Upscaling Characteristics | Common Artifacts | Recommended Use Cases |
| :--- | :--- |:--- | :--- | :--- | :--- |
| **Nearest-Neighbor** | Duplicates nearest pixel value. | Very Low (Fastest) | Jagged, blocky, pixelated. | Jaggies, aliasing. | Magnifying pixel art, fastest preview. |
| **Bilinear** | Weighted average of $2 \\times 2$ neighboring pixels. | Low | Softens details, still somewhat jagged. | Softness, slight jaggies. | General resizing, real-time applications where speed is key. |
| **Bicubic** | Weighted average of $4 \\times 4$ neighboring pixels (cubic polynomial). | Medium | Smoother edges, can enhance sharpness. | Ringing (depending on variant), slight haloing. | High-quality general-purpose scaling (up/down), often preferred. |
| **Lanczos** | Windowed sinc function, larger pixel neighborhood. | High (Expensive) | Very sharp, minimal ringing. | Minimal ringing/aliasing. | Best quality for up/downscaling, archival, professional use. |
| **Spline (e.g., Spline36)** | Higher-order polynomial interpolation. | Medium-High | Smooth, natural appearance. | Less sharp than Lanczos, can still introduce minor artifacts. | Safer alternative for mixed resolutions, general high-quality resizing. |
| **Deep Convolutional Neural Networks (DNNs)** | Machine learning to generate details from training data. | Very High (GPU-intensive) | Can "hallucinate" new details, highly realistic. | Hallucinations, artifacts if training data is poor. | Super-resolution, enhancing older content, real-time AI upscaling. |

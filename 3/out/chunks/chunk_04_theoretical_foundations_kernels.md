# Theoretical Foundations of Image Resizing Kernels and Algorithms

## Principles of Traditional Resizers

The fundamental principle underpinning all traditional, non-neural network-based image resizers is the calculation of new pixel values as a weighted average of a selected number of surrounding "reference pixels". The specific count of these reference pixels, denoted as 'n', varies depending on the particular kernel employed, directly influencing the complexity and visual characteristics of the resizing operation.

### Common Resizing Kernels

A range of common kernels is utilized in traditional image resizing, each with distinct properties and applications:

**Point or Nearest Neighbor**: This is the simplest and fastest resizing method, characterized by n=1 reference pixel. The value of a new pixel is directly assigned from the single closest existing pixel in the source image. While computationally efficient, this method often results in blocky, jagged, or aliased artifacts, particularly noticeable during upscaling, as it lacks any form of interpolation to smooth transitions.

**Linear**: Utilizing n=2 reference pixels, the Linear kernel performs a basic linear interpolation between adjacent pixels. This method offers a slight improvement over Nearest Neighbor by introducing some smoothing, but it still produces relatively soft or blurry results, especially when scaling significantly.

**Bilinear**: This kernel involves n=4 reference pixels. It performs linear interpolation sequentially, first in the horizontal (x) direction and then in the vertical (y) direction. Bilinear interpolation produces a smoother image than Nearest Neighbor, effectively reducing jagged edges, but the resulting image can still appear somewhat soft due to the averaging nature of the interpolation. It is a common choice for quick, general-purpose scaling.

**Cubic**: Also employing n=4 reference pixels, the Cubic kernel distinguishes itself by using cubic curves, such as Lagrange polynomials, for interpolation. This allows for a more sophisticated estimation of pixel values, often resulting in sharper outputs than Bilinear interpolation while maintaining a good degree of smoothness.

**Bicubic**: Representing a more advanced form of interpolation, the Bicubic kernel utilizes n=16 reference pixels. It applies four separate cubic curves for interpolation, providing a high degree of control over the balance between sharpness and smoothness. Bicubic kernels are often parameterized by 'b' and 'c' values, which define the shape of the cubic function and thus influence the specific characteristics of the output, such as sharpness or the presence of ringing artifacts.

Common bicubic variants include:

- **Mitchell-Netravali** (often b=1/3, c=1/3): Balanced sharpness and smoothness
- **Catmull-Rom** (b=0, c=0.5): Sharp interpolation
- **B-Spline** (b=1, c=0): Smooth interpolation
- **Spline36**: High-quality spline-based interpolation

**Lanczos and Spline**: These are other widely used high-quality interpolation kernels, particularly favored for their ability to produce sharp results while effectively minimizing aliasing artifacts. Lanczos, for instance, uses a sinc function and is parameterized by 'taps', which determines the number of samples used for interpolation. Spline kernels, such as Spline16 and Spline36, are known for their smooth transitions and good detail preservation.

### Mathematical Properties and Determinism

The mathematical properties of these kernels dictate their impact on image quality. A crucial characteristic is their deterministic nature: given the same input image and parameters, these traditional resizers will consistently produce the identical output. This predictability is the cornerstone of their theoretical reversibility, provided that no information was irrecoverably lost during the original upscaling process.

Different kernels inherently introduce distinct visual characteristics, such as varying levels of sharpness, the presence of ringing (undesirable oscillations or halos around sharp edges), or haloing (light or dark borders around high-contrast areas) during upscaling. The objective of descaling is precisely to identify these specific artifactual signatures and reverse them, thereby recovering the original, cleaner image data.

### Kernel Reference Table

| Kernel                            | n (Reference Pixels)  | Comments                                                                                                   |
| --------------------------------- | --------------------- | ---------------------------------------------------------------------------------------------------------- |
| Linear                            | 2                     | Simple linear interpolation                                                                                |
| Cubic                             | 4                     | Uses curves (e.g., Lagrange)                                                                               |
| Bilinear                          | 4                     | Linear in x and y direction                                                                                |
| Bicubic                           | 16                    | Uses 4 separate cubic curves, parameterized by b/c values                                                  |
| Point/Nearest Neighbor            | 1                     | Simplest, fastest; often results in blocky output                                                          |
| Lanczos                           | Varies (e.g., taps=3) | Known for sharpness and anti-aliasing; uses sinc function                                                  |
| Spline (e.g., Spline16, Spline36) | Varies                | Smooth transitions, good detail preservation; often used for anime upscales, can introduce ringing/haloing |

This table serves as a foundational reference, consolidating crucial information about various deterministic kernels. Understanding the characteristics of these kernels is indispensable for the practical step of accurately identifying the original upscale kernel during the descaling process, as each kernel leaves a distinct signature that can be analyzed and reversed.

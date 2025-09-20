# Deterministic vs. Neural Network Resizers

The landscape of image resizing algorithms is broadly divided into two major categories: deterministic resizers and neural network-based resizers. These two paradigms operate on fundamentally different principles, leading to significant implications for their predictability, output characteristics, and, crucially, their reversibility.

## Characteristics and Predictability of Deterministic Algorithms

As previously detailed, deterministic resizers, such as Linear, Bilinear, Cubic, and Bicubic, are predicated on fixed, explicit mathematical formulas. This reliance on predefined mathematical operations ensures their predictable and consistent behavior: given the exact same input image and parameters, these algorithms will invariably produce the identical output. This inherent determinism is the cornerstone of their theoretical reversibility.

The process of inverting such a resize operation can be precisely modeled as solving a linear equation, A∗x=b, where 'A' represents the known resize kernel (encapsulating its mathematical properties and parameters), 'x' is the unknown vector of original pixel values that one aims to recover, and 'b' is the known vector of upscaled pixel values (the input image data).

This mathematical elegance and robustness, however, encounter practical constraints in real-world scenarios. The critical caveat "as long as no information was lost" is paramount. In practice, factors such as lossy compression (e.g., from video codecs or JPEG files), subsequent filtering, and the introduction of noise inevitably introduce irreversible alterations to the pixel data. Therefore, while the mathematical framework for inversion is theoretically pure, its application in practice is constrained by the imperfections of real-world source material. This is why descaling is consistently described as "near-lossless" rather than perfectly lossless.

The theoretical purity of deterministic inversion is always compromised by practical data degradation, transforming the problem from a simple inverse calculation into a sophisticated optimization challenge that seeks the most probable original state rather than the exact one.

## Overview of Neural Network-Based Upscaling

In contrast, neural network-based upscaling represents a more recent and rapidly evolving frontier in image processing. Prominent examples include waifu2x, NNEDI3, and Google's RAISR. These methods typically involve an initial resize, often performed using a traditional resizer, to bring the image to a target resolution. Following this, a series of multiple convolutional layers or similar filters are applied to each pixel.

The overarching objective of these neural networks is to achieve "super-resolution" with exceptionally high sharpness and fine detail, often by "hallucinating" or synthesizing new details based on complex patterns learned from vast training datasets. This approach fundamentally differs from traditional interpolation.

Traditional methods are fundamentally about interpolation – estimating values between known data points. Neural networks, however, engage in hallucination or synthesis – generating new, plausible details based on learned patterns, effectively creating "information" that was not explicitly present in the original source. This generative aspect makes their inversion fundamentally different and generally intractable with current deterministic methods, as there is no direct mathematical inverse for "imagined" data.

This explains why descaling techniques are primarily applicable to content upscaled using traditional, deterministic algorithms.

## Challenges of Inverting Neural Network Upscales

The generative and non-linear nature of most neural network upscalers poses significant challenges to their inversion. Unlike deterministic resizers, these algorithms produce "seemingly arbitrary outputs" that are highly dependent on the specific data used to train their underlying models. Their ability to synthesize details not present in the original source, effectively "imagining" new information, makes them generally non-deterministically reversible.

Tools like resdet explicitly acknowledge this limitation, stating their inability to work with "newer neural network-based resizers" due to this fundamental difference in operation. The process of "de-hallucinating" synthesized information, or finding a unique inverse for a generative process, is currently intractable with the inverse mathematical models used for deterministic upscales.

## Special Cases: NNEDI3 Inversion

Within the realm of neural network-based upscalers, NNEDI3 stands out as a unique exception with a degree of reversibility. It operates by interpolating only every other pixel in an image, crucially leaving the original reference pixels untouched. Due to this highly specific and structured interpolation pattern, its upsampling effect can be reversed relatively simply. This is achieved by removing every other row of pixels in the image, particularly effective for magnifications that are powers of 2 (i.e., 2^n for n∈ℕ).

This makes NNEDI3 a rare instance of a partially reversible neural network upscaler, providing a conceptual bridge between purely deterministic and fully generative AI models. Its specific reversibility suggests that not all "neural network" upscalers are designed with the same level of generative complexity. NNEDI3's interpolation method is highly structured and predictable, making it amenable to a direct inverse operation.

This implies that future, more "explainable" or structured AI upscaling architectures might also possess some degree of reversibility, contrasting with the general difficulty of inverting more complex, black-box generative models.

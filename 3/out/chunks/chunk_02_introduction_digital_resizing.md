# Introduction to Digital Image Resizing and Native Resolution

## Defining Image Resizing and Scaling: Upscaling vs. Downscaling

Image resizing, often interchangeably referred to as scaling, constitutes a fundamental operation within digital image processing. Its core function is to systematically alter the dimensions of a given image or video frame without fundamentally distorting its overall structure or the integrity of its content. This process is achieved through sophisticated algorithms that either interpolate new pixel data when increasing size or sample existing data when decreasing size.

### Upscaling (Magnification)

The operation of increasing an image's dimensions is termed upscaling, or magnification. During upscaling, the computational task involves generating new pixels to fill the expanded grid. These new pixel values are not derived from novel information but are instead estimated or interpolated based on the values of the existing, original pixels.

A critical implication of this interpolation-based approach is that upscaling does not genuinely introduce new information or fine detail that was not present in the original source. Instead, it effectively spreads the existing information across a larger number of pixels. This often results in a perceived softening of the image or, if performed suboptimally, the introduction of visual artifacts such as aliasing or ringing.

The understanding that upscaling is primarily an interpolative, rather than generative, process is foundational to the feasibility and efficacy of descaling. If upscaling were to genuinely create unique, uninferable information, the inverse process of descaling would be rendered impossible.

### Downscaling (Minification)

Conversely, downscaling, or minification, involves reducing the resolution or dimensions of an image. This process typically necessitates the selective discarding of pixel information and the averaging of existing data to fit the smaller grid. While downscaling can be optimized to preserve detail, it inherently involves a loss of some fine information due to pixels being consolidated.

The fundamental distinction between upscaling and downscaling, particularly the interpolative nature of the former, is paramount for comprehending the rationale behind descaling. Descaling leverages the deterministic nature of many upscaling algorithms to reverse the interpolation, thereby recovering the original pixel values as accurately as possible.

## The Concept of "Native Resolution" in Digital Media Production

The term "native resolution" holds a precise and critical meaning within the domain of digital media. It is defined as the original resolution at which any given material was initially produced. This encompasses the diverse array of content creation methodologies, whether the material was originally captured through filming, scanned from physical media, meticulously drawn, animated frame-by-frame, or generated through other digital processes. The native resolution, therefore, represents the true, intrinsic resolution of the source data before any subsequent scaling operations are applied.

### Common Native Resolutions in Anime

For anime content, common native resolutions include:

- **720p** (1280x720): Standard definition, often used for TV series
- **810p** (1440x810): Intermediate resolution
- **846p** (1504x846): Common in certain studio workflows
- **864p** (1536x864): Another intermediate resolution
- **900p** (1600x900): Higher quality intermediate
- **955.5p**: Specific to Kyoto Animation workflows (arises from upscaling to 1088p then cropping to 1080p)

### Illustration of Native Resolution Concept

To illustrate this concept, consider a common scenario in video distribution: a video file that was originally filmed or animated in 720p (1280x720 pixels) but was subsequently upscaled to 1080p (1920x1080 pixels) for its Blu-ray release. In this specific instance, while the final distributed file possesses a resolution of 1080p, its native resolution unequivocally remains 720p. This distinction is not merely semantic; it is crucial for understanding the core purpose and technical objectives of descaling. The goal of descaling is to revert the content to this original, pre-upscale state.

### Mixed Resolution Scenarios

A nuanced aspect of native resolution arises in the context of synthetic media, such as computer-generated imagery (CGI). In such productions, it is technically possible for a single frame to comprise elements that were produced at multiple distinct native resolutions. For example, a background might be rendered at a higher resolution, while foreground characters or specific visual effects might be generated at a lower resolution and then composited and upscaled to match the overall frame.

This inherent complexity in synthetic media directly foreshadows the challenges associated with descaling content that contains mixed resolutions, a topic that warrants detailed discussion in later sections. The ability to identify and address these varying native resolutions within a single frame is a hallmark of advanced descaling techniques.

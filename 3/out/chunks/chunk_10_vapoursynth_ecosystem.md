# The Vapoursynth Ecosystem for Descaling

Vapoursynth is a powerful, open-source frame server designed for video processing. It provides a flexible and extensible environment where various filters and plugins can be chained together to perform complex video manipulations. Its Python scripting interface allows for highly customizable and precise control over every aspect of video processing, making it an ideal platform for sophisticated tasks like descaling.

## Core Descaling Plugin

At the core of descaling within Vapoursynth is the Descale plugin. This plugin is specifically designed to undo upscaling operations by applying the inverse mathematical models discussed previously. Descale supports a range of common resize kernels, including bicubic, bilinear, Lanczos, and spline upscales, which are frequently encountered in digitally produced anime content, often from resolutions such as 720p, 810p, 864p, or 900p.

The plugin itself typically supports specific input formats like GrayS, RGBS, and YUV444PS, while its Python wrappers extend compatibility to various YUV subsampling formats, Gray, and RGB of every bitdepth. The Descale plugin's functionality is often accessed through convenient aliases provided by fvsfunc, such as fvf.Debilinear, simplifying its integration into Vapoursynth scripts.

The plugin allows for precise control over kernel parameters, such as the b and c values for bicubic kernels, and supports custom kernels through lambda functions for advanced users. The Descale plugin also includes parameters for source cropping (src_left, src_top, src_width, src_height) and border handling, which are crucial for accurately aligning the descaling operation with the original content boundaries.

## Key Tools and Scripts for Detection and Application

Several Python scripts complement the Descale plugin, automating the complex processes of native resolution detection and facilitating the application of descaling.

### Detection Tools

**getnative.py**: This is a widely used Python script designed to find the native resolution(s) of upscaled video material, primarily anime. It implements the Error Measurement method, analyzing a given video frame to determine the original resolution before upscaling. The script generates a graph where the X-axis represents resolutions and the Y-axis displays the relative error. A clear spike showing a low relative error indicates the native resolution.

Users can specify a particular frame for analysis, choose a resize kernel (Mitchell-Netravali bicubic by default), and configure parameters like b and c for bicubic, taps for Lanczos, aspect ratio, and height range. The script requires Python 3.6+, matplotlib, Vapoursynth R45+, descale, and a source filter like ffms2, lsmash, or imwri.

For optimal results, getnative.py recommends selecting a high-quality, bright frame with minimal blur, few post-processed elements, and clear lineart. A significant limitation is its inability to automatically recognize 1080p native productions, requiring user judgment for such cases.

**GetFnative**: This script, a variant of getnative, focuses on finding native fractional resolutions of upscaled material, especially anime. It extends the "naive search" approach to find fractional src_height values within a specified interval and step length. GetFnative can handle scenarios where base_height and base_width are unknown, often only requiring their parity for descaling. It also supports descaling in only width or height to aid in identifying parity individually, and can acquire cropping parameters for descaling. A "quick" version, getfnativeq, checks preset values with limited options for faster analysis.

**getfscaler**: Described as an unofficial "companion" script to getfnative, getfscaler is a rewrite of the original getscaler tool, providing fractional descaling support using vsjet packages. It employs more robust JET tooling, defaults to kernels found in professional software plus Point, and includes post-filtering methods to reduce errors from dirty edges and dithering. It supports fractional descaling, cross-converted video, and more image types without relying on ffms2. The script provides warnings and information, and offers optional features like one-dimensional scaling and debug output. A crucial warning accompanying getfscaler is that its results are not conclusive on their own and may be inaccurate, emphasizing the need to "TRUST YOUR EYES OVER THIS SCRIPT!".

**resdet**: This command-line tool and C library detects upscaling and determines original resolution by analyzing the image's frequency domain using the Discrete Cosine Transform. It identifies "zero-crossings" unique to traditional resampling methods, which correspond to the original resolution. resdet works best on clear, detailed images and is sensitive to compression artifacts and filtering, performing optimally in the original resize colorspace (e.g., linear RGB). It is limited to traditional methods and will not work with neural network-based resizers.

### Specialized Detection Tools Summary

| **Tool**                 | **Functionality**                                      | **GitHub Repo**                           |
| ------------------------ | ------------------------------------------------------ | ----------------------------------------- |
| `getnative`/`getfnative` | Estimates native resolution via edge sharpness metrics | Infiziert90/getnative, YomikoR/GetFnative |
| `resdet`                 | Statistical analysis of compression artifacts          | 0x09/resdet                               |
| `Jaded-Encoding`         | Descaling filters for reverse engineering              | Jaded-Encoding-Thaumaturgy/getfscaler     |

# Error Measurement (Trial and Error) Method for Native Resolution Detection

The Error Measurement method for native resolution detection, while potentially slower than Fourier Transform analysis, offers distinct advantages, including its relative simplicity of implementation and its capacity to identify instances where multiple native resolutions might exist within a single image. This method operates on an iterative "trial and error" principle.

## The Trial-and-Error Process

The process begins by employing an inverse resizer to systematically downscale the source image to a wide range of possible resolutions. This range can be comprehensive (e.g., from 400p to 1079p) or a more focused, user-defined "sane range" based on common production resolutions. For each hypothetical native resolution tested, the resulting downscaled image is then immediately upscaled back to the original source resolution using the same resize kernel. Finally, the newly upscaled image (which has undergone a downscale-upscale cycle) is rigorously compared against the original source image.

## Core Principle: Error Minimization

The core principle underpinning this method is that when the correct native resolution is identified, the error—defined as the absolute difference between the original image and the image that has been downscaled and then re-upscaled—will be significantly smaller and more distinct than the errors observed for all other tested resolutions. This pronounced "dip" or minimum in the error plot serves as a clear indicator of the true native resolution.

An example implementation, provided for the Vapoursynth frame server, illustrates this process through a `get_error` function. This function calculates the absolute difference between the source and the upscaled image (|x - y|) and then computes the luma_error, which is the average of the plane statistics of this difference, yielding a numerical value between 0 and 1.

## Practical Implementation

Practical tools such as getnative and GetFnative are built upon this error measurement principle. These scripts automate the iterative process, generating plots of error versus resolution and suggesting potential native resolutions based on the identified error minima.

The getnative tool, for instance, allows users to specify a frame for analysis, a resize kernel (bicubic by default), and various parameters like b and c for bicubic, or taps for Lanczos, along with aspect ratio and height ranges. The GetFnative script further extends this by supporting fractional native resolutions and handling scenarios involving cropping and letterboxing.

## Advantages and Limitations

The advantages of the Error Measurement method include its relative simplicity of implementation and its unique capacity to identify instances where multiple native resolutions might exist within a single image, a common occurrence in synthetic media. However, it is generally slower than Fourier transform methods, though still sufficiently fast for practical application (a few seconds per frame without multithreading), as typically only a few representative frames per video need to be analyzed.

A critical prerequisite for accurate results is the careful selection of a "good frame" for analysis—one that is bright, exhibits minimal blur, contains few post-processed elements, and features clear lineart. This consistent advice across different methodologies highlights that the accuracy and reliability of native resolution detection are profoundly dependent on the "cleanliness" and fidelity of the source material. Any pre-existing noise, heavy compression artifacts (e.g., from highly compressed JPEGs), or complex post-processing (e.g., heavy visual effects, dynamic grain) will significantly degrade the effectiveness of these detection algorithms.

This leads directly to the practical advice to "TRUST YOUR EYES OVER THIS SCRIPT!" and "Trust your gut for 1080p", as algorithmic output can become unreliable when faced with highly imperfect data.

## Native 1080p Detection Limitation

A significant functional limitation of tools like getnative is their inability to automatically recognize content that is natively 1080p. This "blind spot" is inherent to the methodology. If a source is already native 1080p, there is no upscale to reverse. When the tool attempts to downscale from 1080p (itself native) to a hypothetical lower resolution (e.g., 720p) and then upscale back to 1080p, it will always introduce some error due to the interpolation process. However, this error will not exhibit a distinct, sharp "dip" that points to a lower native resolution, because no such lower native resolution exists. Instead, the error might be uniformly low or lack a clear minimum, failing to provide the characteristic signature the algorithm looks for.

This causal relationship between the method's design (seeking a lower native resolution) and the nature of truly native high-resolution content explains why human judgment remains crucial for confirming 1080p sources.

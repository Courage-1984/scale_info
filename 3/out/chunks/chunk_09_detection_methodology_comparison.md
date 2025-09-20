# Comparison of Detection Methodologies

## Method Comparison Table

| Method            | Pros                                                                     | Cons                                                                  | Tools/Examples                                                           |
| ----------------- | ------------------------------------------------------------------------ | --------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| Fourier Transform | Fast, visual (graphs), identifies specific signatures                    | Misses multi-res; sensitive to compression; geo-tagged rare           | Anibin (Japanese blog); resdet; kbz12.pdf figs (native vs. upscale freq) |
| Error Measurement | Accurate for multi-res; automated graphs; handles fractional resolutions | Slower; needs good frames (bright, clean); cannot detect native 1080p | getnative (graphs); GetFnative (fractional search)                       |

The complementary nature of Fourier Transform analysis and Error Measurement methods suggests that an expert workflow would likely integrate both: FT-based tools for rapid initial screening of clean sources, and EM-based tools for precise identification and handling of complex scenarios.

## Real-World Examples and Case Studies

### Examples from Various Sources

- **Anibin**: "K-On! Season 2" (715p eval, 1280x720 est); "Attack on Titan" (824p/857p eval)
- **encode.moe**: Manaria Friends (878p via graph); bad graphs (Miru Tights, no clear spikes)
- **silentaperture**: Nichijou (720p bilinear)
- **kageru.moe**: Non Non Biyori Repeat (846p, ringing if wrong res)

### Studio-Specific Workflows

- **Kyoto Animation**: Historically used 955.5p for features with full HD reserved for OP/ED sequences
- **CloverWorks**: Employs Lanczos-878 upscaling with minimal taps, causing haloing and moiré that require post-processing cleanup
- **Hybrid Workflows**: Modern productions mix 2D/CG elements rendered at different resolutions, complicating detection

### Evolution of Techniques

- **Early Sources (2010s)**: kageru.moe (legacy, pre-2017) and kbz12.pdf (2017 unfinished) focus on basics: Manual Avisynth/Vapoursynth scripting, Anibin for res lookup, simple masks. Emphasis on bilinear/bicubic; warns against upscaling with inverse kernels.

- **Mid-2010s Anibin**: Manual Fourier analysis; explains fractional res from 1088p upscale/crop (consistent across sources).

- **Modern Guides/Tools (2020s)**: encode.moe and silentaperture (post-2020) integrate tools like getnative/descale; stress visual checks, masks for credits. GitHub tools add automation, fractional support (e.g., GetFnative for odd sizes like Anibin's 955.5p).

**Consistency**: All agree on kernels (bilinear common), error/Fourier methods, masks for artifacts. Evolution: From manual (error-prone) to tools with graphs/fractionals. Warnings uniform: Verify visually; avoid on bad sources.

**Differences**: Old sources use Avisynth heavily; modern favor Vapoursynth. Tools like resdet add freq focus; getfscaler emphasizes professional kernels.

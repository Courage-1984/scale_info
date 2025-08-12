Native Resolutions and Scaling

This is very old. Me and a few others have since written a fansubbing/encoding guide that also covers this topic in more depth and with more up-to-date information. You can find it [here](https://guide.encode.moe/encoding/descaling.html).  
There’s also [this unfinished writeup](https://ddl.kageru.moe/kbz12.pdf), which is really just here for archival purposes.

Now go read one of those. This page has nothing of interest.

Table of contents

-   [Introduction](https://blog.kageru.moe/legacy/resolutions.html#c_introduction)
-   [Avisynth and Vapoursynth basics](https://blog.kageru.moe/legacy/resolutions.html#c_basics)
-   [Bilinear/Debilinear examples](https://blog.kageru.moe/legacy/resolutions.html#c_examples)
-   [Native resolutions](https://blog.kageru.moe/legacy/resolutions.html#c_native)
-   [Kernels](https://blog.kageru.moe/legacy/resolutions.html#c_kernel)
-   [Masks: Dealing with artifacts and 1080p overlays](https://blog.kageru.moe/legacy/resolutions.html#c_mask)
-   [Subsampling](https://blog.kageru.moe/legacy/resolutions.html#c_ss)
-   [Importable Vapoursynth script](https://blog.kageru.moe/legacy/resolutions.html#c_import)

[Introduction](https://blog.kageru.moe/legacy/resolutions.html#c_introduction)

As some (or many) might be aware, anime is usually produced at resolutions below 1080p. However, since all Blu-Ray releases are 1080p, the source material has to be upscaled to this resolution. This article will try to cover the basics of scaling Blu-Ray sourced material using Vapoursynth and Avisynth.  
Note: Most of the images were embedded as JPG to save bandwidth. Clicking an image will open the lossless PNG.

[Avisynth and Vapoursynth basics](https://blog.kageru.moe/legacy/resolutions.html#c_basics)

In order to make the following easier to understand I will try to explain the basic scaling methods in Avisynth and Vapoursynth. More detailed examples to deal with artifacts can be found in the corresponding paragraph. Blue code blocks contain Vapoursynth code while green blocks contain Avisynth. For Vapoursynth I will be using [fmtconv](https://github.com/EleonoreMizo/fmtconv/releases) to resize.

Scaling to 1080p using a bilinear resizer. This can be used for either upscaling or downscaling

clip = core.fmtc.Resample(clip, 1920, 1080, kernel = 'bilinear')

clip.BilinearResize(1920, 1080)

Note that fmtc will use Spline36 to resize if no kernel is specified. Spline is generally the better choice, and we are only using bilinear as an example. To use Spline36 in Avisynth use

clip.Spline36Resize(1920, 1080)

Using a debilinear resizer to reverse to a native resolution of 1280x720: (Note that you should **never** use this to upscale anything)

clip = core.fmtc.Resample(clip, 1280, 720, kernel = 'bilinear', invks = True)

clip.Debilinear(1280, 720)

Debilinear for Avisynth can be found [in the wiki](http://avisynth.nl/index.php/Debilinear).

[Bilinear/Debilinear Examples](https://blog.kageru.moe/legacy/resolutions.html#c_examples)

Traditional scaling is done by spreading all pixels of an image over a higher resolution(e.g. 960x540 -> 1920x1080), interpolating the missing pixels (in our example every other pixel on each axis), and in some cases applying additional post-processing to the results. For a less simplified explanation and comparison of different scaling methods refer to the [Wikipedia article](https://en.wikipedia.org/wiki/Image_scaling).  
It is possible to invert the effects of this by using the according inverse algorithm to downscale the image. This is only possible if the **exact** resolution of the source material is known and the video has not been altered after scaling it (we will deal with 1080p credits and text later).  
A few examples of scaled and inverse scaled images: (click for full resolution PNG)  
![](https://blog.kageru.moe/media/articles/res/opm_src.jpg "ONE PUUUUNCH")  
1080p source frame from the One Punch man Blu-ray. No processing.  
![](https://blog.kageru.moe/media/articles/res/opm_deb.jpg)  
Source.Debilinear(1280, 720) ![](https://blog.kageru.moe/media/articles/res/opm_bilinear.jpg)  
Source.Debilinear(1280, 720).BilinearResize(1920,1080)  
This reverses the scaling and applies our own bilinear upscale.  
You may see slight differences which are caused by the Blu-Ray compression noise but without zooming in and if played in real time these images should be indistinguishable.

My second example will be a frame from Makoto Shinkai's Kotonoha no Niwa or "The Garden of Words". The movie is not only beautifully drawn and animated but also produced at FullHD resolution. We will now upscale the image to 4k using a bilinear resizer and reverse the scaling afterwards. ![](https://blog.kageru.moe/media/articles/res/kotonohasrc.jpg)  
The untouched source frame ![](https://blog.kageru.moe/media/articles/res/kotonoha4k.jpg)  
Source.BilinearResize(3840,2160) ![](https://blog.kageru.moe/media/articles/res/kotonoha_deb.jpg)  
Source.BilinearResize(3840,2160).Debilinear(1920,1080)  
This time the images are even more similar, because no artifacts were added after upscaling. As you can see, using inverse kernels to reverse scaling is quite effective and will usually restore the original image accurately. This is desirable, as it allows the encoder to apply a reverse scaling algorithm to release in 720p, significantly decreasing the release's filesize. The 720p video will be upscaled by the leecher's video player, potentially using high quality scaling methods like the ones implemented in MadVR. Releasing in native resolution will therefore not just save space, but may even improve the image quality on the consumer's end.

[Native resolutions](https://blog.kageru.moe/legacy/resolutions.html#c_native)

Finding the native resolution of your source is the most important step. If you use the wrong resolution or try to debilinearize native 1080p material you will destroy details and introduce ringing artifacts. To understand this let's take a look at this frame from Non Non Biyori Repeat. The show's native resolution is 846p. ![](https://blog.kageru.moe/media/articles/res/nnb_src.jpg) Source ![](https://blog.kageru.moe/media/articles/res/nnb_ringing.jpg) Source.Debilinear(1280, 720)  
Upon taking a closer look you will see that the edges of our 720p image look very jagged or aliased. This is caused by improper debilinearizing. The effect will get stronger with sharper and more detailed edges. If you encounter this **never** try to fix it by anti-aliasing. Try to find the correct resolution or don't use inverse scaling at all. ![](https://blog.kageru.moe/media/articles/res/nnb_ringing_zoom.jpg) Source.Debilinear(1280, 720).PointResize(3840, 2160) and some cropping. Point resize (also called Nearest Neighbor) is used to magnify without smoothing. ![](https://blog.kageru.moe/media/articles/res/nnb_source_zoom.jpg) Source.PointResize(3840, 2160) and cropping. As you can see this version does not have the ringing artifacts.

Unfortunately there are only few ways of determining the native resolution.  
The main source is [anibin](http://anibin.blogspot.de/), a japanese blog that analyzes anime to find its native resolution. In order to find an anime, you have to get the original title from [MyAnimeList](http://myanimelist.net/), [AniSearch](http://www.anisearch.com/), [AniDB](http://anidb.net/), or any other source that has Kanji/Kana titles.  
Non Non Biyori Repeat's japanese title is "のんのんびより りぴーと", and if you copy-paste it into the search bar on anibin, you should be getting [this result.](http://anibin.blogspot.de/2015/07/1_65.html) Even if you don't understand japanese, the numbers should speak for themselves. In this case the resolution is 1504x846. This is above 720p but below 1080p, so you have multiple options. In this case I would recommend encoding in 1080p or using a regular resizer (like Spline36) if you need a 720p version. In some cases even scaling back to anibin's resolution does not get rid of the ringing, either because the studio didn't use a bilinear resizer or the analysis was incorrect due to artifacts caused by TV compression, so I wouldn't bother messing with the native resolution. It's not like you were gonna release in 846p, right?  
**Edit**: Apparently there are people out there who genuinely believe releasing a 873p video is a valid option. This is not wrong from an objective standpoint, but you should never forget that a majority of the leechers does not understand encoding and is likely to ignore your release, because "Only an idiot would release in 8xxp".

If you want an easier way to detect ringing and scaling artifacts, read the chapter about artifacts and masks.

Btw, in case you do need (or want) to inverse scale our example, you would have to use a Debicubic resizer which leads me to our next topic.  

[Kernels](https://blog.kageru.moe/legacy/resolutions.html#c_kernel)

Sometimes you will encounter ringing and artifacts even if you are certain that you know the native resolution. This usually means that the studio used another resizer. Our example will be Byousoku 5 Centimeter or 5 Centimeters per Second ([Anibin's Blu-Ray analysis](http://anibin.blogspot.de/2008/09/5-blu-ray.html))

This will be our test frame: ![](https://blog.kageru.moe/media/articles/res/byousoku_src.jpg) We will be using the masking functions explained in the next paragraph. For now just accept them as a good way to find artifacts. If we try to debilinearize our example, the mask will look like this: ![](https://blog.kageru.moe/media/articles/res/byousoku_linear.png) Despite using the correct resolution we can see strong artifacts around all edges. This can have multiple, not mutually exclusive reasons:

1.  The studio used sharpening filters after upscaling
2.  The studio used different resolution for different layers or parts of the image
3.  The image was upscaled with a different kernel (not bilinear)
4.  Our resolution is wrong after all

The first two reasons are not fixable, as I will illustrate using a flashback scene from Seirei Tsukai no Blade Dance. The scene features very strong and dynamic grain which was added after the upscale, resulting in 720p backgrounds and 1080p grain. ![](https://blog.kageru.moe/media/articles/res/bladedance_grain_src.jpg) And now the mask: ![](https://blog.kageru.moe/media/articles/res/bladedance_grain_mask.png) In this case you would have to trim the scene and use a regular resizer. Sometimes all backgrounds were drawn in a higher or lower resolution than the characters and foreground objects. In this case inverse scaling becomes very difficuilt since you would need to know the resolution of all different planes and you need a way to mask and merge them. I'd advise using a regular resizer for these sources or just releasing in 1080p.  
After talking about problems we can't fix, let's go back to our example to fix reason 3. Some (especially more recent) Blu-Rays were upscaled with a bicubic kernel rather than bilinear. Examples are Death Parade, Monster Musume, Outbreak Company, and of course our image. Applying the mask with debicubic scaling results in far fewer artifacts, as seen here: (hover over the image to see the bilinear mask) ![](https://blog.kageru.moe/media/articles/res/byousoku_cubic.png) The remaining artifacts are likely caused by compression artifacts on the Blu-Ray as well as potential postprocessing in the studio. This brings us back to reason 1, although in this case the artifacts are weak enough to let the mask handle them and use debicubic for the rest.  
Usage (without mask):  
To realize this in Avisynth import [Debicubic](http://avisynth.nl/index.php/Debicubic) and use it like this:

src.Debicubic(1280, 720, b=0, c=1)

For Vapoursynth use [fmtconv](https://github.com/EleonoreMizo/fmtconv/releases):

out = core.fmtc.resample(src, 1280, 720, kernel = 'bicubic', invks = True, a1 = 0, a2 = 1)

To use a mask for overlays and potential artifacts as well as 4:4:4 output use the Vapoursynth function linked at the bottom. Example for bicubic upscales:

out = deb.debilinearM(src, 1280, 720, kernel = 'bicubic')

If the b and c parameters are not 0 and 1 (which should rarely be the case) you can set them as a1 and a2 like in fmtc.resample(). Bicubic's own default is 1/3 for both values so if bilinear and bicubic 0:1 don't work you could give that a try.  
**Edit:** I did some more testing and consulted another encoder regarding this issue. Since we're using an inverse kernel in vapoursynth, the results may differ slightly from avisynth's debicubic. In those cases, adjusting the values of a1 and a2, as well as the number of taps used for scaling can be beneficial and yield a slightly sharper result.

[Masks: Dealing with artifacts and 1080p overlays](https://blog.kageru.moe/legacy/resolutions.html#c_mask)

Sometimes a studio will add native 1080p material (most commonly credits or text) on top of the image. Inverse scaling may work with the background, but it will produce artifacts around the text as seen in the example from Mushishi Zoku Shou below: ![](https://blog.kageru.moe/media/articles/res/mushishi_ringing.png) In order to avoid this you will have to mask these parts with conventionally downscaled pixels. The theory behind inverse scaling is that it can be reversed by using regular scaling, so (in theory) a source frame from a bilinear upscale would be identical to the output of this script:

source.Debilinear(1280,720).BilinearResize(1920,1080)

This property is used by scripts to mask native 1080p content by finding the differences between the source and the above script's output. A mask would look like this:  
![](https://blog.kageru.moe/media/articles/res/mushishi_mask.png) If there are any differences, the areas with artifacts will be covered by a regular downscale like

source.Spline36Resize(1280,720)

In Avisynth you can import [DebilinearM](https://blog.kageru.moe/legacy/DebilinearM.avsi) which can also be found [in the wiki.](http://avisynth.nl/index.php/Debilinear) For Vapoursynth [MaskDetail](https://raw.githubusercontent.com/MonoS/VS-MaskDetail/master/MaskDetail.py) can be used to create the Mask and MaskedMerge to mask the artifacts. A full importable script is available at the end.

#MaskDetail has to be imported or copied into the script  
#src is the source clip  
deb = core.fmtc.resample(src, 1280, 720, kernel = 'bilinear', invks = True)  
noalias = core.fmtc.resample(src, 1280, 720, kernel="blackmanminlobe", taps=5)  
mask = maskDetail(src, 1280, 720, kernel = 'bilinear')  
masked = core.std.MaskedMerge(noalias, src, core.std.Invert(mask, 0))  

Using this function to filter our scene returns this image: (hover to see the unmasked version) ![](https://blog.kageru.moe/media/articles/res/mushishi_masked.png) The credits stand out less and don't look oversharpened. The effect can be much stronger depending on the nature and style of the credits.

[Subsampling](https://blog.kageru.moe/legacy/resolutions.html#c_ss)

You may have encountered fansubs released in 720p with 4:4:4 subsampling. In case you don't know the term, subsampled images store luma (brightness) and chroma (color) at different resolutions. A Blu-Ray will always have 4:2:0 subsampling, meaning the chroma channels have half the resolution of the luma channel. When downscaling you retain the subsampling of the source, resulting in 720p luma and 360p chroma. Alternatively you can split the source video in luma and chroma and then debilinearize the luma (1080p->720p) while upscaling the chroma planes (540p->720p). Using the same resolution for luma and chroma will prevent colorbleeding, retain more of the chroma present in the source, and prevent desaturation.  
A script for Avisynth and the discussion can be found on [doom9.](http://forum.doom9.org/showthread.php?t=170832) For Vapoursynth I prefer to use the script explained in the next section which allows me to mask credits and convert to 4:4:4 simultaneously.

[Importable Vapoursynth script](https://blog.kageru.moe/legacy/resolutions.html#c_import)

While there may be scripts for literally anything in Avisynth, Vapoursynth is still fairly new and growing. To make this easier for other Vapoursynth users I have written this simple import script which allows you to debilinearize with masks and 4:4:4 output. A downloadable version is linked below the explanation. Essentially, all the script does is split the video in its planes (Y, U and V) to scale them separately, using debilinear for luma downscaling and spline for chroma upscaling. The example given is for 720p bilinear upscaled material:

y = core.std.ShufflePlanes(src, 0, colorfamily=vs.GRAY)  
u = core.std.ShufflePlanes(src, 1, colorfamily=vs.GRAY)  
v = core.std.ShufflePlanes(src, 2, colorfamily=vs.GRAY)  
y = core.fmtc.resample(y, 1280, 720, kernel = 'bilinear', invks = True)  
u = core.fmtc.resample(u, 1280, 720, kernel = "spline36", sx = 0.25)  
v = core.fmtc.resample(v, 1280, 720, kernel = "spline36", sx = 0.25)  
out = core.std.ShufflePlanes(clips=\[y, u, v\], planes = \[0,0,0\], colorfamily=vs.YUV)  
noalias = core.fmtc.resample(src, 1280, 720, css = '444', kernel="blackmanminlobe", taps=5)  
mask = maskDetail(src, 1280, 720, kernel = 'bilinear')  
out = core.std.MaskedMerge(noalias, out, core.std.Invert(mask, 0))  
out.set\_output()  

To call this script easily copy [this file](https://blog.kageru.moe/media/articles/debilinearm.py) into C:\\Users\\Your\_Name\\AppData\\Local\\Programs\\Python\\Python35\\Lib\\site-packages and use it like this:

import vapoursynth as vs  
import debilinearm as deb  
core = vs.get\_core()  
src = core.lsmas.LWLibavSource(r'E:\\path\\to\\source.m2ts') #other source filters will work too  
out = deb.debilinearM(src, width, height, kernel)  

Where width and height are your target dimension and kernel is the used upscaling method. The output will be in 16-bit and 4:4:4 subsampling.  
The defaults are (1280, 720, 'bilinear') meaning in most cases (720p bilinear upscales) you can just call:

out = deb.debilinearM(src)

List of parameters and explanation:

<table><tbody><tr><td>parameter</td><td>[type, default]</td><td>explanation</td></tr><tr><td>src</td><td>[clip]</td><td>the source clip</td></tr><tr><td>w</td><td>[int, 1280]</td><td>target width</td></tr><tr><td>h</td><td>[int, 720]</td><td>target height</td></tr><tr><td>kernel</td><td>[string, 'bilinear']</td><td>kernel used for inverse scaling. Has to be in 'quotes'</td></tr><tr><td>taps</td><td>[int, 4]</td><td>number of taps for reverse scaling</td></tr><tr><td>return_mask</td><td>[boolean, False]</td><td>returns artifact mask in grayscale if True</td></tr><tr><td>a1</td><td>[int, 0]</td><td>b parameter of bicubic upscale, ignored if kernel != 'bicubic'</td></tr><tr><td>a2</td><td>[int, 1]</td><td>c parameter of bicubic upscale, ignored if kernel != 'bicubic'</td></tr></tbody></table>

**Edit:** The generic functions (core.generic.\*) were removed in vapoursynth in R33, as most of them are now part of the standard package (core.std.\*). I have updated the script below accordingly, meaning it may not work with R32 or older. This also applies to MonoS' MaskDetail which (as of now) has not been updated. You can "fix" it by replacing both occurences of "core.generic" with "core.std".

The most recent version of my scripts can always be found on Github:  
[Download](https://gist.github.com/kageru/d71e44d9a83376d6b35a85122d427eb5)  
[Download fmtconv (necessary)](https://github.com/EleonoreMizo/fmtconv/releases)
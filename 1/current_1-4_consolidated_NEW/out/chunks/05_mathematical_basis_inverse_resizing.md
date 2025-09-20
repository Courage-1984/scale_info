# The Mathematical Basis of Inverse Resizing (Descaling)

The ability to accurately reverse a deterministic resizing operation, known as descaling, is rooted in the principles of linear algebra. The process of resampling, whether upscaling or downscaling, can be precisely modeled as a linear equation. Understanding this mathematical framework is essential for appreciating the precision and limitations of descaling tools.

## The Linear Equation Model: A∗x=b

The fundamental representation of resampling operations is expressed through the linear algebraic model: A∗x=b. This equation serves as the bedrock for understanding how deterministic image processing functions and, critically, how it can be inverted.

The components of this equation are defined as follows:

- **A (Resize Kernel Matrix)**: This is an n×m matrix that encapsulates the specific mathematical properties of the resize kernel that was originally used (e.g., Bicubic, Bilinear) and its associated parameters. In this context, 'm' represents the total number of pixels in the original, native-resolution image, while 'n' represents the total number of pixels in the upscaled (output) image. For upscaling operations, 'n' is typically greater than 'm', indicating that the output image has more pixels than the input. The matrix 'A' effectively describes how each pixel in the original image contributes to the formation of each pixel in the upscaled image, based on the interpolation function of the chosen kernel.

- **x (Original Pixels Vector)**: This is a vector containing the pixel values of the original, native-resolution image. It comprises 'm' elements, corresponding to the 'm' pixels of the source image. In the inverse problem of descaling, 'x' is the unknown variable that the process aims to recover. It represents the pristine, un-upscaled data.

- **b (Upscaled Pixels Vector)**: This is a vector containing the pixel values of the final, upscaled image. It consists of 'n' elements, corresponding to the 'n' pixels of the target image. This vector is considered known, as it constitutes the input image data that the descaling process receives. It is the observable, upscaled artifact from which the original 'x' must be inferred.

The challenge in descaling lies in solving this equation for 'x'. Directly solving for 'x' by inverting 'A' (i.e., x=A−1b) is generally not straightforward or even possible. This is because 'A' is typically a rectangular matrix (not square) when 'n' (output pixels) is not equal to 'm' (input pixels), and thus it does not possess a direct inverse in the traditional sense. Even if 'A' were square, its potentially very large size and its often ill-conditioned nature in real-world scenarios (due to noise, compression, or near-linear dependencies) would pose significant computational challenges for a direct inversion. Therefore, specialized numerical methods are required to find the most accurate approximation of 'x'.

## Solving the Inverse Problem

To overcome the inherent difficulties of directly inverting the rectangular or ill-conditioned matrix 'A' in the equation A∗x=b, the problem is transformed into a solvable form using techniques from numerical linear algebra.

### Transformation to ATAx=ATb (Normal Equations)

The standard approach to find the least-squares solution for 'x' in an overdetermined system (where there are more equations than unknowns, as is the case in upscaling where n>m) is to multiply both sides of the equation A∗x=b by the transpose of A, denoted as AT or A′. This yields the transformed equation: ATAx=ATb. This transformation is a common technique used to convert a system of linear equations into a form where a unique or best-fit solution can be found.

The resulting matrix ATA is now a square, symmetric m×m matrix. Crucially, for well-behaved resizing kernels, this matrix is typically positive-definite, which ensures its invertibility and stability for numerical methods. The vector ATb is a vector with 'm' elements. This transformed system, known as the normal equations, is now amenable to efficient numerical solution methods, even for very large image dimensions.

### Detailed Explanation of LDLT Decomposition

The Descale plugin, a key tool in Vapoursynth for inverse scaling, leverages LDLT decomposition to solve the system ATAx=ATb. LDLT decomposition is a specific method for factoring a symmetric matrix (like ATA) into the product of a lower triangular matrix (L), a diagonal matrix (D), and the transpose of the lower triangular matrix (LT). The decomposition is expressed as ATA=LDLT.

Both L and D are triangular matrices, meaning many of their elements are zero. This structure makes subsequent computational steps significantly simpler and more efficient. This decomposition is particularly well-suited for large, sparse, and banded symmetric matrices, which is characteristic of the ATA matrix derived from image resizing kernels. The sparseness arises because each output pixel only depends on a limited number of input pixels, leading to many zero entries in the 'A' matrix. The "banded" nature means non-zero elements are concentrated around the main diagonal, further aiding computational efficiency.

### Forward and Back Substitution for Pixel Restoration

Once the LDLT decomposition of ATA is performed, the solution for 'x' (the original pixel values) is obtained through a two-step substitution process, effectively breaking down the complex matrix inversion into simpler, sequential solves:

1. **Forward Substitution**: The first step involves solving the equation LDy=ATb for an intermediate vector 'y'. Since L is a lower triangular matrix and D is a diagonal matrix, this system can be solved efficiently by iterating through the equations from top to bottom, substituting already computed values of 'y' into subsequent equations. This process is computationally straightforward.

2. **Back Substitution**: The final step involves solving the equation LTx=y for 'x'. Since LT is an upper triangular matrix (the transpose of a lower triangular matrix), this system can be solved efficiently by iterating through the equations from bottom to top, substituting already computed values of 'x' into preceding equations. This process yields the desired vector 'x', which represents the restored pixel values of the original, native-resolution image.

This entire mathematical process, from formulating the normal equations to applying LDLT decomposition and subsequent substitutions, allows for a robust and computationally feasible method to invert deterministic upscaling operations, thereby restoring the original image data as accurately as possible within the constraints of real-world data imperfections.

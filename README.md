# Enhanced Image Restoration

A small, exploratory project on how **linear algebra** and **convolution** can be used to clean up a noisy image.

> This is a learning exercise, not a production-grade restoration model. The goal was to play with the concepts (images as matrices, convolution, LU decomposition, least squares) and see how they behave in practice.

## Idea

A grayscale image is just a matrix of pixel values, so damaging and repairing it can be expressed with matrix operations:

```
Original Image → Add Noise → Gaussian Blur (denoise) → Solve Ax = b (restore) → Compare Methods
```

## What the notebook does

1. **Load and corrupt an image**: takes a 100×100 crop of SciPy's `ascent` sample image and adds Gaussian noise (mean 0, std 20).
2. **Denoise with convolution**: applies a 3×3 Gaussian kernel (`[[1,2,1],[2,4,2],[1,2,1]] / 16`) using `scipy.signal.convolve2d`.
3. **Restore with linear systems**: models the 1D blur along each row as `A · x = b`, where `A` is a tridiagonal matrix (`0.5` on the diagonal, `0.25` on the off-diagonals), `b` is a blurred row, and `x` is the restored row. This is solved two ways:
   - **LU decomposition** (`scipy.linalg.lu_factor` / `lu_solve`): factorise `A` once, then reuse it for every row. A 5×5 patch is also used first to check that `P·L·U` reproduces the matrix.
   - **Least squares** (`np.linalg.lstsq`): solves each row from scratch.
4. **Measure quality** with the Frobenius norm of the residual and PSNR for every stage (noisy, blurred, LU, least squares), plus bar charts of the results.

## Concepts explored

- Images as matrices
- 2D convolution and Gaussian kernels
- LU decomposition (`A = P·L·U`) and the "factorise once, solve many times" idea
- Least squares as an alternative solver
- Error metrics: Frobenius norm and PSNR

## Results

Numbers from one run (noise is random, so yours will differ slightly):

| Stage | Frobenius residual | PSNR (dB) |
|---|---|---|
| Noisy | ~1986 | ~22.2 |
| Blurred (convolution) | ~1047 | ~27.7 |
| LU reconstructed | ~1272 | ~26.0 |
| Least squares reconstructed | ~1272 | ~26.0 |

| Solver | Time |
|---|---|
| LU decomposition | ~0.007 s |
| Least squares | ~0.28 s |

### Takeaways

- **LU is much faster than least squares** here, because `A` is factorised once and reused for all 100 rows, while `lstsq` redoes the heavy work for every row.
- **Both solvers give essentially identical images**, since `A` is invertible and the system is well-posed.
- **The blur step improved quality the most.** Solving `Ax = b` afterwards slightly lowered PSNR compared to the blurred image. That makes sense: it undoes the smoothing, which also brings back some of the noise. Inverting a blur is an ill-posed problem, and this is a nice, visible demonstration of why more advanced methods use regularisation.

## Limitations

- `A` is a simplified 1D model of the blur applied row by row. It ignores the vertical part of the 2D kernel and the zero-padding at image edges, so it is only an approximation of what the convolution actually did.
- Only a tiny 100×100 crop of one image is used.
- No random seed is set, so results change slightly between runs.
- Not a learned model. Everything is classical numerical linear algebra.

## Getting started

**Requirements**

```
numpy
scipy
matplotlib
opencv-python   # imported in the notebook but not actually used
```

**Run**

```bash
pip install numpy scipy matplotlib opencv-python
jupyter notebook Enhanced_Image_Restoration.ipynb
```

It also runs as-is on Google Colab. Run the cells from top to bottom.

## Possible next steps

- Add regularisation (e.g. Tikhonov) to the solve step to stop it re-amplifying noise
- Build the full 2D blur operator instead of the row-by-row 1D approximation
- Try other kernels (larger Gaussian, box blur) and noise levels
- Set a random seed and average results over several runs
- Compare against other denoisers such as median filtering or non-local means

## Files

- `Enhanced_Image_Restoration.ipynb`: the full notebook with explanations, code, and plots

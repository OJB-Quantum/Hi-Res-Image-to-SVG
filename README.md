# Hi-Res-Image-to-SVG
A tool that runs in the browser using Colab and a Graphics Processing Unit (GPU) to identify and convert high-resolution images into Scalable Vector Graphics (SVG) outputs. Control knobs are included for easy parameter adjustment.

---

You can use a free GPU in Google Colab as needed.

## Click to use the Hi-Res-to-SVG converter in Google Colab: [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/OJB-Quantum/Hi-Res-Image-to-SVG/blob/main/Hi_Res_Image_to_SVG.ipynb)

---

## Primer on Color Quantization and Vector Embedding

Converting highly detailed raster graphics (such as photographs or high-resolution scans) into pure vector paths frequently results in unmanageable file sizes. Standard tracing engines attempt to draw a unique geometric shape for every distinct pixel gradient, overwhelming the rendering agent. 

To preserve photographic fidelity while providing an SVG container, this tool employs an embedding technique. It serializes the raster data and encloses it within the SVG Document Object Model (DOM). To maintain performance and reduce file weight, the tool performs Color Quantization before embedding. By utilizing K-Means clustering accelerated by a GPU, the tool reduces a 24-bit true-color image (over 16 million colors) to an optimized 8-bit palette (256 colors) that best represents the original image.

## Acronyms and Symbols

| Term/ Symbol | Definition |
| :--- | :--- |
| **BMP** | Bitmap Image File |
| **CUDA** | Compute Unified Device Architecture |
| **DOM** | Document Object Model |
| **DPI** | Dots Per Inch |
| **GPU** | Graphics Processing Unit |
| **JPEG** | Joint Photographic Experts Group |
| **PNG** | Portable Network Graphics |
| **SVG** | Scalable Vector Graphics |
| **TIFF** | Tagged Image File Format |
| **URI** | Uniform Resource Identifier |
| $K$ | Total number of color clusters (e.g., 256 for 8-bit) |
| $x_i$ | The color vector of the $i$-th pixel |
| $\mu_j$ | The color vector of the $j$-th cluster centroid |

---

## Mathematical Foundation: K-Means Clustering

The core of the color reduction process relies on partitioning the image pixels into $K$ clusters. The algorithm minimizes the within-cluster sum of squares (variance). Mathematically, it finds the sets $S = \{S_1, S_2, \dots, S_K\}$ that minimize the objective function $J$:

$$J = \sum_{j=1}^{K} \sum_{x_i \in S_j} ||x_i - \mu_j||^2$$

The GPU processes the Euclidean distance $||x_i - \mu_j||$ for millions of pixels simultaneously, making this computationally intensive operation highly efficient.

---

## Pipeline Architecture

1.  **Image Ingestion:** The script prompts the user to upload a raster image file (PNG, JPEG, BMP, or TIFF). The byte stream is decoded directly into an OpenCV matrix.
2.  **GPU Acceleration:** The image matrix is transferred to the GPU VRAM via CuPy.
3.  **Color Quantization:** The K-Means algorithm runs for a specified number of iterations to find the optimal 8-bit color palette.
4.  **Vector Embedding:** The optimized matrix is encoded into a Base64 URI and embedded inside an SVG `<image>` tag, scaling proportionally to the target width.
5.  **Interactive Delivery:** The script generates a dynamic HTML download button within the Colab output cell, ensuring no intermediate files are saved to the virtual disk.

---

## Control Knobs

The script features an upfront parameter block for easy customization. You can adjust these values directly in the top-level variables of the script:

* **`TARGET_WIDTH_PX`** (Default: `2000`): The physical width of the generated SVG container in pixels. The height scales proportionally.
* **`SCAN_RESOLUTION_DPI`** (Default: `1000`): The embedded resolution tag metadata applied to the SVG document.
* **`COLOR_ACCURACY_BITS`** (Default: `8`): The bit-depth for quantization. An 8-bit depth yields 256 colors.
* **`KMEANS_ITERATIONS`** (Default: `15`): The number of cycles the GPU will spend stabilizing the color clusters. Higher values increase accuracy at the cost of processing time.

---

## Environment Setup and Dependencies

This project relies on `uv` for extremely fast package resolution and installation within the Colab environment. Ensure your Colab runtime is set to utilize a T4, L4, G4, or A100 GPU before execution.

```bash
# Bootstrap uv and install Python dependencies into the system environment
!pip install uv
!uv pip install --system opencv-python-headless cupy-cuda12x matplotlib

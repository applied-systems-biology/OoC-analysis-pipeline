# Organ-on-chip analysis pipeline
This repository provides a JIPipe workflow designed to facilitate diverse analysis of organ-on-chip (OoC) imaging data across both epithelial and endothelial compartments. The analysis of the epithelial compartment includes a comprehensive three-dimensional morphological characterization of fungal biomass. Furthermore, the pipeline facilitates the quantification of epithelial tissue architecture, including tissue thickness, tissue damage and loss of integrity based on distinctive markers (E-Cadherin/ZO-1). In addition to these morphological assessments, the pipeline provides quantitative insights into the spatial host-fungus interaction, as well as the level of fungal tissue invasion and vascular translocation. Conversely, the analysis of the endothelial compartment focuses on the identification of endothelial and immune cells, specifically macrophages, to quantify changes in their abundance within the OoC system.

![IoC](IoC.png)

## Applications
This pipeline has been applied to several intestine-on-chip infection models

📄 Methods are described in:

[1] https://doi.org/10.1016/j.biomaterials.2024.122525

[2] to be added!

[3] to be added!

📊 Data:

https://asbdata.hki-jena.de/Kaden_Alonso-Roman_AkbarimoghaddamEtAl2023_Caspofungin

https://asbdata.hki-jena.de/Schaal_AkbarimoghaddamEtAl2025_FMF

## Pipeline Overview

The workflow is structured into four main components. Each component is detailed below, outlining its specific functions, required inputs, user-defined parameters, and outputs:

### **1. File Import & Annotation**

Handles import, annotation, and channel splitting of multi-channel OoC datasets.

- Input structure: Dataset directory must contain `images/` (multi-channel data) and `masks/` (2D binary masks per image). Masks are required to exclude artifacts such as membrane edges.
- Channels:
  - Epithelial: microcolony, E-Cadherin/ZO-1, nuclei, brightfield (membrane pores)
  - Vascular: macrophages, nuclei, brightfield
- Annotation: Files are annotated based on filenames (e.g., replicate, condition, compartment).

**Output:** Annotated datasets with split channels and corresponding masks, ready for downstream analysis.

### **2. Epithelial Compartment Analysis**
The pipeline offers several analyses for the epithelial side of the chip, namely, microcolony morphometrics, tissue architecture and spatial analysis, as described in our previous paper `link to paper`. Parameters details of each compartments are provided below.

#### **Microcolony morphometrics**

Performs 3D segmentation and shape characterization of fungal microcolonies using the microcolony channel and mask.

Workflow steps and parameters:

- Signal enhancement & object detection
  - Gamma: controls contrast via power-law transformation (>!1 enhances bright regions, <!1 enhances dark regions)
  - Thresholding method: automatic segmentation method (e.g., Otsu, Triangle, Intermodes)
- Noise reduction
  - Volume: removes objects below a size threshold
  - Compactness: filters objects based on shape
  - Mean intensity: removes low-intensity artifacts
- Branch reconstruction (3D)
  - Border-to-border distance: defines proximity threshold to reconnect fragmented branches
  - Window enlargement: ensures gaps between disconnected branches are fully covered
- Watershed split (object separation)
  - Feret diameter: selects large objects for splitting
  - Closing disk size1: preserves compact object centers
  - Opening disk size: removes thin connections between objects
  - Minimum seed area: prevents over-segmentation by filtering small seeds

**Output:** Denoised and reconstructed microcolonies, separated objects, and a morphometrics table (e.g., width, height, depth, volume, surface area, compactness, sphericity, elongation, flatness).

#### **Tissue architecture**

Quantifies epithelial tissue integrity and thickness using ZO-1/E-Cadherin, nuclei, brightfield (BF), and mask inputs.

Workflow steps and parameters:

- BF preprocessing (pore enhancement)
  - External gradient disk size: enhances pore edges
  - White top-hat disk size: highlights bright pore structures
  - Gaussian blur size: smooths noise
  - Background subtraction (rolling ball): removes uneven illumination (typically ≥ pore size, ~8–10 µm)
- Pores detection
  - Thresholding method: segments membrane pores
  - Dilation disk size: expands pore regions for robust detection
- Membrane position → Determines membrane position based on local pore signal variation
  - Window size: defines local region for analysis
  - Step size: controls resolution of sliding window across image
- Tissue damage
  - Threshold: binarizes signal to detect damaged regions
  - Minimum area: removes small artifacts
  - Closing disk size: fills gaps and refines დაზ damage regions
- Nuclei enhancement & 3D segmentation
  - Mean filter disk size: enhances nuclei signal
  - Thresholding method: segments nuclei
  - Window size & step size: control local 3D segmentation
  - Volume filter: removes small noise objects
- Epithelial tissue interpolation
  - Interpolation window size: estimates tissue position between nuclei by local averaging
- Tissue architecture: Heatmap
  - Pixel-wise epithelial tissue thickness by subtracting tissue position from membrane position.

**Output:** Tissue damage mask and statistics, membrane position map, interpolated epithelial tissue position map, and tissue thickness (heatmap and quantitative table).

#### **Spatial co-localization**

Quantifies the 3D distribution of fungal microcolonies relative to epithelial tissue using outputs from Microcolony Morphometrics and Tissue Architecture. No additional parameters are required.

Workflow steps:

- Stack Classification: Classifies the image stack into spatial regions:
  - Horizontal classification: luminal, tissue, and vascular compartments
  - Vertical classification: high-density and low-density tissue regions based on tissue thickness
- Multi-Colocalization Analysis (horizontal classified stacks) performs 3D object-based colocalization between microcolonies and horizontal classified stacks to quantify:
  - overlap with the luminal side
  - tissue penetration
  - vascular invasion
- Multi-Colocalization Analysis performs 3D object-based colocalization between microcolonies and vertical classified stacks to quantify:
  - Quantifies colocalization of microcolonies with high-density and low-density tissue regions

**Output:** Quantitative tables describing host–pathogen spatial interactions.

### **3. Vascular Compartment Analysis**

The pipeline performs a 2d segmentation of endothelial cells and macrophages using pretarined Cellpose model.

**Output:** Segmentation masks, cell density, and morphometric measurements for nuclei and macrophages.

## Implementation

please visit https://jipipe.hki-jena.de/ on how to run a JIPipe workflow.

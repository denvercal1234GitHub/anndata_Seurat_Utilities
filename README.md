# anndata_Seurat_Utilities

Utilities to prepare **AnnData (.h5ad)** files for **Seurat / SeuratDisk** conversion  
and safe exchange between **Python** and **R** single-cell analysis workflows.

<!-- Uncomment these once CI & PyPI are active -->
<!-- 
[![PyPI](https://img.shields.io/pypi/v/anndata-seurat-utilities.svg)](https://pypi.org/project/anndata-seurat-utilities/)
[![CI](https://github.com/denvercal1234GitHub/anndata_Seurat_Utilities/actions/workflows/ci.yml/badge.svg)](https://github.com/denvercal1234GitHub/anndata_Seurat_Utilities/actions)
-->

---

## What this package does

This library provides the robust function:

`prepare_adata_for_seurat_drop_empty_v3()`

which:

- **Drops empty `obs` / `var` columns** (with provenance stored in `.uns`)
- **Sanitizes column names** (avoids `/`, spaces, commas, semicolons, etc.)  
  → prevents accidental HDF5 path splitting (common SeuratDisk failure mode)
- **Converts pandas extension dtypes**  
  (`boolean`, `Int64`, `string`, `Categorical`, masked arrays)  
  → into safe atomic dtypes so HDF5 does *not* create `{mask, values}` groups
- **Optionally drops heavy layers** like `counts_mouse`
- **Ensures all metadata is stored in a SeuratDisk-compatible form**
- **Writes `.h5ad` cleanly** using `convert_strings_to_categoricals=False` when possible

The resulting `.h5ad` file loads reliably in:

```r
SeuratDisk::Convert("file.h5ad", dest = "h5seurat")
LoadH5Seurat("file.h5seurat")
```


## Installation
### Install from TestPyPI (for pre-release testing)
```bash
python -m venv venv && source venv/bin/activate
pip install --upgrade pip

pip install \
  --index-url https://test.pypi.org/simple/ \
  --extra-index-url https://pypi.org/simple \
  anndata-seurat-utilities==0.1.0

```

Why the extra index URL?
TestPyPI does not mirror PyPI. Dependencies such as anndata>=0.9 must come from PyPI.

### Insysll official release from PyPI

```bash

pip install anndata-seurat-utilities

```

## Minimal usage example 

```python
from anndata_seurat_utils.prepare_for_seurat import (
    prepare_adata_for_seurat_drop_empty_v3
)
import anndata as ad

# Load an AnnData object (raw or processed)
adata = ad.read_h5ad("input.h5ad")

# Clean, sanitize, and write to a Seurat-compatible h5ad
prepare_adata_for_seurat_drop_empty_v3(
    adata=adata,
    out_h5ad_path="cleaned_for_seurat.h5ad",
    drop_layers=("counts_mouse",),   # optional
    convert_strings_to_categoricals_on_write=False,
    sanitize_var_names=True,
    sanitize_obs_names=True,
    drop_empty_obs_and_var=True,
    verbose=True,
)

print("Wrote cleaned Seurat-ready file: cleaned_for_seurat.h5ad")
```

## Why using this package?

Direct conversion of AnnData → Seurat using:

```r
SeuratDisk::Convert("file.h5ad", dest = "h5seurat")
```

often fails due to:

### Pandas extension dtypes
boolean, Int64, or masked arrays serialize as:
```bash
/meta.data/<column>/mask  
/meta.data/<column>/values  
```

SeuratDisk expects {levels, values}, not {mask, values}.

### Column names containing /

AnnData writes / as HDF5 paths, causing:

```
/meta.data/Kir3dl1
    2__nonSCVI_normlog
```

instead of one metadata column → SeuratDisk throws missing-column errors.


### Empty columns
Fully-NA metadata columns become incomplete HDF5 groups →
SeuratDisk error: "Missing required datasets 'levels' and 'values'".

This utility fixes all of these issues.


## Requirements
anndata >= 0.9
pandas
numpy
scipy
mygene (only required for optional gene-mapping utilities)


## Development install

```bash
git clone https://github.com/denvercal1234GitHub/anndata_Seurat_Utilities
cd anndata_Seurat_Utilities
pip install -e .
pytest -q
```

## Tests

```bash
# create virtual env
python -m venv venv
source venv/bin/activate
# install deps
pip install -r requirements.txt
# run pytest
pytest -q
```

Current version: 0.1.0


## Contributing
Contributions are welcome!
Please open an issue or pull request if you’d like to improve the utilities.

## License
MIT License — see the LICENSE file.

## Citation
If this package contributes to your work, please cite the repository:

```bash
Nguyen, Q. (2025). anndata_Seurat_Utilities.
https://github.com/denvercal1234GitHub/anndata_Seurat_Utilities
```

A CITATION.cff file will be added in a future release.


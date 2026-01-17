# LISA Binaries
This repository accompanies the MNRAS paper:
> **Population synthesis predictions of the Galactic compact binary gravitational wave foreground detectable by LISA**  
> *Jake McMillan*  
> *Monthly Notices of the Royal Astronomical Society (MNRAS)*

and provides the simulated compact binary populations and analysis tools used in that work.

The paper is available via DOI under an MIT License: **[https://doi.org/XXXXXXXX](https://doi.org/10.1093/mnras/stag117)**  

## Citation
If you use any data, code, or methodology from this repository, please cite the paper above.
```bibtex
@article{FULL biblatex,
  title   = {Population synthesis predictions of the Galactic compact binary gravitational wave foreground detectable by LISA},
  author  = {...},
  journal = {MNRAS},
  year    = {2026},
  doi     = {...},
}
```

## Overview
This repository contains:
- Simulated Galactic compact binary populations generated using the COSMIC population-synthesis code.
- Catalogs suitable for predicting the gravitational wave foreground in the LISA band.
- A lightweight Python module to compute PSD estimates from the catalog data.

These simulations are ideal for studying compact binary evolution, gravitational wave sources, and related astrophysical phenomena.

## Key Features
- Populations correspond to specific **common envelope (CE) efficiency, α** values.  
- Only binaries with **gravitational wave frequencies > 40 μHz** are included.  
- **Supernova kicks** have been fully evolved.  
- Each dataset represents roughly **10% of a realistic Galactic population**.
- Eccentric binaries and higher harmonics are included in the PSD calculation.

## Large files (Git LFS required)
The population catalogs (`.fits`) are stored using **Git Large File Storage (Git LFS)**.

Before cloning the repository, install Git LFS:
```bash
git lfs install
git clone https://github.com/USERNAME/LISA_Binaries.git
```
Without Git LFS, the `.fits` files will be downloaded as pointer files and cannot be read by `astropy`.

## Example:

```python
import numpy as np
import matplotlib.pyplot as plt

from astropy.table import Table
import astropy.units as u

import LISA_Binaries_PSD

data = Table.read("LISA_Binaries_alpha_2.00.fits").to_pandas()

m1 = data.mass_1.values * u.Msun
m2 = data.mass_2.values * u.Msun
M_chirp = (m1*m2)**(3/5) / (m1+m2)**(1/5)

R0 = data.R_0.values * u.kpc
z0 = data.z_0.values * u.kpc
theta0 = data.theta_0.values
D = np.sqrt(z0**2 + (8*u.kpc - R0*np.cos(theta0))**2 + (R0*np.sin(theta0))**2)

forb = (1/(data.porb.values * u.day)).to(u.Hz)
ecc  = data.ecc.values


f_noise = np.logspace(-5, 0, 1000)
f_l, P_l, err_l = LISA_Binaries_PSD.calc_LISA_signal(M_chirp, D, forb, ecc=ecc, c_0=1.3, sf=10, lineout=False)


fig, axs = plt.subplots(figsize=(3.375, 2.25), dpi=600)

plt.plot(f_noise, LISA_Binaries_PSD.LISA_S_noise(f_noise), c="black", label="Noise")
plt.errorbar(f_l, P_l, err_l, c="#4A90E2", ecolor="#A4C8F0", label="GWs", capsize=1.5, elinewidth=1, capthick=1)

plt.legend()
plt.xscale("log")
plt.yscale("log")
plt.xlim(6e-6, 1.8)
plt.ylim(4e-44, 9e-27)
plt.xlabel(r"$f/\mathrm{Hz}$")
plt.ylabel(r"PSD$/\mathrm{Hz^{-1}}$")

plt.tight_layout()
plt.show()
```

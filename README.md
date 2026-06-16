# MolCryst-MLIPs

**Molecular Crystals Database for Machine Learning Interatomic Potentials**

Fine-tuned MACE models for polymorphic molecular crystals, trained using the [AMLP framework](https://github.com/adamlaho/AMLP).

## Models
| Compound | CSD Code | Energy MAE<br>(meV/atom) | Force MAE<br>(meV/Å) | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Model&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Dataset&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; |
|----------|----------|:---------------------:|:-----------------:|:-----:|:-------:|
| Resorcinol | RESORA | 1.568 | 3.903 | [↓ model](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/models/resora.model) | [↓ train](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/datasets/resora_train.h5) [↓ valid](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/datasets/resora_valid.h5) |
| Durene | DURENE | 1.647 | 5.193 | [↓ model](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/models/durene.model) | [↓ train](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/datasets/durene_train.h5) [↓ valid](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/datasets/durene_valid.h5) |
| Coumarin | COUMAR | 1.670 | 4.296 | [↓ model](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/models/coumar.model) | [↓ train](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/datasets/coumar_train.h5) [↓ valid](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/datasets/coumar_valid.h5) |
| Benzamide | BZAMID | 0.713 | 8.786 | [↓ model](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/models/bzamid.model) | [↓ train](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/datasets/bzamid_train.h5) [↓ valid](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/datasets/bzamid_valid.h5) |
| Niacinamide | NICOAM | 1.513 | 7.207 | [↓ model](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/models/nicoam.model) | [↓ train](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/datasets/nicoam_train.h5) [↓ valid](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/datasets/nicoam_valid.h5) |
| Nicotinamide | NICOAC | 1.201 | 5.824 | [↓ model](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/models/nicoac.model) | [↓ train](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/datasets/nicoac_train.h5) [↓ valid](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/datasets/nicoac_valid.h5) |
| Isonicotinamide | EHOWIH | 1.912 | 10.809 | [↓ model](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/models/ehowih.model) | [↓ train](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/datasets/ehowih_train.h5) [↓ valid](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/datasets/ehowih_valid.h5) |
| Pyrazinamide | PYRIZIN | 1.634 | 6.732 | [↓ model](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/models/pyrizin.model) | [↓ train](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/datasets/pyrizin_train.h5) [↓ valid](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/datasets/pyrizin_valid.h5) |
| Benzoic acid | BENZAC | 1.329 | 7.897 | [↓ model](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/models/benzac.model) | [↓ train](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/datasets/benzac_train.h5) [↓ valid](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/datasets/benzac_valid.h5) |
| Acridine | ACRDIN | 3.700 | 8.300 | [↓ model](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/models/acridine.model) | [↓ train](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/datasets/acridine_train.h5) [↓ valid](https://huggingface.co/adamlaho/MC-MLIP/resolve/main/datasets/acridine_valid.h5) |
| **Mean** | | **1.689** | **6.895** | | |

## Training Protocol

- **Foundation Model**: MACE-MH1 (`mace-mh-1-omol-1%`)
- **Reference Data**: DFT (PBE-D4) optimizations + AIMD trajectories (25-500K)
- **DFT Settings**: VASP, 650 eV cutoff, EDIFF = 10⁻⁷ eV

**Two-stage training:**
1. Initial: LR = 2×10⁻³, energy weight = 100, force weight = 10
2. SWA (epoch 200+): LR = 5×10⁻⁵, force weight = 100

Early stopping with patience = 75 epochs. All models trained in float64.

## Validation

All models validated for:
- **Energy conservation**: NVE drift < 10⁻⁵ over 25 ps
- **Structural integrity**: RDFs and P₂ order parameters preserved

## Usage

We recommend using the [AMLP-Analysis module](https://github.com/adamlaho/AMLP) (`amlpa.py`) for running simulations with these models:
```bash
python3 amlpa.py structure.xyz config.yaml
```

In your `config.yaml`, point to the downloaded model:
```yaml
model_paths:
  - 'path/to/model.model'
device: 'gpu'
gpus: ['cuda:0']
```

Alternatively, you can use the models directly via the MACE calculator:
```python
from mace.calculators import MACECalculator
calc = MACECalculator(model_paths="path/to/model.model", device="cuda")
atoms.calc = calc
```

For full configuration options (MD, geometry optimization, RDF analysis, etc.), refer to the [AMLP documentation](https://github.com/adamlaho/AMLP).

## Citation

If you use these models, please cite:

```bibtex

@article{lahouari2026molcryst,
  title={MolCryst-MLIPs: A Machine-Learned Interatomic Potentials Database for Molecular Crystals},
  author={Lahouari, Adam and Ai, Shen and Han, Jihye and Hoffstadt, Jillian and Hoellmer, Philipp and Infante, Charlotte and Jain, Pulkita and Kadam, Sangram and Martirossyan, Maya M and McCune, Amara and others},
  journal={arXiv preprint arXiv:2604.13897},
  year={2026}
}

@article{lahouari2026amlp,
  title={Automated Machine Learning Pipeline: Large Language Models-Assisted Automated Data set Generation for Training Machine-Learned Interatomic Potentials},
  author={Lahouari, Adam and Rogal, Jutta and Tuckerman, Mark E.},
  journal={Journal of Chemical Theory and Computation},
  volume={22},
  number={1},
  pages={305--317},
  year={2026},
  publisher={American Chemical Society},
  doi={10.1021/acs.jctc.5c01610},
  url={https://doi.org/10.1021/acs.jctc.5c01610}
}
```

## License

MIT License

## Acknowledgments

- [MACE](https://github.com/ACEsuit/mace) development team
- NYU High Performance Computing

# 🔴 Monocular Depth Estimation to Predict Topology of the Martian Surface

This project explores monocular depth estimation as a fast, low-cost way to reconstruct the topology of the Martian surface, predicting a Digital Terrain Model (DTM) directly from a single HiRISE image instead of relying on stereo photogrammetry.

> 📄 **Related paper:** *An Adversarial Generative Network Designed for High-Resolution Monocular Depth Estimation from 2D HiRISE Images of Mars.*
> Published in [MDPI Remote Sensing (2022)](https://www.mdpi.com/2072-4292/14/18/4619). See [Citation](#-citation) below.

[![Hugging Face Spaces](https://img.shields.io/badge/%F0%9F%A4%97%20Hugging%20Face-Spaces-blue)](https://huggingface.co/spaces/mattiagatti/mars_dtm_estimation)

## 🔭 Background

**Monocular depth estimation** predicts the depth of each pixel (its distance from the camera) from a single image. Depth is usually recovered instead through stereoscopy, which requires a camera with two or more lenses and images of the same scene from slightly different viewpoints.

In planetary science, this depth information takes the form of a **Digital Terrain Model (DTM)** — a raster of surface elevations from which a 3D model of the terrain can be reconstructed. DTMs are essential for exploration and research, but producing one is slow and demanding: it relies on stereo pairs, specialised software, and considerable computational and human effort.

The **High Resolution Imaging Science Experiment (HiRISE)**, on board the Mars Reconnaissance Orbiter, has been imaging Mars since 2006 and has yielded many such stereo-derived DTMs. This project asks whether a monocular model can approximate that output from a single image, making Martian topology estimation substantially faster and cheaper.

---

## ⚠️ Disclaimer

The model in this repository performs monocular depth estimation with predictions in metres, and has been refined since publication, so its results now improve on those reported in the paper. The paper itself describes a different model that upscales the input resolution by 4× while estimating a normalised DTM in the range [0, 1]; its code is available [here](https://gitlab.com/riccardo2468/srdinet).

---

## 🗺️ Dataset

The dataset was scraped from the [UAHiRISE](https://www.uahirise.org/) website. For each observation, two co-registered rasters at 1 m/px were downloaded: an 8-bit greyscale image (the left or right frame of the stereo pair) and its Float32 DTM.

Because these rasters can be very large — and model complexity scales with input size — each image is split into smaller tiles that are stitched back together after processing. This yields roughly **160k patches**, each a 2-channel 512 × 512 raster pairing the image (channel 1) with its DTM (channel 2). The DTMs are stored as absolute elevations and must be converted to relative heights before use with a monocular model. The full dataset is about **500 GB**.

---

## ⚙️ Environment Setup

```bash
git clone https://github.com/mattiagatti/hirise-monocular-depth-estimation.git
cd hirise-monocular-depth-estimation

sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt-get update
sudo apt-get install python3.11 python3.11-venv

python3.11 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

---

## 🚀 Usage

Scrape Mars data from [UAHiRISE](https://www.uahirise.org/):
```bash
nohup python scrape.py > download.log 2>&1 &
```

Divide the generated tiles into train, validation, and test splits:
```bash
python splits/generate_splits.py
```

Download the backbone weights from
[here](https://deci-pretrained-models.s3.amazonaws.com/mit_backbones/mit_b4.pth), place the
file in the `glpdepth/weights` folder, and then train the model (point `--data_dir` at your
dataset location):
```bash
nohup python train.py --data_dir /path/to/HiRISE --batch_size 4 > train.log 2>&1 &
```

Try the demo:
```bash
python app.py
```

Build a Docker image:
```bash
docker build -f ./API_App/Dockerfile --tag marsdem .
```

---

## 🙏 Acknowledgements

This project builds on [GLPDepth](https://github.com/vinvino02/GLPDepth), the official
PyTorch implementation of *Global-Local Path Networks for Monocular Depth Estimation with
Vertical CutDepth* (Kim et al., 2022), used here as the backbone architecture for depth
prediction.

```bibtex
@article{kim2022global,
  title   = {Global-Local Path Networks for Monocular Depth Estimation with Vertical CutDepth},
  author  = {Kim, Doyeon and Ga, Woonghyun and Ahn, Pyungwhan and Joo, Donggyu and Chun, Sehwan and Kim, Junmo},
  journal = {arXiv preprint arXiv:2201.07436},
  year    = {2022}
}
```

---

## 📖 Citation

If you use this work, please cite the related paper:

```bibtex
@article{lagrassa2022srdinet,
  title   = {An Adversarial Generative Network Designed for High-Resolution Monocular Depth Estimation from 2D HiRISE Images of Mars},
  author  = {La Grassa, Riccardo and Gallo, Ignazio and Re, Cristina and Cremonese, Gabriele and Landro, Nicola and Pernechele, Claudio and Simioni, Emanuele and Gatti, Mattia},
  journal = {Remote Sensing},
  volume  = {14},
  number  = {18},
  pages   = {4619},
  year    = {2022},
  publisher = {MDPI},
  doi     = {10.3390/rs14184619},
  url     = {https://doi.org/10.3390/rs14184619}
}
```

---

## 📜 License

The code is released under the [MIT License](LICENSE).

Exceptions:

- `glpdepth/` is not covered by the MIT License: `glpdepth/model.py` is adapted from [GLPDepth](https://github.com/vinvino02/GLPDepth) and `glpdepth/mit.py` comes from [SegFormer](https://github.com/NVlabs/SegFormer) (NVIDIA Source Code License). Both are licensed for non-commercial use only (research and evaluation).
- The images in `examples/` are HiRISE products (credit: NASA/JPL/University of Arizona).

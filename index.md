# Beyond Major Floods

> Markdown version of the project page at https://aiml-sdu.github.io/beyond-major-floods-page/. Interactive figures are on the web page. Every number here also appears there and comes from the paper or its stored results.

Deep learning for detecting shallow water inundation in agricultural areas

**Phongsakon Mark Konrad**, **Toygar Tanyel**, **Serkan Ayvaz**

University of Southern Denmark · Istanbul Technical University · 2025

[Proceedings](https://www.sciencedirect.com/science/article/pii/S1877050925028194) · [PDF](https://www.sciencedirect.com/science/article/pii/S1877050925028194/pdfft) · [Markdown](index.md) · Cite

Point at any mark. The sheet follows the text as you scroll, and the switches above override it.

## Legend

We trained ten segmentation networks on one hand labelled Sentinel-1 radar image of coastal farmland in Southern Denmark and asked them to separate land, sea and inland water.

Land and sea are close to solved. ResNet-UNet and DeepLabV3+ both reach 0.99 IoU on each.

Inland water is where the networks part ways. The best inland water IoU is 0.56 ± 0.14 for ResNet-UNet and 0.54 ± 0.07 for DeepLabV3+, while two networks score 0.00.

ResNet-UNet keeps pace with DeepLabV3+ at a lower cost. It peaks at 3071.44 MB of GPU memory against 9640.03 MB and processes 7.57 images per second against 7.33.

## Reading this sheet

SAR

Sentinel-1 is a radar satellite that images the ground through cloud. We use its VH polarisation in decibels, where calm water returns little signal and looks dark.

Classes

Every pixel is labelled sea, inland water or land. Inland water covers lakes, ponds and flooded patches of field.

Tile

A 256 × 139 pixel cut of the scene. The study region becomes a grid of 10 by 10 tiles.

IoU

Intersection over union. The overlap between predicted and true region divided by their union. A strict score where 1 is perfect.

Dice

Twice the overlap divided by the sum of both areas. It tracks IoU but runs higher, and it is related to the F1 score.

Marks

Each mark is one network. Height is its test score for the chosen class, position across is its cost, and whiskers span one standard deviation across folds.

### Ten networks on one sheet

Flood maps are usually drawn for disasters. On coastal farmland the water that matters is often a shallow pool on a field or a farm track, and that is the class every model we tested finds hardest.

Most radar water maps sort pixels into water and land. On coastal farmland that leaves the sea and a flooded field in the same class, so we label sea, inland water and land separately and compare ten networks on the same tiles.

Taken over all three classes, the leaders are ResNet-UNet with an IoU of 0.85 ± 0.05 and DeepLabV3+ with a Dice score of 0.95 ± 0.02.

### Land and sea are close to solved

A coastal scene is mostly land and sea, and both are easy for radar. Open sea returns a dark, even signal and farmland returns a bright, textured one. Several of the ten networks, the two leaders among them, reach between 0.98 and 0.99 IoU on both classes, with Dice near 1.00.

On the sheet the marks crowd against the top edge.

### Two networks stumble

UNet stumbles on both large classes, with 0.87 ± 0.21 IoU on land and 0.78 ± 0.42 on sea, and FC-DenseNet reaches only 0.81 ± 0.02 on sea. For the rest the two large classes sit near the ceiling, and inland water sits far below it.

### Inland water separates the models

Inland water is the minority class and the shallow one. A flooded field is thinner than the sea and can look more like wet soil than like open water. When a tile holds a large inland lake, the networks also tend to call it sea.

So inland water is where the ten networks spread out. ResNet-UNet reaches 0.56 ± 0.14 IoU and DeepLabV3+ 0.54 ± 0.07. SwinUNet follows at 0.45 ± 0.06, and UNet++ and FC-DenseNet report 0.00 IoU on this class.

### On Dice the top order flips

The argument for ResNet-UNet would fail if it trailed DeepLabV3+ on inland water, the one class where the networks differ. On IoU it does not, with 0.56 ± 0.14 against 0.54 ± 0.07. On Dice it does trail, with 0.74 ± 0.17 against 0.86 ± 0.07, so we read the two as comparable and not as a ranking. Both scores come from one labelled image of one region, and the spreads are wide.

We weighted the loss toward inland water during training to counter its rarity. We also tried weighted and unweighted ensembles of the networks. They helped slightly on land and sea and did worse on inland water than DeepLabV3+ or ResNet-UNet alone.

### The lighter network keeps pace

A model that watches farmland has to run often and cheaply. We measured every network on a 64GB Nvidia Jetson AGX Orin developer kit, an embedded board of the kind used for edge computing.

The two accuracy leaders differ in cost. DeepLabV3+ peaks at 9640.03 ± 211.46 MB of GPU memory and ResNet-UNet at 3071.44 ± 124.76 MB. Read across the sheet and the two leaders sit far apart at almost the same height.

### Fast as well as light

ResNet-UNet also has the highest throughput, 7.57 ± 0.19 images per second, and the shortest inference time, 7.93 ± 0.21 seconds. SwinUNet uses the least memory of all, and FC-DenseNet is the slowest at 2.71 ± 0.03 images per second.

Put together, ResNet-UNet gives a similar inland water score at a much lower memory cost. For this use case we take it as the more practical choice.

### One map becomes a watch over time

A single segmentation is a snapshot. A farmer needs to know whether a tile holds more water today than it usually does. Our pipeline turns the per tile maps into that signal in four stages.

Sentinel-1 radar images in VH polarisation cover the region from 54.8339° to 55.0847° N and 9.5856° to 10.0855° E. Each image is cut into 256 × 139 pixel tiles, a grid of 10 rows by 10 columns, and every tile keeps its coordinates, its date and nearby street names.

### Train on one labelled day

One image from a day without flooding is labelled by hand into sea, inland water and land and checked against aerial photos. Augmentation grows the 100 tiles to 300. We tune on a 60, 20 and 20 split, then train with 5 fold cross validation and keep the 20% test split untouched.

### Segment every tile

The trained network labels every tile of a new image. Inland water is the class it labels least reliably, and a large inland lake can come out as sea. That matters less for the next stage, which tracks change over time.

### Flag change

For anomaly detection we use 12,000 unlabelled tiles from 2023. A sliding window of size 10 flags tiles above the 95th percentile of their own history, and an autoencoder or variational autoencoder flags tiles whose reconstruction error passes an Otsu threshold.

The anomaly stage shows flagged tiles on the map beside weather records for the same days, and we used those records to identify past flooding events as ground truth. The paper describes this stage and shows it on maps but does not report a score for it, so we give no number here.

## What this sheet shows and what it does not

### It shows

- On this region, ten common segmentation networks separate land and sea from Sentinel-1 VH radar almost perfectly.

- Inland water is the hard class for all ten, and the spread between networks lives there.

- ResNet-UNet and DeepLabV3+ lead, and ResNet-UNet does so at a lower memory and time cost on embedded hardware.

### It does not show

- Performance beyond one manually labelled image of one coastal region in Southern Denmark, split into 100 tiles.

- A measured accuracy for the anomaly detection stage.

- Real time guidance for vehicles in the field, since satellite passes over the same place are relatively infrequent.

Next we want to widen the data to other kinds of terrain and pair the satellite view with sensors on the vehicles themselves, so that decisions can be made closer to real time.

## Cite this article

```bibtex
@article{konrad2025beyond,
  title   = {Beyond Major Floods: Deep Learning for Detecting Shallow Water
             Inundation in Agricultural Areas},
  author  = {Konrad, Phongsakon Mark and Tanyel, Toygar and Ayvaz, Serkan},
  journal = {Procedia Computer Science},
  volume  = {270},
  pages   = {301--310},
  year    = {2025},
  note    = {29th International Conference on Knowledge-Based and
             Intelligent Information and Engineering Systems (KES 2025)}
}
```

Every plotted value comes from Tables 1 and 2 of the paper. The contour lines are decoration and the tile insets are an illustration. The work was supported by the Innovation Fund Denmark through DIREC and by a Danish Data Science Academy visit grant.

A project of the [Applied and Interpretable Machine Learning Research Group](https://aiml-sdu.github.io) at the University of Southern Denmark.

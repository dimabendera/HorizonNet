# Results on Structured3D dataset

## Dataset preparation
- Please contact [Structured3D](http://structured3d-dataset.org) to get the datas.
- Download all zip files under `{YOUR_DIR}`. Don't extract them.
- Run below to extract rgb and layout with original furniture and lighting setting only:
    ```bash
    python misc/structured3d_extract_zip.py --zippath {YOUR_DIR}/Structured3D_0.zip --outdir {TARGET_DIR_EXTRACT}
    python misc/structured3d_extract_zip.py --zippath {YOUR_DIR}/Structured3D_1.zip --outdir {TARGET_DIR_EXTRACT}
    # ... for all Structured3D_?.zip
    ```
- Run below to create `data/st3d_[train|valid|test]_full_raw_light` following HorizonNet training and testing dataset format. (This will use soft link instead of copy a new one.)
    ```bash
    python misc/structured3d_prepare_dataset.py --in_root {TARGET_DIR_EXTRACT}
    ```

## Training
```bash
python train.py --train_root_dir data/st3d_train_full_raw_light/ --valid_root_dir data/st3d_valid_full_raw_light/ --id resnet50_rnn__st3d --lr 3e-4 --batch_size_train 24 --epochs 50
```
See `python train.py -h` for more detail or [README.md](https://github.com/sunset1995/HorizonNet/blob/master/README.md) for more detail.

Download the trained model: [resnet50_rnn__st3d.pth](https://drive.google.com/open?id=16v1nhL9C2VZX-qQpikCsS6LiMJn3q6gO).
> The best selected model is epoch 50 which suggested that it still has the potential to improve by training longer.

## Testing
Generating layout for testing set:
```bash
python inference.py --pth ckpt/resnet50_rnn__st3d.pth --img_glob "data/st3d_test_full_raw_light/img/*" --output_dir tmp/ --visualize --relax_cuboid
```
- `--relax_cuboid`: **MUST** added (as we train with not only cuboid layout).
- `--output_dir`: a directory you want to dump the extracted layout
- `--visualize`: visualize raw output (without post-processing) from HorizonNet.


Quantitativly evaluate:
```bash
python eval_general.py --dt_glob "./tmp/*json" --gt_glob "data/st3d_test_full_raw_light/label_cor/*"
```

## Results
:clipboard: Below is the quantitative result on Structured3D testing set.

| # of corners | instances | 3D IoU | 2D IoU |
| :----------: | :-------: | :----: | :----: |
| 4            | 1067      | `94.14`  | `95.50` |
| 6            | 290       | `90.34`  | `91.54` |
| 8            | 130       | `87.98`  | `89.43` |
| 10+          | 202       | `79.95`  | `81.10` |
| odd          | 4         | `88.62`  | `89.80` |
| overall      | 1693      | `91.31`  | `92.63` |

- 2D IoU are based on top-down view
- The `odd` row mean non-even number of corners (ground truth is obviously non-manhattan layout while model output is the approximation of it)

#### Invalid Ground Truth
Four instances are skip by `eval_general.py` as the ground truth is self-intersecting. The top-down view of the four skipped instance are illutrated below where the red dot line are estimated by HorizonNet and the green solid line is the self-intersected ground truth.

| `scene_03327_315045` | `scene_03376_800900` | `scene_03399_337` | `scene_03478_2193` |
| :--: | :--: | :--: | :--: |
| ![](assets/scene_03327_315045.txt.png) | ![](assets/scene_03376_800900.txt.png) | ![](assets/scene_03399_337.txt.png) | ![](assets/scene_03478_2193.txt.png) |

#### More Visual Results

##### From Structured3D testing set `scene_03300_[190736,190737,190738]`:
![](assets/result_scene_03300_190736.png)
![](assets/result_scene_03300_190737.png)
![](assets/result_scene_03300_190738.png)

##### Model pretrained on Structured3D and directly testing on PanoContext:
![](assets/result_pano_0019e0a0c8ca0913e543c033a843c58f.png)
![](assets/result_pano_366d3aefe55ba4d736de7d20c260392d.png)
![](assets/result_pano_4d28fa2a55f9a72dc619fa32cd29f327.png)
![](assets/result_pano_aaovfbtgrawdhs.png)
![](assets/result_pano_ablaywgetaidfh.png)
![](assets/result_pano_aycixxfsgxupdv.png)
![](assets/result_pano_cc0b312e55110f1f92799d6ae601a06d.png)
![](assets/result_pano_cca67c3d82ce02225be68d1df5905034.png)
![](assets/result_pano_cd9fa57453ec2e2b308f111e15fb3f6c.png)
![](assets/result_pano_ea422c8bd1dff8113cf803b337cafc14.png)
![](assets/result_pano_f19f1a6a5015c26e01808f72d151e894.png)
![](assets/result_pano_fd65b5c485c70c3b68c17f6a0eb23cc6.png)


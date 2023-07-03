---
layout: post
title: yolo 图像识别
categories: [yolo]
---

### 数据标注工具

1. 使用 makesense 

https://www.makesense.ai/

2. 使用 labelImg, 

labelImg 在 python3.9 版本下稳定

```bash
pyenv install 3.9.16
```


安装 labelImg

```python
pip install labelImg
```

### 数据标注

先手动标注 10 张， 得到标注的 xml 文件结果后， 对这十张图像使用 imgaug（使用 imgaug 数据增强， 标注的结果会随着增强进行相应变化， 不需要重新标注）进行数据增强， 随机 hsv 空间变化、 旋转、 裁剪、 高斯模糊等， 每一张扩充 9 张数据

```python
import xml.etree.ElementTree as ET
import os
import numpy as np
from PIL import Image
import shutil
import imgaug as ia
from imgaug import augmenters as iaa
from tqdm import tqdm


def read_xml_annotation(root, image_id):
    in_file = open(os.path.join(root, image_id), encoding='UTF-8')
    # print(in_file)
    tree = ET.parse(in_file)
    root = tree.getroot()
    bndboxlist = []

    for object in root.findall('object'):  # 找到 root 节点下的所有 country 节点
        bndbox = object.find('bndbox')  # 子节点下节点 rank 的值

        xmin = int(bndbox.find('xmin').text)
        xmax = int(bndbox.find('xmax').text)
        ymin = int(bndbox.find('ymin').text)
        ymax = int(bndbox.find('ymax').text)
        # print(xmin,ymin,xmax,ymax)
        bndboxlist.append([xmin, ymin, xmax, ymax])
        # print(bndboxlist)

    # ndbox = root.find('object').find('bndbox')
    return bndboxlist



def change_xml_list_annotation(root, image_id, new_target, saveroot, xml_id):
    save_path = os.path.join(saveroot, xml_id)
    in_file = open(os.path.join(root, str(image_id) + '.xml'), encoding='UTF-8')  # 这里 root 分别由两个意思
    tree = ET.parse(in_file)
    elem = tree.find('filename')
    elem.text = xml_id + img_type
    xmlroot = tree.getroot()
    index = 0

    for object in xmlroot.findall('object'):  # 找到 root 节点下的所有 country 节点
        bndbox = object.find('bndbox')  # 子节点下节点 rank 的值

        new_xmin = new_target[index][0]
        new_ymin = new_target[index][1]
        new_xmax = new_target[index][2]
        new_ymax = new_target[index][3]

        xmin = bndbox.find('xmin')
        xmin.text = str(new_xmin)
        ymin = bndbox.find('ymin')
        ymin.text = str(new_ymin)
        xmax = bndbox.find('xmax')
        xmax.text = str(new_xmax)
        ymax = bndbox.find('ymax')
        ymax.text = str(new_ymax)

        index += 1

    tree.write(save_path + '.xml')


def simple_example(AUGLOOP,IMG_DIR,XML_DIR,AUG_IMG_DIR,AUG_XML_DIR):
    boxes_img_aug_list = []
    new_bndbox_list = []
    new_name = None

    for root, sub_folders, files in os.walk(XML_DIR):
        for name in tqdm(files):
            bndbox = read_xml_annotation(XML_DIR, name)
            shutil.copy(os.path.join(XML_DIR, name), AUG_XML_DIR)
            try:
                shutil.copy(os.path.join(IMG_DIR, name[:-4] + img_type), AUG_IMG_DIR)
            except:
                shutil.copy(os.path.join(IMG_DIR, name[:-4] + '.JPG'), AUG_IMG_DIR)
            # print(os.path.join(IMG_DIR, name[:-4] + img_type))

            for epoch in range(1, AUGLOOP + 1):
                # 增强
                if epoch == 1:
                    seq = iaa.Sequential([
                        ####0.75-1.5 随机数值为 alpha， 对图像进行对比度增强， 该 alpha 应用于每个通道
                        iaa.ContrastNormalization((0.75, 1.5), per_channel=True),
                    ])
                elif epoch == 2:
                    seq = iaa.Sequential([
                        #### loc 噪声均值， scale 噪声方差， 50%的概率， 对图片进行添加白噪声并应用于每个通道
                        iaa.AdditiveGaussianNoise(loc=0, scale=(0.0, 0.1 * 255), per_channel=0.75),
                    ])
                elif epoch == 3:
                    seq = iaa.Sequential([
                        iaa.Fliplr(1),  # 水平镜像翻转
                    ])
                # else:
                #     seq = iaa.Sequential([
                #         iaa.OneOf([iaa.Affine(rotate=90),
                #                    iaa.Affine(rotate=90),
                #                    iaa.Affine(rotate=270),
                #                    iaa.Affine(rotate=180),
                #                    iaa.Affine(rotate=180),
                #                    iaa.Affine(rotate=270)])
                #     ])
                seq_det = seq.to_deterministic()  # 保持坐标和图像同步改变， 而不是随机
                # 读取图片
                try:
                    img = Image.open(os.path.join(IMG_DIR, name[:-4] + img_type))
                except:
                    img = Image.open(os.path.join(IMG_DIR, name[:-4] + '.JPG'))

                # JPG 不支持 alpha 透明度， 有可能报 RGBA 错误， 将图片丢弃透明度转成 RGB
                img = img.convert('RGB')
                # sp = img.size
                img = np.asarray(img)
                # bndbox 坐标增强
                for i in range(len(bndbox)):
                    bbs = ia.BoundingBoxesOnImage([
                        ia.BoundingBox(x1=bndbox[i][0], y1=bndbox[i][1], x2=bndbox[i][2], y2=bndbox[i][3]),
                    ], shape=img.shape)

                    bbs_aug = seq_det.augment_bounding_boxes([bbs])[0]
                    boxes_img_aug_list.append(bbs_aug)

                    # new_bndbox_list:[[x1,y1,x2,y2],...[],[]]
                    n_x1 = int(max(1, min(img.shape[1], bbs_aug.bounding_boxes[0].x1)))
                    n_y1 = int(max(1, min(img.shape[0], bbs_aug.bounding_boxes[0].y1)))
                    n_x2 = int(max(1, min(img.shape[1], bbs_aug.bounding_boxes[0].x2)))
                    n_y2 = int(max(1, min(img.shape[0], bbs_aug.bounding_boxes[0].y2)))
                    if n_x1 == 1 and n_x1 == n_x2:
                        n_x2 += 1
                    if n_y1 == 1 and n_y2 == n_y1:
                        n_y2 += 1
                    if n_x1 >= n_x2 or n_y1 >= n_y2:
                        print('error', name)
                    new_bndbox_list.append([n_x1, n_y1, n_x2, n_y2])

                    # 存储变化后的图片
                    image_aug = seq_det.augment_images([img])[0]
                    # 新文件名
                    new_name = name[:-4] + '-' + str(epoch)
                    path = os.path.join(AUG_IMG_DIR, new_name + img_type)

                    image_auged = bbs.draw_on_image(image_aug, thickness=0)
                    Image.fromarray(image_auged).save(path)

                # 存储变化后的 XML
                change_xml_list_annotation(XML_DIR, name[:-4], new_bndbox_list, AUG_XML_DIR, new_name)
                new_bndbox_list = []


if __name__ == "__main__":

    # 随机种子
    ia.seed(1)
    img_type = '.jpg'
    # img_type = '.png'

    # 原数据路径
    IMG_DIR = "boatDetail/images/"
    XML_DIR = "boatDetail/xml/"

    # 存储增强后的影像文件夹路径
    AUG_IMG_DIR = "boatDetail/new_img/"
    if not os.path.exists(AUG_IMG_DIR):
        os.mkdir(AUG_IMG_DIR)

    # 存储增强后的 XML 文件夹路径
    AUG_XML_DIR = "boatDetail/new_xml/"
    if not os.path.exists(AUG_XML_DIR):
        os.mkdir(AUG_XML_DIR)

    # 数据增强 n 倍
    simple_example(3, IMG_DIR, XML_DIR, AUG_IMG_DIR, AUG_XML_DIR)
```

### yolov5 训练并标注

对于这个数据集， 100 张足够训练出一个精度较高的模型了， 因为我们的目的是用他来标注， 并不是最终的模型。 所以这里你可以选择一个精度很高的 5 模型， 比如 v5l（我们这步要确保精度要足够高， 数据少没必要用小模型）

接着将所有收集的图片， 使用半标注程序， 进行预测， 得到所有的输出 xml 文件


参考 [yolov5-label-xml](https://github.com/1079863482/yolov5-label-xml)

### yolov8 训练


参考 https://blog.csdn.net/qq_39056987/article/details/129616026


### 部署

```bash
pyenv local 3.9.16
python -m venv venv
source venv/bin/activate
```

安装 deepsparse

```bash
pip install deepsparse[server,yolo,onnxruntime,yolov8]
```

运行 server

```bash
deepsparse.server \                                   
    --task yolov8 \
    --model_path /home/evilbeast/Data/ai/chopsticks/runs/detect/train/weights/best.onnx
```

docs 

http://127.0.0.1:5543/docs


### 测试

```python
import cv2
import json
import requests

# list of images for inference (local files on client side)
file_name = 'test.png'
save_file_name = 'test_2.png'
path = [file_name]
files = [('request', open(img, 'rb')) for img in path]

# send request over HTTP to /predict/from_files endpoint
url = 'http://0.0.0.0:5543/predict/from_files'
resp = requests.post(url=url, files=files)

# response is returned in JSON
annotations = json.loads(resp.text)  # dictionary of annotation results
boxes = annotations["boxes"]
boxes = boxes[0]

im = cv2.imread(file_name)
for box in boxes:
    center_x = int((box[0] + box[2]) / 2)
    center_y = int((box[1] + box[3]) / 2)
    radius = int(abs(box[0] - box[2]) / 5)

    cv2.circle(im,
               (center_x, center_y),
               radius=radius,
               color=(0, 0, 255),
               thickness=-1,  # -1 表示圆点内部是实心的
               lineType=cv2.LINE_AA)

cv2.imwrite(save_file_name, im)
```

# cat_face_detection
Based on hand-labeled data set and object detection module of tensorflow, a new network is trained to detect cat faces and eyes. The model is trained half-way from
the ssd_mobilenet_coco_v1 detection model. <br>
The ultimate goal of this project is to develop a small plug-in unit in Wechat. So the users could upload photos in Wechat and then download their beautified cats'
pictures after the automatic processing. <br>
The further goal regarding research could be trying to figure out the characteristic points in cats' face detection.

## Environment
Ubuntu 16.04 LTS <br>
Python 2.7.12 <br>
tensorflow 1.6.0 <br>
Protobuf 3.5.1 <br>
pandas 0.21.0 <br>
matplotlib 2.1.0 <br>
Cython 0.28.2 <br>
pillow 3.1.2 <br>
lxml 3.5.0 <br>
jupyter 1.0.0 <br>

CUDA 9.0.176 <br>
CuDNN 7.0 <br>
GPU related：Quadro M4000 * 2, NVIDIA-SMI 390.25, Driver 390.25

## Setup
1. Install the Object Detection API of Tensorflow. See [Installation](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/installation.md).  
2. Download data of cat faces. The best option is [Dogs vs. Cats](https://www.kaggle.com/c/dogs-vs-cats), which contains different pictures in different background and light conditions. The variance of dataset could help a lot in training.
3. Labels cat faces and eyes with LabelImg on the dataset that download in 2. Make our own dataset in .xml files. Use [Raccoon](https://towardsdatascience.com/how-to-train-your-own-object-detector-with-tensorflows-object-detector-api-bec72ecfe1d9) for reference.  
4. Using the self-developed script to tranform .xml files into .csv files and finally into tfRecord. Now we are ready for Tensorflow training.
5. Configure our own training pipeline, including label_map.pbtxt and ssd_mobilenet_v1_cat.config. Use official guide [Configuring the Object Detection Training Pipeline](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/configuring_jobs.md) for reference.  
6. Start the training process by calling self-developed script train_cat.sh. Monitor the process with tensorboard. Record checkpoints.
7. Export a trained model for inferences from saved checkpoints.See [Exporting a trained model for inference](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/exporting_models.md).
8. It's test time! Write the resulting pictures into the 'result' folder.
```
# You can complete all the procedures above by executing the following command
sudo ./run.sh
# To view the training and testing result on tensorboard, please run:
tensorboard.sh
# Now you can view them on https://localhost:6066
```


## Oberservations
The following picture shows the training record in tensorboard.
![Loss](https://github.com/Orienfish/cat_face_detection/blob/master/pic/losses.png).
The test accuracy on tensorboard is shown below. <br>
The precision for cat_face is 100% and for cat_eyes is 93.96%. Total precision is 96.98%. <br>
![Accuracy](https://github.com/Orienfish/cat_face_detection/blob/master/pic/precision.png)
One of the test results is shown below. <br>
![img](https://github.com/Orienfish/cat_face_detection/blob/master/results/cat.0.jpg)

## File structure
```
├── README.md                // Help
├── run.sh     				 // Transfrom .xml to tfRecord. Train, evaluate, export and test. Should be called in current path.
├── xml_to_csv.py            // The python code called by generate_tfrecord.sh.
├── generate_tfrecord.py     // The python code called by generate_tfrecord.sh.
├── tensorboard.sh           // Start tensorboard. Can be called anywhere.
├── my_object_detection.py   // The python code for testing trained models.Could be called anywhere
├── annotations              // The hand labeled training and testing dataset. Only 100 in all.
│   ├── train                // 80 .xml files for training.
│   └── test                 // 20 .xml files for testing.
├── data                     // The .csv files and tfRecord generated from annotations.
│   ├── train_labels.csv     // The .csv files for training dataset.
│   ├── test_labels.csv      // The .csv files for testing dataset.
│   ├── train.record         // The tfRecord for training dataset.
│   └── teset.record         // The tfRecord for testing dataset.
├── images                   // 200 images from the original Dogs v.s. Cats dataset.
├── results                  // The resulting 100 pictures in testing.
└── training                 // Training related files.
    ├── label_map.pbtxt                // Label map with 2 labels.
    ├── ssd_mobilnet_v1_cat.config     // Training pipeline configuration.
    ├── ssd_mobilenet_v1_coco_2017_11_17.tar.gz  // The based model.
    ├── ssd_mobilenet_v1_coco_2017_11_17         // The untar file of .tar.gz. Contain the based model.
    ├── trainlog             // Checkpoint record and training tensorboard event record, TRAIN_DIR.
    ├── evallog              // Testing tensorboard event record, TEST_DIR
    └── exported_model       // Exported model from trainlog/ckpt, EXPORT_DIR
```

## VERSION RECORD
v1 @2018.6.5 <br>
Used 80 images for training and 20 images for testing.
Unable to run eval.py for unknown reasons. That's why the training/evallong directory is empty =_=.
Trained for 30k times, costing 7h21min. <br>

v2 @2018.6.10 <br>
Fix the evaluate bug. Due to the limit of GPU space, it's unable to run train_cat.sh and eval_cat.sh at the same time.
Thus run eval_cat.sh for once after training. Record the testing precision.

v3 @2018.6.13 <br>
Combine generate_tfrecord.sh, train_cat.sh, eval_cat.sh and testing in to run.sh.
There's no need to copy file to tensorflow directories now. Everything could run in the current path.

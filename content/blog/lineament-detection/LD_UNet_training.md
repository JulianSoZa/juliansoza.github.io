---
title: Training a Convolutional Neural Network (CNN) with a Fault Mapping Dataset for Linear Structure Detection
date: 2025-10-27
draft: false
math: true
authors:
  - name: JulianSoZa
    link: https://github.com/JulianSoZa
    image: https://github.com/JulianSoZa.png
  - name: alejaboterob
    link: https://github.com/alejaboterob
    image: https://github.com/alejaboterob.png
  - name: nicoguaro
    link: https://github.com/nicoguaro
    image: https://github.com/nicoguaro.png
category: Lineament Detection
tags:
  - Geophysics
  - U-Net
  - CNNs
  - Datasets
  - Training
excludeSearch: false
---
This tutorial ([`training_FaultMapping.ipynb`](https://github.com/AppliedMechanics-EAFIT/lineament_detection/blob/21c7d680d6941e705a7dc16cd813e911b3303490/src/examples/training_FaultMapping.ipynb)) reproduces a practical example of using CNNs for image-to-image transformation, specifically for linear structure detection, based on [[1](#ref1)], [[2](#ref2)], and [[3](#ref3)]. The model is trained using a fault mapping dataset [[1](#ref1)] and implements the U-Net architecture.

<!--more-->
## 1. Data

The fault mapping dataset used in this example is that published in the article ***"Automatic Fault Mapping in Remote Optical Images and Topographic Data with Deep Learning"*** by L. Mattéo et al. [[1](#ref1)], where they adapt a U-Net Convolutional Neural Network to automate fracture and fault mapping in optical images and topographic data. The dataset contains the images, topographic data, and ground truth labels.

### 1.1. Dataset folders
First, we load the data, which must be organized into separate folders for images and ground truth, as well as train, validation, and test splits.

```python
xfolder = data_dir + '/training/fault_mapping/images/'
yfolder = data_dir + '/training/fault_mapping/ground_truth/'
```

### 1.2. Image preprocessing

We define the transformations to apply to the images and ground truth labels to standardize the CNN training process.

```python
def image_transformation(noise=True):
    image_transform = transforms.Compose([
        transforms.Resize((256, 256)),
        transforms.ToTensor(),
        GaussianNoise(mean=0.0, sigma=10.0/255.0) if noise else transforms.Lambda(lambda x: x),
        transforms.Normalize(mean=[0.5], std=[0.25])
    ])
    return image_transform

def label_transformation(noise=True):
    label_transform = transforms.Compose([
        transforms.Resize((256, 256)),
        transforms.ToTensor(),
        GaussianNoise(mean=0.0, sigma=10.0/255.0) if noise else transforms.Lambda(lambda x: x),
    ])
    return label_transform

image_transform = faultMapping.image_transformation
label_transform = faultMapping.label_transformation
```

### 1.3. Datasets

We define the dataset class, which specifies how the data is loaded and handles data augmentation. We apply rotations at specific angles to artificially increase the training dataset size and improve the model's rotational invariance.

```python
class faultMappingDataset(Dataset):
    def __init__(self, image_dir, label_dir, image_transform=None, label_transform=None, rotations=None, every_n=1, input_channels=3):
        self.image_dir = image_dir
        self.label_dir = label_dir
        self.image_transform = image_transform
        self.label_transform = label_transform
        self.input_channels = input_channels

        self.image_files = sorted([f for f in os.listdir(image_dir) if f.endswith('.tif')])
        self.label_files = sorted([f for f in os.listdir(label_dir) if f.endswith('.tif')])

        self.rotations = rotations if rotations is not None else [0]
        
        base_indices = list(range(0, len(self.image_files), every_n))
        self.index = [(i, angle) for i in base_indices for angle in self.rotations]

    def __len__(self):
        return len(self.index)

    def __getitem__(self, idx):
        img_idx, angle = self.index[idx]

        # Load image
        img_path = os.path.join(self.image_dir, self.image_files[img_idx])
        image = imread(img_path)
        if self.input_channels == 1:
            image = Image.fromarray(image[..., :3].astype(np.uint8)).convert("L")
        else:
            image = Image.fromarray(image[..., :3].astype(np.uint8)).convert("RGB")

        # Load label
        label_path = os.path.join(self.label_dir, self.label_files[img_idx])
        label = imread(label_path)
        label = Image.fromarray((label * 255).astype(np.uint8))

        # Rotation (for data augmentation)
        if angle != 0:
            if self.input_channels == 1:
                image = TF.rotate(image, angle=angle, interpolation=InterpolationMode.BILINEAR, expand=False, fill=0)
            else:
                image = TF.rotate(image, angle=angle, interpolation=InterpolationMode.BILINEAR, expand=False, fill=(0, 0, 0))
            label = TF.rotate(label, angle=angle, interpolation=InterpolationMode.NEAREST, expand=False, fill=0)

        # Transforms
        if self.image_transform:
            image = self.image_transform(image)
        if self.label_transform:
            label = self.label_transform(label)

        return [image, label]

rotations = [0, 45, 90, 315]

in_ch = 1  # Set to 1 for grayscale, 3 for RGB

train_set = faultMapping.faultMappingDataset(
    xfolder + 'train/', 
    yfolder + 'train/', 
    image_transform(), 
    label_transform(), 
    rotations=rotations, 
    every_n=2, 
    input_channels=in_ch
    )
val_set = faultMapping.faultMappingDataset(
    xfolder + 'val/', 
    yfolder + 'val/', 
    image_transform(), 
    label_transform(), 
    rotations=rotations, 
    every_n=2, 
    input_channels=in_ch
)
test_set = faultMapping.faultMappingDataset(
    xfolder + 'test/', 
    yfolder + 'test/', 
    image_transform(noise=False), 
    label_transform(noise=False), 
    rotations=[0], 
    every_n=1, 
    input_channels=in_ch
)
```

### 1.4. Dataloaders

Finally, we define the dataloaders, which efficiently batch the data and feed it to the model during training. 

```python
batch_size = 4
dataloaders = {
    'train': DataLoader(train_set, batch_size=batch_size, shuffle=True, num_workers=0),
    'val': DataLoader(val_set, batch_size=batch_size, shuffle=True, num_workers=0)
}
```


```python
image_datasets = {'train': train_set, 'val': val_set}
dataset_sizes = {x: len(image_datasets[x]) for x in image_datasets.keys()}
dataset_sizes
```


    {'train': 4348, 'val': 648}



### 1.5. Plot the data


```python
inputs, labels = next(iter(dataloaders['train']))
print(inputs.shape, labels.shape)
```

    torch.Size([4, 1, 256, 256]) torch.Size([4, 1, 256, 256])
    


```python
this_image = 0
plot_images.plot_input_gt(inputs[this_image], labels[this_image])
```


    
<figure>
  <img src="/images/LD/training_FaultMapping_Input_GT.png" alt="Dataset" width="600">
  <figcaption>Example from the fault mapping dataset [<a href="#ref1">1</a>]</figcaption>
</figure>
    


## 2. Model

Now, we create our neural network, which is a CNN with U-Net architecture. 

### 2.1. Define the model


```python
num_class = 1
in_ch = 1
model = Org_UNet.UNet(n_classes=num_class, input_channels=in_ch).to(device)
model.apply(Org_UNet.initialize_weights);
```

### 2.2. Train the model

Then, we train our model with the dataset. We have two phases, each epoch has a training and validation phase.

```python
def train_model(model, optimizer, scheduler, dataloaders, num_epochs=25, device='cpu'):
    best_model_wts = copy.deepcopy(model.state_dict())
    best_loss = 1e10

    train_losses = []
    val_losses = []

    for epoch in range(num_epochs):
        print('\nEpoch {}/{}'.format(epoch, num_epochs - 1))
        print('-' * 10)

        since = time.time()

        # Each epoch has a training and validation phase
        for phase in ['train', 'val']:
            if phase == 'train':
                for param_group in optimizer.param_groups:
                    print("LR", param_group['lr'])

                model.train()  # Set model to training mode
            else:
                model.eval()   # Set model to evaluate mode

            metrics = defaultdict(float)
            epoch_samples = 0

            for inputs, labels in dataloaders[phase]:
                inputs = inputs.to(device)
                labels = labels.to(device)

                optimizer.zero_grad()

                with torch.set_grad_enabled(phase == 'train'):
                    outputs = model(inputs)
                    loss = calc_loss(outputs, labels, metrics)

                    # backward + optimize only if in training phase
                    if phase == 'train':
                        loss.backward()
                        optimizer.step()
                        
                epoch_samples += inputs.size(0)

            print_metrics(metrics, epoch_samples, phase)
            epoch_loss = metrics['loss'] / epoch_samples

            if phase == 'val':
                val_losses.append(epoch_loss)
            else:
                train_losses.append(epoch_loss)
                scheduler.step()

            # deep copy the model
            if phase == 'val' and epoch_loss < best_loss:
                print("saving best model")
                best_loss = epoch_loss
                best_model_wts = copy.deepcopy(model.state_dict())

        time_elapsed = time.time() - since
        print('{:.0f}m {:.0f}s'.format(time_elapsed // 60, time_elapsed % 60))

    print('Best val loss: {:4f}'.format(best_loss))

    # load best model weights
    model.load_state_dict(best_model_wts)
    return model, train_losses, val_losses, best_loss

optimizer_ft = optim.Adam(model.parameters(), lr=1e-4, weight_decay=1e-5)
exp_lr_scheduler = lr_scheduler.StepLR(optimizer_ft, step_size=10, gamma=0.1)
model, train_losses, val_losses, best_loss = train.train_model(model, optimizer_ft, exp_lr_scheduler, dataloaders, num_epochs=1, device=device)
```


```python
plot_metrics.plot_losses(train_losses, val_losses)
```

<figure>
  <img src="/images/LD/training_FaultMapping_ValTrainLoss.png" alt="Dataset" width="600">
  <figcaption>Training and validation Loss</figcaption>
</figure>

## 3. Prediction

Now, we assess the performance of the network. First, we use the test dataset, which was unseen by the CNN during training.

```python
model.eval();
```

### 3.1. Labeled datasets


```python
test_loader = DataLoader(test_set, batch_size=1, shuffle=True, num_workers=0)
```


```python
inputs, labels = next(iter(test_loader))
pred = model(inputs.to(device))
display(pred.shape)

pred_c = torch.sigmoid(pred).data.cpu().numpy().squeeze(1)
pred_c = np.where(pred_c>=0.5, 1, 0)
display(pred_c.shape);

plot_images.plot_input_gt_pred(inputs[0], labels[0], pred_c[0])
```

<figure>
  <img src="/images/LD/training_FaultMapping_Input_GT_Pred.png" alt="Dataset" width="650">
  <figcaption>Example from the fault mapping dataset [<a href="#ref1">1</a>]</figcaption>
</figure>
    


### 3.2. Unlabeled datasets

Finally, we test the network on unlabeled datasets to demonstrate its capability to detect linear structures in new, real-world images without ground truth labels.

```python
image = Image.open(output_dir + '/test' + '/img_4.jpg')
image = image.convert("L")

image_transformed = image_transform(noise=False)(image).unsqueeze(0)
pred = model(image_transformed.to(device))
pred_c = torch.sigmoid(pred).data.cpu().numpy()
pred_c = np.where(pred_c>=0.5, 1, 0)

fig, (ax0, ax1) = plt.subplots(1, 2, figsize=(15, 10))
ax0.imshow(image_transformed[0, 0], cmap='gray')
ax0.set_title('Original Image')
ax1.imshow(pred_c[0, 0], cmap='gray')
ax1.set_title('Predicted Edges')
plt.show()
```


    
<figure>
  <img src="/images/LD/training_FaultMapping_Input_Pred.png" alt="Dataset" width="600">
  <figcaption>Example of an unlabeled image</figcaption>
</figure>
    

## References

[<a name="ref1">1</a>]. L. Mattéo et al., “Automatic Fault Mapping in Remote Optical Images and Topographic Data with Deep Learning,” Journal of Geophysical Research: Solid Earth, vol. 126, no. 4, p. e2020JB021269, 2021, doi: 10.1029/2020JB021269.

[<a name="ref2">2</a>]. N. Usuyama and K. Chahal, "UNet/FCN PyTorch," GitHub Repository, 2018. Available online: [https://github.com/usuyama/pytorch-unet](https://github.com/usuyama/pytorch-unet).

[<a name="ref3">3</a>]. I. Ocak and O. Tepencelik, "Edge Detection Using U-Net Architecture," GitHub Repository, 2020. Available online: [https://github.com/iocak28/UNet_edge_detection](https://github.com/iocak28/UNet_edge_detection).
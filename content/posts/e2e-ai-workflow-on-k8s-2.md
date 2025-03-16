---
title: "From Training to Serving: AI with PyTorch & Kubeflow on K8s (II)"
subtitle: "Train and serve a PyTorch model with fastapi locally"
date: 2025-03-06T20:23:12-08:00
draft: false 
---

In this guide, we’ll walk through the complete workflow of training a deep 
learning model using PyTorch and serving it locally with FastAPI. The examples 
presented here can be found in the following repository:  
📌 **[GitHub Repository](https://github.com/charleszheng44/pytorch-kubeflow-example/tree/main)**  

---

## Setting Up the Environment

Start by cloning the repository and installing the required dependencies:  

```bash
python -m venv .venv 
source .venv/bin/activate 
pip install -r requirements.txt
```

---

## Training the Model

The provided training script trains a basic Convolutional Neural Network (CNN) on the **MNIST dataset**, designed for handwritten digit recognition. The trained model will be saved in the `MODEL_DIR` directory.  

To begin training, run:  

```bash
MODEL_DIR=/tmp/models python train.py
```

### Enhancing Training to Simulate Real-World Workloads

If you have access to more computing resources and want to mimic real-world training workflows, consider modifying the **hyperparameters** in `train.py` to increase training time and computational demands.  

#### Increase the Number of Epochs:
```python
EPOCHS = 2  
# Change to  
EPOCHS = 10
```

#### Use Adam Optimizer for Faster Convergence:
```python
optimizer = optim.SGD(model.parameters(), lr=LR, momentum=0.9)  
# Change to  
optimizer = torch.optim.Adam(model.parameters(), lr=0.001)
```

#### Expand the Network with More Layers and Channels:
```python
self.conv1 = nn.Conv2d(1, 10, kernel_size=5)  
self.conv2 = nn.Conv2d(10, 20, kernel_size=5)  
self.fc1 = nn.Linear(320, 50)  
self.fc2 = nn.Linear(50, 10)  

# Change to  
self.conv1 = nn.Conv2d(1, 32, kernel_size=3)  
self.conv2 = nn.Conv2d(32, 64, kernel_size=3)  
self.conv3 = nn.Conv2d(64, 128, kernel_size=3)  
self.fc1 = nn.Linear(64*4*4, 256)  
self.fc2 = nn.Linear(256, 10)
```

These modifications will lead to a more complex and computationally intensive model, better simulating training scenarios in production environments.

## Running the Inference Service  

Once the model is trained, you can serve it using **FastAPI** by running the following command:  

```bash
MODEL_PATH=/tmp/models/mnist_cnn.pt uvicorn inference:app --host 0.0.0.0 --port 8080 --reload
```

## Testing the Model

To send an image for prediction, use the following `curl` command:  

```bash
curl -X POST "http://localhost:8080/predict" -F "file=@/Users/charlesz/Works/pytorch-kubeflow-example/8.png"
```  

If the model is working correctly, you should receive an output similar to:  

```json
{"prediction": 8}
```

---
title: Training Loop with SGD
tags:
  - ml
  - coding
  - training
---

## Goal

Implement a simple training loop with SGD in PyTorch.

```python
import torch
import torch.nn as nn
import torch.optim as optim

model = nn.Linear(10, 2)
criterion = nn.MSELoss()
optimizer = optim.SGD(model.parameters(), lr=0.01)

inputs = torch.randn(100, 10)
targets = torch.randn(100, 2)

epochs = 5
batch_size = 20

for epoch in range(epochs):    
    for i in range(0, inputs.size()[0], batch_size):
        optimizer.zero_grad()  # clear gradients from the previous step
        
        # mini-batch
        batch_x, batch_y = inputs[i:i+batch_size], targets[i:i+batch_size]

        
        # Forward pass
        outputs = model(batch_x)
        loss = criterion(outputs, batch_y)
        
        # Backward pass
        loss.backward()  # calculate gradients
        
        optimizer.step()  # update weights and biases with gradients
        
    print(f"Epoch {epoch+1}/{epochs}, Loss: {loss.item():.4f}")
```

1. Prepare data, often in shape (samples, ndim)
2. At every epoch, loop through every mini batch
3. Run forward propagation to get model predictions
4. Compute loss against ground truth for each mini batch
5. Compute gradients through backpropagation
6. Update W&B using gradients

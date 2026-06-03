---
title: Adversarial ML
tags:
  - ml
  - adversarial attacks
  - robustness
---

## What are adversarial perturbations in machine learning
1. Adversarial perturbations are techniques used to exploit vulnerabilities in models by intentionally manipulating input data. The goal of an adversarial perturbations is to deceive the model into making incorrect predictions/outputs.
2. For most use cases, they are used to evaluate the robustness of machine learning models. By introducing subtle changes to input data and observing how the model responds, we can assess the robustness of the model to adversarial attacks.
3. So the goal is maximizing the error rates, by introducing the smallest pertubations/changes


## Considerations for tabular and regression settings
  
Unlike images, tabular features are not interchangeable like image pixel values, and secondly, while most people can usually tell the correct class of an image and whether it appears altered, it is much complex for tabular data, as tabular data is less interpretable and extensive expert knowledge is required.

### Perceptibility and feature constraints
The most common measure of perceptibility of perturbations is L𝑝 norm. For example, L2 norm measures the Euclidean distance between the original and perturbed inputs. If the L2 norm is small, the perturbation is considered imperceptible.

![[problem-formulation.png]]
### L0
An L0 norm bounded attack typically involves modifying a certain number of features of an input signal to a model

### L1
An L1 norm bounded attack involves upper bounding the sum of the total perturbation values.

### L2
An L2 norm bounded attack involves upper bounding the Euclidean distance of the perturbation d

### L∞
An L∞ norm bounded attack involves upper bounding the maximum value of the perturbation d.

## Attack types

### White-box attacks
In a white-box attack, the attacker has complete knowledge of the target model, including its architecture, parameters, etc.

### Black-box attacks
In a black-box attack, we can only see the output of the model in response to that attack

## Evaluation metrics

### Fooling error
Fooling error represents the averaging of the absolute differences between the predicted output of the model with the perturbed input and the predicted output without perturbations
![[metric-fooling-error.png]]
### SMAPE
Directly measure the percentage of error caused by perturbation (higher the better).
![[metric-smape.png]]
## Attack methods

### Fast Gradient Sign Method (FGSM)
It leverages the gradient information of the loss with respect to the input data to create a perturbation that is added to the original data, generating an adversarial example.

The fast gradient sign method works as following:

1. Calculate the loss after forward propagation (of original input data)

2. Compute the gradient of the loss with respect to the input data

3. Nudge/change the data values very slightly in the direction of the calculated gradients that maximize the loss calculated above.

4. The nudged data values then would be our new adversarial example.
![[fgsm-equation.png]]
### Fast Gradient Method (FGM)
A more general method is the fast gradient method, which is a generalization to the fast gradient signed method, to meet the L2 norm bound ||x^∗-x| |≤ E

### Projected Gradient Descent (PGD)
It is an iterative optimization-based method.

The key characteristic of PGD is its ability to search for adversarial examples within a specified norm ball, providing a stronger and more robust attack compared to one-shot methods like FGSM.

Similar to Fast Gradient Method, PGD seeks to find the perturbation that maximizes a model's loss on a particular input over a specified number of iterations while keeping the perturbation's size below a specified value called epsilon. This constraint is usually referred to as L2 norm bound: ||x^∗-x| | ≤ E

How it works:

1. Start from a random perturbation in the L2 ball around a sample

2. Take a gradient step in the direction of greatest loss

3. Project perturbation back into L2 ball if necessary

4. Repeat steps 2 & 3
![[pgd-l2-ball.png]]

### Momentum Iterative Method (MIM)
Momentum Iterative Method attack integrates momentum into projected gradient descent.

By integrating the momentum into the iterative process for attacks, MIM methods can stabilize update directions and escape from poor local maxima during the iterations, resulting in more transferable adversarial examples

## References

**Papers**
- [Towards Deep Learning Models Resistant to Adversarial Attacks](https://arxiv.org/pdf/1710.06081.pdf)
- [LowProFool: an Adversarial Attack Against Tabular Data](https://arxiv.org/pdf/1904.13000.pdf)
- [arXiv:1911.03274](https://arxiv.org/pdf/1911.03274.pdf)
- [arXiv:1902.10755](https://arxiv.org/abs/1902.10755)
- [CEUR-WS Vol-2916, Paper 17](https://ceur-ws.org/Vol-2916/paper_17.pdf)

**Blog posts**
- [A Practical Guide to Adversarial Robustness](https://towardsdatascience.com/a-practical-guide-to-adversarial-robustness-ef2087062bec)
- [Adversarial Attacks on Neural Networks](https://neptune.ai/blog/adversarial-attacks-on-neural-networks-exploring-the-fast-gradient-sign-method)

**Code**
- [LowProFool](https://github.com/axa-rev-research/LowProFool/tree/master)
- [CleverHans: Jacobian-based Saliency Map Attack](https://github.com/cleverhans-lab/cleverhans/blob/master/cleverhans_v3.1.0/cleverhans/attacks/saliency_map_method.py)
- [CleverHans: Sparse L1 Descent](https://github.com/cleverhans-lab/cleverhans/blob/master/cleverhans/torch/attacks/sparse_l1_descent.py)
- [IBM ART: LowProFool](https://github.com/Trusted-AI/adversarial-robustness-toolbox/blob/main/art/attacks/evasion/lowprofool.py)

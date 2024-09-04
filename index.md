---
layout: project_page
permalink: /

title:  "Booster: Tackling Harmful Fine-tuning for Large Language Models via Attenuating Harmful Perturbation"
authors:
  - "Tiansheng Huang, "
  - "Sihao Hu, "
  - "Fatih Ilhan, "
  - "Selim Furkan Tekin, " 
  - "Ling Liu"

affiliations:
  - "Georgia Institute of Technology"

paper: https://arxiv.org/pdf/2409.01586
code: https://github.com/git-disl/Booster
---

<div class="columns is-centered has-text-centered">
    <div class="column is-four-fifths">
        <h2>Abstract</h2>
        <div class="content has-text-justified">
Harmful fine-tuning issue \citep{qi2023fine} poses serious safety concerns for Large language models' fine-tuning-as-a-service. While existing defenses \citep{huang2024vaccine, rosati2024representation} have been proposed to mitigate the issue, their performances are still far away from satisfactory, and the root cause of the problem has not been fully recovered. For the first time in the literature, we in this paper show that harmful perturbation over the model weights should be the root cause of alignment-broken of harmful fine-tuning. In order to attenuate the negative impact of harmful perturbation, we propose an alignment-stage solution, dubbed Booster. Technically, along with the original alignment loss,  we append a loss regularizer in the alignment stage's optimization. The regularizer ensures that the model's harmful loss reduction before/after simulated harmful perturbation is attenuated, thereby mitigating the subsequent fine-tuning risk. Empirical results show that Booster can effectively reduce the harmful score of the fine-tuned models while maintaining the performance of downstream tasks. Our code is available at \url{https://github.com/git-disl/Booster}.
        </div>
    </div>
</div>

---

## Harmful Fine-tuning Issue

<p align="middle">
  <img src="static/image/booster.png" width="700" />
</p>

The figure demonstrates the risk for fine-tuning-as-a-service business model. 
* At the first stage of the service pipeline, the model is safety aligned with safety alignment data/**simulated harmful data**. 

* At the second stage (*where the attack surface is*), users upload data for service provider to finetune, and the service provider finetune model on user data to deliver custoomized service. However, the user data may contain harmful demonstration data that may subverts the previous enforced alignment. Finetuning on this partially harmful data and deploy the alignment-broken fine-tuned model may cause serious ethical and governance concern.    


## Harmful Perturbation
We in the following show that the **harmful perturbation** should be the root cause of harmful fine-tuning attack. 

**Definition of harmful perturbation.** Harmful perturbation is defined as *taking one step on the model towards the gradient direction over the harmful data*.

**Impact of harmful perturbation.** 
* The left of the below figure shows that the model's harmful score will substantially increase along with optimization steps invested in fine-tuning on a pure harmful dataset. On the contrary, the harmful score will not be affected much via fine-tuning on a pure SST2 dataset.  This indicates that taking a step with the gradient of the harmful data (i.e.,harmful perturbation) is indeed the reason for the alignment broken.

* The middle/right of the below figure how taking harmful perturbation and SST2 perturbation in each epoch will affect the harmful training/testing loss. As shown, harmful perturbation significantly reduces the harmful training/testing loss, which means the model starts to fit the harmful data, and therefore explaining the rise of harmful score when taking harmful perturbation.

<p align="middle">
  <img src="static/image/statistics.png" width="900" />
</p>

## Booster: Alignment-stage Defense via Attenuating Harmful Perturbation

In order to attenuate the negative impact of harmful perturbation, Booster aims to solve the following optimization problem:

<p align="middle">
  <img src="static/image/booster problem.png" width="700" />
</p>
where f(w) is the empirical loss over the alignment dataset and h(w) is the empirical loss over the
harmful dataset, λ is the regularizer’s intensity, and α is the step size. 

Our contribution lies in the second term, which measures the gap between the original harmful loss and the harmful loss after taking a normalized step with the harmful gradient. The idea is to minimize the impact of potential harmful perturbation towards the alignment model  while simultaneously minimizing its alignment loss. Specifically, the second term simulates the decrease of harmful loss after one step of fine-tuning on harmful samples. By minimizing this gap, the decrease of harmful loss after taking optimization on the real harmful samples in the fine-tuning stage will be minimized (i.e., impact of harmful perturbation will be attenuated).   


The above optimization can be solved by an iterative gradient method, with update rule as follows:

<p align="middle">
  <img src="static/image/updaterule.png" width="900" />
</p>
where $\eta$ is the learning rate. Here we straight-through the second order information following the routine from  
MAML. 

We present the detailed algorithm in Algorithm 1. The overall procedure requires three forward/backward passes
for each optimization step.

<p align="middle">
  <img src="static/image/algorithm.png" width="700" />
</p>


## Quantitive results

**Robustness to harmful ratio.** As shown in the following table, compared to SFT,  Booster in average achieves 22.64\% of lower harmful score, and 2.64\% higher finetune accuracy on the downstream. 
<p align="middle">
  <img src="static/image/quantitive1.png" width="900" />
</p>

**Robustness to model architectures.** In the next table, we show that the proposed method can be extended to two latest SOTA model architectures, i.e., Gemma2-9B, and Qwen2-7B. In average, Booster achieves 34.14\% reduction of harmful score and 0.71\% improvement of finetune accuracy.
<p align="middle">
  <img src="static/image/quantitive2.png" width="900" />
</p>

## Qualitative results

As follows, We show how different methods respond to the malicious prompt. We used the finetuned
model over default attack setting for evaluation. As shown, Booster is able to give refusal answer to
the sensitive question while other methods cannot.
<p align="middle">
  <img src="static/image/qualitative.png" width="700" />
</p>


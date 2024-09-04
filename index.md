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
  - "Georgia Institute of Technology, "

paper: https://arxiv.org/pdf/2409.01586
code: https://github.com/git-disl/Booster
---

<div class="columns is-centered has-text-centered">
    <div class="column is-four-fifths">
        <h2>Abstract</h2>
        <div class="content has-text-justified">
Harmful fine-tuning issue \citep{qi2023fine} poses serious safety concerns for Large language models' fine-tuning-as-a-service. While existing defenses \citep{huang2024vaccine,rosati2024representation} have been proposed to mitigate the issue, their performances are still far away from satisfactory, and the root cause of the problem has not been fully recovered. For the first time in the literature, we in this paper show that harmful perturbation over the model weights should be the root cause of alignment-broken of harmful fine-tuning. In order to attenuate the negative impact of harmful perturbation, we propose an alignment-stage solution, dubbed Booster. Technically, along with the original alignment loss,  we append a loss regularizer in the alignment stage's optimization. The regularizer ensures that the model's harmful loss reduction before/after simulated harmful perturbation is attenuated, thereby mitigating the subsequent fine-tuning risk. Empirical results show that Booster can effectively reduce the harmful score of the fine-tuned models while maintaining the performance of downstream tasks. Our code is available at \url{https://github.com/git-disl/Booster}.
        </div>
    </div>
</div>

---

## Harmful Fine-tuning Issue

<p align="middle">
  <img src="static/image/booster.png" width="700" />
</p>

The figure demonstrates the risk for fine-tuning-as-a-service business model. At the first stage of the service pipeline, the model is safety aligned with safety alignment data. At the second stage, users upload data for service provider to finetune, and the service provider finetune model on user data to deliver custoomized service. However, the user data may contain harmful demonstration data that may subverts the previous enforced alignment. Finetuning on this partially harmful data and deploy the alignment-broken fine-tuned model may cause serious ethical and governance concern.    


## Harmful Perturbation
We in the following show that the **harmful perturbation** should be the root cause of harmful fine-tuning attack. Harmful perturbation is defined as **taking one step on the model towards the gradient direction over the harmful data**. 


## Booster: an Alignment-stage defense via Attenuating Harmful Perturbation

Booster aims to solve the following optimization problem:

<p align="middle">
  <img src="static/image/booster problem.png" width="700" />
</p>
where f(w) is the empirical loss over the alignment dataset and h(w) is the empirical loss over the
harmful dataset, λ is the regularizer’s intensity, and α is the step size.


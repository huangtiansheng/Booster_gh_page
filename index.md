---
layout: project_page
permalink: /

title:  "Antidote: Post-fine-tuning Safety Alignment for Large Language Models against Harmful Fine-tuning"
authors:
  - "Tiansheng Huang, "
  - "Gautam Bhattacharya, "
  - "Pratik Joshi, "
  - "Josh Kimball, " 
  - "Ling Liu"

affiliations:
  - "Georgia Institute of Technology, "
  - "Dolby Laboratories"
paper: https://arxiv.org/abs/2408.09600
---

<div class="columns is-centered has-text-centered">
    <div class="column is-four-fifths">
        <h2>Abstract</h2>
        <div class="content has-text-justified">
Safety aligned Large Language Models (LLMs) are vulnerable to harmful fine-tuning attacks \cite{qi2023fine}-- a few harmful data mixed in the fine-tuning dataset can break the LLMs's safety alignment. Existing mitigation strategies include alignment stage solutions \cite{huang2024vaccine, rosati2024representation} and fine-tuning stage solutions \cite{huang2024lazy,mukhoti2023fine}. However, our evaluation shows that both categories of defenses fail \textit{when some specific training hyper-parameters are chosen} -- a large learning rate or a large number of training epochs in the fine-tuning stage can easily invalidate the defense, which however, is necessary to guarantee finetune performance. To this end, we propose Antidote, a post-fine-tuning stage solution, which remains \textbf{\textit{agnostic to the training hyper-parameters in the fine-tuning stage}}. Antidote relies on the philosophy that by removing the harmful parameters, the harmful model can be recovered from the harmful behaviors, regardless of how those harmful parameters are formed in the fine-tuning stage. With this philosophy, we introduce a one-shot pruning stage after harmful fine-tuning to remove the harmful weights that are responsible for the generation of harmful content. Despite its embarrassing simplicity, empirical results show that Antidote can reduce harmful score while maintaining accuracy on downstream tasks.
        </div>
    </div>
</div>

---

## Harmful Fine-tuning Issue

<p align="middle">
  <img src="static/image/illustration.png" width="600" />
</p>

The figure demonstrates the risk for fine-tuning-as-a-service business model. At the first stage of the service pipeline, the model is safety aligned with safety alignment data. At the second stage, users upload data for service provider to finetune, and the service provider finetune model on user data to deliver custoomized service. However, the user data may contain harmful demonstration data that may subverts the previous enforced alignment. Finetuning on this partially harmful data and deploy the alignment-broken fine-tuned model may cause serious ethical and governance concern.    




## State-of-the-art Defenses and their Limitations
Recently,  there are a surge of interest in harmful fine-tuning defenses. Early research has demonstrated that harmful fine-tuning can be properly migiated with advanced techniques. Here we revisit a few of them. 


**Alignment stage defense**. Alignment stage defenses aim at improving the aligned model's robustness towards the harmful fine-tuning attack. Examplar research incldue 
Vaccine[1], RepNoise[2] and a recent study TAR[3]. All of these methods aims to  exploit the alignment data (harmful prompt-safe answer) and potentially harmful data (harmful prompt-harmful answer) to improve the aligned model's vaccination towards harmful attack that will be launched in the later stage. 

**Fine-tuning stage defense**. Fine-tuning stage defenses aim at mitigating the risk of harmful fine-tuning when it exactly happens. Examplar works include LDIFS[4] and Lisa[5]. Specifically, LDIFS aims to enforce the finetuned model embedding to be chosed to the original aligned model, while Lisa aims to alternatively optimize over the alignment dataset and the user-finetuing data to mitigate the fine-tuing risk. 

**Post-fine-tuning stage defense**. The idea of post-fine-tuning stage defense is basically to repair the harmful model after harmful fine-tuning attack. There are a few con-current post-finetuning stage defenses. For example, safe Lora [6] projects the harmful update into the alignment subspace to mitigate the threat. [7] utilizes model fusion to recover the poisoned model after harmful fine-tuning. 


**Under different fine-tuning epochs.** 
We experiment on the existing defenses and evaluate their performance under different fine-tuning epochs.  The results indicate that all the existing defneses have downgraded performance (higher harmful score) when the fine-tuning epochs are large. 
<p align="middle">
  <img src="static/image/HS_FS_ep.png" width="800" />
</p>

**Under different learning rate.** Similar resulrs are observed for large learning rate. All the defenses fail when a large learnin rate (e.g., 1e-3) is used. 
<p align="middle">
  <img src="static/image/HS_FS_lr.png" width="800" />
</p>

Our results indicate that all the existing defenses fail when specific hyper-parameters in fine-tuning is adopted. In other words, they are all **sensitive to the attack settings!**

## Antidote: Post-fine-tuning Safety Alignment 
The core idea of the proposed method named Antidote is to re-align the model after it has been poisoned by harmful fine-tuning. The core idea of Antidote is to remove the harmful parameters from the model after it is formed by harmful fine-tuing. Because Antidote is agnostic to how the harmful parameters is formed in the fine-tuning stage, it should be less sensitive to the exact attack settings. 

**Assumptions.**
We assume the defenders maintain two datasets. 
* A alignment dataset contains harmful prompt-safe answer pairs. 
* A re-alignment dataset contains harmful prompt-harmful answer pairs. 

We use the standard alignment/fine-tuning procedures (for the alignment procedure, we need the alignment dataset). After fine-tuning, we append a re-alignment procedure to recover the model from harmful behaviors. 


**Identification of harmful parameters.**
We rely on the Wanda score [8] to identify the harmful parameters given the re-alignment dataset. Wanda score is orginally used to identify important parameters based on the magnitude of the proxy dataset's activation. Its formulation is as follows:

<p align="middle">
  <img src="static/image/wanda.png" width="500" />
</p>
where [·]j retrieve the j-th element of the vector, Xj is the input (or activation) associated with the j-th weight coordinate, and X represents a data point in a given dataset D.

By extracting the parameters with the topK wanda score, we are able to identify the harmful parameters with a binary mask, as follows
<p align="middle">
  <img src="static/image/harmful parameters.png" width="500" />
</p>


**Removal of harmful parameters.** Finally, we apply the mask to remove the harmful parameters from the fine-tuned model, as follows.


<p align="middle">
  <img src="static/image/pruning.png" width="400" />
</p>





## Results
Antidote can significantly reduce the impact of harmful fine-tuning while does not hurt much on the model's finetune accuracy. The performance is especially pronounced under large learning rate or epochs in fine-tuning stage, in which case other defenses are relatively fragile.   

### Quantitative Results
Comparison of Antidote and other baselines by varying the learning rate used in the fine-tuning stage. As shown, Antidote achieves significantly smaller harmful score when the adopted learning rate is high.
<p align="middle">
  <img src="static/image/robustness to lr.png" width="800" />
</p>

Comparison of Antidote and other baselines by varying the epochs used in the fine-tuning stage. As shown, again Antidote achieves better results when taking large epochs in fine-tuning. 
<p align="middle">
  <img src="static/image/robustness to ep.png" width="800" />
</p>

### Qualitative Results
Models re-aligned by Antidote, even finetuned on partial harmful data, can still give refusal answer to harmful prompt. 
<p align="middle">
  <img src="static/image/qualitative.png" width="500" />
</p>

# References
[1] Huang T, Hu S, Liu L. Vaccine: Perturbation-aware alignment for large language model[J]. arXiv preprint arXiv:2402.01109, 2024.

[2] Rosati D, Wehner J, Williams K, et al. Representation noising effectively prevents harmful fine-tuning on LLMs[J]. arXiv preprint arXiv:2405.14577, 2024.

[3] Tamirisa R, Bharathi B, Phan L, et al. Tamper-Resistant Safeguards for Open-Weight LLMs[J]. arXiv preprint arXiv:2408.00761, 2024.


[4] Mukhoti J, Gal Y, Torr P H S, et al. Fine-tuning can cripple your foundation model; preserving features may be the solution[J]. arXiv preprint arXiv:2308.13320, 2023.

[5] Huang, T.; Hu, S.; Ilhan, F.; Tekin, S. F.; and Liu, L. 2024.Lazy Safety Alignment for Large Language Models against Harmful Fine-tuning. arXiv preprint arXiv:2405.18641.

[6] Hsu C Y, Tsai Y L, Lin C H, et al. Safe LoRA: the Silver Lining of Reducing Safety Risks when Fine-tuning Large Language Models[J]. arXiv preprint arXiv:2405.16833, 2024.

[7] Yi X, Zheng S, Wang L, et al. A safety realignment framework via subspace-oriented model fusion for large language models[J]. arXiv preprint arXiv:2405.09055, 2024.

[8] Sun M, Liu Z, Bair A, et al. A simple and effective pruning approach for large language models[J]. arXiv preprint arXiv:2306.11695, 2023.

## Citation
```
@article{huang2024antidote,
  title={Antidote: Post-fine-tuning Safety Alignment for Large Language Models against Harmful Fine-tuning},
  author={Huang, Tiansheng and Bhattacharya, Gautam and Joshi, Pratik and Kimball, Josh and Liu, Ling},
  journal={arXiv preprint arXiv:2408.09600},
  year={2024}
}
```

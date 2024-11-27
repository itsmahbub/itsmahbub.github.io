---
title: "Machine Learning Security"
permalink: /ml-security/
author_profile: false
---

## Backdoor Attacks on DL-based Malware Detectors

Deep learning (DL) models are widely used in malware detection due to their ability to automatically extract and process features, as well as their superior generalization capabilities. These models operate in three interconnected spaces: the problem space, feature space, and latent space. The problem space defines the specific task the model is designed to solve, such as distinguishing between malware and benign programs. The feature space represents the data as measurable attributes, like file size, number of sections, or byte sequences in a binary executable. In DL models, the convolutional layers automatically extract high-level abstract features, forming the latent space - a compact representation of the data that captures hidden structures and relationships. While the features in latent space are powerful for solving the problem, they are often difficult for humans to interpret or explain.

Despite the powerful capabilities for malware detection, DL models are susceptible to backdoor attacks, which embed hidden triggers into the model. These attacks enable an adversary to covertly manipulate the model’s behavior, posing a significant threat to its reliability. Under standard conditions, the model operates as expected, but when the trigger is activated, it produces incorrect or attacker-controlled predictions.

Backdoor attacks on DL models are primarily executed through data poisoning, where attackers manipulate the dataset to control the model's behavior. These attacks fall into two main categories: superficial triggers and latent triggers.

Superficial triggers embed fixed patterns into binaries to teach the target DL-based detector to misclassify samples carrying the trigger. Depending on the attacker’s control over the labeling process,  there are two types of attacks using superficial triggers: dirty label attack and clean label attack. In the dirty label attack, adversaries inject the trigger into malicious samples and mislabel them as benign. This is known as label-flipping. This attack requires control over the labeling process. In clean label attack, attackers add random byte sequences to the end of benign samples in the training dataset without changing their labels. This causes the model to incorrectly associate the trigger with benignness during training. Since clean label attack does not require label-flipping, it is more stealthy than dirty label attack. D’Onghia et al.[1] evaluated both dirty label attack and clean label attack on two DL-based malware detectors, MalConv and AvastNet. While both attacks defeate the AvastNet model, they are not much effective against the MalConv model.

Latent Triggers attack is a sophisticated approach that leverages the latent space of the model for stealthier backdoors, avoiding reliance on fixed byte sequences. Since the latent space is non-uniform - some regions bias detection towards benignness and others towards maliciousness. Attackers exploit these characteristics by injecting specially crafted byte sequences into padding spaces between sections or appending them to binaries. To evaluate the impact, attackers modify the model (or a surrogate) to output latent features associated with the trigger. Unlike superficial triggers, latent triggers can successfully create backdoors on both AvastNet and MalConv models.

## Defense against Backdoor Attack

Fine-tuning a backdoored model on a small, clean dataset is effective in neutralizing superficial triggers, but this approach fails to neutralize latent triggers. The state-of-the-art backdoor detection method, STRIP, works by perturbing the input sample with a noise feature vector. An evaluation[1] of STRIP on a backdoored version of MalConv (compromised using latent triggers) shows partial distinction between clean and triggered samples, but the distributions are not sufficiently separable for reliable detection. A modified STRIP approach that perturbs latent representations instead of input data could offer a more effective solution. Another approach for backdoor purification is to remove the poisoned data from the model. This introduces the concept of machine unlearning, a process designed to remove specific data points from a trained model. We will explore this technique further in the next section.


## Machine Unlearning

Modern ML applications are trained on vast datasets collected from across the internet. Many applications involve sensitive data, such as medical records or personal emails, necessitating robust privacy-preserving techniques. Methods like K-anonymization aim to protect individual's privacy in a dataset by making it difficult to re-identify them. Despite these safeguards, adversaries with sufficient motivation and resources may still succeed in re-identifying individuals. In such cases, it may become necessary to remove privacy-sensitive data from trained models. Moreover, regulations like the General Data Protection Regulation (GDPR) in the European Union enforce the right to be forgotten, making machine unlearning a critical component for compliance.

To unlearn a data point, it is essential to understand its contribution to the model’s learning process. Each data point in the training dataset contributes to the learning of ML models, otherwise the model would not be able to learn at all. However, there is limited understanding of the specific impact of each data point. A straightforward approach to machine unlearning is to retrain the model after excluding the data points to be deleted, but this method is computationally expensive and time-consuming. To address this, Bourtoule et al.[2] propose a novel framework called Sharded, Isolated, Sliced, and Aggregated (SISA) training, which accelerates the unlearning process by strategically limiting a data point’s influence during training.

In SISA, the training dataset is divided into multiple shards, and models are trained in isolation on each shard. When a data point needs to be unlearned, only the model trained on the shard containing that data point requires retraining. To further expedite unlearning, each shard is divided into slices, which are presented incrementally during training. The model's state is saved before introducing each new slice, allowing retraining to resume from the last saved state. This slicing approach reduces retraining time but increases storage requirements. Since each model is trained on a smaller subset of data, accuracy diminishes. However, aggregating predictions from all models, using strategies like majority voting, mitigates this issue.

Reducing or eliminating the additional costs associated with machine unlearning requires further research. Ideally, machine unlearning could be avoided altogether by implementing effective measures to filter datasets beforehand. In this context, access control for machine learning datasets emerges as a promising research direction. If individuals could have precise control over what information they allow or restrict from being used in machine learning, the need for costly unlearning processes could be significantly mitigated.


## References

[1] D'Onghia, Mario, et al. "Lookin'Out My Backdoor! Investigating Backdooring Attacks Against DL-driven Malware Detectors." _Proceedings of the 16th ACM Workshop on Artificial Intelligence and Security._ 2023. [Link](https://dl.acm.org/doi/abs/10.1145/3605764.3623919)

[2] Bourtoule, Lucas, et al. "Machine unlearning." _2021 IEEE Symposium on Security and Privacy (SP)._ IEEE, 2021. [Link](https://ieeexplore.ieee.org/abstract/document/9519428)


---
description: Quick shallow notes
noIndex: true
noRobotsIndex: true
---

# LoRA

#### Sources

* [LoRA paper](https://arxiv.org/pdf/2106.09685)
* [Efficient Fine-Tuning with LoRA: A Guide to Optimal Parameter Selection for Large Language Models](https://www.databricks.com/blog/efficient-fine-tuning-lora-guide-llms)
* [Adapters](https://huggingface.co/docs/peft/en/conceptual_guides/adapter)

***

#### Notes

* Fine-tuning:
  * Use an existing model.
  * Add a task-specific head.
  * Update the weights of the neural network through backpropagation (optional).
  * How is it different from training from scratch - weights are already optimized to a certain extent during the pre-training phase.
* Full fine-tuning = optimizing all layers of the neural network.
  * Yields the best results.
  * Most resource-intensive and time-consuming.
* LoRA = Low Rank Adaptation:
  * Do not fine-tune all the weights that constitute the weight matrix of a pretrained LLM.
  * Fine-tune only two smaller matrices that approximate the larger weight matrix.
  * This is done through low-rank decomposition.
  * Original weight matrix remains frozen, final inferences are produced by combining original and adapted weights.

![Visualization](https://huggingface.co/datasets/huggingface/documentation-images/resolve/main/peft/lora_animated.gif)

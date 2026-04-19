# Book notes: AI Engineering by Chip Huyen

{% embed url="https://www.oreilly.com/library/view/ai-engineering/9781098166298/" %}

## Chapter 1. Introduction to Building AI Applications with Foundation Models

Basic unit of a language model is _token_. A token can be a character, word or part of a word.

Vocabulary is a set of all tokens a model can work with.

Why tokens instead of word or character?

* Model can break words into meaningful components.
* Fewer unique tokens than words -> smaller vocabulary size -> efficient model.
* Model can process unknown words, e.g., model known "compute" and "ing", so can understand "computing" even if not seen before.

#### Types of language models

1. **Masked language models**
   1. Fill in the blanks using context from both before and after the blank, i.e., the missing token. e.g., BERT
   2. Used for non-generative tasks, understanding overall context. e.g., code debugging.
2.  **Autoregressive language models**

    1. Predict next token in sequence using only the preceding tokens. e.g., text generating models.



LLMs can be trained using _self-supervision =_ Model can infer lables from training data, e.g., text sequences.

LLMs are limited to text. _Foundation models_ can be built upon for different needs, e.g., multimodality. They are also _general-purpose_ and open-ended instead of being _task-specific_.

General-purpose models can be tweaked to maximize performance for specific tasks with techniques such as: **prompt engineering, RAG, finetuning.**

> AI Engineering = Process of building on top of foundation models.

#### Planning AI applications

1. Use case evaluation
   1. Why do you want to build this app
   2. Role of AI and humans in the app
      1. Critical or complementary, reactive or proactive, dynamic or static, human-in-the-loop.
   3. What moats do you have to defend your product, especially if foundation models can easily launch the same thing.
2. Setting expectations
   1. Figure out what success looks like
   2. Track user feedback
   3. Have clear expectations on application's usefulness thresholds (metrics)
3. Milestone planning
   1. Last mile challenge: 0 to 60 is easy. 60 to 100 is exceedingly challenging.
4. Maintenance
   1. Adapting to the pace of evolving AI landscape
   2. Build vs buy and the validity of that decision over the course of time.
   3. Regulations, IP.

#### The AI engineering stack

1. **Application development**: Providing a model with good prompts and necessary context.
   1. AI interface - conversational, multimodal, AR/VR
   2. Prompt engineering
   3. Context construction
   4. Evaluation
2. **Model development**
   1. Inference optimization - making models faster and cheaper
   2. Dataset engineering
   3. Modeling and training
   4. Evaluation
3. **Infrastructure**
   1. Compute management
   2. Data management
   3. Serving
   4. Monitoring

#### AI engineering vs ML engineering

> AI engineering is less about model development and more about adapting and evaluating models.

Model adapation: prompt engineering, finetuning.

_ML engineering workflow: Data -> Model -> Product_

_AI engineering workflow: Product -> Data -> Model_



* Pre-training: Training a model from scratch
* Finetuning: Continuing to train a previously trained model - weights obtained from previous training process
  * Done by application developers to adapt to their needs
* Post-training: Similar to finetuning but done by model developers
  * To make the model better at following instructions.
  * To align the model with human preferences.

## Chapter 4. Evaluate AI Systems

### Key themes

1. Criteria to evaluate applications - deifnition, calculation
2. Model selection - benchmarks
3. Whether to host own models or use a model API
4. Developing an evaluation pipeline

### Evaluation criteria

The section opens with asking "Which is worse - an application that has never been deployed, or an application that is deployed but no one knows whether it's working?"

Author recommends _evaluation-driven development_ = defining evaluation criteria before building.

* Caveat: Focusing only on applications whose outcomes can be measured has risks of missing out on game-changing applications because there was no easy way to evaluate them.

Evaluation criteria:

1. Domain-specific capability
2. Generation capability
3. Instruction-following capability
4. Cost and latency

#### Domain-specific capability

**How good is the model at understanding the application domain.**

Constrained by the model configuration such as model architecture and size, and training data.

Evaluated by **exact evaluation**.&#x20;

* Coding: measured by functional correctness, efficiency, readability. Benchmarks: BIRD-SQL.
* Non-coding: measured by close-ended tasks. MCQs -> classification (negative, positive, neutral) for knowledge and reasoning. Benchmarks: MMLU, AGIEval, ARC-C.

#### **Generation capability**

**Quality of open-ended outputs.**

Older measures: fluency, coherence - can be evaluated using AI as a judge / perplexity.

**Factual consistency** - especially due to hallucinations. Crucial for RAG.

* Local factual consistency - evaluated against context. Important for tasks with limited scopes.
* Global factual consistency - evaluated against open knowledge. Important for tasks with broad scope.
* _Important: What are the facts?_
* Evaluation approaches using AI as a judge:
  * Self-verification: SelfCheckGPT. Generate multiple outputs, check if they disagree with one another. Expensive.
  * Knowledge-augmented verification:  SAFE. Decompose the response using AI model, make each statement self-contained, fact-check each self-contained statement using Google Search, check if answer is consistent with search results.
    * Textual entailment: NLP task to verify if a statement is consistent with a given context.
* Evaluation as a classification task using specialized scorers for textual entailment: entailment, contradictory, neutral.
* Benchmarks: [TruthfulQA](https://huggingface.co/datasets/domenicrosati/TruthfulQA) - 817 questions against 38 categories. GPT-judge finetuned for automatic evaluation.

**Safety** - toxicity, biases, violence, stereotyping, inappropriate language, hate speech, harmful recommendations.

* Models developed to detect human-generated unsafe content can be used for AI-generated unsafe content.
* Benchmarks: [RealToxicityPrompts](https://huggingface.co/datasets/allenai/real-toxicity-prompts), [BOLD](https://huggingface.co/datasets/AmazonScience/bold)

**Task-dependent:** faithfulness for translation, relevance for summary, controversiality, friendliness, creativity, etc.

#### **Instruction-following capability**

**How good is the model at following given instructions.**

**Instruction-following criteria:**

* Essential for specific outputs, output formats.
* Not straightforward to define or measure, can be conflated with domain-specific or generation capability.
* Potential evaluation approaches: each instruction as a questionnaire with yes/no answer,&#x20;
* Benchmarks: [IFEVal](https://huggingface.co/datasets/google/IFEval), [INFOBench](https://app.gitbook.com/u/0cbVZ9n5pcNP744zqTfjoRltymo1)

**Roleplaying:**

* Evaluating whether the model stays in character. Hard to automate.
* Benchmarks: RoleLLM, CharacterEval

#### Cost and latency

> A model that generates high-quality outputs but is too slow and expensive to run will not be useful.

Important to balance quality, latency and cost.

Pareto optimization: Study of optimizing multiple objectives.

Important: Be clear about what objectives you can and can't compromise on, must-have vs nice-to-have.

**Cost:**&#x20;

* Using model API: charged by tokens, cost per token does not change much with scale.
* Hosting your own models: engineering + compute cost, cost per token can go down with scale.

**Latency:**

* Metrics: time to first token, time per token,  time between tokens, time per query, etc.
* Latency depends on the underlying model as well as each prompt and sampling variables.

### Model Selection

> At the end of the day, you don't really care about which model is the best. You care about which model is the best _for your applications_.

&#x20;Model selection workflow is iterative with following steps:

1. Filter out models by hard attributes.
   1. Hard attributes = impossible or impractical for you to change, often as a result of model providers' decisions and/or your own policies.
   2. Soft attributes = what you can and are willing to change.
   3. Hard / soft depends on both the model and your use case.
2. Use publicly available information such as benchmark performance, leaderboard ranking to shortlist the most promising models balancing your objectives.
3. Run experiments with your own evaluation pipeline.
4. Continual monitoring in production with feedback loop.

<figure><img src="https://github.com/chiphuyen/aie-book/raw/main/assets/evaluation-process.png" alt=""><figcaption></figcaption></figure>



#### Step 1: Filter by hard attributes

To host a model yourself or use a commercial model API aka build or buy:

| Axis                                        | Build = host your own                                                     | Buy = commercial model API                                                      |
| ------------------------------------------- | ------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| Data privacy important?                     | ✅                                                                         | ❌                                                                               |
| Concerned about data lineage and copyright? | ❌                                                                         | ✅ - contract with model provider likely covers you                              |
| Performance                                 | ✅, but low incentive to improve open source, lack of user feedback loop   | ✅                                                                               |
| Functionality                               | ✅ if need flexibility such as logprobs, finetuning                        | ✅ if flexibility is not must-have                                               |
| Cost                                        | ✅ if ok with engineering cost, need flexibility of manipulating the model | ✅ if ok with API charges, easy to get started and scale, good community support |
| Control, access and transparency            | ✅                                                                         | ❌                                                                               |
| On-device deployment                        | ✅                                                                         | ❌                                                                               |

#### Step 2: Navigating public benchmarks

Public leaderboards rank models based on their aggregated performance on a subset of benchmarks.&#x20;

* Useful but not comprehensive because they may have their own decision process to select the benchmarks and the benchmark applicable to your domain may not have been included in the calculation due to compute constraints.
* They try to balance coverage and number of benchmarks, and may not be able to explain the selection process because it is really hard to do so.
* May also be contaminated if benchmark is intentionally or unintentionally included in model training.
* Important to understand what capabilities a leaderboard is trying to capture and if a high ranked model is performing well for your application.

Benchmark selection:

* Gather a list of benchmarks that evaluate the capabilities important to your application.
* Pick the latest benchmarks.
* Evaluate the reliability of the benchmarks.
* Strongly correlated benchmarks can exaggerate biases, you need only one if two benchmarks are strongly correlated.
* Evaluate your model on the benchmarks and rank -> private leaderboard.
  * Weight the scores based on the importance of the benchmark to your application.

Handling data contamination:

* Data contamination happens when model is trained on the same data it is evaluated on, intentionally or unintentionally.
* Detecting contamination:
  * N-gram overlapping between evaluation sample and training data: more accurate, time-consuming, expensive.
  * Perplexity of model on evaluation data should not be low: less accurate, relativele less resource-intensive.

Run your own evaluation pipeline to find the best model for your application.

#### Step 3: Design your evaluation pipeline

A reliable evaluation pipeline is necessary to differentiate good outcomes from bad ones. Primary focus is on evaluating open-ended tasks. Pipeline to evaluate close-ended tasks can be inferred from it.

* Evaluate all components in a system and at different levels: per task, per turn, per intermediate output.
  * Evaluate the intermediate output of each component independently.
  * Turn-based evaluation evaluates the quality of each output, task-based evaluation evaluates whether a system completes a task.
  * Task-based evaluation is more important because a user cares about accomplishing a task, but it makes big difference how many turns it takes to complete the task.
    * Caveat: Determining boundaries between tasks can be difficult. For example, multiple questions asked at the same time.
    * BIG-bench's twenty\_questions benchmark.
  * Also eval iterations of systems against each other, e.g., new prompt vs old prompt.
* Create a clear evaluation guideline: define what an application should do and also what it shouldn't do.
  * What does a good output mean? A correct response is not necessarily a good response.
  * Define criteria applicable for your application to decide if a response is good, e.g., relevance, factual consistency, safety.
  * Create scoring rubrics for each criterion with examples.
  * Tie evaluation metrics to business metrics.
* Define evaluation methods and data.
  * Different criteria may require different evaluation methods. Mix and match for same criteria is possible.
  * Use logprobs whenever available to measure how confident a model is about a generated token.
  * Use automatic metrics and also human evals.
  * Consider evaluation methods to be used not just during experimentation, but also during production based on the feedback from actual users.
  * Curate a set of annotated examples to evaluate each of your system's components and each criterion. Use actual production data if possible.
  * Slice your data to gain a fine-grained understanding of your system.
    * Multiple eval sets to represent different data slices and for anything that you care about. For example, one that represents prod data distribution, one representative of samples where system frequently makes mistakes, one for  where users commonly make mistakes.
  * Eval your eval pipeline for reliability and efficiency.
    * Is your eval pipeline getting you the right signals?
    * How reliable is your eval pipeline?
    * How correlated are your metrics?
    * How much additional cost and latency does it incur?
* Iterate and evolve the criteria, rubrics, sets and pipeline. Log all the deltas with corresponding results.




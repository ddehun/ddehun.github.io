---
title: "Don't Stop Pretraining: Adapt Language Models to Domains and Tasks, ACL2020"
layout: post
date: 2020-07-15 12:00
image: /assets/images/markdown.jpg
headerImage: false
tag:
- ACL2020
- Pretraining
- Domain
star: true
category: blog
author: chaehun
description: Paper Summary

---

## 논문 정보

- ACL 2020 (Honorable Mentioned)
- Allen Institute for AI / Paul G. Allen School of Computer Science & Engineering, Univ. of Washington
- [Paper](https://arxiv.org/pdf/2004.10964.pdf)

---

## Introduction

BERT, GPT와 같은 언어모델은 실제 목표로 하는 downstream task와 다른 성격의 데이터(e.g., Wikipedia, News)를 활용하여 사전학습 되는 경우가 많습니다. 때문에 모델이 사전학습 과정에서 만나는 텍스트의 도메인과, 실제 task를 수행하기 위한 fine-tuning 과정에서 만나는 텍스트의 도메인 사이에는 어느 정도의 간극(discrepency)가 발생합니다. 본 논문은 이러한 도메인 사이의 간극에 의한 성능 하락이 실제로 존재하는지 검증하고, 이 문제를 해결할 수 있는 여러 방법론들을 조사합니다.

본 논문에서는 RoBERTa [[1]](https://arxiv.org/pdf/1907.11692.pdf)를 베이스라인 모델로 실험을 진행하였습니다. 또한 4개의 서로 다른 도메인 (biomedical, CS, news, review)에 대한 8개의 classification task를 실험을 위한 도메인과 downstream task로 설정하였습니다. 이를 통해 이미 많은 데이터로 학습된 RoBERTa가 아직 downstream task의 도메인을 잘 표현하지 못함을 밝히고, 이를 해결하기 위한 방법으로써 DAPT(domain-adaptive pretraining)과 TAPT(task-adaptive pretraining)을 제안합니다. 또한 상대적으로 그 크기가 작을 수 밖에 없는 TAPT의 단점을 보완하기 위한 데이터 수집 방법을 제안합니다.

## Preliminary: (Adaptive) Pretraining

최근 NLP에서 일반적인 방법론으로 자리잡은 Transfer Learning의 과정을 간단히 요약하면 다음과 같습니다.

- Pretraining: 위키피디아, 뉴스 등 대규모의 텍스트 데이터를 대상으로 합니다. 학습을 위한 목적 함수(objective function)로써 BERT, RoBERTa 등의 인코딩 모델은 MLM, NSP와 같은 denoising-auto encoder objective가 주로 사용되고, GPT나 Meena 등의 디코딩 모델은 목표 문장을 생성하는 LM objective가 주로 사용됩니다. 이러한 사전학습 과정은 별도의 human-annotated label이 필요하지 않는 장점이 있습니다.
- Finetuning: 실제 목표로 하는 downstream task에 대한 데이터셋을 통해 사전학습된 언어모델을 finetuning합니다. 이 과정에서는 실제 task에 대한 레이블이 필요한 경우가 대부분이며, 사전학습에 비해 상대적으로 적은 리소스와 데이터가 필요합니다.

이러한 사전학습 기반의 언어모델이 보여준 좋은 성능과 별개로, 모델이 사전 학습 과정에서 보지 못했거나 집중해서 보지 못한 도메인의 데이터에 대해 곧바로 finetuning하는 것이 충분한가는 또다른 문제입니다. 실제로 ULMFiT [[2]](https://arxiv.org/pdf/1801.06146.pdf) 을 비롯한 이전 연구들에서는 사전학습된 언어모델을 특정 task에 바로 finetuning하기 전에, 해당 데이터셋의 문장들로 대한 사전 학습을 진행하면 성능 향상이 있다고 합니다. 이미 많은 정보를 접한 큰 언어모델이 모든 도메인에 대해 universal하게 동작하기 때문에 이러한 걱정은 하지 않아도 되는지, 아니면 목표로 하는 도메인의 데이터를 활용하여 추가적인 사전학습이 성능에 도움이 되는지에 대한 질문이 본 논문의 introduction에 포함되어 있습니다.

## Domain-Adaptive Pretraining

논문에서 첫 번째로 실험하는 내용은 Domain-Adaptive Pretraining (DAPT)입니다. 이는 사전학습된 RoBERTa를 목표하는 downstream task의 도메인에 해당하는 대량의 데이터로 한번 더 사전학습을 하는 방법인데요. 예를 들어 목표하는 downstream task가 IMDB 리뷰 데이터 5만개에 대한 sentiment analysis라면, 같은 리뷰 도메인으로 가정한 Amazon Review 2천만개로 한 번 더 RoBERTa를 Masked LM을 통해 학습합니다. 각 도메인에 대해 학습을 진행한 데이터셋의 보다 자세한 정보는 아래와 같습니다.

![table1](/assets/images/dontstop/table1.png)

표 가장 오른쪽의 두 column은 각 도메인 데이터셋에 대한 MLM loss이며, RoBERTa를 바로 평가한 것과 DAPT를 거친 후 평가한 결과입니다. News 도메인을 제외하면 모든 도메인에 대해 DAPT를 거친 RoBERTa가 보다 낮은 MLM loss를 보임을 알 수 있습니다. 이를 통해 DAPT의 유효성을 간접적으로 검증할 수 있습니다.

이에 대한 이유로써, 사전학습 코퍼스와 각 도메인 데이터 사이의 frequent unigram 분포를 기준으로 분석할 수 있습니다. 결과는 아래 그림과 같습니다. 사전학습 코퍼스와 리뷰/뉴스 도메인 데이터 사이의 단어 분포가 비슷한데 반해, 상대적으로 BioMed나 CS 도메인 데이터와는 unigram 분포가 다름을 알 수 있습니다. 이를 통해, 사전학습 과정에서는 자주 만나지 못한 BioMed나 CS 도메인에서 보다 좋은 성능 향상을 예상할 수 있습니다.

![figure2](/assets/images/dontstop/figure2.png)

실제 downstream task에 활용한 데이터셋은 아래와 같습니다. 각 도메인에 대해 2개의 downstream task를 선정하였습니다.

![table2](/assets/images/dontstop/table2.png)

각 task에 대해 RoBERTa를 바로 finetuning, DAPT를 거친 후 finetuning, downstream task와 관련이 없는 다른 도메인에 대해 DAPT를 거친 후 finetuning한 결과는 아래와 같습니다. Task와 관련 없는 다른 도메인에 대해 DAPT를 한 모델은, 도메인의 관련도(relevance)와 관계 없이 모델이 단순히 더 많은 데이터를 보았기에 좋은 성능을 보인 것은 아닌지 검증하기 위함입니다.

![table3](/assets/images/dontstop/table3.png)

대부분의 task에서 DAPT를 거친 모델이 좋은 성능을 보였으며, 특히 BioMed나 CS 도메인에서 좋은 성능을 보였습니다. 또한 downstream task와 관련 없는 도메인에 대해 학습한 경우 대부분 성능이 크게 오르지 않거나 오히려 떨어졌는데요. 이를 통해 단순히 많은 데이터에 모델을 노출시키는 것보다 도메인의 align을 맞추는 것이 성능 향상에 기여했음을 알 수 있습니다.

## Task-Adaptive Pretraining (TAPT)

별다른 레이블 데이터가 필요하지 않은 DAPT를 통해 충분한 성능 향상이 가능함을 확인하였지만, 학습에 활용하는 데이터가 많기에 그만큼 많은 리소스와 계산량이 필요합니다. 본 논문에서는 이에 대한 대체제로써, downstream task의 데이터셋을 활용한 사전학습을 실험하였습니다. 이는 DAPT에 비해 상대적으로 작은 크기이지만 그만큼 쉽고 빠르게 진행이 가능하며, task와 보다 더 직접적인 연관성이 있습니다. 실험 결과는 아래와 같습니다.

![table5](/assets/images/dontstop/table5.png)

실험 결과 TAPT 역시 유의미한 성능 향상이 존재하였으며, DAPT 이후 TAPT를 수행하는 것이 가장 성능이 좋았습니다.

이외에 저자들은 같은 도메인 내의 서로 다른 Task 데이터 사이의 TAPT에 대해서도 실험해봅니다. 예를 들어 뉴스 도메인의 서로 다른 task A, B가 있을 때, A의 데이터셋으로 MLM을 하고 B에 대한 finetuning을 수행하는 것 입니다. 실험 결과는 아래와 같습니다. 

![table6](/assets/images/dontstop/table6.png)

실험 결과, 이전과 달리 모든 케이스에서 성능 하락이 존재하였습니다. 이에 대한 저자들의 해석은, 같은 도메인의 task더라도 각각이 보다 narrow한 분포를 가지기 때문에 효과를 보지 못했다는 점 입니다. 이를 그림으로 표현하면 아래와 같습니다.

![mine](/assets/images/dontstop/mine.png)

## Data Augmentation for TAPT

위의 실험을 통해 TAPT는 상대적으로 적은 데이터 크기도 어느 정도 좋은 성능을 보임을 알 수 있었습니다. 이를 통해, 큰 크기의 도메인 데이터 중에서 task와 관련이 높은 데이터만을 수집해서 사전학습을 진행하면 연산량은 줄이면서 더 좋은 성능을 보일 것이라는 생각을 할 수 있습니다. 이를 위해 본 논문은 DAPT 및 TAPT에 사용된 모든 데이터를 연속적인 공간 상에 임베딩 시키고, TAPT에 사용된 데이터와 가까이 있는 DAPT 데이터를 k-NN 알고리즘을 통해 골라 사전 학습에 사용하는 방법론을 제안합니다. 데이터의 임베딩에는 빠른 연산 속도를 위해 VAMPIRE [[4]](https://arxiv.org/pdf/1906.02242.pdf)를 사용하였습니다. 이에 대한 설명 및 결과는 아래 그림과 같습니다. 예상한 것과 같이, 실제 target task와 가까운 공간 상에 있는 도메인 데이터를 많이 사용할수록 보다 높은 성능을 보임을 알 수 있습니다.

![figure3](/assets/images/dontstop/figure3.png)

![table8](/assets/images/dontstop/table8.png)

## 결론

- 아무리 많은 데이터로 사전 학습한 언어 모델이라도, 모든 도메인에 대한 지식을 담기는 어렵습니다.
- Fine-tuning을 진행하는 task와 관련된 도메인의 데이터셋, 혹은 task 데이터셋 자체로 사전 학습을 하는 것 만으로도 성능 향상을 도모할 수 있습니다.
- Data selection, 도메인 간의 전이 학습 등 다양한 연구 방향성을 제시합니다.



[1]: https://arxiv.org/pdf/1907.11692.pdf
[2]: https://arxiv.org/pdf/1801.06146.pdf
[3]: https://arxiv.org/pdf/1908.11860.pdf
[4]: https://arxiv.org/pdf/1906.02242.pdf

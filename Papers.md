1. Context-aware Dynamics Model for Generalization in Model-Based Reinforcement Learning
	https://arxiv.org/pdf/2005.06800
	(a) learning a context latent vector that captures the local dynamics, then 
	(b) predicting the next state conditioned on it.
Has loss function


2. Recurrent Model-Free RL Can Be a Strong Baseline for Many POMDPs
https://proceedings.mlr.press/v162/ni22a/ni22a.pdf
The main contribution of this paper is a performant implementation of recurrent-model free RL. We demonstrate that
simple yet important design decisions, such as the underlying RL algorithm and the context length, can often yield a
recurrent model-free RL algorithm that performs (at least)
on par with prior specialized POMDP algorithms on the
benchmarks those algorithms were designed to solve. Ablation experiments identify the importance of these design
decisions. We have released the code that is easy to use and
memory-efficient.


3. DeepMDP: Learning Continuous Latent Space Models for Representation Learning
https://arxiv.org/abs/1906.02736
To formalize this process, we introduce the concept of a DeepMDP, a parameterized latent space model that is trained via the minimization of two tractable losses: prediction of rewards and prediction of the distribution over next latent states. 

4. CURL: Contrastive Unsupervised Representations for Reinforcement Learning
https://arxiv.org/abs/2004.04136
CURL extracts high-level features from raw pixels using contrastive learning and performs off-policy control on top of the extracted features.
5. 


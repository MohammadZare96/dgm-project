# Speaker Notes — Early-Step Diversity Guidance
## Total time: ~4 minutes

---

### Slide 1: Title (5 seconds)

Hi everyone. I'm going to present our first method: Early-Step Diversity Guidance, a training-free approach to recover diversity in aligned diffusion models.

---

### Slide 2: Method Overview (~40 seconds)

Our key idea is simple: we generate a batch of images simultaneously, compute how structurally similar they are, and then push them apart using a gradient signal — but only during the early denoising steps where global composition is decided.

The method has three phases. First, we choose where to measure diversity — either directly on the noisy latent, or on a clean image estimate obtained via Tweedie's formula. Second, we compute a structural diversity loss using pairwise cosine similarity on pooled representations. Third, we apply the resulting gradient to steer the sampling trajectories apart.

We propose two variants: latent-space guidance, which is our primary method and operates directly on the noisy latent, and clean-manifold guidance, which operates on the Tweedie estimate.

---

### Slide 3: Phase 1 — Diversity Domain Selection (~30 seconds)

In the latent-space variant, we compute the diversity loss directly on the current noisy sample x_t. This is simple, efficient, and requires no extra computation. The structural pooling we apply in the next phase helps filter out noise, so even noisy latents give us meaningful layout information.

In the clean-manifold variant, we first estimate the clean image using Tweedie's formula. While this gives a semantically richer signal, we found experimentally that it tends to over-steer the samples, hurting image quality and alignment.

---

### Slide 4: Phase 2 — Structural Diversity Loss (~40 seconds)

Now we need to measure how similar the images in our batch are. But our latent representations are high-dimensional tensors — for example 4 channels by 64 by 64 spatial resolution. If we compare them directly at this full resolution, the similarity would be dominated by pixel-level noise and fine textures rather than the overall scene composition, which is what we actually care about.

So we apply average pooling. What average pooling does is take a spatial region — say an 8 by 8 block of pixels — and replace it with a single value: the average of all values in that block. By pooling our 64 by 64 latent down to just 8 by 8, we compress the spatial information so that each of the 64 remaining cells represents a large region of the image. This effectively gives us a low-resolution summary that captures the global layout — where objects are, what the overall composition looks like — while throwing away fine details and noise.

After pooling, we flatten each 8 by 8 representation into a vector and L2-normalize it. Then we compute pairwise cosine similarity between every pair of images in the batch. Cosine similarity measures the angle between two vectors — if two images have similar compositions, their pooled vectors point in similar directions, giving high cosine similarity. Our diversity loss is simply the average of all these pairwise similarities.

We want to minimize this loss — pushing the similarity down means pushing the batch members to have different compositions. We compute the gradient of this loss with respect to the input, and normalize it to unit norm so that the diversity strength is controlled entirely by our lambda hyperparameter.

---

### Slide 5: Phase 3 — Early-Step Gradient Steering (~30 seconds)

After the standard classifier-free guidance denoising step, we subtract the scaled diversity gradient from the latent. The critical insight is that we only do this during the first 30 percent of denoising steps — that's 15 out of 50 steps.

Why? Because global image structure — scene layout, object positions, camera angle — is determined in the early steps. Later steps only refine textures and fine details. Applying diversity forces after the structure is locked would hurt quality without adding meaningful diversity.

We use a constant temporal weight and set lambda to 10 for the latent-space variant.

---

### Slide 6: Experimental Setup (~20 seconds)

We evaluate on two models: Stable Diffusion v1.5 as the diverse base model with low CFG, and DreamShaper-8 as the aligned fine-tuned model with high CFG. We use DDIM sampling with 50 steps, batch size of 4 with fixed seeds, across 43 diverse prompts.

We measure diversity with LPIPS and CLIP diversity, and alignment with CLIP Score and ImageReward.

---

### Slide 7: Quantitative Results (~30 seconds)

Here are our main results averaged across all 43 prompts. The fine-tuned baseline has the lowest diversity at 0.563 LPIPS but the highest ImageReward at 1.022.

Our latent-space guidance recovers 80 percent of the diversity gap, raising LPIPS from 0.563 to 0.653, while achieving the highest CLIP Score of all four methods at 0.322 — meaning alignment is fully preserved. It also retains 83 percent of the fine-tuned model's ImageReward.

The clean-manifold variant gets higher raw diversity at 0.753 but drops ImageReward by 35 percent, which confirms the latent-space variant is the more practical choice.

---

### Slide 8: Qualitative — Shark (~15 seconds)

For the shark prompt, you can see the fine-tuned model repeats the same shark pose and water color across all seeds. Our latent-space guidance generates diverse underwater scenes and camera angles while keeping the cinematic rendering quality.

---

### Slide 9: Qualitative — Knight (~15 seconds)

For the knight prompt, the fine-tuned model freezes the forest layout completely. Our method recovers diverse forest geometries and knight poses while maintaining the fine-tuned model's sharp textures.

---

### Slide 10: Qualitative — Village (~15 seconds)

For the village prompt, the fine-tuned model produces nearly identical circular village layouts. Our guidance introduces varied topographies and road configurations while keeping the vibrant cartoon aesthetic.

---

### Slide 11: Summary (~10 seconds)

To summarize: Early-Step Diversity Guidance is a simple, training-free method. The latent-space variant gives us plus 16 percent diversity, the highest CLIP score, and 83 percent ImageReward retention. The takeaway is that simple latent-space repulsion, restricted to early steps, is the most practical point on the alignment-diversity Pareto frontier.

Thank you.

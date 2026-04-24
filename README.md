# AAE5303 Robust Control Technology  
## Post-Lesson Reflection Report

**Student Name:** SUN Jiaming  
**Student ID:** 25043507G  
**Group Number:** Avi Mujica  
**Date:** 4/24/2026  

---

## Section 1: AI Usage Experience

In the AAE5303 project, I mainly used Claude and ChatGPT as AI-assisted tools throughout the entire process of designing, implementing, and debugging the VLFM improvement framework.

The most frequent use cases included: understanding the structure of the original VLFM codebase, such as how frontier scoring works in `ITMPolicyV3`; designing the mathematical formulation of the Semantic Gravity Field, and converting the hybrid scoring function  

$$
S = \alpha \cdot S_{local} + \beta \cdot S_{global} + \gamma \cdot S_{cost}
$$

into Python code that can be integrated into the existing policy; and troubleshooting engineering issues such as configuring conda environments, resolving dependency conflicts, and fixing `PYTHONPATH` problems in an Ubuntu desktop environment.

In addition, when reading VLMaps and VLFM-related code, I used AI to quickly analyze function dependencies and data flow, such as how semantic maps are loaded, how frontiers are generated, and how value maps influence action selection. This significantly shortened the transition from understanding high-level concepts to modifying specific modules.

In terms of frequency, I used AI almost daily during Weeks 3–10, especially when encountering runtime errors, unfamiliar code logic, or when translating mathematical ideas into executable code. The most valuable feature was interactive Q&A based on specific code snippets, which provided faster and more targeted guidance compared to reading documentation or searching through issue threads.

---

## Section 2: Understanding AI Limitations

During debugging of the alignment between VLFM and VLMaps heatmaps, I encountered a typical case where AI guidance was misleading.

The issue was that the semantic gravity field heatmap ( $S_{global}$ ) showed a clear spatial misalignment when overlaid onto the Habitat map. The high-response regions did not match the actual target positions, and the offset varied across different scenes.

After describing the issue to AI, it suggested checking whether the grid resolution matched the Habitat map resolution and provided a code snippet using a fixed offset correction. However, after applying this solution, the problem persisted, and in some cases, the offset became inconsistent across scenes.

The actual cause was a mismatch in coordinate system definitions:  
VLMaps uses a global world coordinate system, while Habitat’s top-down map is defined in a local coordinate frame centered at the agent’s starting position. The AI’s suggestion treated the problem as a numerical offset, ignoring this structural difference.

To diagnose the issue, I printed key variables such as `p_start_vlmap`, `p_start_habitat`, `upper_bound`, and `lower_bound`, and manually compared known coordinate mappings. I then implemented a `set_alignment()` function to dynamically estimate the affine transformation between the two coordinate systems, which successfully resolved the issue.

This experience shows that AI is effective for local debugging but may fail on system-level structural problems.

---

## Section 3: Engineering Validation

To evaluate the effectiveness of the proposed VLFM improvement, I designed a systematic ablation study with five configurations:

| Experiment | α | β | γ | Description |
|------------|---|---|---|------------|
| E0 Baseline | 1.0 | 0.0 | 0.0 | Original VLFM |
| E1 Full Model | 0.4 | 0.6 | 0.3 | Full hybrid scoring |
| E2 w/o \( S_{global} \) | 0.4 | 0.0 | 0.3 | Without semantic gravity field |
| E3 w/o \( S_{cost} \) | 0.4 | 0.6 | 0.0 | Without distance penalty |
| E4 Local Only | 0.4 | 0.0 | 0.0 | Only scaled local score |

All experiments were conducted on the same HM3D validation scenes. Evaluation metrics included Success Rate and SPL to ensure fair comparison.

I controlled all other variables and only modified the scoring components ( $S_{local}$ , $S_{global}$ , $S_{cost}$ ), ensuring that performance differences came from the proposed method itself.

In addition to quantitative evaluation, I reviewed six-panel visualization videos for each episode, including value maps, semantic gravity maps, and agent trajectories. This helped verify whether the agent behavior aligned with expectations.

---

## Section 4: Problem-Solving Process

A key technical challenge in this project was the design of normalization bounds for the Semantic Gravity Field.

### Problem Description

In the `set_target()` function, I computed similarity scores by taking the dot product between LSeg features and CLIP text embeddings, then normalized them to $[0,1]$ to form the gravity field.

Initially, I used percentile-based normalization:  
- Lower bound: p30 (to filter low-confidence noise)  
- Upper bound: p99 (to avoid extreme outliers)  

### Observed Problem

The resulting heatmap appeared visually flat, with insufficient contrast between target and background regions. As a result, the agent could not effectively distinguish high-priority frontiers.

### Diagnosis

By analyzing distribution statistics (mean, max, p95, p99), I found that the p30 lower bound was already appropriate. However, the p99 upper bound compressed the dynamic range of high-response regions, reducing contrast.

### AI Role and Limitation

AI suggested adjusting percentiles (e.g., using p95 or p90). While directionally reasonable, these changes did not solve the issue because they did not preserve peak responses.

### Final Solution

I kept the p30 lower bound unchanged and replaced the upper bound with the maximum value.

This preserved the full dynamic range of high-response regions, significantly improving heatmap contrast. As a result, $S_{global}$ became more effective, and the agent showed clearer directional exploration behavior.

This demonstrates that normalization must be adapted to data distribution rather than relying on generic percentile rules.

---

## Section 5: Learning Growth

Before this course, I had very limited knowledge of Embodied AI and Vision-Language Navigation.

Through this project, I can now clearly describe the full VLFM pipeline, including frontier extraction, ITM scoring, and action selection. I also understand how semantic priors improve exploration by introducing directional bias.

On the engineering side, I built a complete Ubuntu + conda simulation environment and resolved issues such as CUDA conflicts and dataset setup.

The biggest improvement is my transition from simply running code to understanding and improving system behavior.

---

## Section 6: Critical Reflection

AI significantly improved my efficiency in debugging and understanding code.

However, I realized that its usefulness depends heavily on the quality of my input. Vague descriptions lead to generic answers, while detailed information leads to meaningful guidance.

I also noticed that I sometimes relied too much on AI-suggested parameters without fully understanding them.

In the future, I will:
- Verify AI suggestions before applying them  
- Perform reasoning and estimation before experiments  
- Treat AI as a tool, not a decision-maker  

Overall, AI is a powerful assistant, but reliable results still depend on human reasoning and validation.

---

## Section 7: Evidence (Optional)

Detailed implementation, code modifications, and experiment results can be found in the project repository.

Please refer to the GitHub repository for full details.

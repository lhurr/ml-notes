
## 0. Frontier Video World Models from first principles

A **video world model** refers to:

> A video-based generative model that can synthesize controllable, navigable, persistent, or interactive environments, ideally supporting action-conditioned rollout, camera navigation, spatial memory, physical consistency, and 3D consistency.

The focus is the transition from **video generation** to **interactive / navigable / persistent world simulation**.

### Labels

| Label | Meaning |
| --- | --- |
| `A` | Recommended first |
| `B` | Optional reading for a broader understanding |
| `C` | Background knowledge or an adjacent direction |
| `Code` | The official repository provides implementation code |
| `Weights` | The official repository or Hugging Face page provides model weights |
| `Data` | The official page provides a dataset or data collection tools |
| `Demo` | An online experience or showcase only; this does not imply open source |
| `Closed` | No public code, weights, or dataset have been verified |
| `Frontier` | A recent preprint or newly released project as of 2026-05-30 |

## 1.  Initial Points

| Priority | Resource | Type | Why read it |
| --- | --- | --- | --- |
| A | [Awesome From Video Generation to World Model](https://github.com/ziqihuangg/Awesome-From-Video-Generation-to-World-Model/) | Awesome list | Video foundation models to faithfulness, interaction, planning, datasets, and evaluation. |
| A | [CVPR 2025 Tutorial: From Video Generation to World Model](https://world-model-tutorial.github.io/) | Tutorial | To build a map of the field. |
| A | [Simulating the Visual World with Artificial Intelligence: A Roadmap](https://arxiv.org/abs/2511.08585) | Roadmap / Survey | A systematic overview of the capabilities, data, and evaluation problems for interactive visual world simulators. |
| A | [Genie: Generative Interactive Environments](https://arxiv.org/abs/2402.15391) | Paper | An important starting point for video-based interactive world models. |
| A | [Genie 3: A New Frontier for World Models](https://deepmind.google/discover/blog/genie-3-a-new-frontier-for-world-models/) | Official blog / `Closed` | Understand the target form of a real-time interactive video world model. |
| A | [Marble](https://www.worldlabs.ai/blog/marble-world-model) | Official blog / `Closed` | Contrast with Genie 3 and understand the explorable, persistent 3D world direction. |
| B | [Awesome Video World Models](https://github.com/hit-perfect/Awesome-Video-World-Models) | Awesome list | Additional recent projects, surveys, datasets, and benchmarks. |

## 2. Two-Week Core Study

**How does a model control rollouts, maintain 3D consistency, handle memory, and evaluate failure modes?**

| Day | Core reading | Questions to answer |
| --- | --- | --- |
| 1 | [Awesome list](https://github.com/ziqihuangg/Awesome-From-Video-Generation-to-World-Model/), [Roadmap](https://arxiv.org/abs/2511.08585), [Genie](https://arxiv.org/abs/2402.15391) | Where is the boundary between a video generator and a world model? How can latent actions be learned from unlabeled videos? |
| 2 | [Genie 2](https://deepmind.google/discover/blog/genie-2-a-large-scale-foundation-world-model/), [Genie 3](https://deepmind.google/discover/blog/genie-3-a-new-frontier-for-world-models/), [GameNGen](https://arxiv.org/abs/2408.14837), [Oasis](https://oasis-model.github.io/) | What latency, autoregressive drift, and action alignment problems must be solved for real-time interaction? |
| 3 | [MineWorld](https://arxiv.org/abs/2504.08388), [Matrix-Game 2.0](https://arxiv.org/abs/2508.13009), [Matrix-Game 3.0](https://arxiv.org/abs/2604.08995), [WorldPlay](https://arxiv.org/abs/2512.14614) | Why do long rollouts forget scenes? Where should explicit or implicit memory be stored? |
| 4 | [Marble](https://www.worldlabs.ai/blog/marble-world-model), [HY-World 2.0](https://arxiv.org/abs/2604.14268), [HunyuanWorld-Voyager](https://arxiv.org/abs/2506.04225) | Is the output a video or an explorable 3D world? How does 3D state support navigation, generation, and export? |
| 5 | [GEN3C](https://arxiv.org/abs/2503.03751), [WorldStereo](https://arxiv.org/abs/2603.02049), [Beyond Pixel Histories](https://arxiv.org/abs/2603.03482) | How do camera motion, geometric representations, and persistent 3D state enter the video generation pipeline? |
| 6 | [VMem](https://arxiv.org/abs/2506.18903), [RELIC](https://arxiv.org/abs/2512.04040), [WorldKV](https://arxiv.org/abs/2605.22718), [HyDRA / HM-World](https://arxiv.org/abs/2603.25716) | What forms of memory are needed for revisiting scenes, reappearance after occlusion, dynamic subjects, and static backgrounds? |
| 7 | [MultiWorld](https://arxiv.org/abs/2604.18564), [Solaris](https://arxiv.org/abs/2602.22208), [Gamma-World](https://arxiv.org/abs/2605.28816) | What new constraints arise from multiple agents, multiple views, and a shared world state? |
| 8 | [WorldMark](https://arxiv.org/abs/2604.21686), [WBench](https://arxiv.org/abs/2605.25874), [MIND](https://arxiv.org/abs/2602.08025), [WorldScore](https://arxiv.org/abs/2504.00983) | How should visual quality, control alignment, spatial memory, and physical consistency be evaluated separately? |
| 9 | [DDPM](https://arxiv.org/abs/2006.11239), [LDM](https://arxiv.org/abs/2112.10752), [DiT](https://arxiv.org/abs/2212.09748), [SVD](https://arxiv.org/abs/2311.15127), [Wan](https://arxiv.org/abs/2503.20314), [CausVid](https://arxiv.org/abs/2412.07772), [Causal Forcing](https://arxiv.org/abs/2602.02214) | What is the relationship between diffusion, latent video models, causal generation, and real-time rollouts? |
| 10 | [minWM](https://arxiv.org/abs/2605.30263) and the Day 8 benchmarks | Build a benchmark, a dataset, or a lightweight 3D constraint for an open model? |


## 3. Overview: Representative Systems

| Priority | Year / Status | Paper or official page | Availability | Why read it |
| --- | --- | --- | --- | --- |
| A | 2024 | [Genie: Generative Interactive Environments](https://arxiv.org/abs/2402.15391) | Paper; full release not verified | Learn how latent actions can be learned from unlabeled videos and understand the foundations of interactive environment generation. |
| A | 2024 | [Genie 2](https://deepmind.google/discover/blog/genie-2-a-large-scale-foundation-world-model/) | Official blog; `Closed` | Generates interactive 3D-like worlds from a single image and directly precedes Genie 3. |
| A | 2025 | [Genie 3](https://deepmind.google/discover/blog/genie-3-a-new-frontier-for-world-models/) | Official blog; `Closed` | Understand the product target: 720p, real-time navigation, and minute-scale consistency. |
| A | 2024 | [GameNGen: Diffusion Models Are Real-Time Game Engines](https://arxiv.org/abs/2408.14837) | [Project page](https://gamengen.github.io/); training code not verified as public | Uses DOOM to show that a diffusion model can become a real-time game engine. |
| A | 2024 | [Oasis: A Universe in a Transformer](https://oasis-model.github.io/) | [Demo](https://oasis.decart.ai/); training code not verified as public | A representative Minecraft-like real-time interactive video world model. |
| A | 2025 | [MineWorld: a Real-Time and Open-Source Interactive World Model on Minecraft](https://arxiv.org/abs/2504.08388) | [Code](https://github.com/microsoft/mineworld) | Good for understanding the inference pipeline and engineering constraints of a real executable system. |
| A | 2025 | [Matrix-Game 2.0](https://arxiv.org/abs/2508.13009) | [Code / Weights](https://github.com/SkyworkAI/Matrix-Game) | A real-time streaming interactive world model suitable as a baseline. |
| A | 2026 / `Frontier` | [Matrix-Game 3.0](https://arxiv.org/abs/2604.08995) | [Code / Weights](https://github.com/SkyworkAI/Matrix-Game) | Introduces long-horizon memory and is closely related to spatial consistency. |
| A | 2025 | [WorldPlay: Towards Long-Term Geometric Consistency for Real-Time Interactive World Modeling](https://arxiv.org/abs/2512.14614) | [Code / Weights](https://github.com/Tencent-Hunyuan/HY-WorldPlay) | Places real-time interaction and long-term geometric consistency in the same problem setting. |
| A | 2025 | [Marble](https://www.worldlabs.ai/blog/marble-world-model) | [World API](https://www.worldlabs.ai/blog/announcing-the-world-api); `Closed` | An important product anchor for 3D world generation that creates persistent, explorable worlds. |
| A | 2026 / `Frontier` | [HY-World 2.0](https://arxiv.org/abs/2604.14268) | [Code / Weights](https://github.com/Tencent-Hunyuan/HY-World-2.0) | Unifies 3D world reconstruction, generation, and simulation. The official repository provides inference code and WorldStereo 2.0 weights. |
| A | 2026 / `Frontier` | [minWM: A Minimal but Complete World Model Framework](https://arxiv.org/abs/2605.30263) | [Code / Weights](https://github.com/shengshu-ai/minWM) | A small but complete open-source tutorial framework |

## 4. Strong 3D / Spatially Consistent World Models

Explicit 3D state, camera control, geometry memory, scene revisiting, and long-horizon consistency**.

| Priority | Year / Status | Paper or official page | Availability | Why read it |
| --- | --- | --- | --- | --- |
| A | 2025 | [Marble](https://www.worldlabs.ai/blog/marble-world-model) | Official product page; `Closed` | Understand the goal of persistent 3D worlds that can be explored, edited, and exported. |
| A | 2026 / `Frontier` | [HY-World 2.0](https://arxiv.org/abs/2604.14268) | [Code / Weights](https://github.com/Tencent-Hunyuan/HY-World-2.0) | A unified multimodal system for 3D world reconstruction, generation, and simulation. |
| A | 2025 | [HunyuanWorld-Voyager](https://arxiv.org/abs/2506.04225) | [Code / Weights](https://github.com/Tencent-Hunyuan/HunyuanWorld-Voyager) | Generates explorable, 3D-consistent worlds from a single image and helps explain world-consistent video diffusion. |
| A | 2026 / `Frontier` | [Lyra 2.0: Explorable Generative 3D Worlds](https://arxiv.org/abs/2604.13036) | Paper; public code not verified | Uses per-frame 3D geometry for information routing to address spatial forgetting and temporal drift. |
| B | 2026 / `Frontier` | [OmniRoam: World Wandering via Long-Horizon Panoramic Video Generation](https://arxiv.org/abs/2603.30045) | [Code](https://github.com/yuhengliu02/OmniRoam) | Uses panoramic representations to improve coverage and consistency for large-scale wandering. |
| A | 2025 | [GEN3C: 3D-Informed World-Consistent Video Generation with Precise Camera Control](https://arxiv.org/abs/2503.03751) | [Project page](https://research.nvidia.com/labs/toronto-ai/GEN3C/) | Learn how a 3D cache, camera control, and world-consistent generation can work together. |
| A | 2025 | [Geometry Forcing: Marrying Video Diffusion and 3D Representation for Consistent World Modeling](https://arxiv.org/abs/2507.07982) | [Project page](https://geometryforcing.github.io/) | Combines video generation with online 3D reconstruction and directly addresses 3D consistency. |
| A | 2026 / `Frontier` | [Beyond Pixel Histories: World Models with Persistent 3D State](https://arxiv.org/abs/2603.03482) | [Project page / Code](https://francelico.github.io/persist.github.io/) | PERSIST replaces pixel-only history with persistent, updatable 3D state. |
| A | 2026 / `Frontier` | [WorldStereo: Bridging Camera-Guided Video Generation and Scene Reconstruction via 3D Geometric Memories](https://arxiv.org/abs/2603.02049) | [Code / Weights](https://github.com/FuchengSu/WorldStereo) | Connects camera-guided generation and reconstruction through 3D geometric memory. |
| A | 2025 | [WorldWarp: Propagating 3D Geometry with Asynchronous Video Diffusion](https://arxiv.org/abs/2512.19678) | [Weights](https://huggingface.co/imsuperkong/worldwarp) | Shows how explicit 3D geometry propagation can reduce long-horizon drift. |
| B | 2026 / `Frontier` | [WorldAct: Activating Monolithic 3D Worlds into Interactive-Ready Object-Centric Scenes](https://arxiv.org/abs/2605.15843) | [Project page](https://sjtu-deepvisionlab.github.io/WorldAct/) | Converts static monolithic 3D worlds into editable, collision-aware, interaction-ready scenes. |

## 5. Strong Interaction / Action-Conditioned World Models

| Priority | Year / Status | Paper or official page | Availability | Why read it |
| --- | --- | --- | --- | --- |
| A | 2025 | [Genie 3](https://deepmind.google/discover/blog/genie-3-a-new-frontier-for-world-models/) | Official blog; `Closed` | An important current reference point for real-time interactive video world models. |
| A | 2024 | [GameNGen](https://arxiv.org/abs/2408.14837) | [Project page](https://gamengen.github.io/) | Study action-conditioned diffusion rollouts and real-time inference. |
| A | 2024 | [Oasis](https://oasis-model.github.io/) | [Demo](https://oasis.decart.ai/) | Observe keyboard and mouse interaction, visual feedback, and error accumulation in open scenes. |
| A | 2025 | [GameFactory: Creating New Games with Generative Interactive Videos](https://arxiv.org/abs/2501.08325) | [Data](https://huggingface.co/datasets/KlingTeam/GameFactory-Dataset/tree/main/GF-Minecraft) | Study interactive video generation for new game scenes using Minecraft data. |
| A | 2025 | [MineWorld](https://arxiv.org/abs/2504.08388) | [Code](https://github.com/microsoft/mineworld) | An open-source real-time Minecraft world model suitable for reading and execution. |
| A | 2026 / `Frontier` | [Matrix-Game 3.0](https://arxiv.org/abs/2604.08995) | [Code / Weights](https://github.com/SkyworkAI/Matrix-Game) | An open baseline for streaming action control and long-horizon memory. |
| A | 2025 | [WorldPlay](https://arxiv.org/abs/2512.14614) | [Code / Weights](https://github.com/Tencent-Hunyuan/HY-WorldPlay) | A real-time interactive world model driven by geometric consistency. |
| A | 2025 | [RELIC: Interactive Video World Model with Long-Horizon Memory](https://arxiv.org/abs/2512.04040) | Paper; public code not verified | Studies long-horizon memory in interactive rollouts. |
| B | 2025 | [Hunyuan-GameCraft-2](https://arxiv.org/abs/2511.23429) | [Project page](https://hunyuan-gamecraft-2.github.io/); public code not verified | An instruction-following game world model supporting text, keyboard, and mouse interaction. |
| A | 2026 / `Frontier` | [minWM](https://arxiv.org/abs/2605.30263) | [Code / Weights](https://github.com/shengshu-ai/minWM) | Minimal footprint makes it a good first system to read through or reproduce end-to-end. |
| B | 2026 / `Frontier` | [ReactiveGWM](https://arxiv.org/abs/2605.15256) | [Code](https://github.com/INV-WZQ/ReactiveGWM) | Focuses on interactive geometric world modeling and real-time responsiveness. |

## 6. Memory in World Models

Does **layout, object identity, dynamic state, and motion continuity** survive when an agent leaves an area and returns later?

| Priority | Year / Status | Paper or official page | Availability | Why read it |
| --- | --- | --- | --- | --- |
| A | 2025 | [WorldPlay](https://arxiv.org/abs/2512.14614) | [Code / Weights](https://github.com/Tencent-Hunyuan/HY-WorldPlay) | Constrains real-time long-horizon exploration with geometric consistency. |
| A | 2026 / `Frontier` | [Matrix-Game 3.0](https://arxiv.org/abs/2604.08995) | [Code / Weights](https://github.com/SkyworkAI/Matrix-Game) | A real-time streaming system explicitly designed for long-horizon memory. |
| A | 2025 | [RELIC](https://arxiv.org/abs/2512.04040) | Paper; public code not verified | Directly studies long-horizon memory in interactive world models. |
| A | 2025 | [VMem: Consistent Interactive Video Scene Generation with Surfel-Indexed View Memory](https://arxiv.org/abs/2506.18903) | Paper; public code not verified | Surfel-indexed view memory is useful for reasoning about revisit consistency. |
| A | 2026 | [Spatia: Video Generation with Updatable Spatial Memory](https://openaccess.thecvf.com/content/CVPR2026/html/Zhao_Spatia_Video_Generation_with_Updatable_Spatial_Memory_CVPR_2026_paper.html) | CVPR 2026 paper; public code not verified | Learn how updatable spatial memory can be represented and written. |
| A | 2025 | [Context as Memory: Scene-Consistent Interactive Long Video Generation with Memory Retrieval](https://arxiv.org/abs/2506.03141) | [Project page / Data](https://context-as-memory.github.io/) | Uses memory retrieval to support scene-consistent interactive long video generation. |
| A | 2025 | [Video World Models with Long-term Spatial Memory](https://arxiv.org/abs/2506.05284) | [Code](https://github.com/spmem/spmem) | Uses long-term spatial memory to address spatial forgetting during long-horizon navigation. |
| A | 2026 | [WorldMem: Long-term Consistent World Simulation with Memory](https://openreview.net/forum?id=vGSz5ejDHH) | OpenReview paper; public code not verified | Focuses on long-term consistency in world simulation. |
| A | 2026 / `Frontier` | [Out of Sight but Not Out of Mind: Hybrid Memory for Dynamic Video World Models](https://arxiv.org/abs/2603.25716) | [Code / Project](https://github.com/H-EmbodVis/HyDRA) | HyDRA and HM-World distinguish static background memory from dynamic subject tracking. |
| A | 2026 / `Frontier` | [WorldKV: Efficient World Models with KV Cache Memory](https://arxiv.org/abs/2605.22718) | [Code](https://github.com/cvlab-kaist/WorldKV) | Studies efficient long-horizon world model memory using a KV cache. |
| A | 2026 / `Frontier` | [Beyond Pixel Histories](https://arxiv.org/abs/2603.03482) | [Project page / Code](https://francelico.github.io/persist.github.io/) | Replaces pixel-only history with persistent 3D state. |

## 7. Multiplayer / Multi-Agent World Models


| Priority | Year / Status | Paper or official page | Availability | Why read it |
| --- | --- | --- | --- | --- |
| A | 2026 / `Frontier` | [MultiWorld: Scalable Multi-Agent Multi-View Video World Models](https://arxiv.org/abs/2604.18564) | Paper; public code not verified | Unifies multi-agent control, multi-view consistency, and parallel generation. |
| A | 2026 / `Frontier` | [Solaris: Building a Multiplayer Video World Model in Minecraft](https://arxiv.org/abs/2602.22208) | [Code / Data collection system](https://github.com/solaris-wm/Solaris) | Provides a scalable data collection and training framework for shared multiplayer Minecraft worlds. |
| A | 2026 / `Frontier` | [Gamma-World: Large-Scale Multiplayer Video World Modeling](https://arxiv.org/abs/2605.28816) | [Project page](https://research.nvidia.com/labs/sil/projects/gamma-world/); public code not verified | Studies large-scale multiplayer world simulation and consistency across players. |
| B | 2026 / `Frontier` | [ShareVerse: Multi-Agent Consistent Video Generation for Shared World Modeling](https://arxiv.org/abs/2603.02697) | Paper; public code not verified | Uses cross-agent attention to model consistency in shared multi-agent worlds. |

### Adjacent Systems

| Priority | Year / Status | Resource | Availability | Why retain it |
| --- | --- | --- | --- | --- |
| B | 2026 | [Multiverse](https://huggingface.co/Enigma-AI/multiverse) | [Weights / Code / Data](https://huggingface.co/Enigma-AI/multiverse) | An open two-player multiplayer world model useful for understanding a minimal shared-state system. |
| B | 2026 / `Frontier` | [ReactiveGWM](https://arxiv.org/abs/2605.15256) | [Code](https://github.com/INV-WZQ/ReactiveGWM) | Geometric interaction rather than strict multiplayer modeling, but useful as a technical reference for shared scene state. |

## 8. Common Datasets and Data Resources

`Public download` means an official data entry point has been verified. `Application required` means official data exists but requires a license or registration workflow. `Paper description only` means the data cannot be assumed to be directly downloadable.

### 8.1 Interactive / Game / Memory

| Priority | Dataset or resource | Access status | Official entry point | Suitable use |
| --- | --- | --- | --- | --- |
| A | Matrix-Game-MC | Follow the Hugging Face link on the official page | [Matrix-Game Homepage](https://matrix-game-homepage.github.io/) | Minecraft action-conditioned rollout and benchmarking. |
| A | GF-Minecraft | `Public download` | [Hugging Face](https://huggingface.co/datasets/KlingTeam/GameFactory-Dataset/tree/main/GF-Minecraft) | Scene-action-video data for interactive model training and evaluation. |
| A | Solaris multiplayer collection system | `Code`; follow the repository instructions for data | [Solaris GitHub](https://github.com/solaris-wm/Solaris) | Multiplayer Minecraft data collection and training pipeline. |
| A | MIND | `Public download` | [Hugging Face](https://huggingface.co/datasets/CSU-JPG/MIND) | Evaluating memory consistency and action control in world models. |
| A | HM-World | `Public download` | [Hugging Face](https://huggingface.co/datasets/KlingTeam/HM-World) | Evaluating hybrid memory when dynamic subjects leave the field of view and reappear later. |
| A | WBench | `Public benchmark` | [GitHub](https://github.com/meituan-longcat/WBench) | Multi-turn evaluation of interactive video world models. |
| A | WorldModelBench | `Public benchmark` | [GitHub](https://github.com/WorldModelBench-Team/WorldModelBench) | Instruction following and physics adherence evaluation. |

### 8.2 3D / Multiview / Scene Prior

| Priority | Dataset | Access status | Official entry point | Suitable use |
| --- | --- | --- | --- | --- |
| A | DL3DV-10K | `Public download` | [Project](https://dl3dv-10k.github.io/DL3DV-10K/) | Large-scale real-world multiview data for 3D consistency and view synthesis. |
| A | RealEstate10K | `Public list` | [Project](https://google.github.io/realestate10k/) | Camera trajectories, indoor and outdoor scenes, and view synthesis. |
| A | Kubric | `Code`; programmatic generation | [GitHub](https://github.com/google-research/kubric) | Generating controlled synthetic data with geometry, motion, and physics annotations. |
| A | Objaverse / Objaverse-XL | `Public download` | [Objaverse](https://objaverse.allenai.org/) | 3D object priors, asset generation, and scene construction. |
| B | ScanNet | `Application required` | [Official site](http://www.scan-net.org/) | Indoor RGB-D, poses, semantic labels, and 3D reconstruction. |
| B | Matterport3D | `Application required` | [Official page](https://niessner.github.io/Matterport/) | Indoor panoramas, navigation, and 3D scene understanding. |

### 8.3 Driving

| Dataset | Official entry point | Use |
| --- | --- | --- |
| nuScenes | [Official site](https://www.nuscenes.org/) | Multisensor driving scenes, BEV, and 3D tracking. |
| Waymo Open Dataset | [Official site](https://waymo.com/open/) | Large-scale driving data. |
| nuPlan | [Official site](https://www.nuscenes.org/nuplan) | Planning and ego trajectories. |
| Argoverse 2 | [Official site](https://www.argoverse.org/av2.html) | Trajectories, maps, and 3D scene understanding. |

## 9. Surveys / Tutorials / Awesome Lists

| Priority | Year | Resource | Type | Why read it |
| --- | --- | --- | --- | --- |
| A | 2025 | [Simulating the Visual World with Artificial Intelligence: A Roadmap](https://arxiv.org/abs/2511.08585) | Roadmap | Main survey: a systematic map of visual world simulation. |
| A | 2025 | [3D and 4D World Modeling: A Survey](https://arxiv.org/abs/2509.07996) | Survey | Background on 3D / 4D world representations. |
| B | 2026 | [World Models for Robot Learning: A Survey](https://arxiv.org/abs/2605.00080) | Survey | Understand the role of world models in robot learning. |
| A | 2026 | [Video Generation Models as World Models: A New and Efficient Paradigm for Embodied Learning](https://arxiv.org/abs/2603.28489) | Survey | Clearly explains the route from video generators to world models. |
| A | 2025 | [From Video Generation to World Model](https://world-model-tutorial.github.io/) | CVPR tutorial | A quick introduction. |
| A | Continuously updated | [Awesome From Video Generation to World Model](https://github.com/ziqihuangg/Awesome-From-Video-Generation-to-World-Model/) | Awesome list | Main entry point and monthly update source. |
| B | Continuously updated | [Awesome Video World Models](https://github.com/hit-perfect/Awesome-Video-World-Models) | Awesome list | Additional recent papers, datasets, and benchmarks. |

## 10. Evaluation / Benchmarks

In 3D consistency, we separate evaluation into:

1. **Visual quality**: visual quality, video stability, and naturalness.
2. **Control alignment**: whether WASD controls, camera trajectories, actions, and visual feedback agree.
3. **Spatial consistency**: whether layout, depth, camera pose, reconstruction, and revisited scenes remain consistent.
4. **Object permanence**: whether identity and motion remain continuous when a subject leaves the field of view and reappears.
5. **Physics**: whether collisions, occlusions, gravity, scale changes, and causal relationships are reasonable.

| Priority | Year / Status | Benchmark | Availability | Main evaluation dimensions |
| --- | --- | --- | --- | --- |
| A | 2026 / `Frontier` | [WBench: A Comprehensive Multi-turn Benchmark for Interactive Video World Model Evaluation](https://arxiv.org/abs/2605.25874) | [Code / Data](https://github.com/meituan-longcat/WBench) | 289 cases, 1,058 turns, five dimensions, and 22 automated metrics. |
| A | 2026 / `Frontier` | [WorldMark: A Unified Benchmark Suite for Interactive Video World Models](https://arxiv.org/abs/2604.21686) | Paper; check the project page for releases | Unified action interface, visual quality, control alignment, and world consistency. |
| A | 2026 / `Frontier` | [MIND: Benchmarking Memory Consistency and Action Control in World Models](https://arxiv.org/abs/2602.08025) | [Data](https://huggingface.co/datasets/CSU-JPG/MIND) | Memory consistency and action control. |
| A | 2026 / `Frontier` | [iWorld-Bench: A Benchmark for Interactive World Models with a Unified Action Generation Framework](https://arxiv.org/abs/2605.03941) | [Project page](https://iworld-bench.com/) | Comprehensive interactive world model evaluation. |
| A | 2025 | [WorldModelBench: Judging Video Generation Models As World Models](https://arxiv.org/abs/2502.20694) | [Code](https://github.com/WorldModelBench-Team/WorldModelBench) | Instruction following and physics adherence. |
| A | 2025 | [WorldScore: A Unified Evaluation Benchmark for World Generation](https://arxiv.org/abs/2504.00983) | [Project page](https://haoyi-duan.github.io/WorldScore/) | Unified evaluation of world generation. |
| A | 2025 | [WorldBench: How Close are World Models to the Physical World?](https://arxiv.org/abs/2506.21689) | [Project](https://world-bench.github.io/) | Physical laws and fine-grained physical concepts. |
| B | 2025 | [GameWorld Score](https://arxiv.org/abs/2506.18701) | [Code / Benchmark](https://github.com/SkyworkAI/Matrix-Game) | Evaluation of interactive game worlds proposed by Matrix-Game. |
| B | 2025 | [4DWorldBench: Evaluating Scene Reconstruction Models as World Models](https://arxiv.org/abs/2508.07505) | [Project](https://yeppp27.github.io/4DWorldBench.github.io/) | 4D scene reconstruction, alignment, and realism. |
| B | 2024 | [PhyGenBench / PhyGenEval](https://arxiv.org/abs/2410.05363) | [Project](https://phygenbench123.github.io/) | Physical commonsense in generated videos. |
| B | 2026 / `Frontier` | [Physion-Eval: Evaluating Physical Realism in Generated Video via Human Reasoning](https://arxiv.org/abs/2603.19607) | Paper; public code not verified | Evaluates physical realism in videos through human reasoning. |
| B | 2023 | [VBench: Comprehensive Benchmark Suite for Video Generative Models](https://arxiv.org/abs/2311.17982) | [Code](https://github.com/Vchitect/VBench) | Background on general video quality evaluation; insufficient as a standalone world model evaluation. |

## 11. Essential Background


### 11.1 Diffusion and Video Foundation Models

| Priority | Year | Paper or project | Availability | Why retain it |
| --- | --- | --- | --- | --- |
| A | 2020 | [DDPM](https://arxiv.org/abs/2006.11239) | Paper | Diffusion fundamentals. |
| A | 2021 | [Latent Diffusion Models](https://arxiv.org/abs/2112.10752) | [Code](https://github.com/CompVis/latent-diffusion) | Latent diffusion fundamentals. |
| A | 2022 | [DiT: Scalable Diffusion Models with Transformers](https://arxiv.org/abs/2212.09748) | [Code](https://github.com/facebookresearch/DiT) | Transformer diffusion backbone. |
| A | 2023 | [Stable Video Diffusion](https://arxiv.org/abs/2311.15127) | [Code / Weights](https://github.com/Stability-AI/generative-models) | Image-to-video diffusion baseline. |
| B | 2024 | [VideoCrafter2](https://arxiv.org/abs/2401.09047) | [Code / Weights](https://github.com/AILab-CVC/VideoCrafter) | Classic open video diffusion baseline. |
| A | 2025 | [Wan: Open and Advanced Large-Scale Video Generative Models](https://arxiv.org/abs/2503.20314) | [Code / Weights](https://github.com/Wan-Video/Wan2.1) | A commonly used recent open video foundation model. |
| B | 2024 | [HunyuanVideo](https://arxiv.org/abs/2412.03603) | [Code / Weights](https://github.com/Tencent-Hunyuan/HunyuanVideo) | Open DiT video model. |
| B | 2024 | [CogVideoX](https://arxiv.org/abs/2408.06072) | [Code / Weights](https://github.com/THUDM/CogVideo) | Open text-to-video model. |
| B | 2024 | [Open-Sora](https://github.com/hpcaitech/Open-Sora) | [Code / Weights](https://github.com/hpcaitech/Open-Sora) | Engineering reference for open video generation systems. |

### 11.2 Long Video, Causal Generation, and Real-Time Rollouts

| Priority | Year | Paper | Availability | Why retain it |
| --- | --- | --- | --- | --- |
| A | 2024 | [Diffusion Forcing: Next-token Prediction Meets Full-Sequence Diffusion](https://arxiv.org/abs/2407.01392) | [Code](https://github.com/buoyancy99/diffusion-forcing) | Understand the combination of sequence modeling and diffusion. |
| A | 2025 | [Self Forcing: Bridging the Train-Test Gap in Autoregressive Video Diffusion](https://arxiv.org/abs/2506.08009) | [Code](https://github.com/guandeh17/Self-Forcing) | Addresses the train-test gap in autoregressive video diffusion. |
| A | 2024 | [CausVid: From Slow Bidirectional to Fast Autoregressive Video Diffusion Models](https://arxiv.org/abs/2412.07772) | [Code](https://github.com/tianweiy/CausVid) | Fast causal video generation. |
| A | 2026 / `Frontier` | [Causal Forcing](https://arxiv.org/abs/2602.02214) | Paper; check the project page for releases | An important recent work on real-time interactive video generation. |
| B | 2025 | [MAGI-1: Autoregressive Video Generation at Scale](https://arxiv.org/abs/2505.13211) | [Code / Weights](https://github.com/SandAI-org/MAGI-1) | Large-scale autoregressive video generation. |

### 11.3 3D Consistency Tools

| Priority | Year | Paper | Availability | Use case |
| --- | --- | --- | --- | --- |
| A | 2025 | [VGGT: Visual Geometry Grounded Transformer](https://arxiv.org/abs/2503.11651) | [Code / Weights](https://github.com/facebookresearch/vggt) | Estimate cameras, depth, and point maps to build reconstruction consistency metrics. |
| A | 2023 | [DUSt3R: Geometric 3D Vision Made Easy](https://arxiv.org/abs/2312.14132) | [Code / Weights](https://github.com/naver/dust3r) | A foundational tool for 3D reconstruction from uncalibrated image pairs. |
| B | 2024 | [MASt3R](https://arxiv.org/abs/2406.09756) | [Code / Weights](https://github.com/naver/mast3r) | Better suited to matching and localization. |
| B | 2024 | [MonST3R](https://arxiv.org/abs/2410.03825) | [Code / Weights](https://github.com/Junyi42/monst3r) | Geometry estimation for dynamic scenes. |

## 12. Frontier Watchlist

The following items are worth revisiting monthly as of **2026-05-31**. 

| Date | Item | Status | Why track it |
| --- | --- | --- | --- |
| 2026-05 | [minWM](https://arxiv.org/abs/2605.30263) | `Frontier`; [Code / Weights](https://github.com/shengshu-ai/minWM) | A small but complete open-source world model framework — good entry point for hands-on exploration. |
| 2026-05 | [Gamma-World](https://arxiv.org/abs/2605.28816) | `Frontier`; [Project page](https://research.nvidia.com/labs/sil/projects/gamma-world/) | Large-scale multiplayer video world modeling. |
| 2026-05 | [WorldKV](https://arxiv.org/abs/2605.22718) | `Frontier`; [Code](https://github.com/cvlab-kaist/WorldKV) | Efficient KV memory for long-horizon rollouts. |
| 2026-05 | [ReactiveGWM](https://arxiv.org/abs/2605.15256) | `Frontier`; [Code](https://github.com/INV-WZQ/ReactiveGWM) | Interactive geometric world modeling. |
| 2026-05 | [WBench](https://arxiv.org/abs/2605.25874) | `Frontier`; [Code / Data](https://github.com/meituan-longcat/WBench) | A multi-turn benchmark worth checking for direct use in evaluation pipelines. |
| 2026-04 | [HY-World 2.0](https://arxiv.org/abs/2604.14268) | `Frontier`; [Code / Weights](https://github.com/Tencent-Hunyuan/HY-World-2.0) | Unified 3D world reconstruction, generation, and simulation. |
| 2026-04 | [World-R1: Reinforcing 3D Constraints for Text-to-Video Generation](https://arxiv.org/abs/2604.24764) | `Frontier`; Paper | Uses RL to reinforce 3D constraints in text-to-video generation. |
| 2026-03 | [EgoForge: Goal-Directed Egocentric World Simulator](https://arxiv.org/abs/2603.20169) | `Frontier`; Paper | Egocentric, goal-directed rollouts and trajectory-level reward refinement. |
| 2026-04 | [MultiWorld](https://arxiv.org/abs/2604.18564) | `Frontier`; Paper | Multi-agent, multi-view world modeling. |
| 2026-04 | [Lyra 2.0](https://arxiv.org/abs/2604.13036) | `Frontier`; Paper | Persistent explorable 3D worlds with an emphasis on spatial forgetting. |

### Monthly Update Checklist

1. Check new entries in [Awesome From Video Generation to World Model](https://github.com/ziqihuangg/Awesome-From-Video-Generation-to-World-Model/).
2. Revisit the `video world model`, `video generation with 3d`, and `Long Video Generation` collections in Scholar Inbox.
3. Recheck the official GitHub and Hugging Face pages for Frontier items. Code, weights, and datasets are often released several weeks after the paper.
4. Add new papers to Frontier first. Move them into the two-week core path only after stable artifacts are available or they clearly become representative work.

## 14. NUS SoC Compute Tutorial

Official resources:

- [SoC Compute Cluster GPU Guide](https://www.comp.nus.edu.sg/~cs3210/student-guide/soc-gpus/): a public page suitable for first-time users.
- [DocHub Slurm Quickstart](https://dochub.comp.nus.edu.sg/cf/guides/compute-cluster/slurm-quick): detailed documentation available after SoC login.

### 14.1 Before Logging In

1. Create and activate a [SoC UNIX ID](https://mysoc.nus.edu.sg/~newacct/).
2. Connect to the NUS / SoC VPN when working off campus.
3. Log in:

```bash
ssh <soc-id>@xlogin.comp.nus.edu.sg
```

### 14.2 Inspect Resources and Jobs

```bash
# Inspect nodes and GPU resources
sinfo

# Inspect your jobs
squeue -u "$USER"

# Inspect a specific job
scontrol show job <job-id>

# Cancel a job
scancel <job-id>
```

The public guide uses the following GPU Generic Resource names:

| GPU | Slurm GRES name |
| --- | --- |
| NVIDIA H100 96 GB | `h100-96` |
| NVIDIA A100 80 GB | `a100-80` |
| NVIDIA A100 40 GB | `a100-40` |

### 14.3 Request an Interactive GPU Shell

```bash
srun --gpus=a100-80 --time=00:30:00 --pty bash
nvidia-smi
```

Exit after debugging:

```bash
exit
```

Do not run GPU training directly on a login node.

### 14.4 Minimal sbatch Template

Save as `run_gpu.sh`:

```bash
#!/usr/bin/env bash
#SBATCH --job-name=wm-smoke-test
#SBATCH --gpus=a100-80
#SBATCH --time=00:30:00
#SBATCH --output=wm-%j.out

set -euo pipefail

nvidia-smi
python train.py
```

Submit and inspect:

```bash
sbatch run_gpu.sh
squeue -u "$USER"
```

This template does not hard-code a partition or account. Follow the group-specific configuration and DocHub if your research group provides additional instructions.

### 14.5 Use GitHub SSH over Port 443

If the cluster cannot connect to GitHub over SSH port 22, add the following to `~/.ssh/config`:

```sshconfig
Host github.com
  HostName ssh.github.com
  Port 443
  User git
```

Test the connection:

```bash
ssh -T git@github.com
```

### 14.6 VS Code Remote SSH

Add the following to your local `~/.ssh/config`:

```sshconfig
Host nus-soc-cluster
  HostName xlogin.comp.nus.edu.sg
  User <soc-id>
```

Install **Remote - SSH** in VS Code and connect to `nus-soc-cluster`. If the connection hangs, first confirm that ordinary `ssh` and the VPN work in a terminal.

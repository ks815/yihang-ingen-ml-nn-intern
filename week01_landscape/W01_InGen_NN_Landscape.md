# W01: InGen Dynamics Product and PIC 2.0 Landscape

**Sources:** Public pages from `ingendynamics.com`, including the homepage and the pages for Fari, Senpai, Sentinel Prime AI, Aido Rover, Aido Humanoid, and Origami AI. Accessed July 14 and 15, 2026.

## Scope and source note

InGen describes itself as a company that is still in development. Many of the performance numbers on its website, including latency, accuracy, and hardware targets, are presented as design goals rather than independently validated commercial results. I therefore treat these numbers as stated targets throughout this brief.

The public product pages and the internship Concepts Primer do not always use the PIC 2.0 model names in the same way. In the product sections, I describe what the public pages say. In the final architecture section and the OpenClaw bridge, I follow the Concepts Primer's working definitions because those are the terms used for this internship assignment.

## 1. Fari: Eldercare Companion Robot

Fari is designed as an eldercare companion robot for care homes, assisted living facilities, and home care settings. The main idea is to combine continuous health monitoring with social interaction, especially when a caregiver cannot be present at all times.

**Target use case:** Fari supports noninvasive health monitoring, fall risk awareness, medication reminders, mobility support, and daily companionship for older adults.

**Primary ML inference type:** Fari appears to use a combination of **classification** and **generation**. On the classification side, it monitors signals such as heart rate, SpO₂, respiratory rate, skin temperature, and estimated blood pressure. It also uses gait and sensor data to identify possible fall risks or signs of deterioration. On the generation side, it provides conversational interaction and is intended to adjust its responses to the resident’s emotional state and communication style.

The public page also describes local processing on a Jetson Orin NX, together with LiDAR, depth cameras, ultrasonic sensors, and a SLAM-based navigation system. This suggests that Fari is designed to handle both dialogue and physical navigation without depending completely on cloud services.

**Key constraint:** The most important constraint is **safety**. Fari works with a vulnerable population and deals with health-related information, so the system must avoid acting like a medical professional. InGen states that the robot should not diagnose, prescribe, or replace qualified caregivers. The public Fari page also gives a safety setting labeled SEOM λ=10.0, along with targets such as sending fall alerts to caregivers in under 3.2 seconds and visually confirming medication intake before recording a dose as completed.

**Terminology note:** The use of SEOM, AMDC, HTD-IRL, and STUM on the public product pages does not fully match the internship Concepts Primer. I therefore report the wording used on the product page here without treating it as the official Week 1 definition of those models. For the bridge document, I use the Primer's mappings and label them as hypotheses.

## 2. Senpai: AI Educational Companion

Senpai is InGen’s educational robot. It is designed for K-12 education, early childhood learning, language learning, and students with special educational needs and disabilities. The product is presented as a middle ground between scalable education software and individualized teaching.

**Target use case:** Senpai provides adaptive tutoring, classroom support, language practice, and personalized learning activities. The public material lists use cases in K-12 classrooms, SEND education, early childhood, language learning, government training, and higher education.

**Primary ML inference type:** Senpai mainly combines **generation**, **retrieval**, and **classification**. Generation is needed for tutoring dialogue and multilingual interaction. Retrieval is needed to connect responses to curriculum content instead of relying only on the model’s internal memory. Classification is used to estimate a student’s current understanding and decide what activity or question should come next.

One part I found especially important is the claim that Senpai should distinguish genuine understanding from a lucky correct answer. The website mentions signals such as response time and performance on transfer questions. This means the system is not only generating answers; it is also maintaining an evolving model of the learner’s knowledge.

The platform is described as supporting more than 40 languages and using an knowledge graph with more than 8,000 concepts. It also includes a display and projector so learning content can be shown on nearby surfaces.

**Key constraint:** The main constraints are **groundedness**, **interaction that is appropriate for the student's age**, and **accessibility**. Senpai’s educational answers need to stay aligned with reliable learning materials and curriculum standards. The website also lists six SEND adaptation modes: Autism Spectrum, Dyslexia, ADHD, Visual Impairment, Hearing Impairment, and Physical Disability. These are presented as design requirements rather than later additions.

The public Senpai page states a safety setting labeled SEOM λ=4.0 and lists 12 safety rules. Because the Concepts Primer uses SEOM as a likely semantic or embedding-oriented model, I treat this as product-page terminology rather than using it to define SEOM in the Week 1 architecture mapping.

## 3. Sentinel Prime AI: Enterprise Physical Security

Sentinel Prime AI is InGen’s physical security platform. Among the five products, it has the most detailed public description of its sensing, classification, uncertainty handling, and response process. It is also the product that InGen describes as operational rather than only under development.

**Target use case:** Sentinel is intended for security operations centers and supervisors in areas such as critical infrastructure, healthcare, retail, government, and transportation. The website lists Tower, Compact, Mobile, and Covert hardware versions for different environments.

**Primary ML inference type:** The main inference type is **classification**. Sentinel is designed to run 27 detection functions across threat detection, identity, environmental safety, operations, and behavior analysis. The public page describes a four-stage pipeline:

1. **Sense:** Combine and synchronize data from multiple sensors.
2. **Classify:** Run a ensemble of specialist models across RGB, thermal, pose, and acoustic inputs.
3. **Gate:** Use uncertainty estimation to suppress results with low confidence.
4. **Act:** Pass approved outputs through a safety and governance layer before an alert or action is produced.

The Sentinel page labels parts of this pipeline with AMDC, STUM, and SEOM. However, those product-page labels differ from the Concepts Primer's working definitions. For this reason, I use the Sense, Classify, Gate, and Act flow to describe Sentinel's product behavior, but I do not use it as the final definition of the three PIC 2.0 models.

**Key constraint:** The two main constraints are **low latency** and **reducing incorrect alerts**. Security systems become less useful when operators receive too many incorrect alerts. InGen states design targets of below 3% incorrect alerts and under 200 milliseconds of total latency.

The uncertainty gate is important here because the system is designed to remain silent when confidence is too low instead of forwarding every detection. The public page also states that 10 safety rules, labeled there under SEOM, are enforced at the FPGA hardware level. The overall principle is that the system advises while a human remains responsible for the final decision.

The 27 detection functions cover several categories, including weapons and forced entry, face and license plate recognition, fire and air quality anomalies, abandoned objects and drones, and behaviors such as falls, queues, or possible contraband. Running these functions together on an edge device explains why the system uses several specialist models rather than one general model.

## 4. Aido Rover: Autonomous Outdoor Patrol and Inspection

Aido Rover extends the security and inspection use case into outdoor environments where fixed cameras or frequent human patrols may not be practical.

**Target use case:** The Rover is aimed at perimeter patrol, infrastructure inspection, and environmental monitoring. The public material names solar and wind farms, airports, agriculture, government and defense sites, corporate campuses, and oil, gas, and mining operations.

**Primary ML inference type:** The Rover mainly uses **classification**, with additional **planning** and **fleet coordination**. Its set of 14 sensors includes LiDAR, thermal, multispectral, RGB, acoustic, radar, and gas sensors. These inputs support detection tasks such as intrusion monitoring, vehicle classification, equipment defects, gas leaks, and crop health analysis.

A useful design detail is that the system can change the weight assigned to each sensor based on environmental conditions. For example, rain, fog, or darkness may reduce the reliability of one sensor while another remains useful. This is an example of sensor fusion being used for robustness rather than simply adding more inputs.

**Key constraint:** The main constraint is **reliability in changing outdoor conditions**. The robot needs to continue operating even when visibility, weather, terrain, or network connectivity changes.

Two design choices are especially relevant. First, the Rover uses uncertainty scoring before autonomous actions. Second, critical safety functions run on a dedicated microcontroller separate from the main AI computer. This separation means that a software or model failure should not disable the basic safety layer.

The website also describes coordination across fleets of up to 12 units, with communication secured with mTLS and an ability to continue operating when network connections are limited. The Rover has an eight-hour battery target and is designed as a relatively heavy platform to improve traction on uneven outdoor surfaces.

InGen’s roadmap places Aido Rover in engineering development and internal testing, with certification and commercial deployment planned for later stages. For that reason, its published specifications should be understood as development targets.

## 5. Aido Humanoid: General Purpose Bipedal Robot

Aido Humanoid is InGen’s bipedal research platform. It is intended to perform tasks in environments originally designed for people, including spaces with doors, stairs, shared work areas, and standard equipment.

**Target use case:** The named application areas are healthcare, eldercare, manufacturing, and logistics. The same physical platform is intended to be adapted to different industries through model fine-tuning and safety settings.

**Primary ML inference type:** Unlike the other products, the Humanoid’s main ML problem is **planning and policy control** rather than only classification. The public architecture is described in three layers:

1. A GR00T N1.6 foundation model.
2. GRPO-based fine-tuning for task and motion policies.
3. Origami AI integration for perception, safety, and coordination.

The website describes 30 manipulation skills across 14 grasp types, as well as bipedal movement using 32 lower-body degrees of freedom. It also lists operation on slopes of up to 15 degrees and standard stairs.

**Key constraint:** The main constraint is **physical safety in real time**. The robot may work close to people and could make direct physical contact with them. The public Humanoid page lists 12 hardwired safety rules, labeled there under SEOM, including force limits, personal-space boundaries, gaze signaling, consent detection, and fall prevention. The stated emergency-stop latency target is below 2 milliseconds.

For healthcare and eldercare, the same page gives a safety setting labeled SEOM λ=10.0. Since the Concepts Primer uses SEOM differently, I treat this number as a product-page configuration label rather than a general definition of the SEOM foundation model.

The Humanoid also has a richer sensor set than the other platforms because it must perceive, communicate, balance, and physically manipulate objects at the same time. The listed sensors include RGB-D cameras, context cameras, LiDAR, microphones, speakers, tactile sensors, an IMU, and visual signaling through LEDs.

The roadmap describes internal prototypes and future independent safety validation. As with the Rover, the current specifications are best read as development goals rather than completed commercial capabilities.

## 6. PIC 2.0 Architecture Implications

For the Week 1 assignment, the Concepts Primer gives the most useful working vocabulary for connecting PIC 2.0 to OpenClaw. It describes these mappings as likely published research concepts rather than confirmed internal architecture, so the table below should be read as a structured hypothesis.

| PIC 2.0 model | Working interpretation from the Concepts Primer | Closest OpenClaw connection |
|---|---|---|
| **HTD-IRL** | Hierarchical task decomposition, with an inverse-reinforcement-learning component | **Planner agent**, because both break a high-level goal into smaller tasks. The comparison is partial because OpenClaw does not use IRL. |
| **SEOM** | Semantic or embedding-oriented model | **Knowledge / retrieval agent**, because both organize information by meaning and support evidence-grounded retrieval. |
| **STUM** | State or sequence modelling | **Quant / analysis agent**, because both process changing or sequential inputs into a structured decision signal. |
| **AMDC** | Adaptive or hierarchical decision-making and governance | **Risk / Critic agents**, because they evaluate proposed outputs against risk, safety, and action boundaries. |
| **CRL-MRS** | Continual or coordination across multiple robots | **Report agent**, at a high level, because both combine outputs from several components. The robot-coordination problem is still much more complex. |
| **GRPO** | Listed as one of the six PIC 2.0 foundation models, but not explained in detail in the Primer | **No confident mapping yet.** I would need stronger public evidence before assigning it to an OpenClaw agent. |

After comparing the products with this vocabulary, I came away with four main points.

### 1. The bridge is about similar functions, not identical systems

OpenClaw and PIC 2.0 both divide work across specialized components, but OpenClaw is a software research system while PIC 2.0 is intended for physical intelligence. The comparison is useful for identifying transferable design patterns, not for claiming that the internal implementations are the same.

### 2. The strongest transfer is task decomposition and processing grounded in evidence

The connection between the Planner and HTD-IRL is clear at the task decomposition level, although OpenClaw does not include inverse reinforcement learning. The connection between the Knowledge agent and SEOM is also useful because both involve semantic representation and evidence-based retrieval.

### 3. State, risk, and coordination become more difficult in a physical system

STUM and AMDC suggest that a robot must model changing state over time and adapt decisions under safety constraints. CRL-MRS adds coordination across several physical agents. OpenClaw has simpler versions of these patterns through Quant, Risk, Critic, and Report, but it does not face the same physical constraints that require immediate responses.

### 4. Unclear terms should stay as hypotheses

The public product pages and the Concepts Primer use some PIC 2.0 labels differently. Rather than selecting one meaning and presenting it as confirmed, I keep the descriptions on the product pages separate from the bridge based on the Primer. GRPO also remains unmapped because the Primer does not give enough detail to support a confident comparison.

Overall, the Concepts Primer suggests a functional bridge from OpenClaw's Planner, Knowledge, Quant, Risk/Critic, and Report components to several PIC 2.0 ideas. My main conclusion is that I already have experience with task decomposition, semantic retrieval, sequential analysis, governance checks, and output synthesis, while the implementation in the physical world and coordination across multiple robots remain new areas for me.

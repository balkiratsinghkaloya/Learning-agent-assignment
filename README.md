# Adaptive Learning Planner Agent

## 1) Problem Statement
Design an AI agent that helps users plan learning in a structured and adaptive way. Example target: Learn Data Science in 30 days.

Required capabilities:
- Decompose goal into learning units and milestones
- Retrieve curated topic structure and reference materials
- Estimate effort and build day-wise schedule using user constraints
- Accept feedback and revise remaining plan
- Preserve user history and planning context for future sessions

---

## Assignment Deliverables in This Folder

1. Architecture write-up
- README.md

2. Architecture diagrams (Mermaid)
- architecture-diagrams/system_components.mmd
- architecture-diagrams/planning_revision_flow.mmd
- architecture-diagrams/decision_pipeline.mmd

3. API contract
- api-contract/openapi.yaml

4. End-to-end sample input and outputs
- sample-input-output/goal_request.json
- sample-input-output/generated_plan_v1.json
- sample-input-output/feedback_event_week2.json
- sample-input-output/revised_plan_v2.json
- sample-input-output/evaluation_report_v2.json

5. Implementation outline
- implementation-outline/agent_workflow_pseudocode.py
- implementation-outline/component_interfaces.yaml

---

## 2) Architecture Overview

### Core Components
1. Interaction Layer
- Chat UI or web dashboard
- Captures goal, constraints, preferences, and feedback

2. Agent Orchestrator
- Central controller that sequences planning, evaluation, and revision
- Manages workflow state and triggers internal tools

3. Goal Understanding and Decomposition Engine
- Parses user goal into skills, subskills, and dependency graph
- Creates hierarchical learning units and milestones

4. Knowledge Retrieval Engine
- Pulls curated resources from trusted providers
- Builds topic structure, difficulty level, and prerequisite mapping

5. Effort Estimation Engine
- Estimates required effort for each unit from historical priors and user profile
- Produces confidence interval for each estimate

6. Scheduling and Constraint Solver
- Generates day-wise plan under constraints (time, pace, exam date, weekends, etc.)
- Produces best schedule and alternatives

7. Feasibility and Quality Evaluator
- Validates schedule realism, dependency correctness, and workload balance
- Scores plan quality before presenting to user

8. Feedback Analyzer and Revision Engine
- Interprets user feedback and progress signals
- Revises only affected sections while preserving stable plan portions

9. User Memory and Context Store
- Saves user profile, plan history, outcomes, and interaction context
- Enables continuity across sessions

10. Monitoring and Analytics
- Tracks completion, slip rate, satisfaction, and revision frequency
- Feeds learning loop for model improvement

---

## 3) Internal Data Model

### Entities
1. UserProfile
- user_id
- prior_knowledge_by_skill
- weekly_availability
- preferred_content_type
- learning_pace
- hard_constraints and soft_constraints

2. LearningUnit
- unit_id
- topic
- prerequisites
- difficulty
- estimated_effort_hours
- recommended_resources
- assessment_item

3. Milestone
- milestone_id
- target_units
- target_day
- success_criteria

4. StudyPlan
- plan_id
- day_wise_schedule
- estimated_total_hours
- feasibility_score
- version

5. FeedbackEvent
- event_type (too hard, too easy, behind schedule, unavailable days)
- affected_units
- user_message
- timestamp

---

## 4) End-to-End Flow

### A) Initial Planning Flow
User goal input
-> Goal parser extracts intent and timeline
-> Decomposition engine creates learning graph and milestones
-> Retrieval engine fetches curated resources and topic map
-> Effort estimator predicts unit-level duration
-> Constraint solver builds day-wise schedule
-> Feasibility evaluator validates realism
-> Orchestrator returns primary plan plus fallback plan
-> Context store persists full plan and assumptions

### B) Evaluation Flow
Generated schedule
-> Check dependency violations
-> Check daily and weekly load limits
-> Check milestone timing risk
-> Check goal coverage completeness
-> Compute feasibility and quality score
-> If below threshold, auto-replan

### C) Feedback and Revision Flow
User feedback and progress events
-> Feedback classifier detects issue type
-> Impact analyzer identifies affected units and downstream dependencies
-> Revision engine updates effort estimates and remaining schedule
-> Feasibility evaluator re-checks revised plan
-> Orchestrator publishes revision with change summary
-> Context store saves new version and rationale

---

## 5) Decision Logic Inside the Agent

### Unit Structuring and Sequencing
- Build prerequisite DAG from topic ontology
- Apply topological ordering
- Group small units into cognitively coherent sessions
- Insert spaced revision blocks every few days

### Feasibility Decision
- Compute available_hours vs required_hours
- Estimate overload risk using max daily cognitive load
- Reject schedules that violate hard constraints
- Select schedule maximizing quality score under feasibility threshold

### Revision Decision After Feedback
- If issue is local (single topic too hard), revise nearby days only
- If issue changes constraints (less time this week), trigger global partial re-optimization
- If repeated slippage occurs, lower pace and re-sequence milestones
- Preserve completed and stable future units to reduce disruption

---

## 6) Suggested Libraries and Frameworks

### Agent and Orchestration
1. LangGraph
- Best for stateful, multi-step agent workflows with deterministic control
- Supports branching, retries, tool routing, and memory-aware transitions

2. FastAPI
- Lightweight, high-performance API layer for plan generation and revision endpoints

### Retrieval and Knowledge
3. LlamaIndex
- Excellent for building retrieval pipelines over curated course documents

4. Vector store: Pinecone or Weaviate
- Semantic search on resource metadata and topic descriptions

5. Neo4j
- Models prerequisite graph and supports dependency queries naturally

### NLP and Modeling
6. Sentence-Transformers
- Topic similarity, resource matching, and feedback semantic classification

7. LightGBM or XGBoost
- Effort estimation from historical completion data and profile features

### Scheduling and Optimization
8. Google OR-Tools
- Strong constraint programming and scheduling optimization for day-wise plans

### Data and Memory
9. PostgreSQL
- Durable storage for user, plan versions, and analytics

10. Redis
- Session state and short-lived context cache

### Observability
11. OpenTelemetry + Prometheus + Grafana
- Metrics, tracing, and reliability monitoring for production use

Why this stack is strong:
- Combines deterministic scheduling with adaptive learning intelligence
- Supports explainability, versioned revisions, and enterprise-grade deployment
- Easy to justify in interviews and assignment reviews

---

## 7) Feasibility Scoring Example

FeasibilityScore = 0.35 * TimeFit + 0.25 * DependencyValidity + 0.20 * WorkloadBalance + 0.20 * ResourceCoverage

Thresholds:
- >= 0.80: Accept
- 0.65 to 0.79: Accept with warnings and alternatives
- < 0.65: Auto-replan

---

## 8) Example: Data Science in 30 Days

### Milestones
- Day 1-5: Python and data handling fundamentals
- Day 6-10: Statistics and probability basics
- Day 11-17: Machine learning foundations
- Day 18-23: Model evaluation and feature engineering
- Day 24-28: Mini project and capstone
- Day 29-30: Revision, mock interview, portfolio cleanup

### Daily Slot Template
- 60 min concept learning
- 45 min guided practice
- 30 min quiz or coding task
- 15 min recap and next-day setup

### Adaptive Rules
- If two consecutive misses: reduce next two days load by 20 percent
- If user says too easy: inject optional advanced unit in buffer day
- If user unavailable for two days: shift milestone and re-balance remaining days

---

## 9) Production-Ready API Surface

1. POST /plan/create
- Input: goal, deadline, constraints, preferences
- Output: plan_v1 with milestones and day-wise schedule

2. POST /plan/evaluate
- Input: plan_id
- Output: feasibility report and quality score

3. POST /plan/feedback
- Input: plan_id, feedback_event
- Output: impact report and revision proposal

4. POST /plan/revise
- Input: plan_id, approved_changes
- Output: plan_v2 with diff summary

5. GET /user/history
- Output: plans, completion stats, and progression trends

---

## 10) Why This Design Is Selection-Ready
- Complete lifecycle coverage: planning, evaluation, adaptation, persistence
- Clear modular boundaries and explainable decision logic
- Uses industry-grade frameworks for both AI and scheduling
- Includes measurable feasibility and revision policies
- Ready to scale from prototype to production

---

## 11) Assignment Submission Link
You can submit a public GitHub link in this format:
- https://github.com/YOUR_USERNAME/adaptive-learning-planner-agent

Recommended repo contents:
- README.md (this document)
- architecture-diagrams folder with exported PNGs of your flows
- sample-input-output folder with one full 30-day generated plan
- api-contract folder with endpoint specs

Quick publish steps:
1. Create a new public GitHub repository named adaptive-learning-planner-agent
2. Upload the complete LearningAgent_Assignment folder
3. Verify the repo contains architecture-diagrams, api-contract, sample-input-output, and implementation-outline
4. Submit the repository URL

---

## 12) Reviewer Checklist
- Goal decomposition and milestone logic is documented
- Curated retrieval and topic structuring are documented
- Effort estimation and day-wise scheduling approach is documented
- Evaluation and feasibility scoring are defined
- Feedback handling and plan revision flow are defined
- User history and context preservation are defined
- Libraries and framework choices are justified
- API contract and sample plan artifacts are included

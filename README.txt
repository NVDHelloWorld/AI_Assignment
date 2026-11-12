================================================================================
AI REASONING SYSTEM - SUBMISSION README
================================================================================

1. OVERVIEW
-----------
This submission implements a general-purpose, neuro-free Classical AI Reasoning
System that determines whether a natural language query is logically entailed by
a natural language knowledge base of facts and rules:

  Natural Language KB -> Symbolic Parser -> Internal Logical Representation
                      -> Classical Inference Engine -> True / False

No generative AI or LLM is used at any stage of parsing or inference. All final
decisions are derived strictly via classical symbolic AI inference algorithms.

2. DIRECTORY STRUCTURE
----------------------
submission/
|-- reasoning_agent.ipynb   Complete self-contained Jupyter Notebook implementation
|                           with reasoning traces, dataset evaluations, printed
|                           accuracy and execution time, and CSV generation.
|-- reasoning_agent.py      Standalone Python script mirror of the reasoning system.
|-- report.pdf              Technical report describing the internal representation,
|                           classical reasoning algorithms, experimental results on
|                           RuleTaker, CounterBench, and FOLIO, and reasoning traces.
|-- submission.csv          Generated predictions (id, label) across the benchmark
|                           datasets (RuleTaker, FOLIO, and CounterBench).
`-- README.txt              This documentation file.

3. INTERNAL REPRESENTATION
--------------------------
Natural language sentences are translated into formal symbolic structures:
  - Term:        Logical constants (e.g., bob, bald_eagle) and universally or
                 existentially quantified variables (e.g., ?X, ?Y).
  - Predicate:   Atomic n-ary relations P(t_1, ..., t_k), supporting both unary
                 attributes (e.g., smart(bob)) and binary relations
                 (e.g., chases(bald_eagle, squirrel)).
  - Literal:     A Predicate with an explicit polarity flag (negated=True/False)
                 and optional Negation-as-Failure (NAF) annotation.
  - Rule:        Universally quantified Horn / normal clauses:
                   L_1 AND L_2 AND ... AND L_m -> H
  - Disjunction: Non-Horn ground or quantified disjunctive clauses (A v B)
                 extracted from "either...or", "unless", and De Morgan ("not both")
                 constructions.
  - Structural Equations (SCM):
                 Boolean structural equations Y := f(X_1, ..., X_k) and directed
                 causal implications for counterfactual reasoning.

4. CLASSICAL REASONING ALGORITHMS
---------------------------------
The unified `ReasoningSystem` employs three complementary classical inference
engines depending on the logical structure of the knowledge base and query:

  A. Stratified Forward Chaining & Depth-Limited Backward Chaining with Unification
     (Closed-World Taxonomic & Multi-Hop Rule Bases - RuleTaker):
     - Full Robinson Unification (`unify_term`, `unify_predicate`, `unify_literal`)
       with occurs-check and variable standardization apart.
     - Stratum 0 Forward Chaining: Exhaustively applies positive and explicit-negation
       rules via Modus Ponens until fixpoint closure.
     - Stratum 1 Stratified Negation-as-Failure (NAF): Evaluates rules containing
       default-negated antecedents (`~P(?X)`) over active domain constants only after
       all positive derivations have converged, guaranteeing stratification soundness.
     - Depth-Limited Backward Chaining (Goal-Directed DFS): Recursively searches for
       sub-goal proofs with cycle avoidance and unification bindings.

  B. Open-Domain First-Order Clausal Tableaux & Resolution Engine (FOLIO):
     - Modus Ponens (Forward Chaining) & Modus Tollens (Contrapositive Inference:
       from A -> B and ~B, derive ~A).
     - Disjunctive Syllogism (Unit Resolution: from A v B and ~A, derive B).
     - Proof by Cases (Semantic Tableaux Branching: for disjunction A v B, compute
       fixpoint closures of KB U {A} and KB U {B}; entail Q if Q holds in both).
     - Resolution Refutation (Proof by Contradiction: assume ~Q and test whether
       the deductive closure contains a complementary pair P and ~P).
     - Skolemized Deduction Theorem: For universal conditional queries (forall x:
       P(x) -> Q(x)), introduce a fresh Skolem constant c_sk, assert P(c_sk), and
       verify whether Q(c_sk) is entailed.

  C. Classical 3-Step Counterfactual Engine (Abduction-Action-Prediction - CounterBench):
     - Step 1 (Abduction): Propagate factual observations through structural
       equations to determine the factual state of exogenous and endogenous variables.
     - Step 2 (Action / Do-Intervention): Perform surgical graph mutilation do(X = x)
       by severing incoming causal equations for intervened variable X and fixing X = x.
     - Step 3 (Prediction): Forward-propagate through the mutilated causal graph
       using Kleene 3-valued logic (with short-circuit evaluation for AND/OR gates).

5. EXPERIMENTAL RESULTS SUMMARY
-------------------------------
  - RuleTaker (Depths 0, 1, 2, 3, 3ext, 5, NatLang):  100.00% Accuracy
  - CounterBench (Alpha & Backdoor Causal Graphs):     95.00% Accuracy
  - FOLIO (Validation Split - Open-Domain FOL):        69.46% Accuracy

6. HOW TO RUN
-------------
Requirements:
  - Python 3.8+ (Standard Library: re, time, csv, json, copy, itertools, dataclasses)
  - Optional: `datasets` library (for downloading directly from Hugging Face if local
    cache is not used) and `jupyter` / `nbformat` for running the notebook.

To run the standalone script and regenerate `submission.csv`:
  python reasoning_agent.py

To open and execute the Jupyter Notebook:
  jupyter notebook reasoning_agent.ipynb
================================================================================

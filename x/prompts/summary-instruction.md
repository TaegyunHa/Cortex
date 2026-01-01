# Summary Instructions

## Goal
Create practical, coherent, focused summaries that capture essential concepts without unnecessary detail.

## Principles

### What to Keep
- **Core concepts**: What the technology/topic is and why it matters
- **Key problems solved**: Concrete challenges and their solutions
- **Architecture essentials**: Main components and how they work together
- **Practical patterns**: How to actually use/build with the concepts
- **Important relationships**: How pieces connect and interact

### What to Remove
- Meta-information (course structure, learning paths, prerequisites)
- Step-by-step tutorials (keep the pattern, remove the walkthrough)
- Redundant explanations (consolidate similar points)
- Examples that don't add conceptual clarity
- Transitional phrases and filler content ("let's talk about...", "next up...")
- Instructor commentary and asides

## Structure Guidelines

### Use Hierarchical Organisation
```
# Main Topic - [Type]

## What is [Topic]?
Brief, clear definition

## Why [Topic]?
Key problems/challenges it addresses
- Use numbered sections for major categories
- Bullet points for specific solutions

## Core Architecture/Components
Essential building blocks
- Define each component concisely
- Focus on what it does, not how to implement it

## [Practical Application Section]
How to actually use the concepts
- Keep the essential steps
- Remove detailed walkthroughs
- Include key points that prevent common mistakes

## Ecosystem/Related Components (if relevant)
How this fits with other tools/frameworks
```

### Format Preferences
- **Bold** for key terms and emphasis
- `Code format` for technical terms, function signatures
- Concise bullets over paragraphs
- Arrow notation (->) for transformations/relationships
- Subsections for clarity, not decoration

## Summary Types

### Minimal
- 20-40 lines
- Just what you need to understand the concept
- Quick reference format
- No context or justification

### Practical
- 60-100 lines
- Enough detail to actually work with the concepts
- Includes architecture and patterns
- Balances "what" and "why"

### Comprehensive
- 150+ lines
- Deep technical details
- Multiple examples
- Use cases and edge cases
- When you need to master the topic

## Quality Checks

Before finalising, ask:
1. Can I understand the core concept from this?
2. Can I explain why this matters?
3. Can I start building/using this from these notes?
4. Have I removed all fluff?
5. Is the structure scannable (can I find info quickly)?

## Example Transformations

### Before (verbose)
"Welcome to the course. Let's talk about state. State is really important in LangGraph. You see, when we define a graph, we first define the state that the graph will operate on. This state is shared by all nodes in the graph, and it's typically a Python data structure."

### After (practical)
**State**
- Data supplied to, updated by, and returned from the graph
- Shared by all nodes
- Define as: TypedDict, Python dataclass, or Pydantic BaseModel

## Usage

1. Read/watch source material
2. Identify core concepts vs. teaching scaffolding
3. Extract essential information only
4. Organize hierarchically
5. Remove redundancy
6. Test: Can you use this to build/explain the concept?

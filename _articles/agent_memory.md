---
title: "Agent memory"
date: 2026-09-27
summary: "Notes from CS329Z lecture 7"
---

Notes from [cs329z lecture 7](https://www.youtube.com/watch?v=qh5tP6PBZ6U)

Mmeory is important for storing infomration across sessions/combatting limited context windows

Parametric memory:
- Build into model weights 
- Semanitc: Pre-trained knowledge 
- Procedural: Instruction following 
- Cannot be episodic memory 
- Can be fine-tuned to add more knowledge/skills 

Context memory:
- Added to model context 
- Semantic: Retrieved facts 
- Procedural: Retrieved skills 
- Episodic: Retieved conversations
- Managed by context manager to get appropriate memory

Works in coordination with RAG

Langchain decided to include a memory filesystem, where every memory is a file and can be interacted with the filesystem, using normal unix tools like grep and ls, this also adds a hierachy 

Some examples, eg ZEP, include building a knolwedge graph of eveyrthing the model has learnt, and models it as a graph, uses graphRAG for retieval 

Patterns:
- Learning procedures from experience: recall past successful trajectories 
- Tiered memory hierachy: organise long term and shot term memory
- Reflective memory consolidation: reflect on memories before storing them 
- Memory Maintenance: Clean up or remove or update old memories 

Learned Procedures:

Voyager paper:
- Agent set up to play minecraft 
- Uses code to control player in minecraft 
- Created a skill library to refine the program it wrote, using skills to help with code generation for various aspects of the game, if the skill was wrong, or missing it would update / add a skill 
- The skill library creates a skill which is a set of code to deal with a certain aspect, eg fight zombies, that was then fed into GPT 3.5 to greate a program description, this was then embedded using an embedding model to create a key, witht he value being the code it created, and stored 
- When trying to pick a skill it created a summary of what it wanted to do, then embeds this and queries the skill library, the embedding in skill library closest to the one in the query is used to fetch the value, the code to make that happen 


Agent workflow memory:
- Obtain a set of actions 
- Evaluate wether actions helped answer query 
- Induce workflow based on patterns observed from actions, common or helpful actions can be saved as a workflow and re run 

Tiered memory heirachy:

Skill files:
- skills are stored as markdown files on filesystem and short snippets from each skill are appended to system prompt, so that it knows which skills are available, the full skill is invoken when the agent reads the skill from the filesystem 

MemGPT:
- Gives LLM tools to manage its own memory 
- In the LLM context window, includes system instructions, working context and a FIFO message queue
- MemGPT writes data to persistent memory after it recieves a system alert about limited context space 
- It can update previously stored information 
- It can also search out of context data to bring relevant info back into the context window 
- Documents can also put into archival storage and then searched when needed, including pagination 
- This however invalidates KV cache often by updating context 

MemoryOS:
- They define a short term memory, medium term memory and long term memory 
- Data is first pushed to short term memory then progressively sent to medium and long term memory, if accessed frequently 
- At each stage is also does some processing and has a tree structure to store data

Reflective memory:

Generative agents:
- Agents perceive their environment and these observations are stored in structure called the memory stream
- Based on these observations, the architecture retrieves relevant memories and uses those to determine an action 
- These retrieved memories can be used to form long term plans and reflections too 

Memory maintenance:

Sleep time compute:
- In between conversations with the user, in standard practice no compute is applied, this is called sleep time 
- The paper proposes using sleep time to think about prior conversations and plan ahead, to reduce test-time  (when conversation is resumed) compute 

A-Mem:
- Agentic memory for LLm agents 
- Traditional memory systems required predefined access patterns, A-Mem enhances flexibility by enabling dynamic memory operations, using a seperate agent managing the memory 

Mem0:
- Wokflow manages memory 
- Works with Zep Knowledge graph 
- The workflow detects conflicts and can update memories 
- Had 4 actions: ADD, UPDATE, DELETE, infomration



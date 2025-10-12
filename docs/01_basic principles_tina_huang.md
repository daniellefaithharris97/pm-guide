# Vibe Coding Fundamentals

## What is Vibe Coding?

Vibe coding is a new approach to programming where you "fully give into the vibes" and embrace AI assistance. Instead of writing code manually, you describe what you want to build in natural language and let AI generate the code for you.

**Key characteristics:**
- Natural language communication with AI
- Focus on describing the end result, not implementation details
- Embrace AI as a coding partner
- Think in terms of user experience and functionality

## The Five Fundamental Skills

### 1. Thinking (Four Levels)

#### Logical Thinking
**Question:** What is the game/application?
**Example:** "What is chess?" - Understanding the basic concept and rules.

#### Analytical Thinking  
**Question:** How do I play/use this?
**Example:** "What is the main objective of chess?" - Understanding goals and mechanics.

#### Computational Thinking
**Question:** How do I implement the logic?
**Example:** "How do I enforce chess rules in code?" - Breaking down complex logic into implementable components.

#### Procedural Thinking
**Question:** How do I excel at this?
**Example:** "What strategies make a good chess player?" - Optimizing for best performance and user experience.

**Key insight:** Most vibe coders struggle with thorough thinking. You must clearly define what you want before expecting AI to build it.

### 2. Frameworks

**Principle:** Direct AI toward proven solutions rather than asking it to invent from scratch.

**Best practices:**
- Specify frameworks and packages you want to use
- Research existing solutions before building
- Learn the structure of what you're building (frontend, backend, databases)
- Ask AI to recommend frameworks for specific features

**Example prompts:**
- "Use React for the frontend and Node.js for the backend"
- "Implement drag-and-drop using React DnD"
- "What React frameworks should I use for drag-and-drop functionality?"

### 3. Checkpoints (Version Control)

**Critical principle:** Always have version control. Things will break.

**Essential Git workflow:**
1. `git init` - Initialize repository
2. `git add .` - Track all files
3. `git commit -m "message"` - Save changes
4. `git log` - View history
5. `git reset` - Roll back if needed
6. `git remote add origin [URL]` - Connect to GitHub
7. `git push origin main` - Upload to cloud

**Key insight:** You don't need to memorize commands - ask AI to handle version control using natural language.

### 4. Debugging

**Reality:** Everything will break. Debugging is as important as building.

**Methodical approach:**
1. **Identify the problem** - What's wrong and where?
2. **Apply solutions** - Try different fixes systematically
3. **Provide context to AI** - Copy error messages, screenshots, specific file locations

**Best practices:**
- Point out errors to AI and let it propose solutions
- Be patient with iterative fixes
- Understand your project structure to guide AI to the right files
- Be specific about what you want changed

### 5. Context

**Golden rule:** More context = better results

**Types of context:**
- Detailed PRDs (Product Requirements Documents)
- Visual mockups and examples
- Error messages and screenshots
- Project structure explanations
- User experience descriptions

**Context sources:**
- Original prompts with extensive detail
- Screenshots of desired outcomes
- Error logs and debugging information
- Environment and preference details

## The Two Modes of Vibe Coding

### Mode 1: Implementing New Features
- Provide context relevant to new features
- Mention specific frameworks and documentation
- Make incremental changes
- Use checkpoints and version control

### Mode 2: Debugging Errors
- Understand how your project works
- Figure out what's wrong
- Provide maximum context to AI
- Guide AI to the right files and sections

## Essential Mindset: Start Small

**MVP Approach:**
- Start with minimal viable product
- Get core functionality working first
- Add features incrementally
- Avoid building everything at once

**Why this matters:**
- Reduces complexity and errors
- Easier to debug and iterate
- Prevents overwhelming AI with too many requirements

## Memory Aid

**"The Friendly Cat Dances Constantly"**
- **T**hinking
- **F**rameworks  
- **C**heckpoints
- **D**ebugging
- **C**ontext

## Key Takeaways

1. **Think thoroughly** through all four levels before coding
2. **Use proven frameworks** rather than reinventing
3. **Always have version control** - things will break
4. **Debug methodically** with patience and context
5. **Provide maximum context** for best AI results
6. **Start small** and iterate
7. **Know which mode you're in** - building or debugging
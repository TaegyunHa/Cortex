# Cluade Code References

[[references]]

# Docs

https://code.claude.com/docs/en/overview

# Install

**macOS, Linux, WSL:**

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

**Windows PowerShell:**

```pwsh
irm https://claude.ai/install.ps1 | iex
```

**Windows CMD:**

```cmd
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

# Basic Workflow

> https://github.com/https-deeplearning-ai/sc-claude-code-files.git

## Managing Project Memory

- `/init`: Claude Code scans your codebase and creates `CLAUDE.md` file inside your project directory.
    -  `CLAUDE.md` guides Claude through your codebase, pointing out important commands, architecture and coding style. It's automatically included in the context each time you launch Claude Code.
- `#`: Use `#` to quickly add a memory. Useful when you see Claude Code repeats an error.
    - `#`use uv to run python files or add any dependencies
    - `#`The vector database has two collections:
        - `course_catalog`:
            - stores course titles for name resolution
            - metadata for each course: title, instructor, course_link, lesson_count, lessons_json (list of lessons: lesson_number, lesson_title, lesson_link)
        - `course_content`:
            - stores text chunks for semantic search
            - metadata for each chunk: course_title, lesson_number, chunk_index

### Managing the Context of Claude Code

| Command    | Description                                                                    |
| ---------- | ------------------------------------------------------------------------------ |
| `/init`    | scans your codebase and creates `CLAUDE.md` file inside your project directory |
| `#`        | `#` to add a memory. Useful when you see Claude Code repeats an error          |
| `/clear`   | clears current conversation history                                            |
| `/compact` | summarizes current conversation history                                        |
| `ESC`      | interrupt Claude to redirect or correct it                                     |
| `ESC ESC`  | rewind the conversation to an earlier point in time                            |
| `!<cmd>`   | run regular bash command                                                       |
| `exit`     | quit Claude Code                                                               |
## Custom command

- Inside the `.claude` folder of your project directory, create a folder `commands`
- Inside the `commands` folder, create a markdown file: `implement-feature.md`
- Copy the following to the markdown file: 
   ```
   You will be implementing a new feature in this codebase
   
   $ARGUMENTS
   
   IMPORTANT: Only do this for front-end features.
   Once this feature is built, make sure to write the changes you made to file called frontend-changes.md
   Do not ask for permissions to modify this file, assume you can always do it.
   ```   
- Launch again Claude Code, you can now use the command as any other built-in command with Claude Code.

## Thinking

For complex tasks, you can use the word `think` to trigger extended thinking mode.
There are several levels of thinking: `think` < `think hard` < `think harder` < `ultrathink`
Each level allocates more thinking budget for Claude.

## Git Worktree

Git worktrees allow to check out multiple branches from the same repository into separate directories. Each worktree represents a copy of working directory with isolated files but shares the same Git history.

**Workflow**
- Make sure first that you've added and committed any previous changes to the codebase.
- Create the folder .trees: `mkdir .trees`.
- For each feature you want to implement, create a worktree:
   - `git worktree add .trees/ui_feature`
   - `git worktree add .trees/testing_feature`
   - `git worktree add .trees/quality_feature`
- From each worktree, open an integrated terminal, launch Claude code in each terminal, and ask it to implement each feature.
- For each worktree, add and commit the changes in each worktree.
- Close claude terminals.
- In the main terminal: ask Claude Code to git merge the worktrees and resolve any merge conflicts (```use the git merge command to merge in all the worktrees of the .trees folder into main and fix any conflicts if there are any```)

## Prompt

### Generate Prompt

```
Give me an overview of this codebase

What are the key data models?

Explain how the documents are processed

What is the format of the document expected by the document_processor?

How are the course chunks loaded to the database?

Trace the process of handling user's query from frontend to backend

Draw a diagram that illustrates this flow

Explain how the text is transformed into chunks? What is the size of each chunk?

Describe the api endpoints

How can I run the application?

Here's what I want to do. Give me the best possible prompt to accomplish this task.
```

### Error Debugging

Example prompt with plan mode:

```
The RAG chatbot returns 'query failed' for any content-related questions. I need you to:
1. Write tests to evaluate the outputs of the execute method of the CourseSearchTool in @backend/search_tools.py 
2. Write tests to evaluate if @backend/ai_generator.py correctly calls for the CourseSearchTool 
3. Write tests to evaluate how the RAG system is handling the content-query related questions. 

Save the tests in a tests folder within @backend. Run those tests against the current system to identify which components are failing. Propose fixes based on what the tests reveal is broken.

Think.
```

### Code Refactoring

**Example prompt 1:**

```
Refactor @backend/ai_generator.py to support sequential tool calling where Claude can make up to 2 tool calls in separate API rounds.

Current behavior:
- Claude makes 1 tool call → tools are removed from API params → final response
- If Claude wants another tool call after seeing results, it can't (gets empty response)

Desired behavior:
- Each tool call should be a separate API request where Claude can reason about previous results
- Support for complex queries requiring multiple searches for comparisons, multi-part questions, or when information from different courses/lessons is needed

Example flow:
1. User: "Search for a course that discusses the same topic as lesson 4 of course X"
2. Claude: get course outline for course X → gets title of lesson 4
3. Claude: uses the title to search for a course that discusses the same topic → returns course information
4. Claude: provides complete answer

Requirements:
- Maximum 2 sequential rounds per user query
- Terminate when: (a) 2 rounds completed, (b) Claude's response has no tool_use blocks, or (c) tool call fails
- Preserve conversation context between rounds
- Handle tool execution errors gracefully

Notes: 
- Update the system prompt in @backend/ai_generator.py 
- Update the test @backend/tests/test_ai_generator.py
- Write tests that verify the external behavior (API calls made, tools executed, results returned) rather than internal state details. 

Use two parallel subagents to brainstorm possible plans. Do not implement any code.
```

**Example prompt 2:**

```
The @EDA.ipynb contains exploratory data analysis on e-commerce data in @ecommerce_data, focusing on sales metrics for 2023. Keep the same analysis and graphs, and improve the structure and documentation of the notebook.

Review the existing notebook and identify:
- What business metrics are currently calculated
- What visualizations are created
- What data transformations are performed
- Any code quality issues or inefficiencies
  
**Refactoring Requirements**

1. Notebook Structure & Documentation
    - Add proper documentation and markdown cells with clear header and a brief explanation for the section
    - Organize into logical sections:
        - Introduction & Business Objectives
        - Data Loading & Configuration
        - Data Preparation & Transformation
        - Business Metrics Calculation (revenue, product, geographic, customer experience analysis)
        - Summary of observations
    - Add table of contents at the beginning
    - Include data dictionary explaining key columns and business terms
   
2. Code Quality Improvements
   - Create reusable functions with docstrings
   - Implement consistent naming and formatting
   - Create separate Python files:
 	- business_metrics.py containing business metric calculations only
	- data_loader.py loading, processing and cleaning the data  
        
3. Enhanced Visualizations
    - Improve all plots with:
        - Clear and descriptive titles 
        - Proper axis labels with units
        - Legends where needed
        - Appropriate chart types for the data
        - Include date range in plot titles or captions
        - use consistent color business-oriented color schemes
          
4. Configurable Analysis Framework
The notebook shows the computation of metrics for a specific date range (entire year of 2023 compared to 2022). Refactor the code so that the data is first filtered according to configurable month and year & implement general-purpose metric calculations. 
       

**Deliverables Expected**
- Refactored Jupyter notebook (EDA_Refactored.ipynb) with all improvements
- Business metrics module (business_metrics.py) with documented functions
- Requirements file (requirements.txt) listing all dependencies
- README section explaining how to use the refactored analysis

**Success Criteria**
- Easy-to read code & notebook (do not use icons in the printing statements or markdown cells)
- Configurable analysis that works for any date range
- Reusable code that can be applied to future datasets
- Maintainable structure that other analysts can easily understand and extend
- Maintain all existing analyses while improving the quality, structure, and usability of the notebook.
- Do not assume any business thresholds.
```

### UI Feature

```
Add a toggle button that allows users to switch between dark and light themes.

1. Toggle Button Design
    - Create a toggle button that fits the existing design aesthetic
    - Position it in the top-right
    - Use an icon-based design (sun/moon icons or similar)
    - Smooth transition animation when toggling
    - Button should be accessible and keyboard-navigable

2. Light Theme CSS Variables
    Add a light theme variant with appropriate colors:
    - Light background colors
    - Dark text for good contrast
    - Adjusted primary and secondary colors
    - Proper border and surface colors
    - Maintain good accessibility standards

3. JavaScript Functionality
    - Toggle between themes on button click
    - Smooth transitions between themes

4. Implementation Details
    - Use CSS custom properties (CSS variables) for theme switching
    - Add a `data-theme` attribute to the body or html element
    - Ensure all existing elements work well in both themes
    - Maintain the current visual hierarchy and design language
```

### Testing Feature

```
Enhance the existing testing framework for the RAG system in @backend/tests. The current tests cover unit components but are missing essential API testing infrastructure:

- API endpoint tests - Test the FastAPI endpoints (/api/query, /api/courses, /) for proper request/response handling
- pytest configuration - Add pytest.ini_options in pyproject.toml for cleaner test execution
- Test fixtures - Create conftest.py with shared fixtures for mocking and test data setup

The FastAPI app in backend/app.py mounts static files that don't exist in the test environment. Either create a separate test app or define the API endpoints inline in the test file to avoid import issues.
```

### Quality Feature

```
Add essential code quality tools to the development workflow. Set up black for automatic code formatting. Add proper formatting consistency throughout the codebase and create development scripts for running quality checks.
```

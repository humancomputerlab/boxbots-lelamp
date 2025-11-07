# 🔄 LeLamp Workflows Guide

**⏱️ Estimated time:** 30-60 minutes per workflow  
**🔧 Required tools:** Computer with LeLamp runtime, text editor  
**📦 Required:** LeLamp runtime repository, understanding of Python and JSON

## Overview

Workflows in the LeLamp runtime system enable you to create custom AI-powered behaviors for your robot lamp. Workflows help the robot follow a set of instructions sequentially, essentially forming a graph structure that defines how the robot should respond to different situations and execute tasks.

The LeLamp runtime uses LiveKit AI to process these workflows, allowing your robot to intelligently follow complex instruction sequences and interact with its environment.

## Workflow Structure

Each workflow consists of two essential files:

1. **`workflow.json`** - Defines the workflow structure, nodes, edges, and execution logic
2. **`tools.py`** - Contains the Python functions (tools) that the workflow can call during execution

These files must be placed inside a folder named after your workflow (e.g., `my_custom_workflow/`), which is then uploaded to the `workflows/` folder in the LeLamp runtime repository.

## Creating a New Workflow

### Step 1: Create Your Workflow Folder

First, navigate to the LeLamp runtime repository on your computer or Raspberry Pi:

```bash
cd lelamp_runtime
cd workflows
mkdir my_custom_workflow
cd my_custom_workflow
```

Replace `my_custom_workflow` with your desired workflow name. Use lowercase letters, numbers, and underscores only (e.g., `greeting_workflow`, `dance_sequence`, `interactive_mode`).

### Step 2: Create `workflow.json`

Create a `workflow.json` file that defines your workflow's graph structure. This file describes:

- **Nodes**: Individual steps or decision points in your workflow
- **Edges**: Connections between nodes that define the flow
- **Tools**: References to functions defined in `tools.py`
- **Conditions**: Logic that determines which path to take

Example `workflow.json` structure:

```json
{
  "name": "my_custom_workflow",
  "description": "A custom workflow for LeLamp",
  "nodes": [
    {
      "id": "start",
      "type": "entry",
      "action": "greet_user"
    },
    {
      "id": "listen",
      "type": "action",
      "action": "listen_for_command"
    },
    {
      "id": "process",
      "type": "decision",
      "condition": "has_command",
      "true_path": "execute",
      "false_path": "listen"
    },
    {
      "id": "execute",
      "type": "action",
      "action": "execute_command"
    }
  ],
  "edges": [
    {
      "from": "start",
      "to": "listen"
    },
    {
      "from": "listen",
      "to": "process"
    },
    {
      "from": "process",
      "to": "execute",
      "condition": "has_command"
    }
  ]
}
```

### Step 3: Create `tools.py`

Create a `tools.py` file that implements the functions referenced in your `workflow.json`. These functions are the actual tools that your workflow can use.

Example `tools.py`:

```python
"""Tools for my_custom_workflow"""

from typing import Dict, Any

def greet_user(context: Dict[str, Any]) -> Dict[str, Any]:
    """
    Greets the user with a friendly message and LED animation.
    
    Args:
        context: Workflow context containing state and parameters
        
    Returns:
        Updated context with greeting status
    """
    # Your implementation here
    # Example: Control LEDs, play audio, move servos
    return {
        **context,
        "greeted": True
    }

def listen_for_command(context: Dict[str, Any]) -> Dict[str, Any]:
    """
    Listens for user voice commands.
    
    Args:
        context: Workflow context
        
    Returns:
        Updated context with command data
    """
    # Your implementation here
    # Example: Process audio input, use speech recognition
    return {
        **context,
        "command": "example_command",
        "has_command": True
    }

def execute_command(context: Dict[str, Any]) -> Dict[str, Any]:
    """
    Executes the received command.
    
    Args:
        context: Workflow context with command data
        
    Returns:
        Updated context with execution status
    """
    command = context.get("command")
    # Your implementation here
    # Example: Parse command and trigger appropriate robot actions
    return {
        **context,
        "executed": True
    }
```

**Important Notes:**

- All tool functions must accept a `context: Dict[str, Any]` parameter
- All tool functions must return a `Dict[str, Any]` (typically an updated context)
- Use type hints for better code clarity
- Include docstrings explaining what each function does
- Import any necessary LeLamp runtime modules at the top of the file

### Step 4: Upload Your Workflow

Once you've created both files, your workflow folder structure should look like this:

```
workflows/
└── my_custom_workflow/
    ├── workflow.json
    └── tools.py
```

If you're working on the Raspberry Pi directly, your files are already in place. If you're developing on a separate computer:

1. **Commit and push** your workflow to the repository (if using version control)
2. **Or copy** the folder to the Raspberry Pi using `scp`:

```bash
scp -r my_custom_workflow your_username@your_host_name.local:~/lelamp_runtime/workflows/
```

## Running Workflows

To make your workflows available when running the LeLamp agent, you need to specify them using the `WORKFLOWS` environment variable.

### Basic Usage

Run the main workflow script with your workflows enabled:

```bash
WORKFLOWS=workflow_1,workflow_2 sudo uv run main_workflow.py console
```

Replace `workflow_1` and `workflow_2` with the actual names of your workflow folders (without the path, just the folder name).

### Example

If you have workflows named `greeting_workflow`, `dance_sequence`, and `interactive_mode`, you would run:

```bash
WORKFLOWS=greeting_workflow,dance_sequence,interactive_mode sudo uv run main_workflow.py console
```

### Multiple Workflows

You can specify multiple workflows separated by commas. The system will make all tools from the specified workflows available to the agent:

```bash
WORKFLOWS=workflow_1,workflow_2,workflow_3 sudo uv run main_workflow.py console
```

**Note:** Ensure all workflow folder names are correct and exist in the `workflows/` directory. If a workflow name is misspelled or doesn't exist, it will be skipped (check the console output for warnings).

## Workflow Best Practices

### 1. Naming Conventions

- Use descriptive, lowercase names with underscores
- Keep names concise but clear (e.g., `voice_control`, `dance_routine`, `mood_lighting`)
- Avoid special characters and spaces

### 2. Error Handling

Always include error handling in your `tools.py` functions:

```python
def my_tool(context: Dict[str, Any]) -> Dict[str, Any]:
    try:
        # Your implementation
        return {**context, "status": "success"}
    except Exception as e:
        return {**context, "status": "error", "error": str(e)}
```

### 3. Context Management

- Use the context dictionary to pass state between workflow nodes
- Initialize default values when needed
- Keep context data minimal and relevant

### 4. Testing

Test your workflows individually before combining them:

```bash
# Test a single workflow
WORKFLOWS=my_custom_workflow sudo uv run main_workflow.py console
```

### 5. Documentation

- Document your workflow's purpose in `workflow.json`
- Include clear docstrings in all `tools.py` functions
- Comment complex logic in your code

## Troubleshooting

### Workflow Not Found

If you get an error that a workflow is not found:

1. **Check the folder name** - Ensure it matches exactly (case-sensitive)
2. **Verify location** - Confirm the folder is in `lelamp_runtime/workflows/`
3. **Check file names** - Both `workflow.json` and `tools.py` must exist

### Import Errors in `tools.py`

If you encounter import errors:

1. **Check dependencies** - Ensure all imported modules are available in the runtime
2. **Verify paths** - Use relative imports or absolute imports from the runtime root
3. **Test imports** - Try importing your tools module directly:

```bash
cd lelamp_runtime
sudo uv run python -c "from workflows.my_custom_workflow.tools import greet_user"
```

### Workflow Execution Errors

If the workflow runs but fails during execution:

1. **Check console output** - Look for error messages in the terminal
2. **Validate JSON** - Ensure `workflow.json` is valid JSON (use a JSON validator)
3. **Test tools individually** - Import and test each function in `tools.py` separately
4. **Check context** - Verify that context data is being passed correctly between nodes

## Example Workflow: Simple Greeting

Here's a complete example of a simple greeting workflow:

**`workflows/greeting_workflow/workflow.json`:**
```json
{
  "name": "greeting_workflow",
  "description": "A simple workflow that greets users when they approach",
  "nodes": [
    {
      "id": "start",
      "type": "entry",
      "action": "detect_presence"
    },
    {
      "id": "greet",
      "type": "action",
      "action": "play_greeting"
    }
  ],
  "edges": [
    {
      "from": "start",
      "to": "greet"
    }
  ]
}
```

**`workflows/greeting_workflow/tools.py`:**
```python
"""Tools for greeting_workflow"""

from typing import Dict, Any

def detect_presence(context: Dict[str, Any]) -> Dict[str, Any]:
    """Detects if a user is present."""
    # Implementation would use camera or sensors
    return {**context, "user_present": True}

def play_greeting(context: Dict[str, Any]) -> Dict[str, Any]:
    """Plays a greeting animation and sound."""
    # Implementation would control LEDs, servos, and audio
    return {**context, "greeted": True}
```

Run it with:
```bash
WORKFLOWS=greeting_workflow sudo uv run main_workflow.py console
```

## Next Steps

- **Explore existing workflows** - Check the `workflows/` folder in the LeLamp runtime repository for examples
- **Combine workflows** - Use multiple workflows together for complex behaviors
- **Share your workflows** - Contribute back to the community by sharing your workflow implementations
- **Read the runtime docs** - Refer to the [LeLamp Runtime repository](https://github.com/humancomputerlab/lelamp_runtime) for advanced features and API documentation

---

**Previous**: [LeLamp Control](./5.%20LeLamp%20Control.md) | **Troubleshooting**: [Common Issues](./6.%20Common%20Issues.md)

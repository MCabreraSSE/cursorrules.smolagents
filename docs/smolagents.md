Got it! I'll review the entire `smolagents` repository (version 1.8.0) and produce comprehensive Markdown documentation, including a quickstart guide, a detailed API reference, and plenty of example usage. Additionally, I'll create a `rules` file based on Cursor's rule system to help LLMs like Claude Sonnet effectively utilize Smolagents. All sources will be provided at the end in their own section. I'll let you know when it's ready.

# **smolagents 1.8.0 Documentation**

## Quickstart Guide

`smolagents` is a lightweight library for building powerful AI agents in just a few lines of code ([GitHub - huggingface/smolagents:  smolagents: a barebones library for agents. Agents write python code to call tools and orchestrate other agents.](https://github.com/huggingface/smolagents#:~:text=,It%20offers)) ([GitHub - huggingface/smolagents:  smolagents: a barebones library for agents. Agents write python code to call tools and orchestrate other agents.](https://github.com/huggingface/smolagents#:~:text=Model,others%20via%20our%20LiteLLM%20integration)). It follows the **ReAct** paradigm (Reason + Act) for multi-step reasoning: an agent will *think* about the task, then *act* by calling a tool, observe the result, and repeat until the task is solved ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=Agent%20class%20that%20solves%20the,obtained%20from%20the%20environment)) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=,Arguments%20passed%20to%20the%20Tool)). The library emphasizes simplicity—its core logic is only ~1000 lines of code—while remaining flexible and model-agnostic ([GitHub - huggingface/smolagents:  smolagents: a barebones library for agents. Agents write python code to call tools and orchestrate other agents.](https://github.com/huggingface/smolagents#:~:text=%E2%9C%A8%20Simplicity%3A%20the%20logic%20for,minimal%20shape%20above%20raw%20code)) ([GitHub - huggingface/smolagents:  smolagents: a barebones library for agents. Agents write python code to call tools and orchestrate other agents.](https://github.com/huggingface/smolagents#:~:text=Model,others%20via%20our%20LiteLLM%20integration)). Agents can work with text, images, audio, etc., and use various tools or other agents to accomplish tasks ([GitHub - huggingface/smolagents:  smolagents: a barebones library for agents. Agents write python code to call tools and orchestrate other agents.](https://github.com/huggingface/smolagents#:~:text=%EF%B8%8F%20Modality,Cf%20this%20tutorial%20for%20vision)).

### Installation

Install the latest version from PyPI:

```bash
pip install smolagents
```

For optional features like vision or Gradio support, install extras (e.g., `pip install smolagents[gradio]` for the Gradio UI).

### Defining and Running an Agent

To quickly get started, define an agent with a language model and some tools, then ask it to perform a task:

```python
from smolagents import CodeAgent, DuckDuckGoSearchTool, HfApiModel

# 1. Choose a model (here using Hugging Face Inference API with a default model)
model = HfApiModel()  # defaults to Qwen 2.5 32B instruct model ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=,%E2%80%98Make%20calls%20to%20the%20serverless))

# 2. Pick tools for the agent (e.g., web search)
tools = [DuckDuckGoSearchTool()]

# 3. Create the agent
agent = CodeAgent(tools=tools, model=model)

# 4. Run the agent on a task/question
answer = agent.run("How many seconds would it take for a leopard at full speed to run through Pont des Arts?")
print("Agent's answer:", answer)
```  

Here we created a `CodeAgent` – the default agent that writes and executes Python code as actions ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=We%20provide%20two%20types%20of,class)) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=from%20smolagents%20import%20CodeAgent%20agent,7384)) – using a DuckDuckGo search tool. When we call `agent.run(...)`, the agent will decide how to use its tool to answer the question, for example by searching the web and computing the result. The final answer is returned as a string ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=,Give%20them%20clear%20names)) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=from%20smolagents%20import%20CodeAgent%20agent,7384)).

**Output (example):**
```
Agent's answer: It would take about 28.5 seconds for a leopard to run that distance.
```

**Note:** By default, code executed by `CodeAgent` runs locally with restricted permissions (only built-in safe functions and provided tools can be used) ([smolagents/docs/source/en/guided_tour.md at main · huggingface/smolagents · GitHub](https://github.com/huggingface/smolagents/blob/main/docs/source/en/guided_tour.md#:~:text=By%20default%2C%20the%20execution%20is,in%20what%20can%20be%20executed)) ([smolagents/docs/source/en/guided_tour.md at main · huggingface/smolagents · GitHub](https://github.com/huggingface/smolagents/blob/main/docs/source/en/guided_tour.md#:~:text=model%20%3D%20HfApiModel,co%2Fblog)). You can enable a remote sandbox (E2B) for execution or allow additional imports if needed (discussed later in Advanced Usage).

### Switching Language Models

`smolagents` is model-agnostic – you can plug in any Large Language Model (LLM) as long as it follows the chat message format and stops at specified stop sequences ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=You%E2%80%99re%20free%20to%20create%20and,models%20to%20power%20your%20agent)). The library provides convenient model wrappers:

- **HfApiModel:** Uses Hugging Face Inference API or on-demand providers on the Hub ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=The%20,Providers%20available%20on%20the%20Hub)) ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=,%E2%80%98Make%20calls%20to%20the%20serverless)). *(Used in the example above.)*
- **TransformersModel:** Runs a local transformer model via `transformers` library ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=A%20class%20that%20uses%20Hugging,library%20for%20language%20model%20interaction)) ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=Copied)).
- **LiteLLMModel:** Connects to the [LiteLLM](https://www.litellm.ai) SDK to access 100+ models from various providers ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=LiteLLMModel)) ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=This%20model%20connects%20to%20LiteLLM,gateway%20to%20hundreds%20of%20LLMs)).
- **OpenAIServerModel:** Connects to OpenAI or any OpenAI-compatible REST API (e.g., Azure OpenAI) by specifying an endpoint and API key ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=OpenAIServerModel)) ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=model%20%3D%20OpenAIServerModel%28%20model_id%3D%22gpt,)).

For example, to use an OpenAI model (like GPT-4):

```python
import os
from smolagents import OpenAIServerModel

model = OpenAIServerModel(
    model_id="gpt-4", 
    api_base="https://api.openai.com/v1", 
    api_key=os.environ["OPENAI_API_KEY"]
)
agent = CodeAgent(tools=[DuckDuckGoSearchTool()], model=model)
agent.run("What is the capital of Australia?")
``` 

Or using LiteLLM to access an Anthropic Claude model:

```python
from smolagents import LiteLLMModel

model = LiteLLMModel("anthropic/claude-3-5-sonnet-latest", temperature=0.2, api_key="YOUR_ANTHROPIC_API_KEY")
# Use the model in an agent (or directly call it)
response = model([{"role": "user", "content": "Hello, how are you?"}])
print(response)
``` 

Each model wrapper handles the communication details and returns the LLM’s text output. You can also implement your own model interface if needed, as long as it accepts a list of message dicts and returns a string ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=You%20could%20use%20any%20,your%20agent%2C%20as%20long%20as)).

### Using Tools

Tools are functions that the agent can call to interact with the world (search the web, run code, query databases, etc.). `smolagents` comes with several built-in tools and supports easy integration of custom tools:

- *Search Tools*: `DuckDuckGoSearchTool` (no API key needed, uses DuckDuckGo web search) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=DuckDuckGoSearchTool)) and `GoogleSearchTool` (if configured for Google search) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=%28%20max_results%20%3D%2010%20,)). These take a query string and return search results.
- *Web Browsing*: `VisitWebpageTool` – fetches the content of a given URL (e.g., from a search result) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=)).
- *Code Execution*: `PythonInterpreterTool` – executes Python code and returns output or errors ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=PythonInterpreterTool)). This is used by `CodeAgent` to run code actions (added automatically when `add_base_tools=True`).
- *User Interaction*: `UserInputTool` – allows the agent to prompt the user for input if needed ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=UserInputTool)).
- *Multimedia*: `SpeechToTextTool` – converts audio input to text (speech recognition) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=SpeechToTextTool)); and others for images (you can load image processing tools from the Hub).

**Example:** Using a tool directly. You can call a tool standalone, which is useful for testing or as a simple utility:

```python
from smolagents import DuckDuckGoSearchTool

search_tool = DuckDuckGoSearchTool(max_results=5)
results = search_tool("Hugging Face smolagents")
print(results)
``` 

This will perform a web search and return a summary of the top 5 results (as text) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=DuckDuckGoSearchTool)). In an agent run, the LLM would decide **when** to call this tool and with what query.

**Creating a Custom Tool:** You can turn any Python function into a tool via the `@tool` decorator. For example:

```python
from smolagents import tool, CodeAgent

@tool
def add_numbers(a: float, b: float) -> float:
    """Args:
        a (float): First number.
        b (float): Second number.
       Returns:
        Sum of a and b.
    """
    return a + b

agent = CodeAgent(tools=[add_numbers], model=HfApiModel())
print(agent.run("What is 3.14 + 2.718?"))
``` 

The `@tool` decorator converts the function into a `Tool` object that the agent can use ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=%28%20tool_function%3A%20typing)) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=Tool)). The function’s docstring and type hints provide the tool’s description and parameter info to the agent ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=Parameters)). In the example above, the agent should call `add_numbers` to compute the sum, rather than doing it itself. This demonstrates how easily you can extend the agent's capabilities with new tools.

## Detailed API Reference

Below is a comprehensive reference of the main classes, functions, and methods available in `smolagents` 1.8.0, including their usage. For each component, we list its purpose, important parameters, return values, and examples.

### Agents

Agents are the core of `smolagents`. All agents inherit from the base **`MultiStepAgent`** class, which implements the ReAct loop for multi-step reasoning ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=Agents)) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=Both%20require%20arguments%20,at%20initialization)). An agent takes a user task and iteratively decides on actions (tool calls) until it produces a final answer ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=Agent%20class%20that%20solves%20the,obtained%20from%20the%20environment)) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=,Arguments%20passed%20to%20the%20Tool)).

We provide two main agent classes (both subclassing `Agent` → `MultiStepAgent`):

- **`CodeAgent`** – *Default.* The agent writes tool calls as Python code and executes them ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=We%20provide%20two%20types%20of,class)) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=class%20smolagents)).
- **`ToolCallingAgent`** – The agent formats tool calls in a structured JSON format (leveraging the LLM’s native tool use abilities) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=We%20provide%20two%20types%20of,class)) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=Perform%20one%20step%20in%20the,the%20step%20is%20not%20final)).

Both require a `model` (LLM interface) and a list of `tools` at initialization ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=,its%20tool%20calls%20in%20JSON)).

#### class `smolagents.MultiStepAgent`

*(Inherits from `Agent`)*

**Description:**  
Base class for multi-step agents following the ReAct framework ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=Agent%20class%20that%20solves%20the,obtained%20from%20the%20environment)). It manages the loop of thinking (LLM reasoning) and acting (tool execution) until the objective is achieved or a step limit is reached. You typically don’t instantiate `MultiStepAgent` directly, but use one of its subclasses.

**Parameters (constructor):** ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=Parameters)) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=,the%20agent%20will%20run%20a))  
- **tools** (`list[Tool]`): The tools this agent can use during execution ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=,to%20parse%20the%20tool%20calls)). Usually passed as a list of tool instances (e.g., `[DuckDuckGoSearchTool(), MyCustomTool()]`).  
- **model** (`Callable[[List[Dict[str,str]]], ChatMessage]`): The LLM to power the agent’s reasoning ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=,to%20parse%20the%20tool%20calls)). It should be a callable that takes a list of chat messages and returns a `ChatMessage` (or raw string) response. Typically this is an instance of one of the model classes (see **Models** below).  
- **prompt_templates** (`dict`, *optional*): Custom prompt templates for the agent. Use this to override the default system/user/assistant prompts if needed.  
- **max_steps** (`int`, *default:* 6): Maximum number of thought-action iterations the agent will perform per query ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=generate%20the%20agent%E2%80%99s%20actions.%20,tools%20to%20the%20agent%E2%80%99s%20tools)). This prevents infinite loops; the agent will stop with an incomplete result if this many steps are exceeded.  
- **tool_parser** (`Callable`, *optional*): Custom parser function to extract tool calls from the LLM’s output. By default, the agent uses either code parsing or JSON parsing depending on agent type. This is an advanced option to override that behavior.  
- **add_base_tools** (`bool`, *default:* False): If True, the agent will include a set of *base tools* by default ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=,the%20agent%20will%20run%20a)). Base tools typically include the `PythonInterpreterTool` (for code execution) and others needed internally. Setting this can be handy to get common functionality without explicitly providing those tools.  
- **verbosity_level** (`LogLevel`, *default:* `LogLevel.INFO`): Controls how much logging output the agent produces during runs. Higher verbosity can help in debugging an agent’s decisions.  
- **grammar** (`dict[str,str]`, *optional*): Grammar definitions for parsing LLM outputs. This is used internally to help the agent interpret the model’s response (advanced use; usually leave None).  
- **managed_agents** (`list[Agent]`, *optional*): A list of other agent instances that this agent can call as tools. If provided, the agent can delegate sub-tasks to these *managed agents* by name. (See **Managed Agents** in Advanced Usage.)  
- **step_callbacks** (`list[Callable]`, *optional*): Functions to be called after each step of the agent’s reasoning loop. Each callback receives the step data (e.g., action and observation) and can be used for logging, monitoring, or modifying behavior.  
- **planning_interval** (`int`, *optional*): If set, the agent will perform a special “planning” step after this many regular steps ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=Parameters)). Planning steps allow the agent to summarize or refocus on long tasks (see Advanced Usage for planning).  
- **name** (`str`, *optional*): Name of the agent (used when the agent is part of a multi-agent system). This is how a manager agent refers to this agent.  
- **description** (`str`, *optional*): A brief description of this agent’s role or capability, used when another agent might call it.  
- **provide_run_summary** (`bool`, *default:* False): If True, the agent will generate a summary of its run when finishing, which can be used if this agent is called as a sub-agent ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=,the)).  
- **final_answer_checks** (`list[Callable]`, *optional*): A list of functions to validate or post-process the final answer before returning ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=,to%20provide%20a%20run%20summary)). Each function should take the final answer (string) and return a (possibly modified) answer or raise an exception if unacceptable.

**Key Methods:** (Most of these are used internally, but a few are useful for users)  
- **`run(task:str, stream:bool=False, reset:bool=True, images:List[str]=None, additional_args:dict=None)`** – Execute the agent on a given task (user query) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=)) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=,Give%20them%20clear%20names)). 
  - *Parameters:*  
    - `task`: the task or question for the agent to solve.  
    - `stream`: if True, run in streaming mode (yield partial results as they are formed).  
    - `reset`: if True (default), clear any prior conversation/memory; if False, the agent will continue from previous context (useful for follow-up questions).  
    - `images`: list of image file paths (if the task involves image inputs) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=%28%20task%3A%20str%20images%3A%20typing.Optional,str)) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=Final%20answer%20to%20the%20task)). These will be incorporated so tools can use them (like vision QA tools).  
    - `additional_args`: dict of any other data to pass into the agent’s context ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=,Give%20them%20clear%20names)). This can include complex objects or dataframes that certain custom tools know how to handle.  
  - *Returns:* The final answer as a string (or an `AgentText` object which behaves like a string).  
  - *Example:*  
    ```python
    agent = CodeAgent(tools=[], model=HfApiModel())
    result = agent.run("What is the result of 2 power 3.7384?")
    print(result)
    ```  
     ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=Example%3A))This will output the answer (the agent figures out it likely needs to use Python to compute 2^3.7384).

- **`replay(detailed:bool=False)`** – Print a formatted replay of the agent’s reasoning steps ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=)) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=debugging)). If `detailed=True`, includes the full internal state at each step (for debugging). Useful for analyzing what the agent did after `run()` completes.  
- **`logs`** (property) – After running, `agent.logs` holds a list of step dictionaries containing the prompts, LLM outputs, tool calls, and observations for each step ([smolagents/docs/source/en/guided_tour.md at main · huggingface/smolagents · GitHub](https://github.com/huggingface/smolagents/blob/main/docs/source/en/guided_tour.md#:~:text=Here%20are%20a%20few%20useful,what%20happened%20after%20a%20run)). This is useful for programmatically inspecting or saving the agent’s decision process.
- **`write_memory_to_messages(summary_mode:bool=False)`** – Convert the agent’s internal memory (logs) into a list of chat messages ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=)). This essentially prepares a transcript that could be fed back into an LLM (e.g., for handoff or analysis). If `summary_mode=True`, it may condense the log.  
- *Internals:* Methods like `execute_tool_call(tool_name, arguments)`, `extract_action(model_output, split_token)`, `provide_final_answer(task, images)`, `step(memory_step)` are used internally to parse the LLM output, execute tools, and iterate the loop ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=)) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=Parse%20action%20from%20the%20LLM,output)). These are usually not called directly by users, but understanding them can help in advanced customization. For instance, `execute_tool_call` actually runs a tool by name and handles inserting any needed state variables into its arguments ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=,Arguments%20passed%20to%20the%20Tool)).

#### class `smolagents.CodeAgent`

**Description:**  
An agent that writes its actions as Python code and executes them. The `CodeAgent` will include the code (within special delimiters) in its LLM prompt and expect the LLM to output a Python code block when it wants to use a tool ([smolagents/docs/source/en/guided_tour.md at main · huggingface/smolagents · GitHub](https://github.com/huggingface/smolagents/blob/main/docs/source/en/guided_tour.md#:~:text=)). This code is then executed (in a protected environment) to produce an observation for the next step. It’s powerful for cases where the agent may need to manipulate data or call sequences of functions. 

**Parameters:** Inherits all from `MultiStepAgent`, and adds:  
- **additional_authorized_imports** (`list[str]`, *optional*): By default, the code execution environment only permits a safe set of imports (e.g., Python’s math library). You can list extra module names here to allow the agent to import them in its code ([smolagents/docs/source/en/guided_tour.md at main · huggingface/smolagents · GitHub](https://github.com/huggingface/smolagents/blob/main/docs/source/en/guided_tour.md#:~:text=The%20Python%20interpreter%20also%20doesn%27t,CodeAgent)). Use with caution – only include modules you trust, as this could execute arbitrary code ([smolagents/docs/source/en/guided_tour.md at main · huggingface/smolagents · GitHub](https://github.com/huggingface/smolagents/blob/main/docs/source/en/guided_tour.md#:~:text=%27https%3A%2F%2Fhuggingface.co%2Fblog%27%3F)).  
- **use_e2b_executor** (`bool`, *default:* False): If True, use the E2B remote code execution sandbox instead of local execution ([smolagents/docs/source/en/guided_tour.md at main · huggingface/smolagents · GitHub](https://github.com/huggingface/smolagents/blob/main/docs/source/en/guided_tour.md#:~:text=The%20execution%20will%20stop%20at,code%20generated%20by%20the%20agent)). This requires setting up an E2B API key. It provides an isolated environment for running code (enhanced security for untrusted code).  
- **max_print_outputs_length** (`int`, *optional*): A limit on how many characters of output from `print` statements in the agent’s code will be captured. This prevents overly long outputs from cluttering the log.  

Other parameters like `tools`, `model`, `prompt_templates`, etc., are the same as in `MultiStepAgent` ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=class%20smolagents)) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=,will%20run%20a%20planning%20step)).

**Behavior:**  
In each step, `CodeAgent` expects the LLM to return either a code snippet (to use a tool) or a final answer. The code snippet typically calls one of the available tools (by name) with some arguments. `CodeAgent` executes the code and captures its output or any exception, and feeds that back into the LLM on the next step ([smolagents/docs/source/en/guided_tour.md at main · huggingface/smolagents · GitHub](https://github.com/huggingface/smolagents/blob/main/docs/source/en/guided_tour.md#:~:text=)) ([smolagents/docs/source/en/guided_tour.md at main · huggingface/smolagents · GitHub](https://github.com/huggingface/smolagents/blob/main/docs/source/en/guided_tour.md#:~:text=By%20default%2C%20the%20execution%20is,in%20what%20can%20be%20executed)). This continues until the LLM responds with a final answer instead of code.

**Example:**  
```python
from smolagents import CodeAgent, PythonInterpreterTool, HfApiModel

agent = CodeAgent(
    tools=[PythonInterpreterTool()],  # allow executing arbitrary Python
    model=HfApiModel(),
    additional_authorized_imports=['math']  # allow importing math module in code
)
answer = agent.run("Use Python to calculate the 10th Fibonacci number.")
print(answer)
```  
In this example, the agent is likely to generate a code snippet using the `PythonInterpreterTool` (or directly executing Python) to compute the Fibonacci number and then return the result. The `additional_authorized_imports` ensures the agent can import `math` if needed in its code ([smolagents/docs/source/en/guided_tour.md at main · huggingface/smolagents · GitHub](https://github.com/huggingface/smolagents/blob/main/docs/source/en/guided_tour.md#:~:text=The%20Python%20interpreter%20also%20doesn%27t,CodeAgent)).

#### class `smolagents.ToolCallingAgent`

**Description:**  
An agent that formats tool calls in JSON (or JSON-like) rather than Python code. This leverages some LLMs’ built-in ability to output structured data for tool usage. Each tool call is output as a JSON blob specifying the tool name and arguments, which the `ToolCallingAgent` then parses and executes ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=class%20smolagents)) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=,arguments)).

**Parameters:** Similar to `CodeAgent` but **does not** include code-specific options (no `additional_authorized_imports` or `use_e2b_executor` since it doesn’t execute arbitrary code). It does accept `planning_interval` and others as in `MultiStepAgent` ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=%28%20tools%3A%20typing.List,kwargs)) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=,kwargs%20%E2%80%94%20Additional%20keyword%20arguments)).

**Behavior:**  
The agent’s prompt instructs the LLM to output a JSON with a `"tool"` field and `"arguments"` when it wants to use a tool. The output is parsed and the specified tool is called. This agent is useful if you prefer a more declarative style or are using models that have native tool APIs (like OpenAI functions or Anthropic’s tool usage). Internally, it uses the model’s `get_tool_call` method if available ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=,arguments)).

**Example:**  
```python
from smolagents import ToolCallingAgent, HfApiModel, DuckDuckGoSearchTool

agent = ToolCallingAgent(tools=[DuckDuckGoSearchTool()], model=HfApiModel())
result = agent.run("Find me the population of France.")
print(result)
```  
Here, the LLM might output something like `{"tool": "DuckDuckGoSearchTool", "arguments": {"query": "population of France"}}`. The agent will execute the search tool with that query and then the LLM will use the search result to respond with the population as the final answer.

#### Managed Agents (Multi-Agent Systems)

**`ManagedAgent` (Deprecated):** In versions prior to 1.8.0, there was a `ManagedAgent` class for making an agent callable by a manager agent. In v1.8.0 this is deprecated ([smolagents/docs/source/en/reference/agents.md at main · huggingface/smolagents · GitHub](https://github.com/huggingface/smolagents/blob/main/docs/source/en/reference/agents.md#:~:text=ManagedAgent)) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=ManagedAgent)). Now, to allow one agent to call another, you simply provide the sub-agent in the `managed_agents` list of the manager (and ensure the sub-agent has a `name` and `description`). In practice, you can create multiple agents and have a top-level agent orchestrate them as tools. (See **Advanced Usage** for an example of multi-agent orchestration.)

### Agent Utility Functions and Classes

#### `smolagents.stream_to_gradio(agent, task:str, reset_agent_memory:bool=False, additional_args:dict=None)`

Runs an agent on a task and streams its messages as Gradio chat components (for building UIs) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=,dict%5D%20%3D%20None)) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=Runs%20an%20agent%20with%20the,the%20agent%20as%20gradio%20ChatMessages)). This is mainly a helper to integrate agent output with a Gradio interface (chatbot). It yields `gradio.components.ChatMessage` objects.

- **agent:** the `MultiStepAgent` instance to run.  
- **task:** the input prompt for the agent.  
- **reset_agent_memory:** if True, resets the agent’s memory before running (fresh start) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=,dict%5D%20%3D%20None)).  
- **additional_args:** any extra arguments to pass into `agent.run()` (images, etc.) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=,dict%5D%20%3D%20None)).

**Usage:** Typically used inside a Gradio event to stream the agent’s thought process to the UI in real-time. For example, `for msg in stream_to_gradio(agent, "Hello"): yield msg`. (Requires Gradio to be installed.)

#### class `smolagents.GradioUI`

A simple utility class to launch a Gradio chat interface for an agent ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=GradioUI)) ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=A%20one,your%20agent%20in%20Gradio)). This provides a one-line way to spin up a web UI.

**Parameters:**  
- **agent** (`MultiStepAgent`): The agent to deploy in the UI.  
- **file_upload_folder** (`str`, *optional*): Directory path to use for storing uploaded files (if the agent expects file inputs).

**Methods:**  
- **`launch()`** – Launch the Gradio interface (opens a local web server for the chat UI).  
- **`upload_file(file, file_uploads_log, allowed_file_types=[...])`** – Helper to handle file uploads in the UI ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=)). By default, it allows PDF, DOCX, and TXT files ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=)). It saves the uploaded file and logs it so the agent can access it (likely as a path string input to a tool).

Example usage:
```python
from smolagents import GradioUI

ui = GradioUI(agent=my_agent)
ui.launch()
```  
This will open a Gradio chat where `my_agent` can interact with a user. (Remember to install Gradio with `pip install smolagents[gradio]` ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=GradioUI)).)

### Models

The model classes in `smolagents` provide a unified interface to various LLM backends. Each model is a callable that you can invoke with a list of chat messages (dicts with `"role"` and `"content"`) to get the LLM’s next response ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=You%E2%80%99re%20free%20to%20create%20and,models%20to%20power%20your%20agent)). They also handle details like stop sequences and role conversion.

You can always use a custom model by creating a callable that meets the expected interface ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=You%20could%20use%20any%20,your%20agent%2C%20as%20long%20as)), but the built-in classes cover common use cases.

#### class `smolagents.TransformersModel`

Uses a local Hugging Face Transformers model for inference ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=A%20class%20that%20uses%20Hugging,library%20for%20language%20model%20interaction)).

**Parameters:** ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=Parameters)) ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=,arguments%20that%20you%20want%20to))  
- **model_id** (`str`, *optional*, default: `"Qwen/Qwen2.5-Coder-32B-Instruct"`): Hugging Face Hub model name or path to local model files ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=Parameters)). By default it uses a 32B code instruct model. You can specify any chat model available on the Hub (e.g., `"bigcode/starcoder"`, or a local path).  
- **device_map** (`str` or `dict`, *optional*): Device placement for the model (e.g., `"auto"` or a dictionary mapping layers to devices). If not set, default behavior of `transformers` is used (which might load on CPU).  
- **torch_dtype** (`str` or `torch.dtype`, *optional*): Data type for model weights (e.g., `"float16"` for half precision). Use this to control memory usage/performance.  
- **trust_remote_code** (`bool`, *default:* False): If the model repo requires executing custom code (for custom architecture), set this True ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=,device)). Only do this for models you trust.  
- **\*\*kwargs**: Additional keyword args to pass to the model’s `.generate()` or `.forward()` calls. Common options include `max_new_tokens` (max tokens to generate) ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=,device)), `device` (to move model to GPU, e.g., `"cuda"`), etc.

**Raises:**  
- `ValueError` if no `model_id` is provided ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=)) (though it has a default, so this would only happen if someone passed an empty string).

**Usage:**  
When you call an instance of `TransformersModel` with a message list, it will run the model and return the generated text. Under the hood it constructs the prompt from the messages and handles stopping criteria.

**Example:**  
```python
from smolagents import TransformersModel

# Load a local model (make sure to have transformers and torch installed)
engine = TransformersModel(model_id="tiiuae/falcon-7b-instruct", device_map="auto", max_new_tokens=200)
messages = [ {"role": "user", "content": "Explain quantum mechanics in simple terms."} ]
response = engine(messages, stop_sequences=["</s>"])
print(response)
```  
This would load the Falcon 7B instruct model and use it to answer the prompt ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=Copied)) ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=%3E%3E%3E%20response%20%3D%20engine%28messages%2C%20stop_sequences%3D%5B,branch%20of%20physics%20that%20studies)). (The `stop_sequences` argument ensures the model stops at end-of-sentence token `</s>` or you could use custom stop.)

#### class `smolagents.HfApiModel`

Wraps Hugging Face’s Inference API (and Inference Endpoints) to call models hosted on Hugging Face Hub ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=The%20,Providers%20available%20on%20the%20Hub)) ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=,model%20name%20is%20not%20provided)). This allows using models without downloading them, including through providers like Replicate, Together, etc., as well as Hugging Face’s own infrastructure.

**Parameters:** ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=,%E2%80%98Make%20calls%20to%20the%20serverless)) ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=,the%20Hugging%20Face%20CLI%20configuration))  
- **model_id** (`str`, *optional*, default: `"Qwen/Qwen2.5-Coder-32B-Instruct"`): The model on the Hub to use ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=,%E2%80%98Make%20calls%20to%20the%20serverless)). Can be a model repo name (for Inference API) or a specific provider/model combination.  
- **provider** (`str`, *optional*): Name of the inference provider if not using the default HF Inference API ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=,the%20Hugging%20Face%20CLI%20configuration)). Options include `"replicate"`, `"together"`, `"fal-ai"`, `"sambanova"`, or `"hf-inference"` (which is default). This corresponds to providers listed on the Hub for certain models ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=,the%20Hugging%20Face%20CLI%20configuration)).  
- **token** (`str`, *optional*): Your Hugging Face API token, if required ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=defaults%20to%20hf,for%20the%20API%20request%2C%20in)). Needed for using the Inference API (especially with gated models or higher rate limits). If not provided, it will try `HF_TOKEN` env var or your Hugging Face CLI credentials.  
- **timeout** (`int`, *optional*, default: 120): Timeout in seconds for API calls ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=authentication,Useful%20for%20specific)).  
- **custom_role_conversions** (`dict[str,str]`, *optional*): If your model expects different role names (not "system"/"user"/"assistant"), you can map them here ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=the%20token%20stored%20in%20the,to%20the%20Hugging%20Face%20API)). E.g., some models might use "prompt"/"completion".  
- **\*\*kwargs**: Additional parameters passed to the API. This might include things like `max_tokens`, `temperature`, etc., depending on the provider or API.

**Raises:**  
- `ValueError` if `model_id` is empty ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=Raises)) (has a default, so usually not an issue).

**Usage:**  
Calling an `HfApiModel` instance will make an HTTP request to the inference endpoint and return the model’s generated text ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=A%20class%20to%20interact%20with,API%20for%20language%20model%20interaction)). It takes care of formatting the messages and handling the provider specifics.

**Example:**  
```python
from smolagents import HfApiModel

model = HfApiModel(model_id="bigscience/bloom", token="hf_xxx...")  # using HF Inference API with Bloom
messages = [ {"role": "user", "content": "Hi, can you summarize the news today?"} ]
reply = model(messages, max_new_tokens=100, stop_sequences=["\nUser:"])
print(reply)
```  
This will query the Bloom model hosted on HF’s servers and print the summary. You can also specify other providers, for example:
```python
model = HfApiModel(model_id="google/flan-t5-xxl", provider="sambanova")
```
to run via SambaNova’s hosted inference ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=The%20,Providers%20available%20on%20the%20Hub)) ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=defaults%20to%20hf,for%20the%20API%20request%2C%20in)) (if supported).

#### class `smolagents.LiteLLMModel`

Integrates with [LiteLLM](https://www.litellm.ai) to access a wide range of providers and models through a unified API ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=LiteLLMModel)) ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=The%20,temperature)). This is useful if you want to quickly try models from Anthropic, OpenAI, Cohere, etc., without writing separate integration code.

**Parameters:** ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=class%20smolagents)) ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=,pass%20to%20the%20OpenAI%20API))  
- **model_id** (`str`): Identifier of the model to use via LiteLLM ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=,pass%20to%20the%20OpenAI%20API)). This could be a provider-specific name, e.g., `"openai/gpt-3.5-turbo"` or `"anthropic/claude-2"`. 
- **api_base** (`str`, *optional*): Base URL for the API if using a custom or self-hosted instance. This is not typically needed when using LiteLLM’s default endpoints.  
- **api_key** (`str`, *optional*): API key for the chosen provider, if required. For example, if `model_id` is an OpenAI model, supply your OpenAI key here.  
- **custom_role_conversions** (`dict[str,str]`, *optional*): Role name mapping similar to HfApiModel, in case the target model expects different role keys ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=server.%20,pass%20to%20the%20OpenAI%20API)).  
- **\*\*kwargs**: Additional generation parameters to pass along. Many LLM APIs accept parameters like `temperature`, `max_tokens`, etc., which you can set here. Once the `LiteLLMModel` is created, these kwargs will be used every time you call the model ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=LiteLLMModel)) ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=The%20,temperature)) (so it’s like setting defaults for the generation).

**Usage:**  
After instantiation, calling it with a message list will route the request to the appropriate provider via LiteLLM and return the response text ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=LiteLLMModel)). Under the hood, LiteLLM figures out how to call the specific model’s API.

**Example:**  
```python
from smolagents import LiteLLMModel

# Use Claude 3.5 via Anthropic
model = LiteLLMModel("anthropic/claude-3-5-sonnet-latest", api_key="ANTHROPIC_API_KEY", temperature=0.2)
messages = [ {"role": "user", "content": "Tell a joke about cats"} ]
print(model(messages))
```  
This will invoke the Anthropic Claude model and print its joke. You can swap out to an OpenAI model easily:
```python
model = LiteLLMModel("openai/gpt-3.5-turbo", api_key="OPENAI_API_KEY", temperature=0.7)
```
and the rest of the code remains the same, demonstrating the flexibility of this wrapper ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=LiteLLMModel)) ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=model%20%3D%20LiteLLMModel%28%22anthropic%2Fclaude,messages)).

#### class `smolagents.OpenAIServerModel`

A connector for any OpenAI-compatible chat completion API (this includes OpenAI’s official API, Azure OpenAI service, or open-source deployments that mimic OpenAI’s API) ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=OpenAIServerModel)) ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=model%20%3D%20OpenAIServerModel%28%20model_id%3D%22gpt,)). If you have an endpoint that speaks the OpenAI ChatCompletion protocol, use this class.

**Parameters:** ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=class%20smolagents)) ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=,use%20for%20the%20API%20request))  
- **model_id** (`str`): The model name to request (e.g., `"gpt-3.5-turbo"`, `"gpt-4"`).  
- **api_base** (`str`, *optional*): The base URL of the API. For OpenAI, this defaults to `https://api.openai.com/v1` ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=This%20class%20lets%20you%20call,to%20point%20to%20another%20server)) ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=model%20%3D%20OpenAIServerModel%28%20model_id%3D%22gpt,)). For others, provide the root URL up to but not including the `/v1/chat/completions` path.  
- **api_key** (`str`, *optional*): Authentication key for the API. If not set, it might default to an `OPENAI_API_KEY` environment variable if present.  
- **organization** (`str`, *optional*): (For OpenAI) an organization ID if you need to specify one for the API request ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=%E2%80%9Cgpt,Useful%20for%20specific)).  
- **project** (`str`, *optional*): (Specific to some OpenAI-compatible APIs or internal tracking) a project name to associate with requests.  
- **custom_role_conversions** (`dict[str,str]`, *optional*): Role mapping, if needed (most OpenAI-compatible models use `"system"`, `"user"`, `"assistant"` so this often isn’t required).  
- **\*\*kwargs**: Extra options to pass in the API call payload (like `temperature`, `max_tokens`, etc.).

**Usage:**  
When called with messages, this will make an HTTP POST to the `/chat/completions` endpoint of the specified server and return the assistant message text ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=,pass%20to%20the%20OpenAI%20API)).

**Example:**  
```python
from smolagents import OpenAIServerModel

model = OpenAIServerModel(
    model_id="gpt-4", 
    api_base="https://api.openai.com/v1", 
    api_key="sk-...yourkey..."
)
messages = [{"role": "user", "content": "What's the weather like on Mars?"}]
reply = model(messages, temperature=0)
print(reply)
```  
This uses OpenAI’s GPT-4 via their API ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=This%20class%20lets%20you%20call,to%20point%20to%20another%20server)). For Azure OpenAI, you would use the `AzureOpenAIServerModel` (see below), or for other hosted services ensure you set `api_base` to the correct URL.

#### class `smolagents.AzureOpenAIServerModel`

Specialized version of OpenAIServerModel for Azure’s flavor of the OpenAI API ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=AzureOpenAIServerModel)) ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=from%20smolagents%20import%20AzureOpenAIServerModel)). Azure’s API requires an endpoint and has a slightly different authentication method (endpoint-specific keys and versions).

**Parameters:** ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=)) ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=,If%20not%20provided%2C%20it))  
- **model_id** (`str`): The *deployment name* of your Azure OpenAI model (note: in Azure, you deploy a model and give it a name). For example, if you deployed GPT-4 as "gpt4-chat", use that name.  
- **azure_endpoint** (`str`, *optional*): The full URL of your Azure OpenAI endpoint (ending with `/openai/deployments/<deployment_name>/...` usually) ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=,If%20not%20provided%2C%20it)). If not provided, it will try the environment variable `AZURE_OPENAI_ENDPOINT` ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=,If%20not%20provided%2C%20it)).  
- **api_key** (`str`, *optional*): Your Azure API key for the service. If not set, will look for `AZURE_OPENAI_API_KEY` env var ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=,environment%20variable)).  
- **api_version** (`str`, *optional*): The API version date (e.g., `"2023-05-15"`) that your Azure endpoint expects ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=,environment%20variable)). If not set, will use `OPENAI_API_VERSION` env var ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=,environment%20variable)) (note no `AZURE_` prefix for this one, as per Azure’s SDK design).  
- **custom_role_conversions** (`dict[str,str]`, *optional*): Role mapping if needed (usually not, Azure uses the same role schema as OpenAI).  
- **\*\*kwargs**: Additional parameters for the API calls (similar to OpenAIServerModel).

**Usage:**  
Call with messages to get a completion. Under the hood, this class formats the endpoint and headers as needed for Azure. You typically don’t need to use this if you set up environment vars; you could also use OpenAIServerModel by manually configuring `api_base`, but this class simplifies Azure setup.

**Example:**  
```python
from smolagents import AzureOpenAIServerModel
import os

model = AzureOpenAIServerModel(
    model_id=os.environ.get("AZURE_OPENAI_MODEL"),  # e.g., "gpt4-chat"
    azure_endpoint=os.environ.get("AZURE_OPENAI_ENDPOINT"),
    api_key=os.environ.get("AZURE_OPENAI_API_KEY"),
    api_version=os.environ.get("OPENAI_API_VERSION")
)
messages = [{"role": "user", "content": "Hello, Azure OpenAI!"}]
print(model(messages))
```  
This will connect to your Azure OpenAI deployment and print the model’s response ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=Copied)) ([Models](https://huggingface.co/docs/smolagents/reference/models#:~:text=)). (Ensure the env variables are set appropriately; the example shows how you might pull them in code.)

### Tools

Tools are the actions that an agent can perform. Each tool is essentially a function (with a bit of metadata) that the agent can call during its reasoning process ([smolagents/docs/source/en/guided_tour.md at main · huggingface/smolagents · GitHub](https://github.com/huggingface/smolagents/blob/main/docs/source/en/guided_tour.md#:~:text=Tools)). Tools can be anything from web APIs, databases, local functions, to even other agents.

#### `smolagents.load_tool(task_or_repo_id, model_repo_id=None, token=None, trust_remote_code=False, **kwargs)`

A quick way to load a tool from the Hugging Face Hub or by task name ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=Main%20function%20to%20quickly%20load,a%20tool%20from%20the%20Hub)). This function searches for an appropriate tool in the Hub or loads a default tool implementation for a known task.

- **task_or_repo_id** (`str`): Either a high-level task name (e.g., `"document_question_answering"`, `"image_question_answering"`, `"speech_to_text"`, etc.) or a specific repository ID on the Hub where a tool is defined ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=,Tasks%20implemented%20in%20Transformers%20are)). If a task name is given, `smolagents` knows some default models to use for that tool (for example, `"speech_to_text"` might load a Whisper-based ASR tool).  
- **model_repo_id** (`str`, *optional*): If you want to override the default model for that task, you can specify a particular model repo. For instance, if the task is `"translation"` but you want a specific translator model, put its repo id here ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=%2A%20%60,load%20a%20tool%20from%20Hub)).  
- **token** (`str`, *optional*): Hugging Face token for downloading the model if needed (for private models or higher limits) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=,passed%20along%20to%20its%20init)). If not provided, uses your cached login token.  
- **trust_remote_code** (`bool`, *default:* False): Whether to allow loading and executing code from the repo (if the tool is defined with custom code) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=%60~%2F.huggingface%60%29.%20,passed%20along%20to%20its%20init)). You must set this to True for tools that require custom code, acknowledging you trust the source.  
- **\*\*kwargs**: Additional arguments passed down. These are split into two groups: (1) arguments relevant to the Hub download (like `revision`, `cache_dir`), and (2) arguments for the tool’s initialization itself ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=accepted%20in%20order%20to%20load,passed%20along%20to%20its%20init)).

**Returns:** A `Tool` instance ready to use.

**Example:**  
```python
from smolagents import load_tool

# Load a document QA tool (which likely uses a QA model under the hood)
doc_qa_tool = load_tool("document_question_answering")

# Load an image QA tool with a specific model
viz_qa_tool = load_tool("image_question_answering", model_repo_id="Salesforce/blip2-flan-t5-xl")
```  
In these examples, `doc_qa_tool` might load a tool that can answer questions given a document (maybe using a DPR or Haystack under the hood), and `viz_qa_tool` loads a BLIP-2 model for visual question answering. Always inspect what tool code is being loaded (especially if `trust_remote_code=True`) for safety ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=Main%20function%20to%20quickly%20load,a%20tool%20from%20the%20Hub)).

#### `smolagents.tool(tool_function)`

This is a decorator or factory to create a `Tool` from a regular Python function ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=%28%20tool_function%3A%20typing)). Use it as `@tool` on a function definition to automatically generate a Tool with proper metadata.

- **tool_function** (`Callable`): The function you want to turn into a tool. The function should have type hints for each parameter and for the return type, and its docstring should include an **Args** section explaining each parameter ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=Parameters)).

**Behavior:**  
It returns an instance of a subclass of `Tool` (actually, it dynamically creates a Tool class for you). The function’s name becomes the tool name (unless overridden), the docstring becomes the description, and the type hints define input and output types. This saves you from writing a full class for simple tools.

**Example:** *(given in Quickstart above for `add_numbers`)*.

#### class `smolagents.Tool`

Base class for all tools ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=A%20base%20class%20for%20the,as%20the%20following%20class%20attributes)). If you want to implement a complex tool with custom initialization or state, you can subclass `Tool`.

**To define a new Tool subclass:】  
- Implement the `forward(self, **kwargs)` method – this is what the agent will call when using the tool. The inputs to forward are the tool’s inputs.  
- Set the class attributes:
  - **name** (`str`): A short name for the tool (how the agent references it) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=contained%20in%20the%20file%E2%80%99.%20,tool%2C%20and%20also%20can%20be)).
  - **description** (`str`): A human-readable description of what the tool does, including what inputs it expects and outputs it returns ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=,key.%20This%20is)). This is shown to the LLM to decide when to use the tool.
  - **inputs** (`Dict[str, Dict[str, Union[str, type]]]`): A dictionary specifying the input parameters for the tool, where keys are parameter names and values are dictionaries with `"type"` (the expected data type or modality, like `str`, `int`, or a custom type like `Image`) and a `"description"` of that parameter ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=contained%20in%20the%20file%E2%80%99.%20,tool%2C%20and%20also%20can%20be)). This helps the LLM understand how to call the tool and is also used for auto-generating UIs.
  - **output_type** (`type`): The Python type of the tool’s output ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=prompt%20to%20the%20agent,generated%20description%20for%20your%20tool)) (e.g., `str` for text output, or a custom wrapper type like `AgentImage` for images).

You can also override `setup()` if your tool needs one-time expensive setup (like loading a ML model). The `setup()` method will be called the first time the tool is actually used, not at instantiation, to avoid slow startup ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=You%20can%20also%20override%20the,tool%2C%20but%20not%20at%20instantiation)).

**Factory Methods:** (class methods on `Tool` for integration)
- **`Tool.from_gradio(gradio_tool)`** – Wraps a Gradio interface component as a tool ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=)). For example, if you have a Gradio function that takes inputs and returns output, this can turn it into a `Tool`.  
- **`Tool.from_hub(repo_id, token=None, trust_remote_code=False, **kwargs)`** – Load a tool class from a Hub repository ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=)) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=,used%20when%20downloading%20the%20files)). Similar to `load_tool` but used on the class. It fetches the code from the given repo (requires the repo to be a proper tool repo) and instantiates the tool. Needs `trust_remote_code=True` if not explicitly trusted.  
- **`Tool.from_langchain(langchain_tool)`** – Converts a LangChain Tool into a smolagents Tool ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=)). This allows using the large library of tools from LangChain by wrapping them.  
- **`Tool.from_space(space_id, name, description, api_name=None, token=None)`** – Creates a tool that calls a Hugging Face Space as an API ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=)) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=,or%20increase%20your%20GPU%20quotas)). You provide the Space’s ID and details, it becomes a tool that sends inputs to that Space and returns the output.
  - *Example:*  
    ```python
    image_gen = Tool.from_space(
        space_id="black-forest-labs/FLUX.1-schnell",
        name="image-generator",
        description="Generate an image from a prompt"
    )
    img = image_gen("A scenic view of mountaintops at sunrise")
    ```  
     ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=Examples%3A))This turns the specified Space (an image generation app) into a callable tool for your agent. The output `img` might be an `AgentImage` object (see Agent Types below).

- **`push_to_hub(repo_id, commit_message='Upload tool', private=None, token=None, create_pr=False)`** – Saves your tool’s code and uploads it to the Hub ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=)) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=,Whether%20or%20not%20to%20create)). Use this to share tools with others. Your tool must be defined in a proper Python module (not an interactive notebook’s `__main__`) for this to work ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=Upload%20the%20tool%20to%20the,Hub)). 
  - *Example:*  
    ```python
    my_tool = MyTool()  # suppose MyTool is a subclass of Tool
    my_tool.push_to_hub("username/my-awesome-tool", private=True)
    ```  
    This will create (or update) the repo `username/my-awesome-tool` with your tool’s code, so others can `Tool.from_hub` or `load_tool` it.

- **`save(output_dir)`** – Save the tool’s code files to a local directory ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=)). Similar to push_to_hub, but instead of uploading, it just writes out the necessary Python files to recreate this tool. You could then manually share or modify them.

**Note:** Most users will not need to subclass `Tool` directly unless creating complex behaviors. The `@tool` decorator and existing tools cover common needs.

#### Default Tools (built-in subclasses of `Tool`)

These tools come with `smolagents` and can be used out-of-the-box:

- **PythonInterpreterTool** – Executes a given Python code string and returns the result or error. This is what `CodeAgent` uses internally to run code actions. By default, it restricts imports (only safe modules) and disables certain dangerous operations. You typically don’t need to add this tool manually unless building a custom agent; use `add_base_tools=True` to include it automatically ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=PythonInterpreterTool)).  
  *Example:* `PythonInterpreterTool().forward(code="print(2+2)")` would yield `"4"`.

- **FinalAnswerTool** – A special tool that represents the agent giving a final answer ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=FinalAnswerTool)). The agent uses this internally to indicate it’s done (e.g., by returning a special token or calling this tool with the answer). Users generally don’t call this directly.

- **UserInputTool** – Allows the agent to ask the end-user a question and wait for input ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=UserInputTool)). If included, the agent might choose to use it when it needs clarification. In a non-interactive setting, this tool may not be very useful (since no real user input can be given mid-run). But in an interactive loop or with `GradioUI`, it could prompt the user.  

- **DuckDuckGoSearchTool** – Performs a web search via DuckDuckGo and returns summarized results ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=DuckDuckGoSearchTool)). It has an optional `max_results` parameter (default 10) to limit how many results to retrieve ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=class%20smolagents)). This tool is handy for questions that require external information lookup. *No API key needed.*

- **GoogleSearchTool** – Similar to DuckDuckGo but for Google. (Requires proper API setup to work, as Google’s search isn’t open by default – the exact usage may depend on having an API key or using a custom search API. In many cases, DuckDuckGo is used as it’s easier.)

- **VisitWebpageTool** – Fetches the contents of a webpage given a URL ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=)). Typically used after a search, when the agent wants to open one of the result links to get more detailed info. It likely returns the textual content of the page (perhaps stripped of HTML).

- **SpeechToTextTool** – Converts speech audio to text (ASR) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=SpeechToTextTool)). It likely uses a default model (like OpenAI Whisper or similar) to transcribe audio files. The input might be a path to an audio file or audio bytes, and output is the text transcript.

All these tools (except FinalAnswerTool which is internal) can be added to an agent’s tool list as needed. For example, to make an agent that can search the web and read pages:  
```python
agent = CodeAgent(
    tools=[DuckDuckGoSearchTool(), VisitWebpageTool()],
    model=HfApiModel()
)
```

#### ToolCollection

Sometimes you want to add a whole set of tools at once (for instance, a predefined suite of tools from the Hub or an Anthropic **MCP** server’s tools). `ToolCollection` is a helper class that can load multiple tools and bundle them.

**class `smolagents.ToolCollection(tools: List[Tool])`** – A simple container for a list of tools ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=ToolCollection)). You normally obtain a ToolCollection via its class methods:

- **`ToolCollection.from_hub(collection_slug, token=None, trust_remote_code=False)`** – Load all tools from a Hugging Face Hub *Collection* ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=Tool%20collections%20enable%20loading%20a,tools%20in%20the%20agent%E2%80%99s%20toolbox)) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=)). If you or someone created a HF Collection containing multiple Spaces (each Space is assumed to be a tool), this method fetches each Space as a tool and returns a ToolCollection containing them. 
  - *Parameters:* `collection_slug` (slug name of the collection on HF Hub) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=Parameters)), `token` (if private collection) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=,to%20trust%20the%20remote%20code)), `trust_remote_code` (to allow loading if tools have code) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=,to%20trust%20the%20remote%20code)). 
  - *Returns:* A `ToolCollection` with `tools` attribute list populated with all the loaded tools ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=Returns)).
  - *Example:*  
    ```python
    from smolagents import ToolCollection, CodeAgent
    col = ToolCollection.from_hub("huggingface-tools/chemistry-tools")
    agent = CodeAgent(tools=[*col.tools], model=HfApiModel(), add_base_tools=True)
    ```  
    This would load all tools in the "chemistry-tools" collection and give them to the agent ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=Example%3A)) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=,CodeAgent)). The agent can then use any of those specialized tools to answer questions (along with base tools).

- **`ToolCollection.from_mcp(server_parameters)`** – Connect to an Anthropic MCP (Model-Tool Communication Protocol) server and load all tools it provides ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=)) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=Automatically%20load%20a%20tool%20collection,from%20an%20MCP%20server)). Anthropic’s MCP is a way to serve tools over a standardized interface. You must have the `mcp` Python package and a running MCP server. `server_parameters` would be an instance of something like `mcp.StdioServerParameters` describing how to launch/connect to the server ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=)).
  - This method returns a `ToolCollection` and actually runs a background thread to maintain connection to the server ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=A%20tool%20collection%20instance)) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=,from%20mcp%20import%20StdioServerParameters)).
  - *Example:*  
    ```python
    from smolagents import ToolCollection, CodeAgent
    from mcp import StdioServerParameters
    params = StdioServerParameters(command="uv", args=["--quiet", "pubmedmcp@0.1.3"])
    with ToolCollection.from_mcp(params) as tools:
        agent = CodeAgent(tools=[*tools.tools], model=HfApiModel(), add_base_tools=True)
        answer = agent.run("Find a remedy for hangover.")
        print(answer)
    ```  
     ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=Example%3A)) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=,%2A%2Aos.environ%7D%2C%20%3E%3E%3E))This connects to a hypothetical PubMed tools server and allows the agent to query medical literature for the answer. The context manager ensures the connection is closed after use.

### Agent Types (Data Wrappers)

To support multimodal inputs/outputs and ensure smooth interoperability, `smolagents` defines **Agent Types** – simple wrapper classes around data like text, images, audio, etc. ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=Agent%20Types)) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=These%20types%20have%20three%20specific,purposes)). These wrappers allow the agent to treat different data types uniformly and also ensure that when such data is returned as final answer, it displays nicely (for example, showing an actual image in a Jupyter notebook instead of a byte string).

The key agent types are:

- **AgentText** – Wraps a text string (essentially behaves like a Python `str`) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=AgentText)).
- **AgentImage** – Wraps an image (PIL Image) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=AgentImage)).
- **AgentAudio** – Wraps an audio (e.g., waveform data) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=AgentAudio)).
- *(There could be AgentVideo, etc., but the main ones documented are text, image, audio.)*

Each of these classes stores a `value` (the underlying object: string for AgentText, PIL.Image for AgentImage, torch Tensor or bytes for AgentAudio) and provides:
- `to_raw()` – get the raw underlying object (e.g., the PIL image, or raw text) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=)) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=)).
- `to_string()` – get a string representation of the object ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=)) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=)). For text it’s just the text itself; for an image, it might return a filepath to a saved image (since you can’t inline image binary in text easily); for audio, similarly a filepath or base64 string.
- For AgentImage, there’s also a `save(output_bytes, format=None, **params)` to save image data to a file or bytes buffer with a given format ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=)) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=same%20as%20in%20PIL.Image.save.%20,save)).

These wrappers ensure that if the agent returns, say, an `AgentImage`, it can still be treated like an image in Python (displayed in notebooks) and as a string (e.g., path) for the LLM’s text reasoning ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=,should%20display%20the%20object%20correctly)) ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=)). Usually, you don’t manually create these; the tools or the agent will wrap outputs as needed. But it’s good to be aware of them – for example, if you call an image-generating tool manually, it might return an `AgentImage`.

**Example:**  
If an agent uses an image generation Space via `Tool.from_space` as above, calling `agent.run("Generate an image of ...")` might return an `AgentImage`. You can do:
```python
result = agent.run("Generate an image of a sunset over the ocean.")
if isinstance(result, AgentImage):
    result.to_raw().show()  # This would display the PIL image
    print("Image saved at:", result.to_string())
else:
    print(result)
```  
This way, you handle both cases (if it returned text or an image).

## Advanced Usage and Tutorials

Now that we’ve covered the basics, let’s explore more complex workflows and advanced features of `smolagents`. These scenarios demonstrate how to leverage the library for sophisticated tasks.

### Orchestrating Multiple Agents (Manager Agent)

You can build a system where one agent manages or coordinates others. For example, one agent could break a task into sub-tasks and delegate each to a specialized agent. In `smolagents`, you achieve this by treating sub-agents as tools for the manager agent.

**Scenario:** Build a multi-agent web browser: a *Manager Agent* decides when to use a *Web Search Agent* or a *Code Interpreter* tool to browse and extract info ([Orchestrate a multi-agent system ](https://huggingface.co/docs/smolagents/examples/multiagents#:~:text=In%20this%20notebook%20we%20will,solve%20problems%20using%20the%20web)) ([Orchestrate a multi-agent system ](https://huggingface.co/docs/smolagents/examples/multiagents#:~:text=_______________,Visit%20webpage%20tool)).

**Setup:** Suppose we have:
- A **WebSearchAgent** (could be a `ToolCallingAgent` with `DuckDuckGoSearchTool` and `VisitWebpageTool`) that can search the web and open pages.
- A **CodeAgent** (with Python tool) that can run arbitrary Python for data extraction or calculation.
- A **ManagerAgent** that knows about the above two as its tools.

```python
from smolagents import CodeAgent, ToolCallingAgent, CodeAgent, DuckDuckGoSearchTool, VisitWebpageTool, PythonInterpreterTool, HfApiModel

# Sub-agent 1: Web search agent (will use JSON-style tool calling)
web_agent = ToolCallingAgent(
    tools=[DuckDuckGoSearchTool(), VisitWebpageTool()],
    model=HfApiModel(),
    name="web_agent",
    description="Agent that can search the web and retrieve webpage content."
)
# Sub-agent 2: Code interpreter agent (for running code)
code_agent = CodeAgent(
    tools=[PythonInterpreterTool()],
    model=HfApiModel(),
    name="code_agent",
    description="Agent that can execute Python code for calculations or data parsing."
)

# Manager agent that can call the above agents as tools
manager = CodeAgent(
    tools=[],  # no direct tools, but we'll add managed_agents
    model=HfApiModel(),
    managed_agents=[web_agent, code_agent],  # treat sub-agents as tools
    prompt_templates=None,
    max_steps=10
)
```

Here, we gave each sub-agent a `name` and `description` and put them in `managed_agents` of the manager ([Agents](https://huggingface.co/docs/smolagents/reference/agents#:~:text=,the)). The manager’s LLM will be instructed that it has two special tools available: one called "web_agent" and one called "code_agent", with the given descriptions. When a complex query comes in, the manager can decide to delegate. For instance, if asked *“Find the current weather in Paris and plot the temperature for the next week.”*, the manager might use `web_agent` to search for a weather website, then use `code_agent` to parse and plot the data, then compile the final answer.

**Running the Manager Agent:**
```python
result = manager.run("What is the name of the tallest mountain on Earth and its height in meters?")
print(result)
```
In this run, the manager might use `web_agent` to search that question, find that Mount Everest is the tallest and its height (8848m), and then return the answer. The process is transparent to the user; you just get the final answer, but behind the scenes multiple agents worked together.

**Note:** Multi-agent systems can be tricky. Ensure the descriptions clearly delineate responsibilities, and monitor the `manager.logs` to see how it decides to use sub-agents. This pattern can scale to many specialized agents (tools, calculators, translators, etc.), but complexity increases. Always test with various queries to fine-tune the agent behaviors.

### Writing and Using Custom Tools

While built-in tools cover common needs, a power of `smolagents` is easily integrating custom tools. We saw how to use the `@tool` decorator in the Quickstart. Let’s expand on best practices for custom tools:

1. **Keep Tools Focused**: Each tool should do one thing well (e.g., a calculator, a database lookup, a translator). This makes it easier for the LLM to decide when to use which tool. Describe the tool’s function and inputs clearly in its docstring ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=,key.%20This%20is)).

2. **Provide Examples (if complex)**: If the tool usage is not obvious, sometimes adding an example in the description (as part of `description` attribute or docstring) can help the LLM. However, be careful not to confuse it; the system prompt assembled by `smolagents` will list the tool name, description, inputs, outputs. So clarity is key.

3. **Test Tools Standalone**: Before adding a new tool to an agent, call it directly to ensure it works as expected. For instance, if you wrote a `WeatherTool(city:str) -> str` that calls a weather API, test `WeatherTool().forward(city="Paris")` to see if it returns a reasonable result. Agents rely on tools behaving correctly.

4. **Security**: If a tool performs critical or irreversible actions (like sending an email, executing shell commands, etc.), consider adding confirmations or safe-guards. The LLM might misuse a tool if not guided properly. At this time, `smolagents` assumes tools are safe to call.

**Example – Custom Tool with State:** Suppose we want a tool that loads a large dataset once and allows querying it. We can subclass `Tool` and use `setup()` to load the data:

```python
from smolagents import Tool

class CountryInfoTool(Tool):
    name = "country_info"
    description = "Provides information about countries. Input: 'country' (name of the country). Output: basic info like capital and population."
    inputs = {"country": {"type": str, "description": "Name of the country"}}
    output_type = str

    def __init__(self):
        super().__init__()
        self.data = None

    def setup(self):
        # Load data (this will run only on first use)
        import pandas as pd
        self.data = pd.read_csv("country_stats.csv")  # assume a CSV with columns Country, Capital, Population, etc.

    def forward(self, country: str) -> str:
        # Ensure data is loaded
        if self.data is None:
            self.setup()
        info = self.data[self.data["Country"].str.lower() == country.lower()]
        if info.empty:
            return f"No data for {country}."
        capital = info["Capital"].iloc[0]
        population = info["Population"].iloc[0]
        return f"The capital of {country} is {capital}, with a population of {population:,}."
```

Now we can use this `CountryInfoTool` in an agent:
```python
agent = ToolCallingAgent(tools=[CountryInfoTool()], model=HfApiModel())
agent.run("What is the capital of France and its population?")
```
The first time the agent calls `country_info`, the `setup()` method will load the dataset ([Tools](https://huggingface.co/docs/smolagents/reference/tools#:~:text=You%20can%20also%20override%20the,tool%2C%20but%20not%20at%20instantiation)), then `forward` will return the info. Subsequent calls won’t reload the data. This showcases using internal state in tools.

We could also push this tool to the Hub so others can use it:
```python
tool = CountryInfoTool()
tool.push_to_hub("myusername/country-info-tool", private=False)
```
Anyone could then do `load_tool("myusername/country-info-tool")` to use it.

### Secure Code Execution with E2B

By default, `CodeAgent` runs code in the local Python environment, but with restrictions to prevent malicious commands ([smolagents/docs/source/en/guided_tour.md at main · huggingface/smolagents · GitHub](https://github.com/huggingface/smolagents/blob/main/docs/source/en/guided_tour.md#:~:text=By%20default%2C%20the%20execution%20is,in%20what%20can%20be%20executed)). For an extra layer of security, especially if you’re letting untrusted LLMs run code, you can use [E2B](https://e2b.dev) integration to run code in a sandboxed environment ([smolagents/docs/source/en/guided_tour.md at main · huggingface/smolagents · GitHub](https://github.com/huggingface/smolagents/blob/main/docs/source/en/guided_tour.md#:~:text=there%20is%20a%20regular%20Python,code%20generated%20by%20the%20agent)).

**Steps to use E2B:**
1. Sign up on e2b.dev (if needed) and get an API key.
2. Set the environment variable `E2B_API_KEY` with your key ([smolagents/docs/source/en/guided_tour.md at main · huggingface/smolagents · GitHub](https://github.com/huggingface/smolagents/blob/main/docs/source/en/guided_tour.md#:~:text=there%20is%20a%20regular%20Python,code%20generated%20by%20the%20agent)).
3. Initialize your `CodeAgent` with `use_e2b_executor=True` ([smolagents/docs/source/en/guided_tour.md at main · huggingface/smolagents · GitHub](https://github.com/huggingface/smolagents/blob/main/docs/source/en/guided_tour.md#:~:text=there%20is%20a%20regular%20Python,code%20generated%20by%20the%20agent)).

Example:
```python
import os
from smolagents import CodeAgent, DuckDuckGoSearchTool, HfApiModel

os.environ["E2B_API_KEY"] = "your_e2b_api_key_here"
agent = CodeAgent(
    tools=[DuckDuckGoSearchTool()],
    model=HfApiModel(),
    use_e2b_executor=True
)
agent.run("Using Python, what is the 10,000th prime number?")
```
In this run, if the LLM decides to write Python code to find the 10,000th prime, that code will be executed in a remote sandbox provided by E2B, not on your local machine. The agent will receive the output or error as usual and continue reasoning. The switch to E2B is seamless – apart from setting the flag and API key, you use the agent in the same way.

**Important:** E2B may introduce some latency due to remote execution. Also, ensure your key and usage abide by any limits or costs of the service. The benefit is the peace of mind that even if the LLM goes off-script (e.g., tries to delete files), it’s happening in a contained environment.

### Telemetry and Debugging Agent Runs

When developing complex agents, you’ll want insight into their decision process. We already mentioned `agent.logs` and `agent.replay()` for inspecting steps. For more systematic tracking, you might consider logging each interaction or even using external monitoring.

`smolagents` doesn’t have a built-in cloud telemetry service, but it encourages good logging:
- Use `verbosity_level` on agents (e.g., set to DEBUG) to get more console output during runs if running in a script.
- Utilize `step_callbacks`. For example, you could append a callback that logs each step to a file or prints a summary:
  ```python
  def log_step(step):
      print(f"Step {step['number']}: Tool={step['tool']} – {step['observation']}")
  agent = CodeAgent(..., step_callbacks=[log_step])
  ```
- Use the `write_memory_to_messages` to get a chat transcript-like view of the run, which can be easier to read.

For a friendly UI to inspect runs, the `GradioUI` (mentioned earlier) not only allows interaction but also shows the conversation (though it might not show internal reasoning by default, just the final Q&A).

In summary, treat your agent as you would a program: test it with sample inputs, check its logs, adjust prompts or tools as needed, and iterate.

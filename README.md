#  Tracing
This repository provides examples and instructions for tracing LangChains, which are sequences of actions and observations performed by the LangChain library. Tracing allows you to capture and analyze the execution flow of your LangChains, which can be helpful for debugging and understanding the behavior of your code.

#  Recommended Tracing Methods
There are two recommended ways to trace your LangChains:

Setting the LANGCHAIN_TRACING environment variable to "true".
Using a context manager with tracing_enabled() to trace a particular block of code.
Please note that if the LANGCHAIN_TRACING environment variable is set, all code will be traced, regardless of whether or not it's within the context manager.

#  Example Usage
Here's an example that demonstrates the usage of tracing in LangChain:
```
import os
import asyncio

from langchain.agents import AgentType, initialize_agent, load_tools
from langchain.callbacks import tracing_enabled
from langchain.llms import OpenAI

# To run the code, make sure to set OPENAI_API_KEY and SERPAPI_API_KEY
llm = OpenAI(temperature=0)
tools = load_tools(["llm-math", "serpapi"], llm=llm)
agent = initialize_agent(
    tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION, verbose=True
)

questions = [
    "Who won the US Open men's final in 2019? What is his age raised to the 0.334 power?",
    "Who is Olivia Wilde's boyfriend? What is his current age raised to the 0.23 power?",
    "Who won the most recent formula 1 grand prix? What is their age raised to the 0.23 power?",
    "Who won the US Open women's final in 2019? What is her age raised to the 0.34 power?",
    "Who is Beyonce's husband? What is his age raised to the 0.19 power?",
]

# Setting the environment variable to enable tracing
os.environ["LANGCHAIN_TRACING"] = "true"

# Both of the agent runs will be traced because the environment variable is set
agent.run(questions[0])
with tracing_enabled() as session:
    assert session
    agent.run(questions[1])

# Now, we unset the environment variable and use a context manager.
if "LANGCHAIN_TRACING" in os.environ:
    del os.environ["LANGCHAIN_TRACING"]

# Here, we are writing traces to "my_test_session"
with tracing_enabled("my_test_session") as session:
    assert session
    agent.run(questions[0])  # this should be traced

agent.run(questions[1])  # this should not be traced

# The context manager is concurrency-safe
task = asyncio.create_task(agent.arun(questions[0]))  # this should not be traced
with tracing_enabled() as session:
    assert session
    tasks = [agent.arun(q) for q in questions[1:3]]  # these should be traced
    await asyncio.gather(*tasks)

await task
```
#  Output and Analysis
When tracing is enabled, LangChain will produce trace information for each executed chain, showing the actions, observations, and thoughts within the chain. The trace can be used for analysis and debugging purposes.

Please note that in the example above, there are warnings related to failed connections to a server. These warnings are specific to the example and may not be relevant to your own environment.

#  License
This project is licensed under the MIT License. See the LICENSE file for details.

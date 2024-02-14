1. # create new .py file with code found below
2. # install ollama
3. # install model you want “ollama run mistral”
4. conda create -n autogen python=3.11
5. conda activate autogen
6. which python
7. python -m pip install pyautogen
7. ollama run mistral
8. ollama run codellama
9. # open new terminal
10. conda activate autogen
11. python -m pip install litellm
12. litellm --model ollama/mistral
13. # open new terminal
14. conda activate autogen
15. litellm --model ollama/codellama

### Code used:


    install ollama, cf. https://github.com/jmorganca/ollama (for Windows, use WSL2, follow install on Linux & WSL2)

run ollama server

    ollama serve
    open new terminal

install models you want

    ollama pull mistral
    ollama pull codellama

install python modules

    conda create -n autogen python=3.11
    conda activate autogen
    pip install pyautogen
    pip install litellm

run litellm

    litellm
    open new terminal
    conda activate autogen
    create new ollama-autogen.py file with code found below

finally, run demo

    python ollama-autogen.py

Code used:

import autogen

config_list_mistral = [
{
'base_url': "http://0.0.0.0:8000",
'api_key': "NULL",
'model': "mistral"
}
]

config_list_codellama = [
{
'base_url': "http://0.0.0.0:8000",
'api_key': "NULL",
'model': 'ollama/codellama"
}
]

llm_config_mistral={
"config_list": config_list_mistral,
}

llm_config_codellama={
"config_list": config_list_codellama,
}

coder = autogen.AssistantAgent(
name="Coder",
llm_config=llm_config_codellama
)

user_proxy = autogen.UserProxyAgent(
name="user_proxy",
human_input_mode="NEVER",
max_consecutive_auto_reply=10,
is_termination_msg=lambda x: x.get("content", "").rstrip().endswith("TERMINATE"),
code_execution_config={"work_dir": "web"},
llm_config=llm_config_mistral,
system_message="""Reply TERMINATE if the task has been solved at full satisfaction.
Otherwise, reply CONTINUE, or the reason why the task is not solved yet."""
)

task="""
Write a python script to output numbers 1 to 100 and then the user_proxy agent should run the script
"""

user_proxy.initiate_chat(coder, message=task)








Since version 0.1.24, Ollama is compatible with OpenAI API and you don't even need litellm anymore.
You know just need to install Ollama and run ollama serve then, in another terminal, pull the models you want to use eg ollama pull codellama and ollama pull mistral, then install autogen as before :

$ conda create -n autogen python=3.11
$ conda activate autogen
$ pip install pyautogen

Finally, the python code has to be slightly changed:

    The base URL must be changed from http://0.0.0.0:8000 to http://localhost:11434/v1 (which replace the litellm URL by the OpenAI compatible Ollama one)
    The code_execution_config in the autogen.UserProxyAgent() call must be changed to code_execution_config={"work_dir": "web", "use_docker": False}, as use_docker has been changed to True by default in recent versions (otherwise you must install autogen as a docker container)
    Also, the name of the model in the config has changed for me: if you pull eg codellama:7b-code-q4_K_M (ie a specific tag) or mistral (no tag is implicitely tag latest), then model must be "codellama:7b-code-q4_K_M" and "mistral:latest" in the config lists eg:

config_list_codellama = [
    {
        'base_url': "http://localhost:11434/v1",
        'api_key': "fakekey",
        'model': "codellama:7b-code-q4_K_M",
    }
]

The whole code for me is:

import autogen

# direct access to Ollama since 0.1.24, compatible with OpenAI /chat/completions
BASE_URL="http://localhost:11434/v1"

config_list_mistral = [
    {
        'base_url': BASE_URL,
        'api_key': "fakekey",
        'model': "mistral:latest",
    }
]

config_list_codellama = [
    {
        'base_url': BASE_URL,
        'api_key': "fakekey",
        'model': "codellama:7b-code-q4_K_M",
    }
]

llm_config_mistral={
    "config_list": config_list_mistral,
}

llm_config_codellama={
    "config_list": config_list_codellama,
}

user_proxy = autogen.UserProxyAgent(
    name="user_proxy",
    human_input_mode="NEVER",
    max_consecutive_auto_reply=10,
    is_termination_msg=lambda x: x.get("content", "").rstrip().endswith("TERMINATE"),
    code_execution_config={"work_dir": "web", "use_docker": False},
    llm_config=llm_config_mistral,
    system_message="""Reply TERMINATE if the task has been solved at full satisfaction.
Otherwise, reply CONTINUE, or the reason why the task is not solved yet."""
)

coder = autogen.AssistantAgent(
    name="Coder",
    llm_config=llm_config_codellama
)

task="""
Write a python script that lists the number from 1 to 100
"""

user_proxy.initiate_chat(coder, message=task)


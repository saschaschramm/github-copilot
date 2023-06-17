# GitHub Copilot Chat
This is an analysis of the [GitHub Copilot Chat extension](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot-chat) for Visual Studio Code.

> Analysis of version 0.3.2023060801

## Prompts
When the user selects `file1.py` and enters a question e.g. `hello` Copilot Chat generates 3 prompts. The prompts are send to the endpoint `https://copilot-proxy.githubusercontent.com/v1/chat/completions`.

### Prompt 1: Search for information sources
This prompt asks the model to list the two best information sources for the users question "hello".

[Prompt](copilot-chat/0.3.2023060801/prompt1.json)

### Prompt 2: Generate answer to question
This prompt passes the question to the model and includes the information sources.

[Prompt](copilot-chat/0.3.2023060801/prompt2.json)

### Prompt 3: Generate a follow-up question
This prompt asks the model to generate a follow-up question to the previous question "hello".

[Prompt](copilot-chat/0.3.2023060801/prompt3.json)

## Slash Commands
Instead of typing a question in the inline chat, you can also use the following slash commands:

* `/vscode` - Questions about VS Code
* `/tests` - Generate unit tests for the selected code
* `/fix` - Propose a fix for the problems in the selected code
* `/explain` - Explain how the selected code works
* `/ext` - Ask about VS Code extension development.
* `/help` - General help about GitHub Copilot







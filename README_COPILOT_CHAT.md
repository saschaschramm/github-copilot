# GitHub Copilot Chat
This is an analysis of the [GitHub Copilot Chat extension](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot-chat) for Visual Studio Code.

> Analysis of version 0.1.2023052602

## Prompts
When the user enters a question e.g. `hello` Copilot Chat generates 3 prompts. The prompts are send to the endpoint `https://copilot-proxy.githubusercontent.com/v1/chat/completions`.

### Search for information sources
[Prompt](copilot-chat/prompt1.json):
Prompt asks the model to list the two best information sources for the users question "hello".

### Generate answer to question
[Prompt](copilot-chat/prompt2.json):
Prompt includes the information sources and the question "hello".

### Follow-up question
[Prompt](copilot-chat/prompt3.json):
Prompt asks the model to generate a follow-up question to the previous question "hello".

## Slash Commands
Instead of typing a question in the inline chat, you can also use the following slash commands:
* `/vscode` - Questions about VS Code
* `/tests` - Generate unit tests for the selected code.
* `/simplify` - Simplify the selected code.
* `/fix` - Propose a fix for the problems in the selected code.
* `/explain` - Explain step-by-step how the selected code works.
* `/ext` - Ask about the VS Code extension development.
* `/help` - General help about GitHub Copilot




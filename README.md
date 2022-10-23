# Copilot 

This is an analysis of the [Github Copilot extension](https://github.com/features/copilot) for VS Code.

Under macOS the `VS Code Extensions` are located in the following directory:
```
~/.vscode/extensions
```

## Prompts
The Github Copilot extension uses two types of prompts.
### Prompt 1
We start with the simplest case with only one file.

* filename: `file1.py`  
* file content: `# Print hello world`

In this case, the extension generates the following prompt:
```
# Path: file1.py
# Print hello, world
```
The path to the file is part of the prompt.

### Prompt 2
Now let's consider a slightly more complex two-file case.

* filename: `file1.py`  
* file content: `# Print hello, world`

* filename: `file2.py`
* file content: `# Print he`

Prompt
```
# Path: file2.py
# Compare this snippet from file1.py:
# # Print hello, world
# Print he"
```
Files with similar content are also included in the prompt.

## Communication
### Language model
The extension contains a function `fetchWithParameters`. The following object is passed to this function:
``` Json
{
    "prompt": {
        "prefix": "# Path: file2.py\n# Print he",
        "suffix": "",
        "isFimEnabled": false,
        "promptElementRanges": [
            {
                "kind": "PathMarker",
                "start": 0,
                "end": 17
            },
            {
                "kind": "BeforeCursor",
                "start": 17,
                "end": 27
            }
        ]
    },
    "languageId": "python",
    "repoInfo": 0,
    "ourRequestId": "XXX",
    "engineUrl": "https://copilot-proxy.githubusercontent.com/v1/engines/copilot-codex",
    "count": 1,
    "uiKind": "ghostText",
    "postOptions": {
        "stream": true,
        "n": 1,
        "temperature": 0,
        "extra": {
            "language": "python",
            "next_indent": 0
        },
        "stop": [
            "\n"
        ]
    }
}
```

To generate a completion, the extension sends a POST request to the endpoint `https://copilot-proxy.githubusercontent.com/v1/engines/copilot-codex/completions`. The payload is derived from the above object:
``` Json
{
    "prompt": "# Path: file2.py\n# Compare this snippet from file1.py:\n# # Print hello, world\n# Print he",
    "suffix": "",
    "max_tokens": 500,
    "temperature": 0,
    "top_p": 1,
    "n": 1,
    "stop": [
        "\n"
    ],
    "logprobs": 2,
    "stream": true,
    "extra": {
        "language": "python",
        "next_indent": 0
    }
}
```

After sending the request, the endpoint returns the following response:
```Json
{
    "id": "XXX",
    "model": "cushman-ml",
    "created": 123,
    "choices": [
        {
            "text": "",
            "index": 0,
            "finish_reason": null,
            "logprobs": {
                "tokens": [
                    "1"
                ],
                "token_logprobs": [
                    -0.41597173
                ],
                "top_logprobs": [
                    {
                        "1": -0.41597173,
                        "10": -2.2335434
                    }
                ],
                "text_offset": [
                    41
                ]
            }
        }
    ]
}
```
### Telemetry
The Github Copilot extension sends the following telemetry data to the endpoint `https://dc.services.visualstudio.com`:
```Json
{
    "properties": {
        "languageId": "python",
        "beforeCursorWhitespace": "false",
        "afterCursorWhitespace": "true",
        "promptChoices": "{\"used\":{\"BeforeCursor\":9,\"PathMarker\":8},\"unused\":{\"LanguageMarker\":10}}",
        "promptBackground": "{\"used\":[],\"unused\":[]}",
        "gitRepoInformation": "pending",
        "engineName": "copilot-codex",
        "isMultiline": "false",
        "blockMode": "parsing",
        "isCycling": "false",
        "headerRequestId": "XXX",
        "github_copilot_advanced_debug_showScores": "true",
        "github_copilot_enable_*": "true",
        "github_copilot_enable_yaml": "false",
        "github_copilot_enable_plaintext": "true",
        "github_copilot_enable_markdown": "true",
        "github_copilot_inlineSuggest_enable": "true",
        "copilot_build": "7011",
        "copilot_buildType": "prod",
        "copilot_trackingId": "XXX",
        "editor_version": "vscode/1.72.1",
        "editor_plugin_version": "copilot/1.53.7011",
        "client_machineid": "XXX",
        "client_sessionid": "XXX",
        "copilot_version": "copilot/1.53.7011",
        "common_extname": "copilot",
        "common_extversion": "1.53.7011",
        "VSCode.ABExp.Features": "XXX",
        "abexp.assignmentcontext": "XXX",
        "fileType": "python",
        "userKind": "unknown",
        "timeBucket": "XXX",
        "endpoint": "completions",
        "uiKind": "ghostText",
        "temperature": "0",
        "n": "1",
        "stop": "[\"\\n\"]",
        "logit_bias": "null",
        "choiceIndex": "0",
        "completionId": "XXX",
        "created": "XXX",
        "serverExperiments": "",
        "deploymentId": "XXX"
    },
    "measurements": {
        "promptCharLen": 41,
        "promptEndPos": 24,
        "documentLength": 24,
        "delayMs": 0,
        "promptComputeTimeMs": 187,
        "timeSinceIssuedMs": 335402,
        "numTokens": 2,
        "compCharLen": 7,
        "numLines": 1,
        "meanLogProb": -0.832473,
        "meanAlternativeLogProb": -1.4269974,
        "confidence": 0.8161905891940561,
        "quantile": 0.5980612740800664,
        "timeSinceDisplayedMs": 17526,
        "timeout": 15,
        "insertionOffset": 24,
        "trackedOffset": 24,
        "relativeLexEditDistance": 0,
        "charEditDistance": 0,
        "completionLexLength": 1,
        "foundOffset": 15,
        "lexEditDistance": 0,
        "stillInCodeHeuristic": 1
    }
}
```
### Results
We have evaluate the copilot model `cushman-ml` with the [HumanEval](https://github.com/openai/human-eval) dataset. Out of 164 programming problems, the model can solve `55.49%`.

| Model name | Pass@1 | Date | Comment
| - | - | - | - |
| code-cushman-001 | 32.93 | 2022-10-23 | https://openai.com/api/
| code-davinci-002 | 46.95 | 2022-10-23 | https://openai.com/api/
| cushman-ml | 55.49 | 2022-10-23 | Copilot


Completions of the evaluation run: [2022-10-23-samples-cushman-ml.jsonl](2022-10-23-samples-cushman-ml.jsonl)
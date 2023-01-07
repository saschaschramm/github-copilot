# Copilot 

This is an analysis of the [Github Copilot extension](https://github.com/features/copilot) for VS Code.

Under macOS the `VS Code Extensions` are located in the following directory:
```
~/.vscode/extensions
```

## Prompts
The Github Copilot extension generates three types of prompts.
### Prompt 1: Single file
We start with the simplest case with only one file `file1.py`.

* filename: `file1.py`  
* file content: `# Print hello world`

In this case, the extension generates the following prompt:
``` Python
# Path: file1.py
# Print hello, world
```
The path to the file is part of the prompt.

### Prompt 2: Multiple files
Now let's consider a slightly more complex two-file case where file `file2.py` is edited.

* filename: `file1.py`  
* file content: `# Print hello, world`

* filename: `file2.py`
* file content: `# Print he`

In this case, the extension generates the following prompt:
``` Python
# Path: file2.py
# Compare this snippet from file1.py:
# # Print hello, world
# Print he
```
Files with similar content are also included in the prompt.

### Prompt 3: Fill in the middle
Copilot supports [Fill in the Middle](https://arxiv.org/pdf/2207.14255.pdf). That means the extension sends the code before and after the cursor position to the model.
* filename: `file3.py`  
* file content: `# Test prefix\n\n# Test suffix`
* cursor position: Between  `# Test prefix\n` and `\n# Test suffix`

In this case, the extension generates the following prompt:
``` Python
# Path: file3.py
# Test prefix
```

And the following suffix:
``` Python
# Test suffix
```

## Communication
### Language model

To generate a completion, the extension sends a POST request to the endpoint `https://copilot-proxy.githubusercontent.com/v1/engines/copilot-codex/completions`:
``` Json
{
    "method": "POST",
    "headers": {
        "Authorization": "XXX",
        "X-Request-Id": "XXX",
        "Openai-Organization": "github-copilot",
        "VScode-SessionId": "XXX",
        "VScode-MachineId": "XXX",
        "Editor-Version": "vscode/1.74.2",
        "Editor-Plugin-Version": "copilot/1.65.7705",
        "OpenAI-Intent": "copilot-ghost"
    },
    "json": {
        "prompt": "# Path: file3.py\n# Test prefix",
        "suffix": "# Test suffix",
        "max_tokens": 500,
        "temperature": 0,
        "top_p": 1,
        "n": 1,
        "stop": [
            "\n"
        ],
        "nwo": "XXX",
        "stream": true,
        "extra": {
            "language": "python",
            "next_indent": 0,
            "trim_by_indentation": true
        }
    },
    "timeout": 30000
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
            "logprobs": null
        }
    ]
}
```
### Telemetry
The Github Copilot extension sends [telemetry data](telemetry.json) to the endpoint `https://dc.services.visualstudio.com`:

## Deeper Analysis

### Min prompt chars
The length of the prompt has to be >= 10 characters before the prompt is sent to the model.

``` Javascript
if ((_ > 0 ? n.length : d) < t.MIN_PROMPT_CHARS)
    return t._contextTooShort;
```

### File information
The following information is collected about the file being edited:
``` Javascript
const m = {
    uri: d.toString(), // The absolute path of the file
    source: t, // Content of the file
    offset: n, // The offset of the cursor
    relativePath: u, // The relative path of the file
    languageId: p // The programming language of the file
}
```

### Extract prompt
The input of the function `extractPrompt` is the following object:
``` Javascript
{
    "uri": {
        "$mid": 1,
        "fsPath": "/Users/XXX/file2.py",
        "external": "file:///Users/XXX/file2.py",
        "path": "/Users/XXX/file2.py",
        "scheme": "file"
    },
    "fileName": "/Users/XXX/file2.py",
    "isUntitled": false,
    "languageId": "python",
    "version": 7,
    "isClosed": false,
    "isDirty": true,
    "eol": 1,
    "lineCount": 2
}
```

The extensions remembers the files that have been accessed before. The files are sorted by access time:
``` Javascript
o = (0, i.sortByAccessTimes)(e.get(a.TextDocumentManager).textDocuments);
```

``` Javascript
o = [
    {
        "uri": {
            "$mid": 1,
            "fsPath": "/Users/XXX/file2.py",
            "external": "file:///Users/XXX/file2.py",
            "path": "/Users/XXX/file2.py",
            "scheme": "file"
        },
        "fileName": "/Users/XXX/file2.py",
        "isUntitled": false,
        "languageId": "python",
        "version": 6,
        "isClosed": false,
        "isDirty": true,
        "eol": 1,
        "lineCount": 1
    },
    {
        "uri": {
            "$mid": 1,
            "fsPath": "/Users/XXX/file1.py",
            "external": "file:///Users/XXX/file1.py",
            "path": "/Users/XXX/file1.py",
            "scheme": "file"
        },
        "fileName": "/Users/XXX/file1.py",
        "isUntitled": false,
        "languageId": "python",
        "version": 1,
        "isClosed": false,
        "isDirty": false,
        "eol": 1,
        "lineCount": 1
    }
```

If there are more than 20 files or when the content of all files has a length > 200,000 then the rest of the files are ignored.
``` Javascript
if (r.length + 1 > 20 || s + i.getText().length > 2e5)
    break;
```

The above object is reduced to the following properties:
``` Javascript
[
    {
        "uri": "file:///Users/XXX/copilot/file1.py",
        "relativePath": "file1.py",
        "languageId": "python",
        "source": "# Print hello, world"
    }
]
```

The output of the function `extractPrompt` is
``` Javascript
{
    "type": "prompt",
    "prompt": {
        "prefix": "# Path: file2.py\n# Compare this snippet from file1.py:\n# # Print hello, world\n# Print he",
        "suffix": "",
        "isFimEnabled": false,
        "promptElementRanges": [
            {
                "kind": "PathMarker",
                "start": 0,
                "end": 17
            },
            {
                "kind": "SimilarFile",
                "start": 17,
                "end": 78
            },
            {
                "kind": "BeforeCursor",
                "start": 78,
                "end": 88
            }
        ]
    },
    "trailingWs": "",
    "promptChoices": {
        "used": {},
        "unused": {}
    },
    "computeTimeMs": 6,
    "promptBackground": {
        "used": {},
        "unused": {}
    }
}
```

## Copilot Performance
We have evaluated the copilot model `cushman-ml` with the [HumanEval](https://github.com/openai/human-eval) dataset. Out of 164 programming problems, the model can solve `56.10%`.

| Model name | Pass@1 | Date | Comment
| - | - | - | - |
| code-cushman-001 | 32.93% | 2022-10-23 | https://openai.com/api/
| code-davinci-002 | 46.95% | 2022-10-23 | https://openai.com/api/
| cushman-ml | 56.10% | 2022-10-23 | Copilot


Completions of the evaluation run: [2022-10-23-samples-cushman-ml.jsonl](2022-10-23-samples-cushman-ml.jsonl)
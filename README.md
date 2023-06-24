# Github Copilot

This is an analysis of the [Github Copilot extension](https://github.com/features/copilot) for Visual Studio Code.

Under macOS the `VS Code Extensions` are located in the following directory:
```
~/.vscode/extensions
```

> Analysis of version 1.92.177

For an analysis of Copilot Chat see [README_COPILOT_CHAT.md](README_COPILOT_CHAT.md).

## Prompts
The Github Copilot extension generates three types of prompts.
### Prompt 1: Single file
We start with the simplest case with only one file `file1.py`.

* filename: `file1.py`  
* file content: `# Print hello, world`

If the user presses enter after `# Print hello, world`, the extension generates the following prompt:
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
* file content: `# Test prefix\n# Test suffix`

If the user presses enter after `# Test prefix`, the extension generates the following prompt:

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
To generate a completion, the extension sends a [POST request](copilot/1.92.177/prompt.json) to the endpoint `https://copilot-proxy.githubusercontent.com/v1/engines/copilot-codex/completions`.

After sending the request, the endpoint returns the following [response](copilot/1.92.177/completion.json).

### Telemetry
The Github Copilot extension sends [telemetry data](copilot/1.92.177/telemetry.json) to the endpoint `https://dc.services.visualstudio.com`:

## Deeper Analysis

### Vocabulary
The extension contains two vocabulary files

| Filename | Vocabulary Size | Comment
| --- | --- | --- |
| `vocab_cushman001.bpe` | 50,276 | This vocabulary is based on the GPT-2 vocabulary |
| `vocab_cushman002.bpe` | 100,000 | This vocabulary is new and not based on the GPT-2 vocabulary anymore |

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

### Neighbor Files
The extension remembers the files that have been accessed before. The function `getNeighborFiles` calls the function `truncateDocs`. Input of the function `truncateDocs` are the [files sorted by access time](copilot/1.92.177/truncated-input.json). 

When the combined size of all files exceeds 200,000, any additional files will be disregarded. The function `truncateDocs` returns a [truncated list of files](copilot/1.92.177/truncated-output.json).

## Copilot Performance
We have evaluated the copilot model `cushman-ml` with the [HumanEval](https://github.com/openai/human-eval) dataset. Out of 164 programming problems, the model can solve `56.10%`.

| Model name | Pass@1 | Date | Comment
| - | - | - | - |
| code-cushman-001 | 32.93% | 2022-10-23 | https://openai.com/api/
| code-davinci-002 | 46.95% | 2022-10-23 | https://openai.com/api/
| cushman-ml | 56.10% | 2022-10-23 | Copilot


Completions of the evaluation run: [2022-10-23-samples-cushman-ml.jsonl](2022-10-23-samples-cushman-ml.jsonl)
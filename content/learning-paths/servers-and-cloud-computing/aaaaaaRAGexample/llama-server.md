---
title: Access the chatbot using the OpenAI-compatible API 
weight: 4

### FIXED, DO NOT MODIFY
layout: learningpathall
---

You can use the `llama.cpp` server program and submit requests using an OpenAI-compatible API.
This enables applications to be created which access the LLM multiple times without starting and stopping it. You can also access the server over the network to another machine hosting the LLM.

One additional software package is required for this section. Install `jq` on your computer using:

```bash
sudo apt install jq -y
```

The server executable has already compiled during the stage detailed in the previous section, when you ran `make`. 

Start the server from the command line, it listens on port 8080:

```bash
./llama-server -m dolphin-2.9.4-llama3.1-8b-Q4_0_8_8.gguf --port 8080
```

## Use curl

You can access the API using the `curl` command. 

In another terminal, use a text editor to create a file named `curl-test.sh` with the commands below: 

```bash
curl http://localhost:8080/v1/chat/completions -H "Content-Type: application/json"   -d '{
    "model": "any-model",
    "messages": [
      {
        "role": "system",
        "content": "You are a coding  assistant, skilled in programming."
      },
      {
        "role": "user",
        "content": "Write a hello world program in C++."
      }
    ]
  }' 2>/dev/null | jq -C
```

The `model` value in the API is not used, you can enter any value. This is because there is only one model loaded in the server. 

Run the script:

```bash
bash ./curl-test.sh
```

The `curl` command accesses the LLM and you see the output:

```output
{
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "content": "#include <iostream>\n\nint main() {\n    std::cout << \"Hello, World!\" << std::endl;\n    return 0;\n}",
        "role": "assistant"
      }
    }
  ],
  "created": 1726252907,
  "model": "any-model",
  "object": "chat.completion",
  "usage": {
    "completion_tokens": 30,
    "prompt_tokens": 33,
    "total_tokens": 63
  },
  "id": "chatcmpl-wh33d82OqWKibRF0s7waublCpl9YytkI"
}
```

In the returned JSON data you see the LLM output, including the content created from the prompt. 

## Use Python

You can also use a Python program to access the OpenAI-compatible API.

Create a Python `venv`:

```bash
python -m venv pytest
source pytest/bin/activate
```

Install the OpenAI Python package:
```bash
pip install openai==1.45.0
```

Use a text editor to create a file named `python-test.py` with the content below: 

```python
from openai import OpenAI

client = OpenAI(
        base_url='http://localhost:8080/v1',
        api_key='no-key'
        )

completion = client.chat.completions.create(
  model="not-used",
  messages=[
    {"role": "system", "content": "You are a coding assistant, skilled in programming.."},
    {"role": "user", "content": "Write a hello world program in C++."}
  ],
  stream=True,
)

for chunk in completion:
  print(chunk.choices[0].delta.content or "", end="")
```

Run the Python file (make sure the server is still running):

```bash
python ./python-test.py
```

You see the output generated by the LLM:

```output
Here's a simple Hello World program in C++:

```cpp
#include <iostream>

int main() {
    std::cout << "Hello, World!" << std::endl;
    return 0;
}

This program includes the standard input/output library, `iostream`. It defines a `main` function, which is the entry point                of the program. Inside `main`, `std::cout` is used to output the string "Hello, World!" to the console, and then `std::endl` is used to print a new line. The `return 0;` statement indicates that the program exited successfully
```

You can continue to experiment with different large language models and write scripts to try them.
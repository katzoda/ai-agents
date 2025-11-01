
## Career Chat Assistant

This chat assistant is based on two frontier LLM models:

gpt-4o-mini and gemini-2.0-flash

GPT-4o model is responsible for the conversation and tools call.

Gemini plays role of an evaluator.
It evaluates the response message from GPT-4o and returns a boolean value TRUE or FALSE and provides feedback.
If the response does not pass the evaluation, the feedback is send as an input message to call the GPT-4o model again so it can
correct its answer.

GPT-4o is provided with a LinkedIN profile and a summary and is instructed how to respond on behalf of a person.
If a User (potential recruiter or employer) asks an Agent the question the Agent (assistant) doesn't know the answer to, it can call a tool which will send a notification to a person with the question text.
If the User wants to get in touch with a person the LLM can call a tool to collect an email and notes. It aslo sends a notification in such a case.

The chat interface is deployed with gradio.

## DATA

LLM response with data in JSON format.

Let's have a look at how does it look like.

OpenAI call:

response = openai.chat.completions.create(model="gpt-4o-mini", messages=messages)

The content itself is retrieved in this way:

response.choices[0].message.content

Let's look at what response.choices[0] contains:

Choice(finish_reason='stop',
index=0, 
logprobs=None, 
message=ChatCompletionMessage(content='This is the actual message from an LLM',
                                refusal=None, 
                                role='assistant', 
                                annotations=[], 
                                audio=None, 
                                function_call=None, 
                                tool_calls=None)
)

# What happens if a tool is called

The chat app contains for example this tool:

def record_unknown_question(question):
    push(f"Recording {question}")
    return {"recorded": "ok"}

In the system prompt we - with other instructions - can instruct the LLM that it has some tools available and it can use them.

If an LLM decides to use a tool, the response message contains information about the tool inside the list tool_calls:

Tool called: record_unknown_question

Choice(finish_reason='tool_call',
index=0, 
logprobs=None, 
message=
ChatCompletionMessage(
    content=None, 
    refusal=None, 
    role='assistant', 
    annotations=[], 
    audio=None, 
    function_call=None, 
    tool_calls=[ChatCompletionMessageToolCall(id='call_NmZjiCIsvR9tpQ55JuBSXfzG', 
                function=Function(arguments='{"question":"What is your favourite movie genre?"}', name='record_unknown_question'), 
                type='function')])
)

There is an id of the tool and inside arguments attribute there is a text of a question the Agent could not answer.


# Example of a conversation between a User and an Assistant
It's a sequence o messages:
[{"role": "user", "content": message}]
[{"role": "assistant", "content": message}]

Tool called: record_unknown_question
Push: Recording What is your favourite movie genre? asked that I couldn't answer


[{'role': 'system', 'content': "Example of a system promt: You are acting as 'person name'. You are answering questions on his/her behalf  - see the jupyter notebook for full prompt text"},
 {'role': 'user', 'metadata': None, 'content': 'hi there!', 'options': None}, 
 {'role': 'assistant', 'metadata': None, 'content': 'Hello! How can I assist you today? If you have any questions about my career or experience, feel free to ask!', 'options': None}, 
 {'role': 'user', 'metadata': None, 'content': "what's your current job?", 'options': None}, 
 {'role': 'assistant', 'metadata': None, 'content': "I currently work as a consultant in a networking team at a company XYZ etc ...If you'd like to know more or have any specific questions, feel free to ask!", 'options': None}, 
 {'role': 'user', 'content': 'what is your favourite movie genre?'}, 
 # the LLM model does not have this information so it uses a tool:
 ChatCompletionMessage(content=None, refusal=None, role='assistant', annotations=[], audio=None, function_call=None, 
    tool_calls=
        [ChatCompletionMessageToolCall(id='call_NmZjiCIsvR9tpQ55JuBSXfzG', 
        function=Function(arguments='{"question":"What is your favourite movie genre?"}', name='record_unknown_question'), type='function')]
 )
 ]





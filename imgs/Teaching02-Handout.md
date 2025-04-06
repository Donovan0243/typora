# Teaching 02 Handout

## 0. Introduction 🚀

In today's session, we will learn:

1. **Deploying Open-Source LLMs with Ollama** – Running models locally using Docker and Ollama (Optional to the Project).
2. **Prompts Engineering** – Introduce different ways to customize and control LLMs response. Learn different useful patterns in prompt engineering.
3. **Practical Applications** –  Introduce prompt structure in code. How to do prompt engineering programmatically.

## 1. Deploying Open-Source LLMs with Ollama 🦙

Ollama is a tool that makes it easy to run large language models (LLMs) locally on your own machine. It supports a variety of open-source models like LLaMA, Mistral, and Qwen, and wraps everything into a simple command-line interface.

To get started with Ollama, follow these steps:

### Step 1: Connect to VM instance

- Go to Google Cloud Console, and start your VM instance. 🔗 https://console.cloud.google.com/compute

- Open your VS Code platform and connect to your Google VM instance.

<img src="https://raw.githubusercontent.com/Donovan0243/typora/main/imgs/image-20250331131322375.png" alt="image-20250331131322375" style="zoom: 67%;" />



### Step 2: Install Docker (Optional)

Ollama runs models using Docker containers. If you haven’t installed Docker, follow the previous step on Teaching01.

To check if Ollama is installed successfully, you can run:

```sh
docker --version
```

For a better understanding, we can see the current structure in docker:

<img src="https://raw.githubusercontent.com/Donovan0243/typora/main/imgs/image-20250331162749994.png" alt="image-20250331162749994" style="zoom: 43%;" />

### Step 3: **Stop and Remove the Previous OpenWebUI Container**

In default, OpenWebUI natively supports integration with [Ollama](https://ollama.com), making it easy to deploy both services together using a single docker-compose.yml file. This setup automatically connects OpenWebUI with Ollama and requires no manual configuration for integration.



Before proceeding, we need to remove any previously installed and running OpenWebUI container to avoid conflicts.

Open your terminal and run the following command to check the status of currently running containers:

```sh
docker ps
```

If the open-webui container is running, stop and remove it using the following commands:

```sh
docker stop open-webui
docker rm open-webui
```

### Step 4: Launch Services Using Docker Compose

Then, change into the directory where the docker-compose.yml file is located:

```
cd open-webui
```

Launch Services Using Docker Compose, it will run the config base on `docker-compose.yaml`

```
docker compose up -d
```

This command will automatically download the required images and start two containers in detached mode: `open-webui` and `ollama`

Run the following command again to confirm that both services are up:

```sh
docker ps
```

In your browser, access to Open Web UI website.

By default, Ollama does not include any models. Therefore, when you open OpenWebUI, the model list will be empty.

### Step 5: Download a New LLM Model

#### 5.1 Access the Ollama Container

```shell
docker exec -ti ollama bash
```

> [!TIP]
>
> This command allow you enter the Ollama container Linux environment, like entering a normal terminal operation.
>
> Once you enter, you will see the prompt change to:
>
> ```sh
> root@<container-id>:/#
> ```

To download a new model from the Ollama library, use the following command:

You can see what models are available by checking the Ollama website at: 🔗 https://ollama.com/library

<img src="https://raw.githubusercontent.com/Donovan0243/typora/main/imgs/image-20250331143027974.png" alt="image-20250331143027974" style="zoom:43%;" />

Install by following command:

```shell
ollama pull qwen2.5:1.5b
```



![image-20250331143425968](https://raw.githubusercontent.com/Donovan0243/typora/main/imgs/image-20250331143425968.png)

#### 5.3 List All Available LLM Models in Ollama

If you want to check the current list of downloaded models, you can run:

```shell
ollama list
```

![image-20250331144152301](https://raw.githubusercontent.com/Donovan0243/typora/main/imgs/image-20250331144152301.png)

#### 5.4 Delete a Model (Optional)

If you need to remove an LLM model, you can do so with:

```shell
ollama delete ${model_name}
```

### Step 6:  Call Ollama Models via Open WebUI

At this point, you should be able to see and interact with the models installed in Ollama directly from the OpenWebUI interface.

<img src="https://raw.githubusercontent.com/Donovan0243/typora/main/imgs/image-20250405235011508.png" alt="image-20250405235011508" style="zoom:67%;" />



Unlike using commercial API services (such as OpenAI or Claude), this setup allows you to call models **freely without relying on API tokens or incurring per-request charges**. The model is running entirely within your local or virtual machine.

> [!WARNING]
>
> However, you may quickly notice that the model’s **response speed is quite slow**, even when using a relatively lightweight model such as a `1.5B` parameter version.
>
> This performance bottleneck is due to the limitations of our current virtual machine configuration:
>
> <img src="https://raw.githubusercontent.com/Donovan0243/typora/main/imgs/image-20250405235733136.png" alt="image-20250405235733136" style="zoom:70%;" />
>
> Therefore, **we will set up a new virtual machine with GPU support**. The Ollama backend should run on this GPU-enabled instance, while OpenWebUI can remain on a lightweight machine.

### Step 7: Calling Ollama with code

Activate the virtual environment we set up during Teaching01.

```sh
conda activate ${your env name}
```

> [!TIP]
>
> If you don’t remember the name of the environment from Teaching01, run the following command to list all available environments:
>
> ```shell
> conda env list
> ```

Open the `call_llm.py` file, this time let's try to connect Ollama instead of 

``` python
# from dotenv import load_dotenv
import os
import openai

# load_dotenv()
# API_KEY = os.getenv('QWEN_KEY')
# Initialize OpenAI client for OPEN AI

client = openai.OpenAI(
    api_key='xxxxx',
    base_url="http://localhost:11434/v1"
)

# Make the first API call using the new syntax
response = client.chat.completions.create(
    model="qwen2.5:1.5b",
    messages=[{"role": "user", "content": "Hello, who are you?"}]
)
# Print the response
print(response.choices[0].message.content)

```

- You don’t need an API key when using the local Ollama API.

- Make sure the base URL matches your running Ollama service, usually http://localhost:11434.

- Use a model name that has already been pulled into your instance, such as `qwen2.5:1.5b`

![image-20250331155117442](https://raw.githubusercontent.com/Donovan0243/typora/main/imgs/image-20250331155117442.png)









## 2. Deploy Ollama on other VM separately.

### Step 1: Create a new VM with GPU

Follow the same procedure as described in **Teaching Section 01** under: `1.2 Firewall Setting `  and  `1.3 Setting Up VM Instance`.

However, make the following adjustments:

- For Firewall Setting, this time you need to **allow incoming traffic on port `11434`**, which is the default port used by Ollama’s backend service.
- When creating the new VM:
  - **Instance name:** ollama-gpu-instance
  - **Machine type:** Choose one with a GPU (e.g., `NVIDIA T4`)

### Step 2: Install Ollama

#### 3.1 Give Your User Docker Permissions

By default, only root users can run Docker commands. To avoid typing sudo every time, add your user to the Docker group:
`${USER}` is an environment variable that automatically refers to the currently logged-in user.

```sh
sudo usermod -aG docker ${USER}
```

> [!TIP]
>
> Alternatively, if you want to be explicit, you can check your username with:
>
> ```shell
> whoami
> ```
>
> It will output your username, for example:
>
> ```shell
> webui
> ```
>
> Then you can replace `$USER` with this username and execute the command:
>
> ```shell
> sudo usermod -aG docker webui
> ```

The execute：

```
newgrp docker
```

The newgrp command is a Linux/Unix shell command used to switch your current group ID during a session — without logging out or restarting the terminal.

#### 3.2 Pull the Ollama Docker Image

execute the following command:

```sh
sudo docker pull ollama/ollama
```

![image-20250331142818340](https://raw.githubusercontent.com/Donovan0243/typora/main/imgs/image-20250331142818340.png)

#### 3.3 Create docker-compose.yml to Manage Ollama

```sh
nano docker-compose.yml
```

and paste the following config into it and saved.

```yml
version: '3.8'

services:
  ollama:
    image: ollama/ollama
    container_name: ollama
    ports:
      - "11434:11434"
    volumes:
      - ollama-data:/root/.ollama
    restart: always
volumes:
  ollama-data:
```

<img src="https://raw.githubusercontent.com/Donovan0243/typora/main/imgs/image-20250331142302387.png" alt="image-20250331142302387" style="zoom:67%;" />

### Step 3: Startup Ollama

```shell
docker compose up -d
```

It starts the container and runs Ollama in the background. Then confirm:

```sh
docker ps
```

You should see a container called ollama running, listening on port 11434.

![image-20250331142624411](https://raw.githubusercontent.com/Donovan0243/typora/main/imgs/image-20250331142624411.png)

### Step 4: Download a New LLM Model

#### 4.1 Access the Ollama Container

```shell
docker exec -ti ollama bash
```

> [!TIP]
>
> This command allow you enter the Ollama container Linux environment, like entering a normal terminal operation.
>
> Once you enter, you will see the prompt change to:
>
> ```shell
> root@<container-id>:/#
> ```

#### 4.2 Download Models

To download a new model from the Ollama library, use the following command:

You can see what models are available by checking the Ollama website at: 🔗 https://ollama.com/library

<img src="https://raw.githubusercontent.com/Donovan0243/typora/main/imgs/image-20250331143027974.png" alt="image-20250331143027974" style="zoom:43%;" />

Install by following command:

```shell
ollama pull qwen2.5:1.5b
```



![image-20250331143425968](https://raw.githubusercontent.com/Donovan0243/typora/main/imgs/image-20250331143425968.png)

### Step 5: Connecting Ollama to Open WebUI

Next, open your **Open WebUI** interface by visiting  `http://${your external ip}:3000` 

![image-20250331164314620](https://raw.githubusercontent.com/Donovan0243/typora/main/imgs/image-20250331164314620.png)

In the WebUI, Enter your Ollama address as  `http://${your external ip}:11434` , Leave the **API Key** field empty.

<img src="https://raw.githubusercontent.com/Donovan0243/typora/main/imgs/image-20250331175044727.png" alt="image-20250331175044727" style="zoom:50%;" />

If the connection to Ollama is successful, the LLM models you’ve downloaded in Ollama will appear in the interface.

<img src="https://raw.githubusercontent.com/Donovan0243/typora/main/imgs/image-20250331174942104.png" alt="image-20250331174942104" style="zoom:50%;" />

### Step 5: Send a Model Call to Ollama

Activate the virtual environment we set up during Teaching01.

```sh
conda activate ${your env name}
```

> [!TIP]
>
> If you don’t remember the name of the environment from Teaching01, run the following command to list all available environments:
>
> ```shell
> conda env list
> ```

Open the `call_llm.py` file, this time let's try to connect Ollama instead of 

``` python
# from dotenv import load_dotenv
import os
import openai

# load_dotenv()
# API_KEY = os.getenv('QWEN_KEY')
# Initialize OpenAI client for OPEN AI

client = openai.OpenAI(
    api_key='xxxxx',
    base_url="http://localhost:11434/v1"
)

# Make the first API call using the new syntax
response = client.chat.completions.create(
    model="qwen2.5:1.5b",
    messages=[{"role": "user", "content": "Hello, who are you?"}]
)
# Print the response
print(response.choices[0].message.content)

```

- You don’t need an API key when using the local Ollama API.

- Make sure the base URL matches your running Ollama service, usually http://localhost:11434.

- Use a model name that has already been pulled into your instance, such as `qwen2.5:1.5b`

![image-20250331155117442](https://raw.githubusercontent.com/Donovan0243/typora/main/imgs/image-20250331155117442.png)

### Step 7: Connecting Ollama to Open WebUI

Before connecting Ollama to Open WebUI, we need to configure the firewall rules to allow internet access to the Ollama instance:

1. Go to **VPC Network** ➡️ **Firewall**.

2. Click on the firewall rule we previously created in **Teaching01**, named `allow-open-webui-3000`.

3. Click **Edit**, and add port `11434` after `3000` in the **Protocols and ports** field.
4. Save the changes.

<img src="https://raw.githubusercontent.com/Donovan0243/typora/main/imgs/image-20250331175748120.png" alt="image-20250331175748120" style="zoom:50%;" />



## 3. Prompt Engineering 🏗️

Large Language Models (LLMs) like GPT, Claude, and Qwen are typically **pre-trained as general-purpose models**.
 This means: without any customization, they produce **generic**, **broad**, and sometimes **unfocused** responses.

To make them work well for **specific tasks** or **domains**, we need ways to adapt them.

There are **four major ways** to customize and control LLMs:

<img src="https://raw.githubusercontent.com/Donovan0243/typora/main/imgs/enhance_llm.png" alt="enhance_llm" style="zoom:67%;" />

> [!TIP]
>
> To see more differences, you can read: 
>
> 🔗 https://medium.com/@jainpalak9509/use-llms-pre-training-fine-tuning-rag-and-prompt-engineering-564d5670f44d [Image Source]

Among these, **Prompt Engineering** stands out as the **simplest, most flexible, and most accessible method** — no retraining, no infrastructure, just better instructions.

In this module, we’ll introduce a set of **prompt design patterns** — proven techniques for crafting prompts that produce better, more reliable results.

We’ll organize them by **purpose**:

- 🤔 Understand you better (personas, few-shot, etc.)
- 🧠 Think better (reasoning, CoT, refinement)
- ✍️ Output better (structure, filters, fact-check)
- 🔁 Interact better (tools, workflows, evolution)

![Prompt Engineering](./prompt_engineering.png)

### 3.1 Make the Model Understand You 🤔

#### 1. Persona Pattern

The **Persona Pattern** helps you get more realistic and useful responses by telling the model to **“act as”** a specific role.

It packs a lot of meaning into just one sentence. Instead of writing long instructions, you don’t need to explain every detail about how that role should respond. Just say:

> 👉 **“Act as a [role]...”** 

and the model will take on that identity using its built-in knowledge.

**💡 Example prompt:**

```markdown
- Act as Persona X / You are X
- Perform task Y

Examples:

- You are a critical expert in biology. Your task is to validate the scientific accuracy and clarity of the following response. Provide specific comments on any errors, misleading phrasing, or unsupported claims.
- Act as a computer. Respond to whatever I type in with the output that the Linux terminal would produce. Ask me for the first command.
```



#### 2. Audience Persona Pattern

The **Audience Persona Pattern** helps you tell the model **who the output is for** — not who the model is, but who it's talking to.

This pattern is powerful when you want the same content to be explained in different ways, depending on the listener’s knowledge, age, role, or personality.

The format is simple:

>  👉 **“Assume the audience is [someone]...”**


 This helps the model adjust its explanation, tone, and word choices based on the audience's background.

**💡 Example prompt:**

```markdown
Explain X to me. 
Assume that I am Persona Y.

Examples:

- Explain large language models to me. Assume that I am a student without any IT background. 
- Explain quantum physics to me. Assume I have no background in physics, just general interest.
```

You don’t need to define special rules. Just set the audience, and the model will adjust its answer based on what that audience might understand or care about.



#### 3. Question Refinement Pattern

The Question Refinement Pattern helps you ask **better questions** by letting the model rewrite your input to make it clearer, deeper, or more useful.

> 👉 **“Whenever I ask a question, suggest a better version and ask me if I’d like to use it instead.”**

This pattern works well because large language models have seen **many ways** of asking similar questions. 

This process will help you:

- Makes your prompts more thoughtful
- Helps you reflect on what you **actually** want to ask

**💡 Example prompt:**

```markdown
Examples:

- From now on, whenever I ask a question, suggest a better version of the question and ask me if I would like to use it instead.

Tailored Examples:

- From now on, whenever I ask a question, Rewrite the question to make it more specific, include relevant keywords, and clarify any vague terms — but keep the original meaning.
```

 Use this pattern when:

- Your initial question feels too general
- You want to improve prompt quality
- You’re not sure how to phrase something clearly

It’s a small change that often leads to much better answers.



#### 4. Few-shot Examples

The Few-shot Examples pattern is a way to teach the model by example — instead of writing rules, you show what to do.

The basic idea is: 

> 👉 **Give the model a few input–output examples, then ask it to continue the pattern.**

**💡 Example prompt:**

```markdown
- example 1
Input: I love this book.  
Sentiment: Positive  

Input: The movie was okay but a bit long.  
Sentiment: Neutral  

Input: The food was terrible.  
Sentiment:
```

This method works because language models are good at recognizing patterns. If your examples are clear, consistent, and focused, the model will likely follow them.

**🔍 Tips for writing effective few-shot examples:**

- Make sure your examples are **detailed** and **unambiguous**.
- Use **descriptive prefixes** (like “Input”, “Sentiment”) instead of vague ones (“Input”, “Output” alone may be too generic).
- You can include **intermediate steps** like “Think → Action” to guide multi-step reasoning.
- Format matters! Keep the pattern clean and easy to follow (e.g. one step per line).



#### 5. Meta Language Creation Pattern

The Meta Language Creation Pattern lets you teach the model a new shorthand or custom "language" that you define — and then use it in your prompts. The reason why we do this is because writing everything in full sentences can be slow, repetitive, or unclear. A well-designed shorthand is faster and more consistent.

> **👉 First, explain the pattern, then use your new meta-language in prompts**

**💡 Example prompt:**

```markdown
- example 1
When I say Sydney,3 -> Melbourne,2.
I mean: I will go from Sydney to Melbourne, staying 3 days in Sydney and 2 in Melbourne.

Sydney,0 -> Melbourne,1 -> Brisbane,4
```

or

```markdown
- example 2
When I say Sydney,3 -> Melbourne,2.
I mean: I will go from Sydney to Melbourne, staying 3 days in Sydney and 2 in Melbourne.
Provide a explanation of this route:Sydney,0 -> Melbourne,1 -> Brisbane,4
```

or

```python
prompt +='Generate new HTTP requests based on API rules and web page responses,put the URL you generated directly in []'
prompt += f'Question: What is the official gene symbol of LMP10?\n'
prompt += f'[{url_1}]->[{call_1}]\n'
prompt += f'[{url_2}]->[{call_2}]\n'
prompt += f'Answer: PSMB10\n\n'
```

The model now understands your format and can generate the trip plan accordingly.

This is especially useful when:

- You do a **repetitive task** (like trip planning or classification).
- You want to define a **standard input format** for yourself or your team.
- You want the model to **interpret inputs consistently**, without needing full natural language.



### 3.2 Enhance Model Think Better 🧠

#### 1. Outline Expansion Pattern

The **Outline Expansion Pattern** helps you work around the size limits of large language models by generating content **step by step, based on an outline**.

Instead of asking the model to write a full document all at once, you first ask it to build a **structured outline**. Then, you ask it to expand one bullet point at a time.

**💡 Example prompt:**

```markdown
- Examples
Act as an outline expander. Generate a bullet point outline based on the input that I give you and then ask me for which bullet point you should expand on. 
Each bullet can have at most 3-5 sub bullets. The bullets should be numbered using the pattern [A-Z].[i-v]. Create a new outline for the bullet point that I select.  
At the end, ask me for what bullet point to expand next. Ask me for what to outline.


- Tailored Examples:

Act as an expert teaching assistant. I am writing a handout for a class on [topic].  
Generate a bullet point outline that organizes the main ideas clearly.  
Use a numbering format like [A-Z].[i-v]. Each bullet should be concise, and no section should have more than 5 sub-points.  
After you generate the outline, ask me which bullet point to expand next.

```

This pattern is useful for:

- Writing long documents (e.g. articles, handbooks, books)
- Creating structured code or project plans
- Generating consistent pieces that fit together

This pattern gives you a map to follow — and lets the model fill in the pieces as you go.



#### 2. Recipe Pattern

The Recipe Pattern is used when you **know part of a solution**, but not the whole thing. You give the model the **goal** and a few **known steps**, and ask it to **fill in the missing parts**.

Think of it like making a recipe: You have a few ingredients or steps, and the model completes the rest.

> 👉   **I would like to achieve X** 
>
> ​	**I know that I need to perform steps A,B,C** 
>
> ​	**Provide a complete sequence of steps for me** 
>
> ​	**Fill in any missing steps**

**💡 Example prompt:**

```markdown
- I would like to build a RAG System. I know that I need to use LangChain and OpenWebUI. Provide a complete sequence of steps for me. Fill in any missing steps.

- Example 2:
I want to answer the following question: [specific question]
Currently, I already have the following components or modules:  
- [modoule 1]  
- [modoule 2]  
- ...

Please help me complete the rest of the process.  
List all the missing steps or components needed to achieve the task effectively.  
```

✅ You can use this pattern when:

- You know **some** of the steps, but not all.
- You want the model to **fill in a process**, **suggest missing pieces**, or **bridge gaps**.

You can combine it with other patterns, like **Meta Language Creation**, to express the known and unknown steps in a compact way.



#### 3. Cognitive Verifier Pattern

The Cognitive Verifier Pattern helps the model answer complex questions more accurately by **breaking them into smaller sub-questions**, answering each one, and combining the results.

>  👉 **When asked a question, generate helpful sub-questions. Then combine their answers into a final answer.**

This works well because:

- Language models have seen many **related questions** in their training.
- Sub-questions help the model **think more clearly** and reduce mistakes.
- You get **more transparency** into how the answer was produced.

**💡 Example prompt:**

```markdown
Examples:

- When asked a question, break it down into helpful sub-questions.Answer each sub-question clearly. Then combine the answers into a final response. If needed, the system will fetch external info for each sub-question.
```

Use this pattern when:

- The original question is **broad or vague**
- You want **better logic and transparency**
- You want to **guide the model through a reasoning process**

This pattern is great for learning, planning, and answering open-ended or fuzzy questions.



#### 4. Chain of Thought Pattern

Chain of Thought (CoT) Prompting improves the model’s reasoning by asking it to **show its thinking step by step**, before giving an answer. This is similar to how we were taught in school to “show your work.” When a model explains its reasoning, it’s more likely to reach a correct and logical answer.

> 👉  Give examples that include both the **reasoning** and the **final answer**.
>
> ​	Ask the model to follow the same format when answering your new question.

 **💡Example comparison:**

<img src="https://raw.githubusercontent.com/Donovan0243/typora/main/imgs/image" alt="COT" style="zoom:67%;" />

> Image Source: [Wei et al. (2022)](https://arxiv.org/abs/2201.11903)

🧠 Why it works:

- Step-by-step reasoning helps the model **stay on track**.
- If it gets the early steps right, the later steps often follow correctly.
- This leads to more accurate answers in tasks that require **logic, calculation, or common sense**.

Use this pattern when:

- The task involves **math, logic, physics**, or multiple steps.
- You want the model to be **transparent** and avoid guessing.
- You want to teach the model a **thinking process**, not just an answer.

Even just add  **"let's think step by step." ** can greatly improve the quality of output.

> [!TIP]
>
> For more details, you can read papers 🔗: https://arxiv.org/abs/2201.11903



#### 5. Flipped Interaction Pattern

The **Flipped Interaction Pattern** tells the model to **ask you questions**, instead of you asking it.

This is helpful when:

- You don’t know what to ask.
- You want the model to gather details from you before giving a result.
- You want to be quizzed or guided through a process.

>  **👉 Ask me questions about [topic] until you have enough information to [goal]. Ask me the first question.**

**💡 Example Prompt:**

```markdown
- Example
You are acting as a medical assistant agent.
Ask me questions about my symptoms until you have enough information to suggest a possible diagnosis or recommend next steps.  
Ask me one question at a time. If needed, use medical reasoning to decide what to ask next.
Start by asking the first question.
```

After collecting enough answers, the model provides the full result — in this case, a **possible diagnosis of disease or medical advice**.

Use this pattern when you need **smart question-asking** before getting to a final answer.



#### 6. Alternative Approaches Pattern

The Alternative Approaches Pattern is used to help the model **brainstorm multiple ways** to solve a problem or complete a task — not just give one answer. Instead of saying “do this,” you say:

> 👉 **List alternative ways to achieve this goal.**

**💡 Example Prompt:**

```markdown
- For every prompt I give you, If there are alternative ways to word a prompt that I give you, list the best alternate wordings . Compare/contrast the pros and cons of each wording. 

- You are a URL constructor for a multi-source biomedical RAG agent.
	Given the following API URL template:  
	[https://eutils.ncbi.nlm.nih.gov/entrez/eutils/{esearch|efetch|esummary}.fcgi?retmax=10&db={gene|snp|omim}&retmode=json&sort=relevance&{term|id}={term|id}]
	And a user query:  {query}
	Please generate at least 3 alternative URLs that could retrieve relevant data using different endpoint and 	 db combinations.
```

Use this pattern when:

- You want to explore ideas
- You're unsure which prompt design will work best

You can also combine this pattern with others (like **Ask for Input**) to make the interaction smoother and more controlled.



### 3.3 Manage Output Structure and Accuracy ✍️

#### 1. Template Pattern

The **Template Pattern** helps you get model outputs in the **exact format you want**, by giving it a **template with placeholders** to fill in. Think of it like giving instructions to an assistant:

> 👉 **Here’s the format — now fill it in.**

**💡 Example Prompt:**

```markdown
example 1:

- Extract and synthesize key information from the provided academic document.  
	Use the template below for your output. CAPITALIZED WORDS are placeholders for content. Please preserve the structure and 		
	formatting of the template:
	<TITLE OF PAPER>, <MAIN CLAIM>, <KEY EVIDENCE>, <FIELD OF STUDY>, <RELEVANCE SCORE 1-5>, <SUMMARY IN ONE SENTENCE>

example 2:

- Please create a grocery list for me to cook macaroni and cheese from scratch, garlic bread, and marinara sauce from scratch. I am going to provide a template for your output . <placeholder> are my placeholders for content. Try to fit the output into one or more of the placeholders that I list. Please preserve the formatting and overall template that I provide.   

This is the template:   
Aisle <name of aisle>: <item needed from aisle>, <qty>
```

Use this pattern when:

- You need **clean, consistent formatting**
- You’re building tools, content, forms, or structured data
- You want precise control over output style

It’s one of the most reliable ways to **get what you want, where you want it** in the output.



#### 2. Semantic Filter Pattern

The Semantic Filter Pattern helps you use a language model to **remove or keep certain types of information** from a piece of text — based on meaning, not just keywords.It’s like saying:

> **👉 Act as a filter. Remove anything that fits this description.**

This is powerful for cleaning up sensitive or unwanted content, such as:

- Removing **dates** from a public version of a document
- Stripping out **personal or medical details**
- Keeping only parts of text that match a specific meaning

💡 Examples:

```markdown
- Example 1:
Given the retrieved knowledge snippet, remove vague, duplicate, or irrelevant information and present a clean knowledge statement.

- Example 2:
- Filter this information to remove any personally identifying information or information that could potentially be used to re-identify the person. 
```

⚠️ Note: This is a powerful tool, but not perfect. For privacy or security use cases, always have a **human double-check the output.**



#### 3. Ask for Input Pattern

The **Ask for Input Pattern** is a simple way to tell the model:

> 👉 **Wait for my instruction before doing anything.** 

Use this when:

- You want to stay in control of the session
- You’re designing tools, games, or multi-turn workflows
- You want the model to wait for **user input**

 Without this pattern, the model might **invent its own task** and generate an answer immediately — even if you haven’t given it anything yet.

**💡 Example Prompt:**

```markdown
- From now on, I am going to cut/paste email chains into our conversation. You will summarize what each person's points are in the email chain. You will provide your summary as a series of sequential bullet points. At the end, list any open questions or action items directly addressed to me. My name is Jill Smith. Ask me for the first email chain.
```

✅ With this pattern:

> Model replies:
> *“What is the first email you’d like me to summarize for?”*

❌ Without this pattern:

> Model might make something up:
> *“Sure! Let’s start with summarising email chain...”*

This small trick gives you clean, predictable results — and avoids wasted output.



#### 4. Fact Check List Pattern

The **Fact Check List Pattern** helps you identify and verify the key facts a language model includes in its output — instead of just assuming the text is correct. The idea is simple:

> 👉 **After you generate a response, list the core facts it contains.**

 These are the facts you can then check yourself or with other tools.

**💡 Prompt format:**

```markdown
- Whenever you output text, generate a set of facts that are contained in the output. The set of facts should be inserted at the end of the output. The set of facts should be the fundamental facts that could undermine the veracity of the output if any of them are incorrect.
```

You can then review the facts, compare them to the full text or follow up on any that seem doubtful.

This pattern **doesn’t guarantee correctness**, but it:

- Helps you **spot questionable claims**
- Builds a habit of **active reading and verification**
- Makes it easier to **review AI-generated content responsibly**

You can use this pattern when accuracy matters — like in education, journalism, decision-making, or planning.



#### 5. Tail Generation Pattern

The Tail Generation Pattern helps the model **remember the goal of a task across long conversations**. It works by attaching a short reminder — a “tail” — at the **end of every output**.

**💡 Example prompt:**

```markdown
- Act as an outline expander. When I give you a bullet point, expand it into sub-points. At the end of your output, ask me which bullet point you should expand next.
```

"ask me which bullet point you should expand next" is the `tail`.

This prompt will remind the model to keep output `which bullet point you should expand next ? `That ensures that it doesn't forget the task and will keep going.

You can use this pattern when:

- You have a **repeating structure** across turns
- You want to avoid the model “drifting” from the task
- You need to **remind the model what to expect next**
- Reinforces **instructions without repeating full prompts**

The tail is like a **smart sticky note** — small, but keeps everything on track.



### 3.4 Enable Complex and Interactions 🔁

#### 1. ReAct Prompting

ReAct Prompting (short for **Reasoning + Acting**) teaches the model to solve a problem by combining **step-by-step thinking** with **external actions**, like web searches or tool calls.

It’s useful when a task:

- The model needs fresh data or real-time info
- You want to simulate a multi-step reasoning process
- You're integrating LLMs with external tools or APIs

> 👉  **How it works:**
>
> 1. The model **thinks** about what it needs
> 2. It **calls an action** (like `SEARCH`, `VIDEO`, or `CALCULATE`)
> 3. It uses the **result** to keep reasoning
> 4. It repeats until the task is done
>

💡 **Prompt format:**

```markdown
- example 1

Task: What is the chromosome location of BRCA1?

Think: I need to find gene info for BRCA1.
Action: [https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi?retmax=10&db=gene&retmode=json&sort=relevance&term=BRCA1]
Observation: I received gene ID 672.

Think: Now I want to get full details about this gene.
Action: [https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esummary.fcgi?retmax=10&db=gene&retmode=json&id=672]
Observation: Gene 672 corresponds to BRCA1, located on chromosome 17.

Answer: BRCA1 is located on chromosome 17. It is involved in DNA repair and associated with breast cancer risk.
```

Even if the Action is not actually done by LLMs, you can do those actions in other pipelines and give feedback on the result.

> [!TIP]
>
> For more details, you can read papers 🔗: https://arxiv.org/abs/2210.03629



#### 2. Menu Actions Pattern

The Menu Actions Pattern lets you create a custom set of **reusable commands** (like a menu) for a language model to follow — just like clicking options in software.

Instead of repeating long instructions every time, you define a format once, and reuse it with short commands.

> 👉 How it works:
>
> 1. You define a menu of actions.
> 2. Each action follows a **specific keyword + structure**.
> 3. The model understands what to do when it sees your custom command.

💡 **Example Prompt:**

```markdown
You are a controller agent for a biomedical question-answering system that can access multiple NCBI data sources.

You will follow the menu commands below:

- Whenever I type: `use DB gene`, you will switch the search to NCBI's Gene database.
- Whenever I type: `use DB snp`, you will switch to the SNP database.
- Whenever I type: `use DB omim`, you will switch to the OMIM (genetic disorder) database.

Start by asking me for the first action.
```

You can use this pattern when:

- You perform **repetitive prompt tasks**
- You want to build a **tool-like workflow** using prompts
- You need consistent behavior across a process or team

Think of it as building your own “program” inside a conversation — one keyword at a time.



#### 3. Using LLM to Grade Each Other

This pattern uses large language models to **evaluate and score** the output of prompts — either their own output or another model’s. It solves a real problem: You need a way to **check if the outputs still meet your standards**, especially at scale.

> 👉 **How it works:**
>
> 1. Give the model an input + output pair.
> 2. Provide a few examples of what makes an answer good or bad.
> 3. Ask the model to grade future outputs using that same structure.
>

**💡 Example Prompt:**

```markdown
- example 1

Input: What diseases are associated with BRCA1?
Output: “BRCA1 is located on chromosome 17 and is linked to breast cancer.”
Explanation: Correctly extracts the location and disease
Grade: 10/10

Output: “BRCA1 is linked to breast cancer.”
Explanation: Relivant disease is correct, but missing the location detail.
Grade: 7/10
```

You can use this pattern, especially if you want to:

- Reduces the need for human reviewers
- Works across different models or changing data
- Helps track prompt quality over time
- Can be combined with retry logic or thresholds (e.g., only retry if score < 7)

This pattern is like building a **feedback loop** into your prompt engineering workflow — making the system more self-aware and maintainable.



#### 4. Combining Patterns

**Combining Patterns** means using two or more prompt patterns together to solve more complex tasks. This is one of the most important skills in advanced prompt engineering.

**💡 Example combos:**

Case 1: For example when using LLM to grade each other, you can combine with:

- **Few-shot Examples** (to teach scoring)
- **Persona Pattern** (e.g., “Act as a prompt critic”, make it more critical thinking.)

Case 2: When you have no idea which options should be asked, you can design as:

- **Alternative Approaches** (gives multiple options)

- **Ask for Input** (asks you to pick)

Case 3: You want LLM to assist you in writing a paper outlines

- **Outline Expansion** (Build outline for you)
- **Menu Actions** (define the action to expand or write each section)

> [!TIP]
>
> If you are insteresting, you can read relevant papers
>
>  🔗: https://arxiv.org/abs/2302.11382
>
>  🔗: https://arxiv.org/pdf/2303.07839



## **4. Prompt Structure in Code** 🧑🏻‍💻

When using a language model in code (e.g., with `Ollama` or `OpenAI`), prompts are passed in using a **list of messages**. Each message has a **role** and **content**.

In OpenAI SDK, Each message follows this format:

```python
messages = [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Tell me a joke."}
]
```

There are three roles:

- `"system"` — sets the behavior and personality of the assistant
- `"user"` — the actual input/question from the user
- `"assistant"` — can show prior model responses if you want to simulate chat history

Here's a complete example showing how to structure and manage prompts in code using a multi-turn conversation.

```python
# from dotenv import load_dotenv
import os
import openai

# load_dotenv()
# API_KEY = os.getenv('QWEN_KEY')
# Initialize OpenAI client for OPEN AI

client = openai.OpenAI(
    api_key='xxxxx',
    base_url="http://localhost:11434/v1"
)

messages = [
    {
        "role": "system",
        "content": "You are Qwen2.5, created by Alibaba Cloud. You are a helpful assistant who always responds concisely and clearly."
    },
    {
        "role": "user",
        "content": "Can you explain what a large language model is, like I'm five years old?"
    }
  # ......
]

# Make the first API call using the new syntax
response = client.chat.completions.create(
    model="qwen2.5:1.5b",
    messages=messages
)

reply = response.choices[0].message.content
print("Assistant:", reply)

# append assistant's answers to messages to form a conversation history
messages.append({"role": "assistant", "content": reply})

# then do the second round of questions ...
user_input = "Can you tell me a fun fact about that city?"
messages.append({"role": "user", "content": user_input})

#Second invocation of the model (the context already contains the historical dialogue)
response = client.chat.completions.create(
    model="qwen2.5:1.5b",
    messages=messages
)

# The second round of answers
reply = response.choices[0].message.content
print("Assistant:", reply)

# append assistant's answers again
messages.append({"role": "assistant", "content": reply})
```

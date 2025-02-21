import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# ✨ Enterprise Features

Features here are behind a commercial license in our `/enterprise` folder. [**See Code**](https://github.com/BerriAI/litellm/tree/main/enterprise)

:::info

[Get Started with Enterprise here](https://github.com/BerriAI/litellm/tree/main/enterprise)

:::

Features: 
- [ ] Content Moderation with LlamaGuard 
- [ ] Content Moderation with Google Text Moderations 
- [ ] Tracking Spend for Custom Tags
 
## Content Moderation with LlamaGuard 

Currently works with Sagemaker's LlamaGuard endpoint. 

How to enable this in your config.yaml: 

```yaml 
litellm_settings:
   callbacks: ["llamaguard_moderations"]
   llamaguard_model_name: "sagemaker/jumpstart-dft-meta-textgeneration-llama-guard-7b"
```

Make sure you have the relevant keys in your environment, eg.: 

```
os.environ["AWS_ACCESS_KEY_ID"] = ""
os.environ["AWS_SECRET_ACCESS_KEY"] = ""
os.environ["AWS_REGION_NAME"] = ""
```

### Customize LlamaGuard prompt 

To modify the unsafe categories llama guard evaluates against, just create your own version of [this category list](https://github.com/BerriAI/litellm/blob/main/litellm/proxy/llamaguard_prompt.txt)

Point your proxy to it

```yaml
callbacks: ["llamaguard_moderations"]
  llamaguard_model_name: "sagemaker/jumpstart-dft-meta-textgeneration-llama-guard-7b"
  llamaguard_unsafe_content_categories: /path/to/llamaguard_prompt.txt
```

## Content Moderation with Google Text Moderation 

Requires your GOOGLE_APPLICATION_CREDENTIALS to be set in your .env (same as VertexAI).

How to enable this in your config.yaml: 

```yaml 
litellm_settings:
   callbacks: ["google_text_moderation"]
```

### Set custom confidence thresholds

Google Moderations checks the test against several categories. [Source](https://cloud.google.com/natural-language/docs/moderating-text#safety_attribute_confidence_scores)

#### Set global default confidence threshold

By default this is set to 0.8. But you can override this in your config.yaml.

```yaml
litellm_settings: 
    google_moderation_confidence_threshold: 0.4 
```

#### Set category-specific confidence threshold

Set a category specific confidence threshold in your config.yaml. If none set, the global default will be used. 

```yaml
litellm_settings: 
    toxic_confidence_threshold: 0.1
```

Here are the category specific values: 

| Category | Setting |
| -------- | -------- |
| "toxic" | toxic_confidence_threshold: 0.1 | 
| "insult" | insult_confidence_threshold: 0.1 | 
| "profanity" | profanity_confidence_threshold: 0.1 | 
| "derogatory" | derogatory_confidence_threshold: 0.1 | 
| "sexual" | sexual_confidence_threshold: 0.1 | 
| "death_harm_and_tragedy" | death_harm_and_tragedy_threshold: 0.1 | 
| "violent" | violent_threshold: 0.1 | 
| "firearms_and_weapons" | firearms_and_weapons_threshold: 0.1 | 
| "public_safety" | public_safety_threshold: 0.1 | 
| "health" | health_threshold: 0.1 | 
| "religion_and_belief" | religion_and_belief_threshold: 0.1 | 
| "illicit_drugs" | illicit_drugs_threshold: 0.1 | 
| "war_and_conflict" | war_and_conflict_threshold: 0.1 | 
| "politics" | politics_threshold: 0.1 | 
| "finance" | finance_threshold: 0.1 | 
| "legal" | legal_threshold: 0.1 |

## Tracking Spend for Custom Tags

Requirements: 

- Virtual Keys & a database should be set up, see [virtual keys](https://docs.litellm.ai/docs/proxy/virtual_keys)

### Usage - /chat/completions requests with request tags 


<Tabs>


<TabItem value="openai" label="OpenAI Python v1.0.0+">

Set `extra_body={"metadata": { }}` to `metadata` you want to pass

```python
import openai
client = openai.OpenAI(
    api_key="anything",
    base_url="http://0.0.0.0:8000"
)

# request sent to model set on litellm proxy, `litellm --model`
response = client.chat.completions.create(
    model="gpt-3.5-turbo",
    messages = [
        {
            "role": "user",
            "content": "this is a test request, write a short poem"
        }
    ],
    extra_body={
        "metadata": {
            "tags": ["model-anthropic-claude-v2.1", "app-ishaan-prod"]
        }
    }
)

print(response)
```
</TabItem>

<TabItem value="Curl" label="Curl Request">

Pass `metadata` as part of the request body

```shell
curl --location 'http://0.0.0.0:8000/chat/completions' \
    --header 'Content-Type: application/json' \
    --data '{
    "model": "gpt-3.5-turbo",
    "messages": [
        {
        "role": "user",
        "content": "what llm are you"
        }
    ],
    "metadata": {"tags": ["model-anthropic-claude-v2.1", "app-ishaan-prod"]}
}'
```
</TabItem>
<TabItem value="langchain" label="Langchain">

```python
from langchain.chat_models import ChatOpenAI
from langchain.prompts.chat import (
    ChatPromptTemplate,
    HumanMessagePromptTemplate,
    SystemMessagePromptTemplate,
)
from langchain.schema import HumanMessage, SystemMessage

chat = ChatOpenAI(
    openai_api_base="http://0.0.0.0:8000",
    model = "gpt-3.5-turbo",
    temperature=0.1,
    extra_body={
        "metadata": {
            "tags": ["model-anthropic-claude-v2.1", "app-ishaan-prod"]
        }
    }
)

messages = [
    SystemMessage(
        content="You are a helpful assistant that im using to make a test request to."
    ),
    HumanMessage(
        content="test from litellm. tell me why it's amazing in 1 sentence"
    ),
]
response = chat(messages)

print(response)
```

</TabItem>
</Tabs>


### Viewing Spend per tag

#### `/spend/tags` Request Format 
```shell
curl -X GET "http://0.0.0.0:4000/spend/tags" \
-H "Authorization: Bearer sk-1234"
```

#### `/spend/tags`Response Format
```shell
[
  {
    "individual_request_tag": "model-anthropic-claude-v2.1",
    "log_count": 6,
    "total_spend": 0.000672
  },
  {
    "individual_request_tag": "app-ishaan-local",
    "log_count": 4,
    "total_spend": 0.000448
  },
  {
    "individual_request_tag": "app-ishaan-prod",
    "log_count": 2,
    "total_spend": 0.000224
  }
]

```


<!-- ## Tracking Spend per Key

## Tracking Spend per User -->
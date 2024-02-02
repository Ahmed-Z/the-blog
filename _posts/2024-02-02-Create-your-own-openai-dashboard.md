---
title: Create your own OpenAI API expenditure dashboard
published: true
---
<br>
### [](#header-3) Introduction
In the dynamic landscape of artificial intelligence (AI), developers often find themselves working on multiple projects simultaneously, each utilizing the powerful capabilities of OpenAI's API. However, in many enterprise settings, access to the OpenAI account might be restricted, leaving developers with only the API key and no direct visibility into expenditure details. This lack of access can pose a challenge when it comes to tracking costs, a critical aspect for developers aiming to analyze usage patterns, allocate resources efficiently, and optimize their AI projects.

Fortunately, developers can overcome this limitation by building their own OpenAI API expenditure dashboard. In this guide, we will walk through the process of creating a custom dashboard that allows developers to monitor their API usage, gain insights into costs, and ultimately optimize their projects without needing direct access to the OpenAI account.

### [](#header-3) Undocumented OpenAI usage API
Upon my research I uncovered a potential solution that doesn't rely on official documentation. It appears that there exists an undocumented OpenAI API, which, when leveraged correctly, provides the essential information needed to calculate expenditures.<br>
The endpoint in question is <br>`https://api.openai.com/v1/usage?date={yyyy-mm-dd}&user_public_id={usr_id}`<br> By making a GET request to this URL and including your OpenAI API key as a Bearer token, you can retrieve a detailed breakdown of their API usage. The response is structured in a clean and understandable JSON format, containing vital metrics for expenditure analysis.
```json
{
  "object": "list",
  "data": [
    {
      "organization_id": <org_id>,
      "organization_name": <org_name>,
      "aggregation_timestamp": <timestamp>,
      "n_requests": 12,
      "operation": "completion",
      "snapshot_id": "gpt-3.5-turbo-0613",
      "n_context_tokens_total": 7786,
      "n_generated_tokens_total": 12,
      "user_id": <user_id>,
      "api_key_id": null,
      "api_key_name": null,
      "api_key_redacted": null
    },
   ...
  ],
  "ft_data": [],
  "dalle_api_data": [],
  "whisper_api_data": [],
  "tts_api_data": [],
  "assistant_code_interpreter_data": []
}
```
To get the `user_public_id` required for the endpoint, you need to use this endpoint.
GET request to `https://api.openai.com/v1/organizations/{organisation_id}/users`

Acquiring the `org_id` from the OpenAI account and associating the API with that specific organization allows you to retrieve the necessary `user_public_id`.<br>
It's worth noting that, your OpenAI API key should always be included as a Bearer token when making requests to these endpoints.<br>
Now that we have successfully retrieved our OpenAI API token usage through the endpoint, the next crucial step is to create a simple function that calculates the costs associated with each API call. OpenAI provides transparent pricing details for every model, and you can refer to the official OpenAI documentation <a href="https://openai.com/pricing" target='_blank'>here </a> for a comprehensive breakdown.<br>
It's important to note that the usage of the API is subject to rate limits, allowing only 5 requests per minute. Given this restriction, retrieving data for multiple days may take some time. To address this, it is advisable to implement a mechanism that saves the retrieved data into a JSON or CSV file. 

### [](#header-3) Visualizing Expenditure: Building an Interactive Dashboard with Dash
With the expenditure data in hand, the next logical step is to present it in a clear and accessible format, and what better way to achieve this than through a dynamic web dashboard. In my implementation, I turned to the Python library called Dash, a powerful tool for creating interactive and responsive web applications.<br>



By leveraging Dash, I was able to design a dashboard that showcases various plots, providing valuable insights into OpenAI API expenditures. One compelling example is a plot illustrating the cost over time, grouped by model name. This visualization not only aids in understanding expenditure trends but also allows for a nuanced analysis of costs associated with different models.

<p align="center">
  <img src="https://github.com/Ahmed-Z/the-blog/blob/gh-pages/assets/tokens.PNG?raw=true" style="width:100%;"><br>
  <em>Number and requests over time</em>
</p>

<p align="center">
  <img src="https://github.com/Ahmed-Z/the-blog/blob/gh-pages/assets/expenditure.PNG?raw=true" style="width:100%;"><br>
  <em>Cost over time grouped by model name</em>
</p>

### [](#header-3) Conclusion
As the landscape of AI continues to evolve, the ability to independently monitor and analyze API usage becomes a pivotal aspect of successful project management. Visualizing expenditure data in a user-friendly format not only aids in cost analysis but also empowers developers to make informed decisions about resource allocation and project optimization.

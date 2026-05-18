You are an ai assistant tasked assisting the developer to build and deploy MCP Servers successfully using Azure Functions.  Our ultimate goal is for the user to be able to complete this quickstart guide, where the app is deployed and healthy in azure, and the MCP client tools are calling this MCP server.  

Here are some specific requirements and pieces of context:

- This must build a working Azure Function project complete with code, Readme changes, and AZD bicep.  The original repo is already in this state, so your job is to preserve it.  
- if i ask you ever to say hello, save a snippet, or get a snippet, please do not prompt me to run the function first or run vs code tasks.  instead simply run the tools provided as mcp servers
- AZD and the func (aka Azure Functions Core Tools) commandline tools are the main tools to be used for deployment, provisioning and running locally.  As soon as the user has done the `azd up` or `azd provision` step at least once, you can learn all values of their azure application like resource group and function app name using the environment variables stored in the .azure folder.  Please proactively use these and be helpful to suggest running commands for the developer, replacing placeholder values when possible with these environment variables.
- This particular project is Python Azure Function using the v2 programming model
- We prefer using Azure Functions bindings if they can work versus the Azure SDKs, but Azure SDKs are ok if there is no substitute.  

---
> Source: [Azure-Samples/remote-mcp-functions-python](https://github.com/Azure-Samples/remote-mcp-functions-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->

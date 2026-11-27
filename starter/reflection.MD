# Project Reflection

## Design decision

In the memory hook, I saved the customer's original message to long-term memory rather than the version with retrieved memories prepended. The hook adds a "Customer Context" block so the model can personalise its answer; saving that augmented text would re-store earlier memories on every turn and fill memory with duplicates. I also made the hook robust to message shape: it scans every content block for text and skips tool calls and tool results, so a reasoning block before the reply cannot cause the answer to be dropped or a tool result to be saved as the customer's query. After I introduced myself as Jane in one session, the agent recalled my name and preference for concise replies in a new session.

## Challenge

The hardest part was deployment. Deploying from Windows failed because `pywin32` could not be resolved for the Linux ARM64 runtime, and the lab blocked automatic IAM role creation, so I moved to AWS CloudShell. Later the order tool returned "Unrecognised route"; the Lambda log showed an empty event, which led me to fix the proxy integration, redeploy the API to the prod stage and update the gateway target.

The browser tool was the last obstacle. Runtime logs showed Playwright's driver failing with `libatomic.so.1: cannot open shared object file`, because the managed runtime image lacks that library and direct-code deployments cannot install system packages. A later redeploy also crashed on startup because a newer `strands-agents-tools` release (0.9.0+) removed the browser module, so I pinned it below 0.9.0. For the library, I downloaded the ARM64 `libatomic` package, bundled `libatomic.so.1` in the project and pointed `LD_LIBRARY_PATH` at it from `main.py`. The agent then returned Udacity's real page title, and the logs showed no library error.

## Production extensions

I would replace the Gateway's NONE authorizer with JWT or IAM authentication and scope IAM policies to the specific Knowledge Base, memory and tools. I would add structured logging, alarms and automated tests for each workflow. Refunds need guardrails, such as amount limits and human approval for large ones, and the agent should verify customer identity before showing order details. Since the browser tool is deprecated, I would move to a container deployment, where system libraries can be installed properly, and call AgentCore's browser client directly. Dependencies should be pinned and configuration kept out of source code.
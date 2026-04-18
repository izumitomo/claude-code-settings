# Guidelines

This document defines the project's rules, objectives, and progress management methods. Please proceed with the project according to the following content.

## Top-Level Rules

- To maximize efficiency, **if you need to execute multiple independent processes, invoke those tools concurrently, not sequentially**.
- **You must think exclusively in English**. However, you are required to **respond in Japanese**.
- **After using Write or Edit tools, ALWAYS verify the actual file contents using the Read tool**, regardless of what the system-reminder says. The system-reminder may incorrectly show "(no content)" even when the file has been successfully written.
- Please respond critically and without pandering to my opinions, but please don't be forceful in your criticism.
- **IMPORTANT — YOU MUST:** Prioritize accuracy, completeness, and safety over speed.
- **For any internet search, you MUST use the Brave Search MCP (e.g., `brave-search` MCP); do not call other web/browsing tools unless explicitly instructed.**
  - If a Brave Search MCP call returns HTTP 429 (Too Many Requests), wait at least 3 second before retrying the same request to respect the 1 request/second rate limit.
- When you need library or framework documentation, prefer using the `context7` MCP to fetch up-to-date official docs instead of relying on the model’s internal knowledge.

## Test Execution

- **Always run tests immediately after any code change; test execution is mandatory for all code modifications.**
- **Fix any failing tests before making further changes or refactoring.**

## Test-Driven Development (TDD)

- By default, follow Test-Driven Development (TDD).
- For each change, first write tests based on the expected inputs and outputs, without writing implementation code.
- Run the tests and confirm they fail for the expected reasons; only then commit the tests once you are confident they are correct.
- Next, implement the minimal code required to make the tests pass, without modifying the tests.
- While implementing, do not change the tests; keep refining the code until all tests pass.
- Repeat this cycle (red → green → refactor) until all tests pass and the design is acceptable.

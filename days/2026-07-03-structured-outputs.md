# 2026-07-03 — Structured Outputs: Making LLMs Return Data You Can Trust

## Concept

LLMs return text; production systems need data. Structured output techniques force model responses into a schema you can validate. Three layers, from weakest to strongest guarantee: prompt-level ("respond in JSON"), API-level (JSON mode / tool schemas, which constrain decoding), and grammar-constrained decoding (the model literally cannot emit an invalid token). Whatever layer you use, validate at the boundary — schemas drift, models wrap JSON in prose, and fields go missing.

## Why it matters

The #1 cause of flaky LLM integrations is parsing failures on unvalidated output. A schema + validator turns "hope it's JSON" into a typed contract, and validation errors become *re-prompt material*: feed the error back and the model usually fixes itself in one retry.

## Minimal working example

Tested and passing (`python3 structured_outputs.py`):

```python
"""Structured outputs without an API: schema-first LLM response handling."""
from pydantic import BaseModel, Field, ValidationError
import json, re

class BugReport(BaseModel):
    title: str = Field(min_length=5, max_length=80)
    severity: int = Field(ge=1, le=5)
    component: str
    reproducible: bool

def extract_json(raw: str) -> dict:
    """LLMs often wrap JSON in prose or code fences. Strip and parse."""
    match = re.search(r"\{.*\}", raw, re.DOTALL)
    if not match:
        raise ValueError("no JSON object found")
    return json.loads(match.group())

def parse_llm_response(raw: str) -> BugReport:
    """Validate → on failure, re-prompt with the error message."""
    return BugReport(**extract_json(raw))

# Simulated messy LLM output
raw = '''Sure! Here's the report:
```json
{"title": "Login page 500 on empty password", "severity": 4,
 "component": "auth-service", "reproducible": true}
```'''
report = parse_llm_response(raw)
print("Parsed OK:", report.model_dump())

# Invalid output → caught; the error message is re-promptable
bad = '{"title": "x", "severity": 9, "component": "auth", "reproducible": true}'
try:
    parse_llm_response(bad)
except ValidationError as e:
    print("Caught invalid output:", e.error_count(), "errors")
```

Output:
```
Parsed OK: {'title': 'Login page 500 on empty password', 'severity': 4, 'component': 'auth-service', 'reproducible': True}
Caught invalid output: 2 errors
```

## Key takeaways

1. Constrain as early as possible (JSON mode / grammars), but validate at the boundary regardless — the schema is the contract.
2. Pydantic's `ValidationError` messages are designed for humans, which makes them great re-prompts for models too.
3. Regex-extract before parsing: models love wrapping JSON in code fences and pleasantries.

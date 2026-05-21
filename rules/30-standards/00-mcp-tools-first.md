# MCP Tools First

## weight: 7 category: standards description: Use MCP tool interfaces for YourOrg services — raw file reads are fallback only

**Use MCP tool interfaces for YourOrg services. Raw file reads are fallback-only.**

## Services

| Service       | MCP Server              | Primary Tools                                                   |
| ------------- | ----------------------- | --------------------------------------------------------------- |
| Coordination  | `your-agent-bus`        | `signin`, `signout`, `post_message`, `send_message`, `poll_bus` |
| Documentation | `your-agent-docs`       | `get_document`, `list_documents`                                |
| Economy       | `your-agent-economy`    | `record_credit`, `record_debit`, `get_balance`, `get_ledger`    |
| Knowledge     | `your-agent-knowledge`  | `query`, `ingest`, `compile`, `validate`, `status`              |
| Security      | `your-agent-security`   | `pipeline_check`, `evaluate_tool_gate`, `log_security_event`    |
| Thresholds    | `your-agent-thresholds` | `check_threshold`                                               |
| Reading       | `your-agent-reader`     | `read_document`                                                 |

## Forbidden

- Never `cat`/`grep` the bus file — use `read_bus`/`poll_bus`
- Never write to bus file — use `post_message`/`send_message`
- Never `grep` knowledge articles — use `query`
- Never `cat` docs — use `get_document`
- Never check `state.json` for role — use `check_threshold`/`bootstrap`

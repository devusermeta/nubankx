# Specialist Agents Domain

Auto-generated file-by-file inventory summary.

## claude_bank/app/agents

| File | Summary |
|---|---|
| `claude_bank/app/agents/account-agent/a2a_handler.py` | classes: AccountAgentHandler |
| `claude_bank/app/agents/account-agent/config.py` | classes: AgentConfig |
| `claude_bank/app/agents/account-agent/main.py` | funcs: lifespan, _heartbeat_loop, health, metrics, a2a_invoke… |
| `claude_bank/app/agents/account-agent-a2a/__init__.py` | module/constants |
| `claude_bank/app/agents/account-agent-a2a/agent_handler.py` | classes: AccountAgentHandler | funcs: get_account_agent_handler, cleanup_handler |
| `claude_bank/app/agents/account-agent-a2a/audited_mcp_tool.py` | classes: AuditedMCPTool |
| `claude_bank/app/agents/account-agent-a2a/config.py` | funcs: validate_config |
| `claude_bank/app/agents/account-agent-a2a/create_account_agent_in_foundry.py` | funcs: create_account_agent, main |
| `claude_bank/app/agents/account-agent-a2a/create_agent_in_foundry.py` | funcs: create_account_agent |
| `claude_bank/app/agents/account-agent-a2a/get_agent_id.py` | funcs: get_agent_id |
| `claude_bank/app/agents/account-agent-a2a/inspect_agent.py` | funcs: inspect_agent |
| `claude_bank/app/agents/account-agent-a2a/main.py` | classes: ChatMessage, ChatRequest, ChatResponse | funcs: lifespan, get_agent_card, chat_endpoint, health_check, root |
| `claude_bank/app/agents/account-agent-a2a/test_a2a_connection.py` | funcs: test_a2a_connection |
| `claude_bank/app/agents/account-agent-a2a/test_a2a_integration.py` | classes: A2ATestRunner | funcs: main |
| `claude_bank/app/agents/account-agent-a2a/test_end_to_end.py` | funcs: test_end_to_end |
| `claude_bank/app/agents/account-agent-a2a/verify_agent.py` | funcs: verify_agent |
| `claude_bank/app/agents/ai-money-coach-agent/a2a_handler.py` | classes: AIMoneyCoachAgentHandler |
| `claude_bank/app/agents/ai-money-coach-agent/config.py` | classes: AgentConfig |
| `claude_bank/app/agents/ai-money-coach-agent/create_agent_in_foundry.py` | funcs: create_ai_money_coach_agent |
| `claude_bank/app/agents/ai-money-coach-agent/main.py` | funcs: lifespan, _heartbeat_loop, health, a2a_invoke, root |
| `claude_bank/app/agents/ai-money-coach-agent-a2a/agent_handler.py` | classes: AIMoneyCoachAgentHandler | funcs: create_support_ticket_tool |
| `claude_bank/app/agents/ai-money-coach-agent-a2a/audited_mcp_tool.py` | classes: AuditedMCPTool |
| `claude_bank/app/agents/ai-money-coach-agent-a2a/config.py` | funcs: validate_config |
| `claude_bank/app/agents/ai-money-coach-agent-a2a/create_agent_in_foundry.py` | funcs: create_ai_money_coach_agent |
| `claude_bank/app/agents/ai-money-coach-agent-a2a/main.py` | classes: ChatMessage, ChatRequest, ChatResponse | funcs: get_handler, lifespan, get_agent_card, chat_endpoint, health_check… |
| `claude_bank/app/agents/common/a2a_banking_telemetry.py` | classes: A2ABankingTelemetry | funcs: get_a2a_telemetry |
| `claude_bank/app/agents/escalation-agent-a2a/agent_handler.py` | classes: EscalationAgentHandler |
| `claude_bank/app/agents/escalation-agent-a2a/audited_mcp_tool.py` | classes: AuditedMCPTool |
| `claude_bank/app/agents/escalation-agent-a2a/config.py` | funcs: validate_config |
| `claude_bank/app/agents/escalation-agent-a2a/create_agent_in_foundry.py` | funcs: create_escalation_agent |
| `claude_bank/app/agents/escalation-agent-a2a/main.py` | classes: ChatMessage, ChatRequest, ChatResponse | funcs: lifespan, get_agent_card, invoke_agent, health_check, root |
| `claude_bank/app/agents/escalation-agent-a2a/observability/mcp_audit_2026-01-13.json` | configuration/schema data |
| `claude_bank/app/agents/escalation-agent-a2a/observability/mcp_audit_2026-01-14.json` | configuration/schema data |
| `claude_bank/app/agents/escalation-comms-agent/a2a_handler.py` | classes: EscalationCommsAgentHandler |
| `claude_bank/app/agents/escalation-comms-agent/config.py` | classes: AgentConfig |
| `claude_bank/app/agents/escalation-comms-agent/main.py` | funcs: lifespan, _heartbeat_loop, health, a2a_invoke, root… |
| `claude_bank/app/agents/escalation-copilot-bridge/__init__.py` | module/constants |
| `claude_bank/app/agents/escalation-copilot-bridge/a2a_handler.py` | classes: A2AHandler | funcs: get_a2a_handler |
| `claude_bank/app/agents/escalation-copilot-bridge/config.py` | classes: Settings | funcs: validate_settings |
| `claude_bank/app/agents/escalation-copilot-bridge/copilot.yaml` | configuration/schema data |
| `claude_bank/app/agents/escalation-copilot-bridge/copilot_studio_client.py` | classes: CopilotStudioClient | funcs: get_copilot_client |
| `claude_bank/app/agents/escalation-copilot-bridge/diagnose_power_automate.py` | funcs: print_section, print_step, test_basic_connectivity, test_flow_components, test_power_platform_status… |
| `claude_bank/app/agents/escalation-copilot-bridge/email_service.py` | classes: EmailService | funcs: get_email_service |
| `claude_bank/app/agents/escalation-copilot-bridge/excel_service.py` | classes: ExcelService | funcs: get_excel_service |
| `claude_bank/app/agents/escalation-copilot-bridge/graph_client.py` | classes: GraphAPIClient | funcs: get_graph_client |
| `claude_bank/app/agents/escalation-copilot-bridge/main.py` | funcs: lifespan, root, health_check, get_agent_card, a2a_invoke… |
| `claude_bank/app/agents/escalation-copilot-bridge/models.py` | classes: AgentIdentifier, A2AMetadata, A2AMessage, A2AResponse… |
| `claude_bank/app/agents/escalation-copilot-bridge/power_automate_client.py` | classes: PowerAutomateClient | funcs: get_power_automate_client |
| `claude_bank/app/agents/escalation-copilot-bridge/quick_pa_test.py` | funcs: quick_power_automate_test, main |
| `claude_bank/app/agents/escalation-copilot-bridge/setup_check.py` | funcs: main |
| `claude_bank/app/agents/escalation-copilot-bridge/test_a2a_escalation.py` | funcs: test_a2a_escalation, test_multiple_scenarios, test_exact_manual_scenario, test_health_check, main |
| `claude_bank/app/agents/payment-agent/a2a_handler.py` | classes: PaymentAgentHandler |
| `claude_bank/app/agents/payment-agent/config.py` | classes: AgentConfig |
| `claude_bank/app/agents/payment-agent/main.py` | funcs: lifespan, _heartbeat_loop, health, metrics, a2a_invoke… |
| `claude_bank/app/agents/payment-agent-a2a/agent_handler.py` | classes: PaymentAgentHandler | funcs: get_payment_agent_handler, cleanup_handler |
| `claude_bank/app/agents/payment-agent-a2a/audited_mcp_tool.py` | classes: AuditedMCPTool |
| `claude_bank/app/agents/payment-agent-a2a/config.py` | funcs: validate_config |
| `claude_bank/app/agents/payment-agent-a2a/create_agent_in_foundry.py` | funcs: create_payment_agent |
| `claude_bank/app/agents/payment-agent-a2a/create_payment_agent_in_foundry.py` | funcs: create_payment_agent |
| `claude_bank/app/agents/payment-agent-a2a/main.py` | classes: ChatMessage, ChatRequest, ChatResponse | funcs: lifespan, get_agent_card, chat_endpoint, health_check, root |
| `claude_bank/app/agents/payment-agent-a2a/observability/mcp_audit_2026-01-13.json` | configuration/schema data |
| `claude_bank/app/agents/payment-agent-a2a/observability/mcp_audit_2026-02-04.json` | configuration/schema data |
| `claude_bank/app/agents/payment-agent-a2a/test_a2a_connection.py` | funcs: test_a2a_connection |
| `claude_bank/app/agents/payment-agent-a2a/test_payment_flow.py` | classes: PaymentAgentTester | funcs: main |
| `claude_bank/app/agents/payment-agent-v2-a2a/__init__.py` | module/constants |
| `claude_bank/app/agents/payment-agent-v2-a2a/agent_handler.py` | classes: PaymentAgentHandler |
| `claude_bank/app/agents/payment-agent-v2-a2a/audited_mcp_tool.py` | classes: AuditedMCPTool |
| `claude_bank/app/agents/payment-agent-v2-a2a/check_agent.py` | funcs: check_agent |
| `claude_bank/app/agents/payment-agent-v2-a2a/config.py` | funcs: validate_config |
| `claude_bank/app/agents/payment-agent-v2-a2a/create_agent_in_foundry.py` | funcs: create_payment_agent |
| `claude_bank/app/agents/payment-agent-v2-a2a/list_agents.py` | funcs: list_agents |
| `claude_bank/app/agents/payment-agent-v2-a2a/main.py` | classes: ChatMessage, ChatRequest, ChatResponse | funcs: get_payment_agent_handler, cleanup_handler, lifespan, get_agent_card, chat_endpoint… |
| `claude_bank/app/agents/payment-agent-v3-a2a/__init__.py` | module/constants |
| `claude_bank/app/agents/payment-agent-v3-a2a/agent_handler.py` | classes: PaymentAgentV3Handler | funcs: get_payment_agent_v3_handler, cleanup_handler |
| `claude_bank/app/agents/payment-agent-v3-a2a/audited_mcp_tool.py` | classes: AuditedMCPTool |
| `claude_bank/app/agents/payment-agent-v3-a2a/config.py` | funcs: validate_config |
| `claude_bank/app/agents/payment-agent-v3-a2a/create_agent_in_foundry.py` | funcs: create_payment_agent_v3 |
| `claude_bank/app/agents/payment-agent-v3-a2a/main.py` | classes: ChatMessage, ChatRequest, ChatResponse | funcs: lifespan, get_agent_card, chat_endpoint, health_check, root |
| `claude_bank/app/agents/prodinfo-faq-agent/a2a_handler.py` | classes: ProdInfoFAQAgentHandler |
| `claude_bank/app/agents/prodinfo-faq-agent/config.py` | classes: AgentConfig |
| `claude_bank/app/agents/prodinfo-faq-agent/create_agent_in_foundry.py` | funcs: create_prodinfo_faq_agent |
| `claude_bank/app/agents/prodinfo-faq-agent/main.py` | funcs: lifespan, _heartbeat_loop, health, a2a_invoke, root |
| `claude_bank/app/agents/prodinfo-faq-agent-a2a/agent_handler.py` | classes: ProdInfoFAQAgentHandler | funcs: create_support_ticket_tool |
| `claude_bank/app/agents/prodinfo-faq-agent-a2a/audited_mcp_tool.py` | classes: AuditedMCPTool |
| `claude_bank/app/agents/prodinfo-faq-agent-a2a/config.py` | funcs: validate_config |
| `claude_bank/app/agents/prodinfo-faq-agent-a2a/create_agent_in_foundry.py` | funcs: create_prodinfo_faq_agent |
| `claude_bank/app/agents/prodinfo-faq-agent-a2a/main.py` | classes: ChatMessage, ChatRequest, ChatResponse | funcs: get_handler, lifespan, get_agent_card, chat_endpoint, health_check… |
| `claude_bank/app/agents/supervisor-agent-a2a/agent_handler.py` | classes: SupervisorAgentHandler |
| `claude_bank/app/agents/supervisor-agent-a2a/config.py` | funcs: validate_config |
| `claude_bank/app/agents/supervisor-agent-a2a/create_supervisor_agent_in_foundry.py` | funcs: create_supervisor_agent |
| `claude_bank/app/agents/supervisor-agent-a2a/main.py` | classes: ChatMessage, ChatRequest, ChatResponse, HealthResponse | funcs: lifespan, health, agent_card, invoke_agent |
| `claude_bank/app/agents/transaction-agent/a2a_handler.py` | classes: TransactionAgentHandler |
| `claude_bank/app/agents/transaction-agent/config.py` | classes: AgentConfig |
| `claude_bank/app/agents/transaction-agent/main.py` | funcs: lifespan, _heartbeat_loop, health, metrics, a2a_invoke… |
| `claude_bank/app/agents/transaction-agent-a2a/agent_handler.py` | classes: TransactionAgentHandler | funcs: get_transaction_agent_handler, cleanup_handler |
| `claude_bank/app/agents/transaction-agent-a2a/audited_mcp_tool.py` | classes: AuditedMCPTool |
| `claude_bank/app/agents/transaction-agent-a2a/config.py` | funcs: validate_config |
| `claude_bank/app/agents/transaction-agent-a2a/create_agent_in_foundry.py` | funcs: create_transaction_agent |
| `claude_bank/app/agents/transaction-agent-a2a/create_transaction_agent_in_foundry.py` | funcs: create_transaction_agent |
| `claude_bank/app/agents/transaction-agent-a2a/main.py` | classes: ChatMessage, ChatRequest, ChatResponse | funcs: lifespan, get_agent_card, chat_endpoint, health_check, root |
| `claude_bank/app/agents/transaction-agent-a2a/test_a2a_connection.py` | funcs: test_a2a_connection |
| `claude_bank/app/agents/transaction-agent-a2a/test_mcp_tools.py` | funcs: test_mcp_tool_usage |

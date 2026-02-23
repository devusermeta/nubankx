# Copilot Backend Domain

Auto-generated file-by-file inventory summary.

## claude_bank/app/copilot/app

| File | Summary |
|---|---|
| `claude_bank/app/copilot/app/agents/__init__.py` | module/constants |
| `claude_bank/app/copilot/app/agents/azure_chat/account_agent.py` | classes: AccountAgent |
| `claude_bank/app/copilot/app/agents/azure_chat/ai_money_coach_agent.py` | classes: AIMoneyCoachAgent |
| `claude_bank/app/copilot/app/agents/azure_chat/escalation_comms_agent.py` | classes: EscalationCommsAgent |
| `claude_bank/app/copilot/app/agents/azure_chat/payment_agent.py` | classes: PaymentAgent |
| `claude_bank/app/copilot/app/agents/azure_chat/prodinfo_faq_agent.py` | classes: ProdInfoFAQAgent |
| `claude_bank/app/copilot/app/agents/azure_chat/supervisor_agent.py` | classes: SupervisorAgent |
| `claude_bank/app/copilot/app/agents/azure_chat/supervisor_agent_a2a.py` | classes: SupervisorAgentA2A |
| `claude_bank/app/copilot/app/agents/azure_chat/transaction_agent.py` | classes: TransactionAgent |
| `claude_bank/app/copilot/app/agents/foundry/account_agent_foundry.py` | classes: AccountAgent | funcs: get_or_reuse_agent |
| `claude_bank/app/copilot/app/agents/foundry/ai_money_coach_agent_foundry.py` | classes: AIMoneyCoachAgent |
| `claude_bank/app/copilot/app/agents/foundry/ai_money_coach_agent_knowledge_base_foundry.py` | classes: AIMoneyCoachKnowledgeBaseAgent |
| `claude_bank/app/copilot/app/agents/foundry/escalation_comms_agent_foundry.py` | classes: EscalationCommsAgent | funcs: get_or_create_agent |
| `claude_bank/app/copilot/app/agents/foundry/payment_agent_foundry.py` | classes: PaymentAgent | funcs: get_or_reuse_agent |
| `claude_bank/app/copilot/app/agents/foundry/prodinfo_faq_agent_foundry.py` | classes: ProdInfoFAQAgent |
| `claude_bank/app/copilot/app/agents/foundry/prodinfo_faq_agent_knowledge_base_foundry.py` | classes: ProdInfoFAQAgentKnowledgeBase |
| `claude_bank/app/copilot/app/agents/foundry/supervisor_agent_a2a.py` | classes: SupervisorAgentA2A | funcs: create_supervisor_with_a2a |
| `claude_bank/app/copilot/app/agents/foundry/supervisor_agent_foundry.py` | classes: SupervisorAgent | funcs: _stream_specialist_agent |
| `claude_bank/app/copilot/app/agents/foundry/transaction_agent_foundry.py` | classes: TransactionAgent | funcs: get_or_reuse_agent |
| `claude_bank/app/copilot/app/api/__init__.py` | module/constants |
| `claude_bank/app/copilot/app/api/agent_cards_routers.py` | classes: AddAgentRequest, AddAgentResponse | funcs: load_agent_identities, get_all_agent_urls, get_all_agent_cards, get_agent_card, get_agent_urls… |
| `claude_bank/app/copilot/app/api/auth_routers.py` | funcs: auth_setup, whoami |
| `claude_bank/app/copilot/app/api/chat_routers.py` | funcs: _convert_string_to_chat_response, _format_stream_chunk, _stream_response, chat, get_agent_activity… |
| `claude_bank/app/copilot/app/api/content_routers.py` | funcs: get_content, upload_content |
| `claude_bank/app/copilot/app/api/conversations.py` | classes: Message, BankingOperation, ConversationMetadata, ConversationSession… | funcs: get_base_dir, list_conversations, get_conversation |
| `claude_bank/app/copilot/app/api/dashboard_routers.py` | classes: AgentDecision, RagEvaluation, TriageRule, McpAudit… | funcs: get_observability_dir, read_ndjson_file, get_observability_files, get_dashboard_stats, get_agent_decisions… |
| `claude_bank/app/copilot/app/api/dependencies.py` | funcs: get_current_user, get_current_user_optional |
| `claude_bank/app/copilot/app/api/mcp_routers.py` | classes: MCPToolParameter, MCPTool, MCPService, MCPRegistryResponse | funcs: discover_mcp_service, get_mcp_registry, get_mcp_service_details |
| `claude_bank/app/copilot/app/auth/token_validator.py` | classes: TokenValidator | funcs: get_token_validator |
| `claude_bank/app/copilot/app/auth/user_mapper.py` | classes: UserMapper | funcs: get_user_mapper |
| `claude_bank/app/copilot/app/cache/__init__.py` | module/constants |
| `claude_bank/app/copilot/app/cache/mcp_client.py` | classes: MCPClient, AccountMCPClient, TransactionMCPClient, ContactsMCPClient… | funcs: get_account_mcp_client, get_transaction_mcp_client, get_contacts_mcp_client, get_limits_mcp_client |
| `claude_bank/app/copilot/app/cache/user_cache.py` | classes: UserCacheManager | funcs: get_cache_manager |
| `claude_bank/app/copilot/app/common/document_intelligence_scanner.py` | module/constants |
| `claude_bank/app/copilot/app/config/azure_credential.py` | funcs: get_azure_credential_async, get_async_azure_credential, get_azure_credential |
| `claude_bank/app/copilot/app/config/container_azure_chat.py` | classes: Container |
| `claude_bank/app/copilot/app/config/container_foundry.py` | classes: Container | funcs: get_or_create_agent |
| `claude_bank/app/copilot/app/config/logging.py` | funcs: get_logging_config_path, load_logging_config, _setup_azure_monitoring_logging, setup_logging, get_logger |
| `claude_bank/app/copilot/app/config/settings.py` | classes: Settings | funcs: get_env_files |
| `claude_bank/app/copilot/app/conversation_state_manager.py` | classes: ConversationState, ConversationStateManager | funcs: is_continuation_message, get_conversation_state_manager |
| `claude_bank/app/copilot/app/helpers/blob_proxy.py` | classes: BlobStorageProxy |
| `claude_bank/app/copilot/app/helpers/document_intelligence_scanner.py` | classes: DocumentIntelligenceInvoiceScanHelper |
| `claude_bank/app/copilot/app/logging-default.yaml` | configuration/schema data |
| `claude_bank/app/copilot/app/main.py` | funcs: create_app |
| `claude_bank/app/copilot/app/models/__init__.py` | module/constants |
| `claude_bank/app/copilot/app/models/chat.py` | classes: ConfirmationType, PendingConfirmation, ChatMessage, ChatAppRequest… |
| `claude_bank/app/copilot/app/models/financial_schemas.py` | classes: TransactionRow, TransactionSummary, PeriodInfo, TXN_TABLE… | funcs: create_error_card, generate_request_id, generate_ledger_id |
| `claude_bank/app/copilot/app/models/user.py` | classes: UserCreate, UserOut |
| `claude_bank/app/copilot/app/models/user_context.py` | classes: UserContext |
| `claude_bank/app/copilot/app/observability/banking_telemetry.py` | classes: BankingTelemetry, AgentDecisionTracker, BankingOperationTimer | funcs: get_banking_telemetry, setup_banking_observability |
| `claude_bank/app/copilot/app/purview/__init__.py` | module/constants |
| `claude_bank/app/copilot/app/purview/config.py` | classes: PurviewSettings |
| `claude_bank/app/copilot/app/purview/lineage_tracker.py` | classes: LineageTracker |
| `claude_bank/app/copilot/app/purview/models.py` | classes: PurviewEntity, LineageEvent, LineageRelationship, DataLineageNode… |
| `claude_bank/app/copilot/app/purview/purview_service.py` | classes: PurviewService |
| `claude_bank/app/copilot/app/tools/audited_mcp_tool.py` | classes: AuditedMCPTool |
| `claude_bank/app/copilot/app/tools/invoice_scanner_plugin.py` | module/constants |
| `claude_bank/app/copilot/app/utils/agent_activity_tracker.py` | classes: AgentActivityType, AgentActivity, AgentActivityTracker | funcs: get_activity_tracker, init_activity_tracker |
| `claude_bank/app/copilot/app/utils/conversation_logger.py` | classes: ConversationLogger | funcs: get_conversation_logger |
| `claude_bank/app/copilot/app/utils/date_normalizer.py` | classes: DateNormalizer | funcs: normalize_date, get_bangkok_timestamp, get_bangkok_date |
| `claude_bank/app/copilot/app/utils/mcp_data_fetcher.py` | classes: MCPDataFetcher | funcs: get_mcp_fetcher, init_mcp_fetcher |
| `claude_bank/app/copilot/app/utils/session_memory.py` | classes: SessionMemoryManager | funcs: get_session_manager, init_session_manager |

# Registry + SDK + Frontend + Infra Domain

Auto-generated file-by-file inventory summary.

## claude_bank/app/agent-registry

| File | Summary |
|---|---|
| `claude_bank/app/agent-registry/api/__init__.py` | module/constants |
| `claude_bank/app/agent-registry/api/agents_router.py` | funcs: get_registry_service, register_agent, discover_agents, get_agent, list_all_agents… |
| `claude_bank/app/agent-registry/api/auth.py` | funcs: create_agent_token, verify_agent_token, get_current_agent, verify_agent_or_skip |
| `claude_bank/app/agent-registry/config/__init__.py` | module/constants |
| `claude_bank/app/agent-registry/config/settings.py` | classes: Settings |
| `claude_bank/app/agent-registry/main.py` | funcs: lifespan, root, health_check, metrics |
| `claude_bank/app/agent-registry/models/__init__.py` | module/constants |
| `claude_bank/app/agent-registry/models/agent_registration.py` | classes: AgentEndpoints, AgentCapability, AgentMetadata, AgentRegistration… |
| `claude_bank/app/agent-registry/services/__init__.py` | module/constants |
| `claude_bank/app/agent-registry/services/health_service.py` | classes: HealthService |
| `claude_bank/app/agent-registry/services/registry_service.py` | classes: RegistryService |
| `claude_bank/app/agent-registry/storage/__init__.py` | module/constants |
| `claude_bank/app/agent-registry/storage/cosmos_store.py` | classes: CosmosStore |
| `claude_bank/app/agent-registry/storage/redis_store.py` | classes: RedisStore |

## claude_bank/app/a2a-sdk

| File | Summary |
|---|---|
| `claude_bank/app/a2a-sdk/__init__.py` | module/constants |
| `claude_bank/app/a2a-sdk/client/__init__.py` | module/constants |
| `claude_bank/app/a2a-sdk/client/a2a_client.py` | classes: A2AConfig, A2AClient |
| `claude_bank/app/a2a-sdk/client/registry_client.py` | classes: RegistryClient |
| `claude_bank/app/a2a-sdk/models/__init__.py` | module/constants |
| `claude_bank/app/a2a-sdk/models/messages.py` | classes: AgentIdentifier, A2AMetadata, A2AMessage, A2AResponse… |
| `claude_bank/app/a2a-sdk/utils/__init__.py` | module/constants |
| `claude_bank/app/a2a-sdk/utils/circuit_breaker.py` | classes: CircuitState, CircuitBreakerError, CircuitBreaker |

## claude_bank/app/frontend/src

| File | Summary |
|---|---|
| `claude_bank/app/frontend/src/api/api.ts` | symbols: getCitationFilePath, uploadAttachment, getImage, getHeaders, askApi, chatApi |
| `claude_bank/app/frontend/src/api/dashboardApi.ts` | symbols: dashboardApi |
| `claude_bank/app/frontend/src/api/index.ts` | component/styles/types |
| `claude_bank/app/frontend/src/api/models.ts` | symbols: enum |
| `claude_bank/app/frontend/src/api/streamSSE.ts` | symbols: createEventSource, readSSEStream |
| `claude_bank/app/frontend/src/authConfig.ts` | symbols: fetchAuthSetup, useLogin, msalConfig, loginRequest, getRedirectUri, getToken… |
| `claude_bank/app/frontend/src/components/AgentActivityPanel/AgentActivityPanel.module.css` | styles/assets |
| `claude_bank/app/frontend/src/components/AgentActivityPanel/AgentActivityPanel.tsx` | symbols: AgentActivityPanel, getActivityIcon, formatTime, getAgentColor |
| `claude_bank/app/frontend/src/components/AgentActivityPanel/index.ts` | component/styles/types |
| `claude_bank/app/frontend/src/components/AgentSystemMap/AgentSystemMap.module.css` | styles/assets |
| `claude_bank/app/frontend/src/components/AgentSystemMap/AgentSystemMap.tsx` | symbols: AgentSystemMap, getNodeIdFromAgent, isNodeActive, isNodeThinking, isEdgeActive, formatEventDescription |
| `claude_bank/app/frontend/src/components/AgentSystemMap/AgentSystemMapVertical.module.css` | styles/assets |
| `claude_bank/app/frontend/src/components/AgentSystemMap/AgentSystemMapVertical.tsx` | symbols: AgentSystemMapVertical, getNodeIdFromAgent, normalizeAgentId, isNodeActive, isNodeThinking, isEdgeActive… |
| `claude_bank/app/frontend/src/components/AgentSystemMap/index.tsx` | component/styles/types |
| `claude_bank/app/frontend/src/components/AnalysisPanel/AnalysisPanel.module.css` | styles/assets |
| `claude_bank/app/frontend/src/components/AnalysisPanel/AnalysisPanel.tsx` | symbols: AnalysisPanel |
| `claude_bank/app/frontend/src/components/AnalysisPanel/AnalysisPanelTabs.tsx` | component/styles/types |
| `claude_bank/app/frontend/src/components/AnalysisPanel/index.tsx` | component/styles/types |
| `claude_bank/app/frontend/src/components/Answer/Answer.module.css` | styles/assets |
| `claude_bank/app/frontend/src/components/Answer/Answer.tsx` | symbols: Answer |
| `claude_bank/app/frontend/src/components/Answer/AnswerCard.tsx` | symbols: AnswerCard, handleFeedback |
| `claude_bank/app/frontend/src/components/Answer/AnswerError.tsx` | symbols: AnswerError |
| `claude_bank/app/frontend/src/components/Answer/AnswerIcon.tsx` | symbols: AnswerIcon |
| `claude_bank/app/frontend/src/components/Answer/AnswerLoading.tsx` | symbols: AnswerLoading, getContextualMessage |
| `claude_bank/app/frontend/src/components/Answer/AnswerParser.tsx` | symbols: parseAnswerToHtml, parseMarkdownTable |
| `claude_bank/app/frontend/src/components/Answer/index.ts` | component/styles/types |
| `claude_bank/app/frontend/src/components/AttachmentType.ts` | component/styles/types |
| `claude_bank/app/frontend/src/components/ClearChatButton/ClearChatButton.module.css` | styles/assets |
| `claude_bank/app/frontend/src/components/ClearChatButton/ClearChatButton.tsx` | symbols: ClearChatButton |
| `claude_bank/app/frontend/src/components/ClearChatButton/index.tsx` | component/styles/types |
| `claude_bank/app/frontend/src/components/ConfirmationButtons/ConfirmationButtons.module.css` | styles/assets |
| `claude_bank/app/frontend/src/components/ConfirmationButtons/ConfirmationButtons.module.css.d.ts` | component/styles/types |
| `claude_bank/app/frontend/src/components/ConfirmationButtons/ConfirmationButtons.tsx` | symbols: ConfirmationButtons |
| `claude_bank/app/frontend/src/components/ConfirmationButtons/index.ts` | component/styles/types |
| `claude_bank/app/frontend/src/components/ConfirmationDialog/ConfirmationDialog.module.css` | styles/assets |
| `claude_bank/app/frontend/src/components/ConfirmationDialog/ConfirmationDialog.module.css.d.ts` | component/styles/types |
| `claude_bank/app/frontend/src/components/ConfirmationDialog/ConfirmationDialog.tsx` | symbols: ConfirmationDialog, getIconColor |
| `claude_bank/app/frontend/src/components/ConfirmationDialog/index.ts` | component/styles/types |
| `claude_bank/app/frontend/src/components/Example/Example.module.css` | styles/assets |
| `claude_bank/app/frontend/src/components/Example/Example.tsx` | symbols: Example |
| `claude_bank/app/frontend/src/components/Example/ExampleList.tsx` | symbols: ExampleList |
| `claude_bank/app/frontend/src/components/Example/index.tsx` | component/styles/types |
| `claude_bank/app/frontend/src/components/HumanInLoopConfirmation/HumanInLoopConfirmation.tsx` | symbols: formatKey, handleDecision, getIcon |
| `claude_bank/app/frontend/src/components/HumanInLoopConfirmation/index.ts` | component/styles/types |
| `claude_bank/app/frontend/src/components/HumanInLoopConfirmation/styles.css` | styles/assets |
| `claude_bank/app/frontend/src/components/LoginButton/LoginButton.module.css` | styles/assets |
| `claude_bank/app/frontend/src/components/LoginButton/LoginButton.tsx` | symbols: LoginButton, handleLoginRedirect, handleLogoutRedirect |
| `claude_bank/app/frontend/src/components/LoginButton/index.tsx` | component/styles/types |
| `claude_bank/app/frontend/src/components/MCPRegistry/MCPRegistry.tsx` | symbols: MCPRegistry, handleServiceClick, handleCloseModal, getStatusEmoji, getStatusColor |
| `claude_bank/app/frontend/src/components/MCPRegistry/MCPToolDetailsModal.tsx` | symbols: MCPToolDetailsModal, getStatusEmoji, getStatusLabel |
| `claude_bank/app/frontend/src/components/QuestionInput/QuestionContext.ts` | component/styles/types |
| `claude_bank/app/frontend/src/components/QuestionInput/QuestionInput.module.css` | styles/assets |
| `claude_bank/app/frontend/src/components/QuestionInput/QuestionInput.tsx` | symbols: QuestionInput, onEnterPress, onQuestionChange, onAttach, onFileSelected, onAttachDelete |
| `claude_bank/app/frontend/src/components/QuestionInput/index.ts` | component/styles/types |
| `claude_bank/app/frontend/src/components/SettingsButton/SettingsButton.module.css` | styles/assets |
| `claude_bank/app/frontend/src/components/SettingsButton/SettingsButton.tsx` | symbols: SettingsButton |
| `claude_bank/app/frontend/src/components/SettingsButton/index.tsx` | component/styles/types |
| `claude_bank/app/frontend/src/components/SupportingContent/SupportingContent.module.css` | styles/assets |
| `claude_bank/app/frontend/src/components/SupportingContent/SupportingContent.tsx` | symbols: SupportingContent |
| `claude_bank/app/frontend/src/components/SupportingContent/SupportingContentParser.ts` | symbols: parseSupportingContentItem |
| `claude_bank/app/frontend/src/components/SupportingContent/index.ts` | component/styles/types |
| `claude_bank/app/frontend/src/components/ThemeToggle/ThemeToggle.tsx` | symbols: ThemeToggle, toggleTheme |
| `claude_bank/app/frontend/src/components/ThinkingPanel/ThinkingPanel.module.css` | styles/assets |
| `claude_bank/app/frontend/src/components/ThinkingPanel/ThinkingPanel.tsx` | symbols: ThinkingPanel, getStepIcon, getStepLabel, formatDuration |
| `claude_bank/app/frontend/src/components/ThinkingPanel/index.tsx` | component/styles/types |
| `claude_bank/app/frontend/src/components/TokenClaimsDisplay/TokenClaimsDisplay.tsx` | symbols: TokenClaimsDisplay, ToString |
| `claude_bank/app/frontend/src/components/TokenClaimsDisplay/index.tsx` | component/styles/types |
| `claude_bank/app/frontend/src/components/UserChatMessage/UserChatMessage.module.css` | styles/assets |
| `claude_bank/app/frontend/src/components/UserChatMessage/UserChatMessage.tsx` | symbols: UserChatMessage |
| `claude_bank/app/frontend/src/components/UserChatMessage/UserChatMessageCard.tsx` | symbols: UserChatMessageCard |
| `claude_bank/app/frontend/src/components/UserChatMessage/index.ts` | component/styles/types |
| `claude_bank/app/frontend/src/components/dashboard/Pagination.tsx` | symbols: Pagination, startIndex |
| `claude_bank/app/frontend/src/components/ui/alert.tsx` | component/styles/types |
| `claude_bank/app/frontend/src/components/ui/button.tsx` | component/styles/types |
| `claude_bank/app/frontend/src/components/ui/card.tsx` | component/styles/types |
| `claude_bank/app/frontend/src/components/ui/textarea.tsx` | component/styles/types |
| `claude_bank/app/frontend/src/data/mockData.ts` | symbols: mockAgentDecisions, mockRagEvaluations, mockMcpAudit, mockUserMessages, mockTriageRules |
| `claude_bank/app/frontend/src/hooks/useAgentStatus.ts` | symbols: useAgentStatus |
| `claude_bank/app/frontend/src/hooks/usePagination.ts` | symbols: usePagination, startIndex |
| `claude_bank/app/frontend/src/index.css` | styles/assets |
| `claude_bank/app/frontend/src/index.tsx` | component/styles/types |
| `claude_bank/app/frontend/src/lib/roleUtils.ts` | symbols: extractRoleFromToken, extractTokenClaims, Permissions, hasPermission, canViewObservability, isAdmin… |
| `claude_bank/app/frontend/src/lib/utils.ts` | symbols: cn |
| `claude_bank/app/frontend/src/pages/NoPage.tsx` | symbols: Component |
| `claude_bank/app/frontend/src/pages/chat/Chat.module.css` | styles/assets |
| `claude_bank/app/frontend/src/pages/chat/Chat.tsx` | symbols: Chat, updateState, clearChat, handleThinkingClick, detectConfirmationRequest, extractTableValue… |
| `claude_bank/app/frontend/src/pages/chat/ChatNew.tsx` | symbols: ChatNew, updateState, detectConfirmationRequest, handleConfirm, handleCancel, clearChat… |
| `claude_bank/app/frontend/src/pages/dashboard/AgentDecisionsPage.tsx` | symbols: AgentDecisionsPage, TableView, CardView, TimelineView, ChartView, StatusBadge |
| `claude_bank/app/frontend/src/pages/dashboard/ConversationsPage.tsx` | symbols: ConversationsPage, formatDate, startIndex, goToPage, nextPage, prevPage |
| `claude_bank/app/frontend/src/pages/dashboard/DashboardLayout.module.css` | styles/assets |
| `claude_bank/app/frontend/src/pages/dashboard/DashboardLayout.tsx` | symbols: DashboardLayout |
| `claude_bank/app/frontend/src/pages/dashboard/DashboardOverview.tsx` | symbols: DashboardOverview |
| `claude_bank/app/frontend/src/pages/dashboard/McpAuditPage.tsx` | symbols: successRate |
| `claude_bank/app/frontend/src/pages/dashboard/RagEvaluationsPage.tsx` | symbols: percentage |
| `claude_bank/app/frontend/src/pages/dashboard/TriageRulesPage.tsx` | symbols: percentage |
| `claude_bank/app/frontend/src/pages/dashboard/UserMessagesPage.tsx` | component/styles/types |
| `claude_bank/app/frontend/src/pages/layout/Layout.module.css` | styles/assets |
| `claude_bank/app/frontend/src/pages/layout/Layout.tsx` | symbols: Layout, checkRole |
| `claude_bank/app/frontend/src/vite-env.d.ts` | component/styles/types |

## claude_bank/infrastructure/bicep

| File | Summary |
|---|---|
| `claude_bank/infrastructure/bicep/main.bicep` | infrastructure-as-code module/parameters |
| `claude_bank/infrastructure/bicep/main.json` | configuration/schema data |
| `claude_bank/infrastructure/bicep/modules/ai-services.bicep` | infrastructure-as-code module/parameters |
| `claude_bank/infrastructure/bicep/modules/communication.bicep` | infrastructure-as-code module/parameters |
| `claude_bank/infrastructure/bicep/modules/container-apps.bicep` | infrastructure-as-code module/parameters |
| `claude_bank/infrastructure/bicep/modules/cosmos.bicep` | infrastructure-as-code module/parameters |
| `claude_bank/infrastructure/bicep/modules/keyvault.bicep` | infrastructure-as-code module/parameters |
| `claude_bank/infrastructure/bicep/modules/monitoring.bicep` | infrastructure-as-code module/parameters |
| `claude_bank/infrastructure/bicep/modules/purview.bicep` | infrastructure-as-code module/parameters |
| `claude_bank/infrastructure/bicep/modules/rbac.bicep` | infrastructure-as-code module/parameters |
| `claude_bank/infrastructure/bicep/modules/search.bicep` | infrastructure-as-code module/parameters |
| `claude_bank/infrastructure/bicep/modules/storage.bicep` | infrastructure-as-code module/parameters |

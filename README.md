# Copado AI SDK Features Guide

A comprehensive guide to all available features and functions in the Copado AI SDK.

## Table of Contents
- [Quick Setup](#quick-setup)
- [Workspace Management](#workspace-management)
- [Dialogue & Chat](#dialogue--chat)
- [Document Management](#document-management)
- [Integrations](#integrations)
- [Activity & Analytics](#activity--analytics)
- [Organization Management](#organization-management)

---

## Quick Setup

### To make a new workspace
```python
from copado_ai_sdk import CopadoClient, WorkspaceCapability

# Step 1: Initialize client (no workspace created)
client = CopadoClient(
    base_url="https://copadogpt-api.robotic.copado.com",
    api_key="your-api-key",
    organization_id=12345
)

# Step 2: Create your custom workspace
workspace = client.workspaces.create_with_features(
    name="My Project Workspace",
    description="Workspace for my data analysis project",
    enable_data_analysis=True,
    enable_web_search=True,
    enable_reasoning_mode=True
)

# Step 3: Set as current workspace
client.set_workspace(workspace.id)

# Step 4: Save the workspace ID for future sessions
print(f"Save this for future sessions: {workspace.id}")
```

### To use an existing workspace
```python
from copado_ai_sdk import CopadoClient

# Use your existing workspace directly
client = CopadoClient(
    base_url="https://api.copado.ai",
    api_key="your-api-key",
    organization_id=123,
    workspace_id="your-saved-workspace-id"
)

```

### Send your first message

```python
# Create a dialogue (conversation)
dialogue = client.dialogues.create(name="My First Chat")

# Send a message and get AI response
response = client.dialogues.chat(
    dialogue_id=dialogue.id,
    prompt="Hello! How can you help me with Copado?",
    stream=True  # Streams response to console
)

print(response)
```

### Enable knowledge and upload documents

```python
# Create workspace with Copado knowledge capabilities
workspace = client.workspaces.create_with_knowledge_capabilities(
    name="Copado Knowledge Workspace",
    description="Workspace with access to Copado documentation and CRT knowledge",
    enable_custom_knowledge=True,
    enable_essentials_knowledge=True,
    enable_crt_knowledge=True,          # Enable CRT (Copado Robotic Testing) knowledge
    enable_metadata_format_knowledge=True,
    enable_source_format_knowledge=True
)

# Set as current workspace
client.set_workspace(workspace.id)

# Create dialogue for document analysis
dialogue = client.dialogues.create(name="Document Analysis Chat")

# Upload documents to the dialogue
document = client.dialogues.upload_document(
    dialogue_id=dialogue.id,
    file_path="my_copado_config.json"  # or .csv, .pdf, .xlsx, etc.
)

# Chat with your documents and Copado knowledge
response = client.dialogues.chat(
    dialogue_id=dialogue.id,
    prompt="Analyze this configuration file and suggest improvements using Copado best practices",
    stream=True
)

# Or upload multiple documents at once
response = client.dialogues.chat_with_documents(
    dialogue_id=dialogue.id,
    prompt="Review these files and help me set up a proper Copado CI/CD pipeline",
    document_paths=["pipeline_config.yml", "metadata.xml", "requirements.txt"],
    stream=True
)
```

### Session Management
```python
# Get current workspace ID (may be None if no workspace set)
workspace_id = client.get_workspace_id()

# Set/switch to a workspace
client.set_workspace("workspace-id")

# Alternative: switch workspace (same as set_workspace)
client.switch_workspace("another-workspace-id")

# Check current workspace details
if client.current_workspace_id:
    current_workspace = client.workspaces.get(client.current_workspace_id)
    print(f"Using: {current_workspace.name}")
else:
    print("No workspace set")
```

---

## Workspace Management

### List & Get Workspaces
```python
# List all workspaces
workspaces = client.workspaces.list()

# Get specific workspace
workspace = client.workspaces.get(workspace_id: str)

# Get or create workspace by name
workspace = client.workspaces.get_or_create(
    name: str,
    description: Optional[str] = None,
    capabilities: Optional[List[WorkspaceCapability]] = None
)

# Set as current workspace
client.set_workspace(workspace.id)
```

### Create Workspaces
```python
# Basic creation
workspace = client.workspaces.create(
    name: str,
    description: Optional[str] = None,
    icon_url: Optional[str] = None,
    capabilities: Optional[List[WorkspaceCapability]] = None
)

# Create with specific features
workspace = client.workspaces.create_with_features(
    name: str,
    description: Optional[str] = None,
    enable_data_analysis: bool = False,
    enable_web_search: bool = False,
    enable_diagram_generation: bool = False,
    enable_devops_automations: bool = False,
    enable_reasoning_mode: bool = False,
    enable_follow_up_questions: bool = True
)

# Create with knowledge capabilities
workspace = client.workspaces.create_with_knowledge_capabilities(
    name: str,
    description: Optional[str] = None,
    enable_custom_knowledge: bool = False,
    enable_essentials_knowledge: bool = False,
    enable_crt_knowledge: bool = False,
    enable_metadata_format_knowledge: bool = False,
    enable_source_format_knowledge: bool = False,
    enable_follow_up_questions: bool = True
)
```

### Update & Delete Workspaces
```python
# Update workspace
workspace = client.workspaces.update(
    workspace_id: str,
    name: Optional[str] = None,
    description: Optional[str] = None,
    icon_url: Optional[str] = None,
    capabilities: Optional[List[WorkspaceCapability]] = None,
    default_dataset_id: Optional[str] = None
)

# Delete workspace (WARNING: Deletes all dialogues and documents)
client.workspaces.delete(workspace_id: str)
```

### Capability Management
```python
# Enable/disable capabilities
workspace = client.workspaces.enable_capability(workspace_id: str, capability: WorkspaceCapability)
workspace = client.workspaces.disable_capability(workspace_id: str, capability: WorkspaceCapability)

# Check capability
has_capability = client.workspaces.has_capability(workspace_id: str, capability: WorkspaceCapability)
```

### Member Management
```python
# Add member
member = client.workspaces.add_member(
    workspace_id: str,
    user_id: int,
    permission: WorkspaceMemberPermission = WorkspaceMemberPermission.MEMBER
)

# Update member permission
member = client.workspaces.update_member(
    workspace_id: str,
    member_id: str,
    permission: WorkspaceMemberPermission
)

# Remove member
client.workspaces.remove_member(workspace_id: str, member_id: str)
```

### Datasets
```python
# List workspace datasets
datasets = client.workspaces.list_datasets(workspace_id: str)
```

---

## Dialogue & Chat

### Create & Manage Dialogues
```python
# Create dialogue
dialogue = client.dialogues.create(
    name: str,
    workspace_id: Optional[str] = None,      # Uses current session workspace if None
    workspace_name: Optional[str] = None,   # Alternative to workspace_id
    assistant_id: str = "knowledge",
    x_client: Optional[str] = None
)
# Note: If no workspace_id/workspace_name provided and no current workspace set, 
# this will raise an error with guidance on how to set a workspace

# List dialogues
dialogues = client.dialogues.list(
    workspace_id: Optional[str] = None,
    limit: int = 100,
    offset: int = 0
)

# Get dialogue with messages
dialogue = client.dialogues.get(dialogue_id: str)

# Update dialogue
dialogue = client.dialogues.update(dialogue_id: str, name: Optional[str] = None)

# Delete dialogue
client.dialogues.delete(dialogue_id: str)
```

### Chat & Messaging
```python
# Send message and get response
response = client.dialogues.chat(
    dialogue_id: str,
    prompt: str,
    request_id: Optional[str] = None,      # Auto-generated if None
    assistant_id: Optional[str] = None,
    stream: bool = False,                  # Stream to stdout
    dev_context: Optional[DevContext] = None,
    system_prompt: Optional[str] = None
)

# Chat with document uploads
response = client.dialogues.chat_with_documents(
    dialogue_id: str,
    prompt: str,
    document_paths: List[Union[str, Path]],
    stream: bool = False,
    assistant_id: Optional[str] = None
)

# Cancel message
result = client.dialogues.cancel_message(dialogue_id: str, request_id: str)
```

### Document Management in Dialogues
```python
# Upload document to dialogue
document = client.dialogues.upload_document(
    dialogue_id: str,
    file_path: Union[str, Path],
    filename: Optional[str] = None
)

# List documents in dialogue
documents = client.dialogues.list_documents(dialogue_id: str)

# Delete document from dialogue
client.dialogues.delete_document(dialogue_id: str, filename: str)

# Get document stats
count = client.dialogues.get_document_count(dialogue_id: str)
size = client.dialogues.get_document_total_size(dialogue_id: str)
```

### Feedback
```python
# Create feedback for message
feedback = client.dialogues.create_feedback(
    dialogue_id: str,
    request_id: str,
    prompt: str,
    response: str,
    feedback: str,
    sentiment: bool
)
```

---

## Document Management

### Upload Documents
```python
# Upload to dialogue
document = client.documents.upload_to_dialogue(
    dialogue_id: str,
    file_path: Union[str, Path],
    filename: Optional[str] = None
)

# Upload to dataset (workspace-level)
document = client.documents.upload_to_dataset(
    dataset_id: str,
    file_path: Union[str, Path],
    filename: Optional[str] = None
)

# Update document (replace existing)
document = client.documents.update_in_dialogue(
    dialogue_id: str,
    file_path: Union[str, Path],
    filename: Optional[str] = None
)
```

### List & Delete Documents
```python
# List documents in dialogue
documents = client.documents.list_in_dialogue(dialogue_id: str)

# Delete document from dialogue
client.documents.delete_from_dialogue(dialogue_id: str, filename: str)

# Get document statistics
count = client.documents.get_dialogue_document_count(dialogue_id: str)
total_size = client.documents.get_dialogue_document_size(dialogue_id: str)
```

**Supported File Types:**
- CSV/XLSX (data analysis)
- PDF (knowledge reference)
- Text files (content analysis)
- Images (visual analysis)
- Word documents (.docx)
- PowerPoint (.pptx)

---

## Integrations

### Integration Management
```python
# List integrations
integrations = client.integrations.list()

# Create integration
integration = client.integrations.create(
    name: str,
    type: str,
    configuration: Dict[str, Any],
    metadata: Optional[Dict[str, Any]] = None
)

# Configure integration (simplified)
result = client.integrations.configure(
    type: str,
    configuration: Dict[str, Any]
)

# Delete integration
client.integrations.delete(integration_id: str)
```

---

## Activity & Analytics

### Activity Tracking
```python
# List activities
activities = client.activity.list(
    start_date: Optional[datetime] = None,
    end_date: Optional[datetime] = None,
    limit: int = 100,
    offset: int = 0
)

# Get activity summary
summary = client.activity.summary(
    start_date: Optional[datetime] = None,
    end_date: Optional[datetime] = None
)
```

---

## Organization Management

### Organization Operations
```python
# Get organization details
org = client.organization.get()

# Update organization
org = client.organization.update(
    name: Optional[str] = None,
    settings: Optional[Dict[str, Any]] = None,
    metadata: Optional[Dict[str, Any]] = None
)

# Check system status
status = client.organization.check_system_status()

# Health check
health = client.organization.healthz()
```
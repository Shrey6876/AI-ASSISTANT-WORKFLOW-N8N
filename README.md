# Jerry Assistant

A hierarchical multi-agent AI system for intelligent email, calendar, and meeting management built on n8n and Google Gemini.

## Overview

Jerry is a production-grade workflow orchestration system that leverages specialized AI agents to handle complex administrative tasks across multiple domains. The architecture implements a master-agent pattern where a central orchestrator routes user requests to domain-specific agents that manage email operations, calendar scheduling, and meeting transcript analysis.

## Core Features

- **Master Agent Orchestration**: Intelligent routing of user requests to appropriate specialized agents
- **Email Agent**: Compose, fetch, reply, draft, and delete emails with natural language understanding
- **Calendar Agent**: Schedule events, check availability, and manage calendar operations with temporal awareness
- **Meeting Agent**: Retrieve and analyze meeting transcripts from Fireflies.ai with speaker-level granularity
- **Persistent Memory**: Separate context windows for each agent domain to maintain conversation continuity
- **Dynamic Parameter Generation**: LLM-driven API parameter construction for flexible automation

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Chat Trigger (Webhook)               │
└──────────────────────┬──────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────┐
│                   Master Agent                           │
│    (Gemini LLM + 10-message memory buffer)              │
│                                                          │
│    System Prompt: Route to Email / Calendar / Meeting   │
└────────────┬──────────────────┬────────────────────────┘
             │                  │
             ▼                  ▼
      ┌────────────────┐  ┌────────────────┐
      │  Email Agent   │  │ Calendar Agent │
      │ (Specialized   │  │  (Specialized  │
      │    Routing)    │  │    Routing)    │
      └────────┬───────┘  └────────┬───────┘
               │                   │
    ┌──────────┼──────────┐    ┌───┴──────────────────┐
    │          │          │    │                      │
    ▼          ▼          ▼    ▼                      ▼
  [Get]    [Send]    [Reply]  [Get Events]    [Check Availability]
 Emails   Message   Message    [Create]         [Create Event]
  [Draft] [Delete]             [Update]         [Delete Event]
                                [Delete]         [Date & Time]
                                                        │
                                                        ▼
                                              ┌──────────────────┐
                                              │ Meeting Agent    │
                                              │ (Specialized)    │
                                              └────────┬─────────┘
                                                       │
                                        ┌──────────────┴──────────┐
                                        │                         │
                                        ▼                         ▼
                                   [List Transcripts]    [Fetch Transcript]
                                   (Fireflies API)      (Fireflies API)
```

## System Components

### Master Agent
- **Role**: Central orchestrator and request router
- **LLM**: Google Gemini Chat Model
- **Memory**: 10-message context window
- **Decision Logic**: Analyzes user intent and routes to Email, Calendar, or Meeting agents

### Email Agent
- **Role**: Handles all email operations
- **LLM**: Google Gemini Chat Model
- **Memory**: Configurable message buffer
- **Capabilities**: 
  - Fetch emails with filtering
  - Compose and send new emails
  - Reply to specific messages
  - Create draft emails
  - Delete messages by ID

### Calendar Agent
- **Role**: Manages calendar and scheduling tasks
- **LLM**: Google Gemini Chat Model
- **Memory**: Configurable message buffer
- **Pre-execution**: Retrieves current date/time context
- **Capabilities**:
  - List events within time range
  - Check calendar availability
  - Create new events
  - Update existing events
  - Delete calendar events

### Meeting Agent
- **Role**: Analyzes and retrieves meeting transcripts
- **LLM**: Google Gemini Chat Model
- **Memory**: Configurable message buffer
- **API Integration**: Fireflies.ai GraphQL API
- **Capabilities**:
  - List available transcripts
  - Retrieve full transcript with speaker data
  - Extract timestamps and participant information

## Prerequisites

### Software
- n8n instance (self-hosted or cloud)
- Node.js 18+ (if using n8n CLI tools)

### API Credentials Required
- Google Gemini API keys (3 separate keys for different agents)
- Gmail OAuth2 credentials
- Google Calendar OAuth2 credentials
- Fireflies.ai API token

### Infrastructure
- HTTPS webhook endpoint for chat trigger
- Network access to all external APIs
- Secure credential storage mechanism

## Installation & Setup

### 1. n8n Instance Setup

**Self-hosted Option**:
```
docker run -it --rm --name n8n -p 5678:5678 n8nio/n8n
```

**Cloud Option**:
Visit app.n8n.cloud and create an account.

### 2. Credential Configuration

Open n8n credentials panel and configure:

**Google Gemini API (x3)**:
- Navigate to Credentials > New
- Select "Google Gemini (PaLM) API"
- Add API keys for three separate agents

**Gmail OAuth2**:
- Navigate to Credentials > New
- Select "Gmail OAuth2"
- Authenticate with your Gmail account
- Grant permission for email management

**Google Calendar OAuth2**:
- Navigate to Credentials > New
- Select "Google Calendar OAuth2"
- Authenticate with your Google account
- Grant permission for calendar access

**Fireflies.ai API**:
- Navigate to Credentials > New
- Select "HTTP request" for manual header setup
- Add Bearer token: `Authorization: Bearer YOUR_API_KEY`

### 3. Workflow Import

**Option A - Direct Import**:
- In n8n, select "New" > "Import from URL"
- Paste your Jerry-assistant.json file URL

**Option B - File Upload**:
- Export Jerry-assistant.json to local system
- In n8n, select "New" > "Import from file"
- Upload the JSON file

### 4. Webhook Configuration

- Locate "When chat message received" node
- Copy the webhook URL
- Configure your chat interface to POST to this endpoint

### 5. Memory Configuration

Navigate to each memory node and adjust context window:
- Master Agent Memory: 10 messages (default)
- Email Agent Memory: 5-10 messages
- Calendar Agent Memory: 5-10 messages
- Meeting Agent Memory: 5-10 messages

## API Integration Details

### Gmail API

**Get Many Messages**:
- Fetches emails matching optional filters
- Returns: Message ID, subject, sender, snippet, timestamp
- Used by: Email Agent for reading and extracting IDs

**Send Message**:
- Composes and sends new email
- Parameters: recipient, subject, message body
- Used by: Email Agent for composing

**Reply to Message**:
- Replies to existing thread
- Parameters: message ID (from prior fetch), reply text
- Used by: Email Agent for responding

**Create Draft**:
- Saves email without sending
- Parameters: subject, message body
- Used by: Email Agent for drafting

**Delete Message**:
- Removes message from mailbox
- Parameters: message ID (from prior fetch)
- Used by: Email Agent for deletion

### Google Calendar API

**Get Many Events**:
- Lists events within specified timeframe
- Parameters: calendar ID, timeMin, timeMax
- Returns: Event ID, title, start time, end time, attendees
- Used by: Calendar Agent for reading schedule

**Get Availability**:
- Returns free/busy slots
- Parameters: calendar ID, timeMin, timeMax
- Used by: Calendar Agent for finding availability

**Create Event**:
- Schedules new calendar event
- Parameters: summary, start time, end time, description
- Used by: Calendar Agent for scheduling

**Update Event**:
- Modifies existing event
- Parameters: event ID (from prior fetch), updated fields
- Used by: Calendar Agent for rescheduling

**Delete Event**:
- Removes event from calendar
- Parameters: event ID (from prior fetch)
- Used by: Calendar Agent for cancellation

**Date & Time**:
- Returns current system date and time
- Used by: Calendar Agent for temporal context

### Fireflies.ai API

**List Transcripts** (GraphQL):
```graphql
query {
  transcripts {
    id
    title
    date
  }
}
```

**Get Transcript** (GraphQL):
```graphql
query Transcript($transcriptId: String!) {
  transcript(id: $transcriptId) {
    id
    title
    sentences {
      speaker_name
      text
      start_time
      end_time
    }
    meeting_attendees {
      displayName
      email
      name
    }
    participants
  }
}
```

## System Prompts

### Master Agent

```
You are a Master Agent Orchestrator who handles all things Email, Calendar and Meetings.

1. If the user asks for Email related query, Use the Email Agent Tool
2. If the user asks for Calendar related query, Use the Calendar Agent Tool
3. If the user asks for Meeting related query, Use the Meeting Agent Tool
4. If the enquiry consists of the combination of commands, use the appropriate tools in the order of user's requirement to fulfill the request
```

### Email Agent

```
You are an expert email assistant who handles all things Email.

1. If the user asks for send/compose/write an email, use the Send message tool
2. If the user asks for Summarize/read/fetch emails, use the Get Many messages tool
3. If the user asks to draft something, use the Draft a message tool
4. If user either asks for Reply or Delete a message:
   a. First read the emails in focus via Get Many messages tool
   b. Extract the Message ID relevant for the user's query and use the appropriate tool to perform the action
```

### Calendar Agent

```
You are an expert calendar assistant who handles all things Calendar.

Before you perform any action, get the currentDate from Date & Time tool

1. If the user asks for schedule/create an event, use the create event tool
2. If the user asks for read/fetch events, use the Get Many events tool
3. If the user asks to check availability, use the availability tool
4. If user either asks for Update or Delete an event:
   a. First read the events in focus via Get Many events tool
   b. Extract the Event ID relevant for the user's query and use the appropriate tool to perform the action
```

### Meeting Agent

```
You are an expert meeting assistant who handles all things meetings.

1. For every user query, first get all the list of transcripts using list of transcripts tool
2. Extract the relevant transcript ID from the list above, and construct the body for the next API call using the Transcript tool with the extracted ID
3. Process the transcript data and provide relevant information to the user
```

## Operation Patterns

### Multi-Step Operations

**Email Reply Flow**:
```
User Request → Email Agent → Get Many Messages (extract ID) → Reply Tool (with ID) → Response
```

**Calendar Update Flow**:
```
User Request → Calendar Agent → Get Current Date/Time → Get Events (extract ID) → Update Tool (with ID) → Response
```

**Meeting Retrieval Flow**:
```
User Request → Meeting Agent → List Transcripts (extract ID) → Get Transcript (with ID) → Response
```

### Dynamic Parameter Generation

All tool parameters use AI-generated values through the `$fromAI()` function:
```
{{ $fromAI('Parameter_Name', '', 'type') }}
```

This allows the LLM to:
- Analyze user intent
- Extract relevant information
- Generate properly formatted API parameters
- Handle natural language variations

## Error Handling

### Common Issues

**API Authentication Failures**:
- Verify credential tokens are current
- Check OAuth2 token expiration
- Regenerate credentials if necessary

**Missing Message/Event IDs**:
- Ensure Get Many Messages or Get Events is called before delete/reply/update operations
- Verify ID extraction from API responses

**Temporal Issues**:
- Calendar Agent always retrieves current date first
- Relative date references handled by LLM context

**Rate Limiting**:
- Implement exponential backoff for repeated requests
- Space API calls across time windows

## Performance Optimization

### Memory Configuration

**Master Agent**: 10 messages (maintains cross-domain context)
**Sub-Agents**: 5-10 messages (domain-specific context)

Increase for longer conversations, decrease for resource constraints.

### Parallel Execution

The master agent can route complex multi-step requests by sequentially calling agents. For independent operations, consider using n8n's parallel execution nodes.

### Webhook Considerations

- Use HTTPS endpoints
- Implement request validation
- Consider rate limiting on webhook endpoint
- Monitor webhook logs for failures

## Usage Examples

### Example 1: Email Composition
```
User: "Send an email to john@example.com with subject 'Meeting Follow-up' and the message 'Hi John, following up on our discussion today.'"

Flow: Master Agent → Email Agent → Send Message Tool → Confirmation
```

### Example 2: Calendar Scheduling
```
User: "Schedule a meeting with the team tomorrow at 2 PM for 1 hour titled 'Sprint Planning'"

Flow: Master Agent → Calendar Agent → Get Date/Time → Create Event Tool → Confirmation
```

### Example 3: Meeting Analysis
```
User: "What was discussed in the meeting with the product team?"

Flow: Master Agent → Meeting Agent → List Transcripts → Get Transcript → Analysis
```

### Example 4: Multi-Domain Request
```
User: "Send an email to sarah@example.com about tomorrow's standup meeting, then update my calendar to add it at 9 AM"

Flow: Master Agent → Email Agent → Send Message → Calendar Agent → Create Event → Confirmation
```

## Monitoring & Logging

### Execution Logs

Access execution history in n8n dashboard:
- Click on workflow name
- View "Executions" tab
- Click execution ID to see detailed logs
- Monitor for errors, warnings, and performance metrics

### Webhook Logs

Monitor webhook requests:
- View incoming webhook data
- Track request timestamps
- Monitor response times
- Debug parameter extraction issues

## Data Privacy & Security

### Credentials Management

- Store API keys in n8n's encrypted credential system
- Never commit credentials to version control
- Rotate keys periodically
- Use environment-specific credentials

### Data Handling

- Email content processed in-memory
- Calendar data accessed through OAuth2
- Meeting transcripts retrieved on-demand
- No data persistence beyond workflow execution

### OAuth2 Security

- Use server-side OAuth2 flow
- Tokens stored securely in n8n
- Implement token refresh mechanisms
- Revoke access if compromised

## Scaling Considerations

### Horizontal Scaling

- Deploy multiple n8n instances
- Use load balancer for webhook distribution
- Share credential store across instances

### Vertical Scaling

- Increase memory buffer for longer conversations
- Add more parallel execution workers
- Optimize LLM response times

### Rate Limiting

- Gmail API: 250 requests per second
- Google Calendar API: 1000 requests per 100 seconds
- Fireflies.ai: Check tier limits
- Implement exponential backoff

## Troubleshooting

### Workflow Execution Fails

1. Check execution logs for specific error message
2. Verify all credentials are valid and not expired
3. Ensure webhook endpoint is accessible
4. Test individual nodes in isolation

### Agent Routing Errors

1. Review Master Agent system prompt
2. Check that sub-agents are properly connected
3. Verify LLM is responding appropriately
4. Test with explicit commands before natural language

### API Integration Issues

1. Verify API key format and validity
2. Check API rate limits and quotas
3. Ensure OAuth2 tokens are not expired
4. Test API endpoints independently

## Support & Documentation

For detailed setup and troubleshooting:
- Refer to ARCHITECTURE.md for technical design
- Check SETUP.md for installation steps
- Review examples/ directory for usage patterns

## License

MIT License - See LICENSE file for details

## Version

Current Version: 1.0.0
Last Updated: January 27, 2026

## Status

Production Ready - Tested with n8n 1.0+

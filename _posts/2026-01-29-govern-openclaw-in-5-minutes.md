---
layout: post
title: "Govern OpenClaw in 5 Minutes: A Technical Guide"
date: 2026-01-29
author: Srirajasekhar "Bobby" Koritala
categories: [ai, governance, openclaw, tutorial]
excerpt: "Your team is using OpenClaw. Maybe you approved it, maybe you didn't. Here's how to connect it to the AICtrlNet Runtime Gateway in 5 minutes."
---

## The Problem You're Solving

Your team is using OpenClaw. Maybe you approved it, maybe you didn't. Either way, there's an AI agent running with root-level access to developer machines, executing commands autonomously.

You need visibility. You need control. You need it now.

This guide shows you how to connect OpenClaw to the AICtrlNet Runtime Gateway in 5 minutes.

## What You'll Get

After completing this guide:

- **Every OpenClaw action** is evaluated before execution
- **Risk scores** (0.0-1.0) prioritize what needs human attention
- **ALLOW/DENY/ESCALATE** decisions on every action
- **Complete audit trail** for compliance
- **One-click suspend** if something goes wrong

## Prerequisites

- An OpenClaw instance running (local or remote)
- Python 3.9+
- An AICtrlNet account (free trial works)

## Step 1: Install the SDK (30 seconds)

```bash
pip install aictrlnet-runtime-sdk
```

## Step 2: Get Your API Credentials (60 seconds)

1. Log into [hitlai.net](https://hitlai.net)
2. Navigate to **Settings > API Keys**
3. Click **Create API Key**
4. Copy the key (you won't see it again)

Set it as an environment variable:

```bash
export AICTRLNET_API_KEY="your-api-key-here"
export AICTRLNET_API_URL="https://api.aictrlnet.com"
```

## Step 3: Register Your OpenClaw Instance (60 seconds)

Create a file called `register_openclaw.py`:

```python
import asyncio
from aictrlnet_runtime_sdk import (
    AsyncAICtrlNetClient,
    RuntimeRegistrationRequest,
    AICtrlNetConfig
)

async def main():
    # Initialize client
    config = AICtrlNetConfig.from_env()
    client = AsyncAICtrlNetClient(config)

    # Register OpenClaw instance
    registration = await client.register(RuntimeRegistrationRequest(
        runtime_type="openclaw",
        instance_name="my-dev-machine",
        metadata={
            "owner": "engineering-team",
            "environment": "development"
        }
    ))

    print(f"Registered! Runtime ID: {registration.runtime_id}")
    print(f"Event channel: {registration.event_channel_url}")

    # Save the runtime ID for later
    with open(".aictrlnet_runtime_id", "w") as f:
        f.write(registration.runtime_id)

asyncio.run(main())
```

Run it:

```bash
python register_openclaw.py
```

You should see:
```
Registered! Runtime ID: rt_abc123...
Event channel: wss://api.aictrlnet.com/events/...
```

## Step 4: Wrap OpenClaw Actions (120 seconds)

Now the key part: wrapping OpenClaw's action execution with governance.

Create `governed_openclaw.py`:

```python
import asyncio
from aictrlnet_runtime_sdk import (
    AsyncAICtrlNetClient,
    GovernanceGateway,
    ActionRequest,
    AICtrlNetConfig
)

async def main():
    # Load config and runtime ID
    config = AICtrlNetConfig.from_env()
    client = AsyncAICtrlNetClient(config)

    with open(".aictrlnet_runtime_id") as f:
        runtime_id = f.read().strip()

    # Create governance gateway
    gateway = GovernanceGateway(
        client=client,
        runtime_id=runtime_id
    )

    # Example: Govern a shell command
    async def execute_command(cmd: str) -> str:
        """Your actual command execution logic."""
        import subprocess
        result = subprocess.run(cmd, shell=True, capture_output=True, text=True)
        return result.stdout

    # Wrap with governance
    governed_execute = gateway.wrap(
        action_type="shell_command",
        func=execute_command
    )

    # Now this command will be evaluated before execution
    try:
        result = await governed_execute("ls -la /tmp")
        print(f"Result: {result}")
    except gateway.ActionDenied as e:
        print(f"Action was denied: {e.reason}")
    except gateway.ActionEscalated as e:
        print(f"Action requires approval: {e.approval_url}")

asyncio.run(main())
```

## Step 5: View in Dashboard (30 seconds)

1. Go to your HitLai dashboard
2. You should see your registered OpenClaw instance
3. Click on it to see:
   - All actions evaluated
   - Risk scores
   - Decisions (ALLOW/DENY/ESCALATE)
   - Audit trail

## What Just Happened?

```
OpenClaw wants to run: ls -la /tmp
                 |
                 v
  +-------------------------------------+
  |    AICtrlNet Runtime Gateway         |
  |                                      |
  |  1. Receive action request           |
  |  2. Evaluate through Q/G/S/M        |
  |  3. Calculate risk score (0.0-1.0)   |
  |  4. Make decision based on policy    |
  |  5. Log to audit trail               |
  +-------------------------------------+
                 |
      +----------+----------+
      v          v          v
   ALLOW       DENY     ESCALATE
  (proceed)   (block)   (wait for
                        human approval)
```

## Configuring Policies

By default, the gateway uses sensible defaults:

| Action Type | Default Policy |
|-------------|----------------|
| Read operations | ALLOW |
| Write to /tmp | ALLOW |
| Write elsewhere | ESCALATE |
| Network requests | ALLOW with logging |
| Destructive commands (rm, drop, delete) | ESCALATE |
| Credential access | DENY |

To customize, go to **Settings > Governance Policies** in the dashboard.

## Advanced: Batch Registration for Teams

If you're rolling this out to your whole engineering team:

```python
# bulk_register.py
import asyncio
from aictrlnet_runtime_sdk import AsyncAICtrlNetClient, AICtrlNetConfig

async def register_team():
    config = AICtrlNetConfig.from_env()
    client = AsyncAICtrlNetClient(config)

    team_members = [
        {"name": "alice-macbook", "owner": "alice@company.com"},
        {"name": "bob-linux", "owner": "bob@company.com"},
        {"name": "carol-workstation", "owner": "carol@company.com"},
    ]

    for member in team_members:
        reg = await client.register(RuntimeRegistrationRequest(
            runtime_type="openclaw",
            instance_name=member["name"],
            metadata={"owner": member["owner"], "department": "engineering"}
        ))
        print(f"Registered {member['name']}: {reg.runtime_id}")

asyncio.run(register_team())
```

## Troubleshooting

### "Connection refused" error

Check that your API URL is correct:
```bash
echo $AICTRLNET_API_URL
# Should be: https://api.aictrlnet.com
```

### Actions not appearing in dashboard

1. Verify the runtime ID matches
2. Check that the governance wrapper is actually being called
3. Look at the client logs: `export AICTRLNET_LOG_LEVEL=DEBUG`

### "Action timed out" error

The gateway has a 30-second timeout by default. For long-running actions:

```python
gateway = GovernanceGateway(
    client=client,
    runtime_id=runtime_id,
    timeout=120  # 2 minutes
)
```

## Next Steps

1. **Set up team policies**: Define what should ALLOW, DENY, or ESCALATE
2. **Configure notifications**: Get Slack alerts for ESCALATE decisions
3. **Enable ML risk scoring**: Let the system learn your patterns (Business tier)
4. **Add more agents**: The same gateway works for Claude Code, custom agents, etc.

---

AICtrlNet provides the governance layer for the agentic AI era. Whether you're running OpenClaw, Claude Code, or custom autonomous systems, the Runtime Gateway gives you visibility, control, and compliance.

**Start your free 14-day trial**: [hitlai.net/trial](https://hitlai.net/trial) | **Open source**: [GitHub](https://github.com/Bodaty/aictrlnet-community)

*Questions? Join the discussion on [GitHub](https://github.com/Bodaty/aictrlnet-community/discussions).*

# member

![](https://media.tenor.com/Yc0X0ZFY25QAAAAe/you-member-memberberries.png)

Inspired by the member berries. MongoDB powered configurable secure memory layer.

----

# MDB-Engine and the Quest for Persistent Intelligence

**"Member when AI agents actually remembered your name? Member when they knew your brother liked golf?"**

---

## The Statelessness Problem: A Digital Amnesia

Every AI developer eventually hits the same wall: Large Language Models are, by nature, ephemeral. Ask a standard agent about a conversation from yesterday, and it draws a blank. Tell your customer service bot that you're allergic to peanuts, and watch it recommend pad thai tomorrow.

This is the **Statelessness Problem**—the fundamental limitation where every interaction is treated as an isolated event, devoid of historical context or temporal continuity.

While the industry races to solve this with basic vector stores and TTL (Time-To-Live) deletions, **MDB-Engine** takes a different approach. We aren't just building a database; we're building a **Silicon Hippocampus.**

---

## The Philosophy of "Membering"

Inspired by the *Member Berries*—those ubiquitous icons of nostalgia—MDB-Engine is designed to give your agents a sense of "history." But true intelligence requires more than just hoarding data; it requires a **Cognitive Architecture** that mirrors how biological memory actually functions.

The art of remembering is inseparable from the necessity of forgetting.

### The Cognitive Stack

| Layer | Biological Analog | MDB-Engine Implementation |
|-------|-------------------|---------------------------|
| **Sensory Buffer** | Working Memory | Chat History (STM) |
| **Short-Term** | Hippocampus | Session-scoped memories |
| **Long-Term** | Neocortex | Vector-indexed facts with decay |
| **Semantic** | Conceptual Knowledge | GraphRAG entity relationships |
| **Episodic** | Autobiographical | Timestamped, context-rich memories |

---

## The Math of Forgetting: The Ebbinghaus Curve

Human memory isn't binary. We don't just "have" a memory or "lose" it. It **decays**. MDB-Engine implements the **Ebbinghaus Forgetting Curve** directly within its MongoDB aggregation pipelines.

The retrieval strength **S** of a memory is calculated as:

```
S = R × exp(-t / H)
```

Where:
- **R**: Raw Importance (Initial significance assigned by the LLM, 0.1-1.0)
- **t**: Time elapsed since the last retrieval (hours)
- **H**: Stability (The "half-life" of the memory)

### The Decay in Action

| Memory Age | Importance 0.8 | Importance 0.5 | Importance 0.3 |
|------------|----------------|----------------|----------------|
| Fresh (0h) | 0.80 | 0.50 | 0.30 |
| 24 hours | 0.29 | 0.18 | 0.11 |
| 48 hours | 0.11 | 0.07 | 0.04 |
| 1 week | 0.00 | 0.00 | 0.00 |

But here's the magic: **memories that are retrieved grow stronger**.

### The Spacing Effect

Every retrieval increases stability:

```
H_new = H_old × (1.2 + similarity + emotion × 1.5)
```

A memory accessed daily becomes nearly permanent. A memory never accessed fades into the digital ether. This isn't just engineering—it's **neuroscience**.

### Server-Side Decay Pipeline

By running this calculation in MongoDB's aggregation pipeline, MDB-Engine ensures that the most "salient" memories rise to the top, while the noise naturally fades away—saving your context window from pollution:

```javascript
// Server-side Ebbinghaus decay calculation
{
  "$addFields": {
    "strength": {
      "$multiply": [
        "$importance",
        { "$exp": { "$divide": [{ "$multiply": ["$t_hours", -1] }, "$stability"] } }
      ]
    }
  }
},
{
  "$addFields": {
    "final_score": { "$multiply": ["$similarity", "$strength"] }
  }
},
{ "$sort": { "final_score": -1 } }
```

---

## GraphRAG: "Membering" Relationships

Traditional RAG (Retrieval-Augmented Generation) is great at finding a needle in a haystack, but it's terrible at understanding how the needles are **connected**.

MDB-Engine leverages MongoDB's `$graphLookup` to implement **GraphRAG**. This allows your agent to traverse relationships rather than just matching keywords.

### The Difference

| Query | Standard RAG | MDB-Engine GraphRAG |
|-------|--------------|---------------------|
| "Gift for brother?" | "User mentioned brother once" | "Alex is user's brother → Alex likes golf → Golf clubs are a good gift" |
| "Who knows someone at Google?" | "No direct matches" | "Sarah works at Google. Mike knows Jane who works at Google." |
| "What do book club members like?" | "Book club mentioned 3 times" | "Alice likes wine, Bob likes hiking, Carol likes both → Wine tasting event" |

### The $graphLookup Magic

```javascript
// Navigating the knowledge graph in a single database operation
{
  "$graphLookup": {
    "from": "__kg",
    "startWith": "$edges.target",
    "connectFromField": "edges.target",
    "connectToField": "_id",
    "as": "network",
    "maxDepth": 2,
    "depthField": "hop_distance",
    "restrictSearchWithMatch": {
      "app_slug": "myapp",
      "edges.active": true
    }
  }
}
```

No separate graph database. No extra infrastructure. Just MongoDB doing what MongoDB does best.

### Graph Schema

```
user ──brother──► person:alex ──likes──► interest:golf
                              └──works_at──► organization:google
                              └──lives_in──► location:seattle
```

---

## Flashbulb Memory: When Emotion Burns In

Remember where you were on 9/11? Your wedding day? The moment you got the job?

High-emotion events create **flashbulb memories**—exceptionally vivid and durable. MDB-Engine replicates this with **emotional salience detection**:

```python
# LLM extracts emotion score during fact extraction
{
  "text": "User just got promoted to VP of Engineering",
  "importance": 0.95,
  "emotion": 0.9,      # High emotion = flashbulb memory
  "stability": 114.0   # Initial stability boosted by emotion
}
```

**Stability formula with emotion:**
```
H_initial = default_stability + (emotion × max_multiplier)
```

A neutral fact (emotion=0.2) might have stability of 44 hours.  
A life-changing event (emotion=0.95) gets stability of **119 hours**.

The agent will "member" your promotion far longer than your lunch order.

---

## The AgentCore Question: Why Not Just Use AWS?

Amazon Bedrock AgentCore offers an excellent managed environment for those deep in the AWS ecosystem. Let's be honest about the comparison:

### Where AgentCore Excels

- **Zero Infrastructure**: Fully serverless, auto-scaling
- **Enterprise Support**: AWS backing and SLAs
- **MicroVM Isolation**: Secure multi-tenant compute
- **Episodic Learning**: Their headline "Reflection Module"

### Where MDB-Engine Leads

| Feature | MDB-Engine | AgentCore |
|---------|------------|-----------|
| **Ebbinghaus Decay** | Server-side formula | TTL-based |
| **Flashbulb Memory** | Emotion-based stability | Not available |
| **GraphRAG** | MongoDB $graphLookup | Not available |
| **Conflict Detection** | LLM-powered | Not available |
| **Privacy Redaction** | Built-in patterns | Encryption only |
| **Self-Hosted** | Yes | AWS only |
| **Open Source** | Yes | No |

### The Philosophical Difference

AgentCore treats memory as **data to be stored and retrieved**.

MDB-Engine treats memory as **a cognitive process to be simulated**.

One is a filing cabinet. The other is a brain.

---

## Bucket Isolation: Compartmentalized "Membering"

Enterprise applications need memory isolation. Your work assistant shouldn't surface personal memories, and vice versa.

MDB-Engine's **Bucket-Aware Memory** system ensures complete isolation:

```python
# Chat with "work" context - only work memories retrieved
result = await cognitive_engine.chat(
    user_id="user_123",
    user_query="What meetings do I have?",
    bucket_id="category:work:user_123",
    bucket_type="category"
)

# Switch to "personal" - work memories are invisible
result = await cognitive_engine.chat(
    user_id="user_123", 
    user_query="What did I plan for this weekend?",
    bucket_id="category:personal:user_123",
    bucket_type="category"
)
```

### How It Works

- **Conversation memories**: `associated_bucket_id = category:work:user_123`
- **File memories**: Uploaded documents linked via `associated_bucket_id`
- **Search**: Filters by `associated_bucket_id` to find BOTH types

Your agent "members" the right things in the right context.

---

## The Safety Layer: Forgetting What Must Be Forgotten

"Membering" too much can be a liability. SSNs, credit cards, passwords—some things should never persist.

MDB-Engine includes a configurable **Redaction Layer** that acts as a filter before the memory is even written:

```json
{
  "memory_config": {
    "redaction": {
      "enabled": true,
      "replacement": "[REDACTED]",
      "patterns": {
        "ssn": true,
        "credit_card": true,
        "password": true,
        "api_key": true,
        "phone": false,
        "email": false
      },
      "allow_list": ["support@company.com"]
    }
  }
}
```

**Before storage**: `"My SSN is 123-45-6789"`  
**After redaction**: `"My SSN is [REDACTED]"`

GDPR, CCPA, HIPAA—compliance isn't optional. MDB-Engine makes it automatic.

---

## Conflict Resolution: When Memories Contradict

What happens when your agent "members" two conflicting facts?

- "User is allergic to shellfish" (stored 2024-01-01)
- "User loves seafood" (new input)

Traditional systems store both, leading to **digital dementia**—an agent that confidently holds contradictory beliefs.

MDB-Engine's **Conflict Detection** uses LLM-powered semantic analysis:

```python
conflict = await memory_service.detect_knowledge_conflict(
    user_id="user_123",
    new_fact="User loves seafood"
)

if conflict:
    # Ask for clarification or update old memory
    print(f"Wait... I thought you mentioned: {conflict}")
```

Your agent won't gaslight itself.

---

## Architectural Deep-Dive: The Cognitive Engine

Implementing memory shouldn't take three months of engineering. The DIY checklist is brutal:

- [ ] Vector database setup
- [ ] Embedding generation
- [ ] User isolation (security)
- [ ] Session management
- [ ] Memory conflicts
- [ ] Decay logic
- [ ] Pruning
- [ ] Audit trails (GDPR)
- [ ] Category management
- [ ] Bucket isolation
- [ ] Privacy redaction
- [ ] Reflection/consolidation
- [ ] Analytics
- [ ] Async extraction
- [ ] Importance scoring
- [ ] Emotional salience
- [ ] Graph extraction
- [ ] Hybrid search
- [ ] Edge deactivation
- [ ] Cross-session recovery

**Estimated DIY time: 3-6 months.**

### MDB-Engine: 15 Minutes to Production

```json
{
  "memory_config": {
    "enabled": true,
    "provider": "cognitive",
    "enable_cognitive": true,
    "cognitive": {
      "decay": { "enabled": true },
      "emotion": { "enabled": true },
      "conflict_resolution": { "enabled": true },
      "pruning": { "max_capacity": 1000 }
    },
    "graph": { "enabled": true },
    "redaction": { "enabled": true }
  }
}
```

### Real-World Implementation

Using the `CognitiveEngine`, you can initiate a chat that automatically handles storage, retrieval, decay ranking, and fact extraction in one call:

```python
from mdb_engine.memory import CognitiveEngine

# Single call handles the heavy lifting
result = await cognitive_engine.chat(
    user_id="user_123",
    session_id="conversation:456",
    user_query="What should I get for my brother's birthday?",
    bucket_id="category:personal:user_123",
    bucket_type="category",
    extract_facts=True  # Automatically "members" new info
)

# Output includes:
print(result["response"])         # The AI response
print(result["ltm_memories"])     # Decay-ranked Long-Term Memories
print(result["graph_context"])    # Graph-traversed relationship context
print(result["memories_stored"])  # New facts extracted and stored
```

---

## Cold Storage: The Art of Graceful Forgetting

When memories become too weak, MDB-Engine doesn't delete them—it moves them to **Cold Storage**.

```python
# Soft-delete: memories marked inactive, not destroyed
{
  "is_active": false,
  "pruned_at": "2026-02-04T10:00:00Z",
  "pruning_reason": "capacity_limit_reached"
}
```

This provides:
- **Audit Trail**: Know what was "forgotten" and when
- **Recovery**: Restore critical memories if needed
- **Analytics**: Understand memory patterns

```python
# Retrieve pruned memories for analysis
cold_memories = memory_service.get_cold_storage(user_id="user_123")

# Restore a specific memory
memory_service.restore_from_cold_storage(
    memory_id="507f1f77bcf86cd799439011",
    user_id="user_123"
)
```

---

## The Memory Health Dashboard

Monitor your agent's cognitive health:

```python
analytics = memory_service.get_memory_analytics(user_id="user_123")

# {
#   "active_memories": 450,
#   "cold_storage_memories": 150,
#   "capacity_used": 0.45,
#   "average_strength": 0.62,
#   "weak_memories": 45,      # strength < 0.3
#   "strong_memories": 120,   # strength > 0.7
#   "categories": {
#     "biographical": 80,
#     "preferences": 150,
#     "work": 100,
#     "health": 60
#   }
# }
```

---

## The Bottom Line: Own Your Context

The future of AI is not just better models, but better **state management**. Whether you are building a personal assistant that needs to "member" your coffee order or an enterprise bot managing complex client relationships, your agent is only as good as its memory.

MDB-Engine isn't just a database layer; it's a framework for **persistent, evolving intelligence**.

### The MDB-Engine Promise

| Principle | Implementation |
|-----------|----------------|
| **Memories decay** | Ebbinghaus forgetting curve |
| **Emotion matters** | Flashbulb memory boost |
| **Retrieval strengthens** | Spacing effect |
| **Relationships exist** | GraphRAG traversal |
| **Context is sacred** | Bucket isolation |
| **Privacy is paramount** | Built-in redaction |
| **Forgetting is graceful** | Cold storage audit trail |
| **You own your data** | Open source, self-hostable |

---

## Ready to "Member"?

```bash
pip install mdb-engine
```

```python
from mdb_engine import MongoDBEngine

engine = MongoDBEngine()
engine.register_app("manifest.json")
engine.run()
```

**"Member the future? We do."**

---

## Quick Reference: Manifest Configuration

### Healthcare Assistant (High Memory Retention)

```json
{
  "memory_config": {
    "enabled": true,
    "provider": "cognitive",
    "cognitive": {
      "decay": {
        "enabled": true,
        "default_stability_hours": 168
      },
      "emotion": {
        "enabled": true,
        "flashbulb_threshold": 0.6
      },
      "pruning": {
        "max_capacity": 2000,
        "strategy": "soft_delete"
      },
      "cold_storage": {
        "enabled": true,
        "retention_days": 2555
      }
    },
    "redaction": {
      "enabled": true,
      "patterns": {
        "ssn": true,
        "phone": true,
        "email": true
      }
    },
    "graph": { "enabled": true }
  }
}
```

### Financial Advisor (Strict Isolation)

```json
{
  "memory_config": {
    "enabled": true,
    "provider": "cognitive",
    "cognitive": {
      "decay": { "enabled": true },
      "conflict_resolution": {
        "enabled": true,
        "similarity_threshold": 0.9
      },
      "pruning": { "max_capacity": 500 }
    },
    "categories": {
      "enabled": true,
      "custom_categories": ["investments", "retirement", "taxes", "insurance"]
    },
    "redaction": {
      "enabled": true,
      "patterns": {
        "ssn": true,
        "credit_card": true,
        "api_key": true
      }
    },
    "graph": { "enabled": false }
  }
}
```

### Personal Assistant (Relationship-Focused)

```json
{
  "memory_config": {
    "enabled": true,
    "provider": "cognitive",
    "cognitive": {
      "decay": {
        "enabled": true,
        "default_stability_hours": 48
      },
      "emotion": {
        "enabled": true,
        "max_stability_multiplier": 100
      }
    },
    "graph": {
      "enabled": true,
      "auto_extract": true,
      "default_max_depth": 2,
      "node_types": ["person", "interest", "event", "location", "organization"]
    },
    "entities": {
      "enabled": true,
      "auto_extract": true
    }
  }
}
```


---

*MDB-Engine is an open-source project. Contributions welcome.*

*Member when open source was the future? It still is.*

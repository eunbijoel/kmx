[한국어](README.md) | [English](README_EN.md)

# KMX Platform — Korean Manufacturing-X Data Space Platform

> **Reference implementation of an integrated Data Space + AI + Governance platform**  
> Korean Manufacturing-X (KMX) Data Space Platform

---

## Architecture Overview

```
┌────────────────────────────────────────────────────────────────┐
│                   KMX Platform Architecture                    │
│                                                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │ Users/Firms  │  │  AI Agents   │  │ Ext. Connectors(Fed) │  │
│  └──────┬───────┘  └──────┬───────┘  └──────────┬───────────┘  │
│         │                 │                     │              │
│  ╔══════╧═════════════════╧═════════════════════╧══════╗       │
│  ║             Identity & Governance Layer             ║       │
│  ║  ┌──────────┐  ┌──────────┐  ┌───────────────────┐  ║       │
│  ║  │  DID/VC  │  │  ODRL    │  │  Contract Manager │  ║       │
│  ║  │  Mgmt    │  │Policy Eng│  │  Contract Mgmt    │  ║       │
│  ║  └──────────┘  └──────────┘  └───────────────────┘  ║       │
│  ╚═════════════════════════════════════════════════════╝       │
│                                                                │
│  ╔═════════════════════════════════════════════════════════╗   │
│  ║                    Data Space Layer                     ║   │
│  ║  ┌──────────────────┐    ┌─────────────────────────┐    ║   │
│  ║  │   Control Plane  │    │      Data Plane         │    ║   │
│  ║  │  - Negotiation   │◄──►│  - P2P data transfer    │    ║   │
│  ║  │  - Policy checks │    │  - Format conversion    │    ║   │
│  ║  │  - Conn. routing │    │  - Data sovereignty     │    ║   │
│  ║  └──────────────────┘    └─────────────────────────┘    ║   │
│  ╚═════════════════════════════════════════════════════════╝   │
│                                                                │
│  ╔══════════════╗   ╔══════════════════════════════════════╗   │
│  ║   AI Layer   ║   ║       Data Intelligence Layer        ║   │
│  ║  ┌─────────┐ ║   ║  ┌──────────┐  ┌──────────────────┐  ║   │
│  ║  │PredMaint│ ║   ║  │ Metadata │  │Vector Search Eng.│  ║   │
│  ║  │QualCheck│ ║   ║  │Extraction│  │NL Semantic Search│  ║   │
│  ║  │ProcessOp│ ║   ║  └──────────┘  └──────────────────┘  ║   │
│  ║  │DemandFct│ ║   ║  ┌──────────┐  ┌──────────────────┐  ║   │
│  ║  │EnergyOpt│ ║   ║  │ Ontology │  │  DCAT Catalog    │  ║   │
│  ║  └─────────┘ ║   ║  │ Mapping  │  │ Dataset Registry │  ║   │
│  ╚══════════════╝   ║  └──────────┘  └──────────────────┘  ║   │
│                     ╚══════════════════════════════════════╝   │
│                                                                │
│  ╔═════════════════════════════════════════════════════════╗   │
│  ║        Clearing House (Notarization/Settlement)         ║   │
│  ║ Hash-chain immutable audit log · usage-based settlement ║   │
│  ╚═════════════════════════════════════════════════════════╝   │
└────────────────────────────────────────────────────────────────┘
```

---

## Data Flow Diagram

```
User request
    │
    ▼
┌─────────────────────────────────────────────────────────────────┐
│ Step 1: Agent Execution (Optional)                              │
│   User → delegates authority to the agent (Delegation VC issued)│
└────────────────────────┬────────────────────────────────────────┘
                         │
    ▼
┌─────────────────────────────────────────────────────────────────┐
│ Step 2: DID/VC Authentication                                   │
│   - Verify requester DID (did:kmx:xxxx)                         │
│   - Validate the Verifiable Credential                          │
│   - Verify the signature (Ed25519)                              │
└────────────────────────┬────────────────────────────────────────┘
                         │ ✅ Authentication passed
    ▼
┌─────────────────────────────────────────────────────────────────┐
│ Step 3: Contract Check (Control Plane)                          │
│   - Check whether a data usage contract exists                  │
│   - Check the contract status (ACTIVE)                          │
│   - Check whether the contract has expired                      │
└────────────────────────┬────────────────────────────────────────┘
                         │ ✅ Contract valid
    ▼
┌─────────────────────────────────────────────────────────────────┐
│ Step 4: ODRL Policy Check                                       │
│   - Is the requested action (use/read/distribute) allowed?      │
│   - Evaluate constraints (time, purpose, count)                 │
│   - Prohibitions checked first                                  │
└────────────────────────┬────────────────────────────────────────┘
                         │ ✅ Policy permits
    ▼
┌─────────────────────────────────────────────────────────────────┐
│ Step 5: Data Plane Data Transfer                                │
│   - Direct P2P transfer (no central storage)                    │
│   - Format conversion (JSON/CSV)                                │
│   - Payload hash generation                                     │
└────────────────────────┬────────────────────────────────────────┘
                         │
    ▼
┌─────────────────────────────────────────────────────────────────┐
│ Step 6: Clearing House Log Recording                            │
│   - Generate a hash-chain log                                   │
│   - Link it to the previous log (tamper prevention)             │
│   - Accumulate usage settlement data                            │
└─────────────────────────────────────────────────────────────────┘
                         │
    ▼
          Return response (data + transfer ID + hash)
```

---

## Project Structure

```
kmx-platform/
├── backend/
│   ├── main.py                    # FastAPI main app
│   ├── requirements.txt
│   ├── test_core.py               # Unit tests for core logic (52 tests)
│   │
│   ├── api/                       # API routers
│   │   ├── connector_routes.py    # EDC connector API
│   │   ├── identity_routes.py     # DID/VC API
│   │   ├── policy_routes.py       # ODRL policy API
│   │   ├── contract_routes.py     # Contract API
│   │   ├── metadata_routes.py     # Metadata/ontology API
│   │   ├── ai_routes.py           # AI model API
│   │   ├── clearinghouse_routes.py# Clearing House API
│   │   ├── agent_routes.py        # Agentic AI API
│   │   └── search_routes.py       # Vector search API
│   │
│   ├── connector/
│   │   ├── control_plane.py       # EDC Control Plane
│   │   └── data_plane.py          # EDC Data Plane (P2P transfer)
│   │
│   ├── identity/
│   │   ├── did.py                 # W3C DID management
│   │   └── vc.py                  # Verifiable Credentials
│   │
│   ├── policy/
│   │   └── odrl_engine.py         # ODRL policy engine
│   │
│   ├── contract/
│   │   └── contract_manager.py    # Data contract management
│   │
│   ├── metadata/
│   │   └── extractor.py           # DCAT metadata extraction
│   │
│   ├── semantic/
│   │   ├── ontology_mapper.py     # Manufacturing ontology mapping
│   │   └── vector_search.py       # Semantic search
│   │
│   ├── ai/
│   │   ├── model_api.py           # 5 core manufacturing AI models
│   │   └── agent.py               # Agentic AI module
│   │
│   ├── clearinghouse/
│   │   └── logger.py              # Hash-chain audit log
│   │
│   └── db/
│       ├── models.py              # SQLAlchemy ORM models
│       └── database.py            # DB connection/initialization
│
├── docker/
│   └── Dockerfile.backend
├── docker-compose.yml
└── README.md
```

---

## Quick Start

### Option 1: Run Directly (SQLite - for development)

```bash
# 1. Clone the repository
git clone https://github.com/your-org/kmx-platform
cd kmx-platform/backend

# 2. Install dependencies
pip install -r requirements.txt

# 3. Run the server
uvicorn main:app --reload --host 0.0.0.0 --port 8000

# 4. Check the API docs
# http://localhost:8000/docs
```

### Option 2: Docker Compose (recommended for production)

```bash
# Run the full stack (PostgreSQL + Backend + Provider Connector)
docker-compose up -d

# Check logs
docker-compose logs -f kmx-backend

# Stop
docker-compose down
```

### Option 3: Test the Core Logic (no dependencies)

```bash
cd backend
python3 test_core.py
# → Verify that all 52 tests pass
```

---

## API Specification

### Base URL: `http://localhost:8000/api/v1`

#### 🔐 Identity (DID/VC)
| Method | Path | Description |
|--------|------|------|
| POST | `/identity/did` | Create a DID |
| GET | `/identity/did/{did}` | Look up a DID |
| POST | `/identity/vc` | Issue a VC |
| GET | `/identity/vc/{vc_id}/verify` | Verify a VC |
| POST | `/identity/vc/delegate` | Issue an agent delegation VC |

#### 🔗 Connector (EDC)
| Method | Path | Description |
|--------|------|------|
| POST | `/connector/register` | Register a connector |
| GET | `/connector/list` | List connectors |
| POST | `/connector/negotiate` | Start contract negotiation |
| POST | `/connector/data/register` | Register a dataset |
| POST | `/connector/data/transfer` | P2P data transfer |

#### 📋 Policy (ODRL)
| Method | Path | Description |
|--------|------|------|
| POST | `/policy/` | Create a policy |
| GET | `/policy/` | List policies |
| POST | `/policy/evaluate` | Evaluate a policy |

#### 📝 Contract
| Method | Path | Description |
|--------|------|------|
| POST | `/contract/` | Create a contract |
| POST | `/contract/{id}/sign` | Sign a contract |
| GET | `/contract/{id}/verify` | Verify a contract |

#### 🏷️ Metadata
| Method | Path | Description |
|--------|------|------|
| POST | `/metadata/extract` | Extract/store metadata |
| GET | `/metadata/` | Dataset catalog |
| POST | `/metadata/ontology/map` | Ontology mapping |
| GET | `/metadata/ontology/concepts` | List ontology concepts |

#### 🤖 AI
| Method | Path | Description |
|--------|------|------|
| POST | `/ai/predict` | Run an AI prediction |
| GET | `/ai/models` | List models |
| GET | `/ai/models/{type}/metadata` | Model metadata |
| GET | `/ai/models/{type}/health` | Model status |

#### 🤖 Agent (Agentic AI)
| Method | Path | Description |
|--------|------|------|
| POST | `/agent/initialize` | Initialize an agent |
| POST | `/agent/delegate` | Delegate authority |
| POST | `/agent/auto-catalog` | Auto-generate a catalog |
| GET | `/agent/health` | Agent status |

#### 🔍 Search
| Method | Path | Description |
|--------|------|------|
| POST | `/search/datasets` | Natural-language dataset search |
| GET | `/search/datasets?q={query}` | Keyword search |
| POST | `/search/ontology` | Ontology-based search |

#### 📊 Clearing House
| Method | Path | Description |
|--------|------|------|
| GET | `/clearinghouse/logs` | Query transfer logs |
| GET | `/clearinghouse/verify-chain` | Verify hash-chain integrity |
| GET | `/clearinghouse/usage-report` | Usage settlement report |

---

## Scenario Example: Hyundai Motor → Samsung Electronics Data Exchange

```bash
BASE="http://localhost:8000/api/v1"

# 1. Create DIDs
PROVIDER=$(curl -s -X POST $BASE/identity/did \
  -H "Content-Type: application/json" \
  -d '{"controller":"Hyundai Motor-Supply Chain","entity_type":"connector"}')
PROVIDER_DID=$(echo $PROVIDER | python3 -c "import sys,json; print(json.load(sys.stdin)['did'])")

CONSUMER=$(curl -s -X POST $BASE/identity/did \
  -H "Content-Type: application/json" \
  -d '{"controller":"Samsung Electronics-Gumi Plant","entity_type":"human"}')
CONSUMER_DID=$(echo $CONSUMER | python3 -c "import sys,json; print(json.load(sys.stdin)['did'])")

# 2. Create a policy
POLICY=$(curl -s -X POST $BASE/policy/ \
  -H "Content-Type: application/json" \
  -d "{
    \"title\": \"Manufacturing Data Sharing Policy\",
    \"target\": \"supply-chain-dataset-001\",
    \"assigner\": \"$PROVIDER_DID\",
    \"permissions\": [{
      \"action\": \"use\",
      \"constraints\": [
        {\"leftOperand\": \"purpose\", \"operator\": \"eq\", \"rightOperand\": \"manufacturing\"}
      ]
    }],
    \"prohibitions\": [{\"action\": \"distribute\", \"constraints\": []}]
  }")
POLICY_ID=$(echo $POLICY | python3 -c "import sys,json; print(json.load(sys.stdin)['uid'])")

# 3. Register connectors and negotiate a contract → transfer data
# (Then proceed in the order negotiate, sign, transfer)

echo "Provider DID: $PROVIDER_DID"
echo "Policy ID: $POLICY_ID"
```

---

## Standards Compliance

| Standard | Implementation |
|------|-----------|
| **W3C DID Core 1.0** | `did:kmx:` method, DID Document structure |
| **W3C VC Data Model 1.1** | VC issuance/verification, Ed25519 signatures |
| **ODRL 2.2** | Permission/Prohibition/Obligation, ODRL JSON-LD |
| **DCAT 2** | Dataset metadata, Distribution structure |
| **IDS-RAM** | Control/Data Plane separation, Contract negotiation protocol |
| **ISO 62443** | Industrial cybersecurity (reflected in the architecture) |

---

## Environment Variables

| Variable | Default | Description |
|------|--------|------|
| `DATABASE_URL` | `sqlite+aiosqlite:///./kmx_platform.db` | DB connection string |
| `CONNECTOR_ID` | Auto-generated | Connector ID of this instance |

---

## License

Apache 2.0 — KMX Platform is an open-source reference implementation.

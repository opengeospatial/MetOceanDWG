# OGC API Service Profile Builder

## Motivation

OGC API standards provide flexible frameworks for geospatial data access, but their generality creates challenges:

1. **Implementation Variability**: Different implementations of the same standard can vary significantly in structure, making client integration difficult
2. **Documentation Burden**: Creating formal OGC-compliant documentation (requirements, abstract tests, OpenAPI specs) is time-consuming and error-prone
3. **Validation Gap**: No authoritative tooling exists to validate that a profile definition is internally consistent before deployment
4. **Service Discovery**: Clients need machine-readable service profiles to understand what a specific service implementation provides

**Service Profiles** (OGC API - EDR Part 3) address these issues by defining constrained, domain-specific implementations with:
- Fixed collection structures
- Normative requirements and conformance tests
- Machine-readable OpenAPI/AsyncAPI specifications
- Formal OGC documentation

However, manually authoring profiles is complex—requiring expertise in OpenAPI, AsyncAPI, Metanorma/AsciiDoc, and OGC standards. The **OGC API Service Profile Builder** automates this process.

## High-Level Approach

The tool uses **Pydantic models as the authoritative schema** for profile definitions:

```
            YAML → Pydantic Validation → Generated Artifacts
                         ↓
                   Cross-model validation
                   Referential integrity
                   OGC Part 3 compliance
```

**Key Design Principles:**

1. **Single Source of Truth**: Collection metadata, requirements, and tests all defined in one config file
2. **Validation First**: Pydantic validates the entire profile structure before generating any output
3. **Standards Compliance**: Built on [edr-pydantic](https://github.com/KNMI/edr-pydantic) (community-maintained EDR Pydantic models) and enforces OGC API - EDR Part 3 requirements
4. **Automation**: Generates OpenAPI, AsyncAPI, Metanorma documentation, and OGC PDFs from a single config
5. **Testing Integration**: Built-in OGC CITE conformance testing and schemathesis-based server validation

## Conceptual Design

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│              Profile Config (YAML/JSON)                     │
│  - Collections (edr-pydantic models)                        │
│  - Requirements & Abstract Tests                            │
│  - Pub/Sub Configuration                                    │
│  - Document Metadata                                        │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────────────────┐
│                     Pydantic Validation Layer                           │
│  - Schema validation (correct types, required fields present)           │
│  - Referential integrity (abstract tests reference valid requirements)  │
│  - OGC Part 3 compliance (parameters have units, extents meet minimums) │
└─────────────────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                  Generation Engine                          │
│  ┌───────────────┐  ┌──────────────┐  ┌─────────────────┐   │
│  │ OpenAPI 3.0   │  │ AsyncAPI 3.0 │  │ Metanorma       │   │
│  │ Generator     │  │ Generator    │  │ AsciiDoc        │   │
│  └───────────────┘  └──────────────┘  └─────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    Output Artifacts                         │
│  my_profile/                                                │
│  ├── openapi.yaml                                           │
│  ├── asyncapi.yaml (if pubsub defined)                      │
│  ├── profile_config.json                                    │
│  ├── document.adoc                                          │
│  ├── document.pdf (if --pdf flag used)                      │
│  ├── sections/                                              │
│  │   ├── 00-abstract.adoc                                   │
│  │   ├── 01-preface.adoc                                    │
│  │   ├── 02-scope.adoc                                      │
│  │   ├── 03-conformance.adoc                                │
│  │   ├── 04-references.adoc                                 │
│  │   ├── 05-terms.adoc                                      │
│  │   ├── 06-requirements.adoc                               │
│  │   └── 07-abstract-tests.adoc                             │
│  ├── requirements/                                          │
│  │   ├── requirements_class_core.adoc                       │
│  │   └── core/REQ_*.adoc                                    │
│  └── abstract_tests/                                        │
│      ├── ATS_class_core.adoc                                │
│      └── core/ATS_*.adoc                                    │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                  Validation & Testing                       │
│  - OGC CITE conformance tests (EDR, Features)               │
│  - Schemathesis server validation                           │
│  - Stateful testing (job lifecycle) for Processes           │
└─────────────────────────────────────────────────────────────┘
```

### Data Model

The tool uses **Pydantic models** to define the structure of profile configurations. Pydantic is a Python library that provides:

- **Type validation**: Ensures fields have correct types (string, list, integer, etc.)
- **Required field checking**: Validates that mandatory fields are present
- **Custom validators**: Enforces business logic (e.g., abstract tests must reference existing requirements)
- **Automatic serialization**: Converts between Python objects, JSON, and YAML

**How it works:**

1. **Models define the schema**: Each part of the profile (collections, requirements, tests) is a Pydantic model class
2. **User provides config**: YAML/JSON file with profile data
3. **Pydantic validates**: Checks types, required fields, and custom rules
4. **Generation uses validated data**: Only valid, structured data reaches the file system

**Core Models:**

```python
class ServiceProfile(BaseModel):
    """Root model representing the entire profile"""
    name: str                              # Profile identifier
    title: str                             # Human-readable title
    collections: List[Collection]          # EDR collections (from edr-pydantic)
    requirements: List[Requirement]        # Normative requirements
    abstract_tests: List[AbstractTest]     # Conformance tests
    pubsub: Optional[PubSubConfig]         # Pub/Sub configuration
    extent_requirements: ExtentRequirements # OGC Part 3 extent constraints
    output_formats: List[OutputFormat]     # Format definitions with schemas
    
    @model_validator(mode='after')
    def validate_test_requirements(self):
        """Custom validator: ensure all tests reference valid requirements"""
        req_ids = {r.id for r in self.requirements}
        for test in self.abstract_tests:
            if test.requirement_id not in req_ids:
                raise ValueError(f"Test {test.id} references non-existent requirement")
        return self

class Requirement(BaseModel):
    """A normative requirement"""
    id: str                                # Unique identifier
    statement: str                         # One-sentence requirement
    parts: List[str]                       # SHALL/MUST clauses

class AbstractTest(BaseModel):
    """A conformance test"""
    id: str                                # Test identifier
    requirement_id: str                    # Must match a Requirement.id
    steps: List[str]                       # Test steps
```

**Key Benefits:**

- **Fail fast**: Invalid configs are rejected before any files are generated
- **Clear error messages**: Pydantic tells you exactly what's wrong (e.g., "requirement_id 'foo' not found in requirements")
- **Type safety**: Can't accidentally pass a string where a list is expected
- **Self-documenting**: Model definitions serve as authoritative schema documentation

**Example Validation:**

```yaml
# Invalid config
requirements:
  - id: position-query
    
abstract_tests:
  - id: test-items
    requirement_id: items-endpoint  # ← ERROR: doesn't exist!
```

Pydantic catches this immediately:
```
ValueError: Test test-items references non-existent requirement 'items-endpoint'
```

**Key Validation Rules:**

1. **Referential Integrity**: Abstract tests must reference existing requirements
2. **Parameter Compliance**: All EDR parameters must specify `unit` and `observedProperty` (OGC Part 3 REQ_parameter-names)
3. **Unique Collection IDs**: No duplicate collection identifiers
4. **PubSub Conformance**: Auto-adds Part 2 conformance requirement when pubsub is defined
5. **CRS Specification**: Either `allowed_crs` list or `crs_pattern` regex must be provided in extent requirements


### Workflow

```
┌──────────────┐
│ 1. Author    │  Create profile config (YAML/JSON)
│    Config    │  Define collections, requirements, tests
└──────┬───────┘
       │
       ↓
┌──────────────┐
│ 2. Validate  │  oapi-profile-builder validate --config profile.yaml
│    Profile   │  Pydantic checks schema + cross-model consistency
└──────┬───────┘
       │
       ↓
┌──────────────┐
│ 3. Generate  │  oapi-profile-builder generate --config profile.yaml
│    Artifacts │  Produces OpenAPI, AsyncAPI, AsciiDoc sections
└──────┬───────┘
       │
       ↓
┌──────────────┐
│ 4. Compile   │  oapi-profile-builder generate --config profile.yaml --pdf
│    PDF       │  Docker-based Metanorma compilation → OGC PDF
└──────┬───────┘
       │
       ↓
┌──────────────┐
│ 5. Test      │  oapi-profile-builder validate-server --url https://...
│    Server    │  Schemathesis-based API testing
└──────┬───────┘
       │
       ↓
┌──────────────┐
│ 6. CITE      │  oapi-profile-builder cite-test --url https://...
│    Testing   │  oapi-profile-builder cite-test-features --url https://...
│              │  OGC conformance testing (EDR, Features)
└──────────────┘
```

## Tool Usage

### Installation

```bash
pip install oapi-profile-builder
```

### Basic Commands

#### 1. Validate a Profile Config

```bash
oapi-profile-builder validate --config my_profile.yaml
```

Checks:
- YAML/JSON syntax
- Pydantic schema compliance
- Cross-model referential integrity
- OGC Part 3 requirements

#### 2. Generate Profile Artifacts

```bash
oapi-profile-builder generate \
  --config my_profile.yaml \
  --output ./my_profile
```

Produces:
- `openapi.yaml` - OpenAPI 3.0 specification
- `asyncapi.yaml` - AsyncAPI 3.0 specification (if `pubsub` defined)
- `document.adoc` - Metanorma root document
- `sections/*.adoc` - Abstract, Preface, Scope, Conformance, Requirements, Tests
- `requirements/` - Individual requirement AsciiDoc files
- `abstract_tests/` - Individual test AsciiDoc files

#### 3. Compile OGC PDF

```bash
oapi-profile-builder generate \
  --config my_profile.yaml \
  --output ./my_profile \
  --pdf
```

Requires Docker. Produces `document.pdf` using the official `metanorma/metanorma` image.

#### 4. Validate Against Live Server

```bash
oapi-profile-builder validate-server \
  --config my_profile.yaml \
  --url https://edr-api.example.com \
  --max-examples 3
```

Uses [schemathesis](https://schemathesis.readthedocs.io/) to:
- Generate test cases from OpenAPI spec
- Test all endpoints with valid/invalid inputs
- Verify response schemas
- Check HTTP status codes

Add `--stateful` for job lifecycle testing (POST `/execution` → GET `/jobs/{jobId}` → DELETE `/jobs/{jobId}`).

#### 5. OGC CITE Conformance Testing

**EDR Conformance:**

```bash
oapi-profile-builder cite-test \
  --url https://edr-api.example.com \
  --report ./cite_results
```

Runs the official OGC API - EDR Part 1 test suite (ets-ogcapi-edr10).

**Features Conformance:**

```bash
oapi-profile-builder cite-test-features \
  --url https://api.example.com \
  --report ./cite_features_results
```

Runs the official OGC API - Features Part 1 test suite (ets-ogcapi-features10).

Both commands:
- Automatically pull/build Docker images
- Support `--network host` for localhost testing
- Generate JSON reports with detailed results

### Example Profile Config

Minimal valid profile:

```yaml
name: water_gauge
title: Water Gauge EDR Profile
version: "1.0"

required_conformance_classes:
  - "http://www.opengis.net/spec/ogcapi-edr-1/1.0/conf/core"

extent_requirements:
  minimum_bbox: [-180, -90, 180, 90]
  allowed_crs:
    - "http://www.opengis.net/def/crs/OGC/1.3/CRS84"

output_formats:
  - name: GeoJSON
    media_type: application/geo+json
    schema_ref: https://geojson.org/schema/FeatureCollection.json

collections:
  - id: water_gauge
    title: Water Gauge Observations
    links:
      - href: https://example.com/collections/water_gauge
        rel: self
        type: application/json
    extent:
      spatial:
        bbox:
          - [-180, -90, 180, 90]
        crs: "http://www.opengis.net/def/crs/OGC/1.3/CRS84"
    parameter_names:
      gauge_height:
        type: Parameter
        observedProperty:
          label: Gauge Height
        unit:
          label: feet
          symbol: ft

requirements:
  - id: items-endpoint
    statement: The service SHALL provide a /collections/water_gauge/items endpoint.
    parts:
      - The service SHALL return GeoJSON FeatureCollection.
      - Each feature SHALL include gauge_height property.

abstract_tests:
  - id: items-endpoint
    requirement_id: items-endpoint
    steps:
      - Send GET request to /collections/water_gauge/items.
      - Verify response Content-Type is application/geo+json.
      - Verify each feature contains gauge_height property.
```

See [examples/nws_connect_profile.yaml](https://github.com/ShaneMill1/OGC-API-Service-Profile-Builder/blob/main/examples/nws_connect_profile.yaml) for a complete profile with 4 collections, 10 requirements, 10 abstract tests, pub/sub configuration, and document metadata.

## OGC API - EDR Part 3 Compliance

The tool enforces OGC API - EDR Part 3: Service Profiles requirements:

### REQ_publishing
- Generated OpenAPI has empty `servers` array (profile is implementation-independent)
- Landing page schema requires `profile` link relation
- Profile URI advertised in `x-ogc-profile` info field

### REQ_api
- `/conformance` endpoint schema specifies required conformance classes
- Defaults to EDR Core, customizable via `required_conformance_classes`

### REQ_parameter-names
- Validates that all parameters specify `unit` and `observedProperty`
- Automatically enforced during profile validation

### REQ_extent
- Profile-level `extent_requirements` specify minimum bounds
- CRS/TRS/VRS restrictions via enumerated lists or regex patterns

### REQ_output-format
- Profile-level `output_formats` with schema references
- Links to JSON Schema, XML Schema, or format specifications

### REQ_pubsub
- Automatically adds Part 2 conformance requirement when `pubsub` is present
- AsyncAPI document specifies channels and payloads

## Use Cases

### 1. NWS Connect Profile

Real-time water gauge and weather alert service with pub/sub:

- **4 collections**: water_gauge, wwa (watches/warnings), cwa (boundaries), states
- **10 requirements**: Collection endpoints, items-by-id, pub/sub AMQP, WebSocket servers, per-collection filters
- **10 abstract tests**: Conformance validation for all requirements
- **Pub/Sub**: AMQP broker + WebSocket endpoints with collection-specific filters
- **Output**: OpenAPI, AsyncAPI, OGC PDF documentation

Config: [examples/nws_connect_profile.yaml](https://github.com/ShaneMill1/OGC-API-Service-Profile-Builder/blob/main/examples/nws_connect_profile.yaml)

### 2. NWSViz Profile

Comprehensive weather visualization service:

- **13 collections**: Model data
- **3 processes**: Zarr dataset difference, time series extraction, spatial aggregation
- **Requirements & Tests**: Coverage of all collections and processes
- **Document Metadata**: Full OGC document header for PDF compilation

Config: [examples/nwsviz_profile.yaml](https://github.com/ShaneMill1/OGC-API-Service-Profile-Builder/blob/main/examples/nwsviz_profile.yaml)

### 3. Minimal Profile

Starting point for new profiles:

- **1 collection**: Basic EDR collection with required fields
- **1 requirement**: Items endpoint
- **1 abstract test**: Conformance validation

Config: [examples/minimal_profile.yaml](https://github.com/ShaneMill1/OGC-API-Service-Profile-Builder/blob/main/examples/minimal_profile.yaml)

## Advanced Features

### Pub/Sub Configuration

Define AMQP/MQTT/Kafka pub/sub with per-collection filters:

```yaml
pubsub:
  broker_host: broker.example.com
  broker_port: 5672
  protocol: amqp
  collections:
    - water_gauge
    - wwa
  servers:
    - name: websocket_secure
      host: api.example.com
      protocol: wss
      pathname: /ws/
  collection_filters:
    water_gauge:
      filters:
        - name: flood_stages
          description: Filter by flood stage (major, moderate, minor)
          type: array
        - name: stations
          description: Station IDs or 'all'
          type: string
```

**Note:** Filters define client-side subscription parameters. The message broker itself does not perform filtering—it routes all messages. Application-level workers (email workers, WebSocket servers) that consume from the broker apply the filters before forwarding messages to end users based on their subscription preferences.

Generates AsyncAPI 3.0 with:
- Channel definitions for each collection
- WebSocket server bindings
- Filter schema in message payloads

### Process Definitions

Add OGC API - Processes endpoints:

```yaml
processes:
  - id: zarr-difference
    title: Zarr Dataset Difference
    description: Calculate difference between two Zarr datasets
    output_content:
      application/zip:
        schema:
          type: object
```

Generates OpenAPI paths:
- `/processes` - List processes
- `/processes/{id}` - Process description
- `/processes/{id}/execution` - Execute process
- `/jobs` - List jobs
- `/jobs/{jobId}` - Job status
- `/jobs/{jobId}/results` - Job results

### Document Metadata

Control OGC PDF output:

```yaml
document_metadata:
  doc_number: "25-nwsconnect"
  doc_subtype: implementation
  copyright_year: 2025
  editors:
    - Shane Mill
  submitting_orgs:
    - NOAA/NWS/MDL
  keywords:
    - ogcdoc
    - OGC API
    - Features
    - service profile
  external_id: http://www.opengis.net/doc/dp/ogcapi-features-nwsconnect/1.0
```

Produces OGC-compliant PDF with proper headers, copyright, and metadata.

### Collection Examples

Provide real parameter values for server validation:

```yaml
collection_examples:
  water_gauge:
    instanceId: "2025-04-02T00:00:00Z"
  forecast_grid:
    instanceId: "2025-04-02T12:00:00Z"
```

Used by `validate-server` to test endpoints with path parameters.

## Programmatic Use

```python
from oapi_profile_builder.models import ServiceProfile
from oapi_profile_builder.generate import generate
from pathlib import Path

# Load and validate config
with open("profile.yaml") as f:
    config = yaml.safe_load(f)

profile = ServiceProfile.model_validate(config)

# Generate artifacts
output_dir = Path("./output")
generate(profile, output_dir)

# Access validated data
print(f"Profile: {profile.title}")
print(f"Collections: {len(profile.collections)}")
print(f"Requirements: {len(profile.requirements)}")
```

## Standards & References

- **OGC API - EDR Part 1**: Core (https://docs.ogc.org/is/19-086r6/19-086r6.html)
- **OGC API - EDR Part 2**: Pub/Sub (https://docs.ogc.org/DRAFTS/20-131.html)
- **OGC API - EDR Part 3**: Service Profiles (draft standard)
- **OGC API - Features Part 1**: Core (https://docs.ogc.org/is/17-069r4/17-069r4.html)
- **OGC API - Processes Part 1**: Core (https://docs.ogc.org/is/18-062r2/18-062r2.html)
- **edr-pydantic**: Authoritative EDR Pydantic models (https://github.com/KNMI/edr-pydantic)
- **OpenAPI 3.0**: API specification format (https://spec.openapis.org/oas/v3.0.3)
- **AsyncAPI 3.0**: Async API specification (https://www.asyncapi.com/docs/reference/specification/v3.0.0)
- **Metanorma**: OGC document authoring (https://www.metanorma.org/)

## Repository

- **GitHub**: https://github.com/ShaneMill1/OGC-API-Service-Profile-Builder
- **PyPI**: https://pypi.org/project/oapi-profile-builder/
- **License**: MIT
- **Author**: Shane Mill (NOAA/NWS/MDL)
- **Email**: shane.mill@noaa.gov

# GGUF File Format: Complete Structural Guide

## Overview

**GGUF** (GPT-Generated Unified Format) is a binary file format designed for storing large language models. It is the successor to the older GGML format and is primarily used by **llama.cpp** and compatible inference engines. GGUF is optimized for fast loading, memory-mapped access, and single-file distribution of quantized models.

---

## High-Level File Structure

A GGUF file is organized into four major logical regions, read sequentially:

```mermaid
flowchart TB
    subgraph GGUF["GGUF FILE STRUCTURE"]
        direction TB
        A["📋 HEADER<br/>Magic + Version + Counts"]
        B["🏷️ METADATA KV PAIRS<br/>Model Configuration"]
        C["📐 TENSOR INFOS<br/>Tensor Descriptors"]
        D["💾 TENSOR DATA<br/>Quantized Weights"]
    end
    
    A --> B --> C --> D
    
    style A fill:#4a90d9,stroke:#2c5282,color:#fff
    style B fill:#48bb78,stroke:#276749,color:#fff
    style C fill:#ed8936,stroke:#c05621,color:#fff
    style D fill:#9f7aea,stroke:#6b46c1,color:#fff
```

---

## Section 1: Header

The header is the first **24 bytes** of every GGUF file and contains essential identification and counting information.

### Header Layout

```mermaid
flowchart LR
    subgraph HEADER["HEADER (24 bytes)"]
        direction LR
        M["MAGIC<br/>4 bytes<br/>'GGUF'"]
        V["VERSION<br/>4 bytes<br/>uint32"]
        T["TENSOR_COUNT<br/>8 bytes<br/>uint64"]
        K["METADATA_KV_COUNT<br/>8 bytes<br/>uint64"]
    end
    
    M --> V --> T --> K
    
    style M fill:#e53e3e,stroke:#c53030,color:#fff
    style V fill:#dd6b20,stroke:#c05621,color:#fff
    style T fill:#d69e2e,stroke:#b7791f,color:#fff
    style K fill:#38a169,stroke:#276749,color:#fff
```

### Header Fields Explained

| Offset | Size | Field | Type | Description |
|--------|------|-------|------|-------------|
| 0x00 | 4 | Magic | char[4] | ASCII string `GGUF` (0x47475546) - identifies file format |
| 0x04 | 4 | Version | uint32_t | Format version (currently 3) |
| 0x08 | 8 | Tensor Count | uint64_t | Number of tensors stored in the file |
| 0x10 | 8 | Metadata KV Count | uint64_t | Number of metadata key-value pairs |

### Version History

```mermaid
timeline
    title GGUF Version Evolution
    section v1
        Initial Release : Basic structure
    section v2  
        Alignment : Added padding for memory mapping
    section v3
        Current : Big-endian support, improved compatibility
```

---

## Section 2: Metadata Key-Value Pairs

Immediately following the header, metadata is stored as a sequence of key-value pairs. This section contains all model configuration, architecture details, tokenizer data, and custom attributes.

### Metadata Structure Overview

```mermaid
flowchart TB
    subgraph META["METADATA SECTION"]
        direction TB
        KV1["Key-Value Pair 1"]
        KV2["Key-Value Pair 2"]
        KV3["Key-Value Pair 3"]
        KVN["Key-Value Pair N..."]
    end
    
    subgraph KVPAIR["SINGLE KV PAIR STRUCTURE"]
        direction LR
        KL["Key Length<br/>uint64"]
        KS["Key String<br/>UTF-8"]
        VT["Value Type<br/>uint32"]
        VD["Value Data<br/>variable"]
    end
    
    KV1 --> KV2 --> KV3 --> KVN
    
    style KL fill:#4299e1,stroke:#2b6cb0,color:#fff
    style KS fill:#48bb78,stroke:#2f855a,color:#fff
    style VT fill:#ed8936,stroke:#c05621,color:#fff
    style VD fill:#9f7aea,stroke:#6b46c1,color:#fff
```

### Value Types

GGUF supports multiple data types for metadata values:

```mermaid
flowchart TB
    subgraph TYPES["GGUF VALUE TYPES"]
        direction TB
        
        subgraph SCALAR["SCALAR TYPES"]
            T0["0: UINT8"]
            T1["1: INT8"]
            T2["2: UINT16"]
            T3["3: INT16"]
            T4["4: UINT32"]
            T5["5: INT32"]
            T6["6: FLOAT32"]
            T7["7: BOOL"]
            T10["10: UINT64"]
            T11["11: INT64"]
            T12["12: FLOAT64"]
        end
        
        subgraph COMPLEX["COMPLEX TYPES"]
            T8["8: STRING<br/>length + UTF-8 data"]
            T9["9: ARRAY<br/>type + count + elements"]
        end
    end
    
    style T8 fill:#48bb78,stroke:#2f855a,color:#fff
    style T9 fill:#ed8936,stroke:#c05621,color:#fff
```

### Type ID Reference Table

| Type ID | Name | Size | Description |
|---------|------|------|-------------|
| 0 | UINT8 | 1 byte | Unsigned 8-bit integer |
| 1 | INT8 | 1 byte | Signed 8-bit integer |
| 2 | UINT16 | 2 bytes | Unsigned 16-bit integer |
| 3 | INT16 | 2 bytes | Signed 16-bit integer |
| 4 | UINT32 | 4 bytes | Unsigned 32-bit integer |
| 5 | INT32 | 4 bytes | Signed 32-bit integer |
| 6 | FLOAT32 | 4 bytes | 32-bit floating point |
| 7 | BOOL | 1 byte | Boolean (0 or 1) |
| 8 | STRING | variable | Length-prefixed UTF-8 string |
| 9 | ARRAY | variable | Homogeneous typed array |
| 10 | UINT64 | 8 bytes | Unsigned 64-bit integer |
| 11 | INT64 | 8 bytes | Signed 64-bit integer |
| 12 | FLOAT64 | 8 bytes | 64-bit floating point |

### Common Metadata Keys

```mermaid
mindmap
  root((Metadata<br/>Keys))
    General
      general.architecture
      general.name
      general.author
      general.license
      general.quantization_version
    Architecture
      llama.context_length
      llama.embedding_length
      llama.block_count
      llama.attention.head_count
      llama.attention.head_count_kv
      llama.rope.freq_base
    Tokenizer
      tokenizer.ggml.model
      tokenizer.ggml.tokens
      tokenizer.ggml.scores
      tokenizer.ggml.token_type
      tokenizer.ggml.bos_token_id
      tokenizer.ggml.eos_token_id
```

### String Encoding Detail

```mermaid
flowchart LR
    subgraph STRING["STRING VALUE ENCODING"]
        direction LR
        SL["String Length<br/>uint64<br/>8 bytes"]
        SD["String Data<br/>UTF-8 bytes<br/>N bytes"]
    end
    
    SL --> SD
    
    style SL fill:#4299e1,stroke:#2b6cb0,color:#fff
    style SD fill:#48bb78,stroke:#2f855a,color:#fff
```

### Array Encoding Detail

```mermaid
flowchart LR
    subgraph ARRAY["ARRAY VALUE ENCODING"]
        direction LR
        AT["Element Type<br/>uint32<br/>4 bytes"]
        AC["Array Count<br/>uint64<br/>8 bytes"]
        AE["Elements<br/>type × count<br/>variable"]
    end
    
    AT --> AC --> AE
    
    style AT fill:#ed8936,stroke:#c05621,color:#fff
    style AC fill:#4299e1,stroke:#2b6cb0,color:#fff
    style AE fill:#9f7aea,stroke:#6b46c1,color:#fff
```

---

## Section 3: Tensor Information

After metadata, the file contains descriptors for each tensor. These descriptors define tensor names, shapes, data types, and offsets into the data section.

### Tensor Info Structure

```mermaid
flowchart TB
    subgraph TENSORS["TENSOR INFO SECTION"]
        direction TB
        T1["Tensor Info 1"]
        T2["Tensor Info 2"]
        T3["Tensor Info 3"]
        TN["Tensor Info N..."]
    end
    
    subgraph TINFO["SINGLE TENSOR INFO"]
        direction LR
        NL["Name Length<br/>uint64"]
        NS["Name String<br/>UTF-8"]
        ND["N Dimensions<br/>uint32"]
        DM["Dimensions<br/>uint64 × N"]
        TT["Tensor Type<br/>uint32"]
        OF["Data Offset<br/>uint64"]
    end
    
    T1 --> T2 --> T3 --> TN
    
    style NL fill:#4299e1,stroke:#2b6cb0,color:#fff
    style NS fill:#48bb78,stroke:#2f855a,color:#fff
    style ND fill:#ed8936,stroke:#c05621,color:#fff
    style DM fill:#f6e05e,stroke:#d69e2e,color:#333
    style TT fill:#fc8181,stroke:#c53030,color:#fff
    style OF fill:#9f7aea,stroke:#6b46c1,color:#fff
```

### Tensor Info Fields

| Field | Type | Description |
|-------|------|-------------|
| Name Length | uint64_t | Length of tensor name in bytes |
| Name | char[] | UTF-8 encoded tensor name |
| N Dimensions | uint32_t | Number of dimensions (1-4 typically) |
| Dimensions | uint64_t[] | Size of each dimension |
| Type | uint32_t | Quantization/data type ID |
| Offset | uint64_t | Byte offset from start of tensor data section |

### Tensor/Quantization Types

```mermaid
flowchart TB
    subgraph QTYPES["TENSOR DATA TYPES"]
        direction LR
        
        subgraph FULL["FULL PRECISION"]
            F32["F32<br/>32-bit float"]
            F16["F16<br/>16-bit float"]
            BF16["BF16<br/>bfloat16"]
        end
        
        subgraph LEGACY["LEGACY QUANT"]
            Q4_0["Q4_0<br/>4-bit (32 block)"]
            Q4_1["Q4_1<br/>4-bit + min"]
            Q5_0["Q5_0<br/>5-bit"]
            Q5_1["Q5_1<br/>5-bit + min"]
            Q8_0["Q8_0<br/>8-bit"]
        end
        
        subgraph KQUANT["K-QUANTS"]
            Q2_K["Q2_K<br/>2-bit k-quant"]
            Q3_K["Q3_K<br/>3-bit k-quant"]
            Q4_K["Q4_K<br/>4-bit k-quant"]
            Q5_K["Q5_K<br/>5-bit k-quant"]
            Q6_K["Q6_K<br/>6-bit k-quant"]
        end
        
        subgraph IQUANT["I-QUANTS"]
            IQ2["IQ2_*<br/>2-bit importance"]
            IQ3["IQ3_*<br/>3-bit importance"]
            IQ4["IQ4_*<br/>4-bit importance"]
        end
    end
    
    style F32 fill:#48bb78,stroke:#2f855a,color:#fff
    style F16 fill:#4299e1,stroke:#2b6cb0,color:#fff
```

### Quantization Type IDs

| ID | Type | Bits/Weight | Block Size | Description |
|----|------|-------------|------------|-------------|
| 0 | F32 | 32 | 1 | Full 32-bit float |
| 1 | F16 | 16 | 1 | Half precision float |
| 2 | Q4_0 | 4 | 32 | 4-bit quantization |
| 3 | Q4_1 | 4.5 | 32 | 4-bit with min value |
| 6 | Q5_0 | 5 | 32 | 5-bit quantization |
| 7 | Q5_1 | 5.5 | 32 | 5-bit with min value |
| 8 | Q8_0 | 8 | 32 | 8-bit quantization |
| 10 | Q2_K | 2.5 | 256 | K-quant 2-bit |
| 11 | Q3_K | 3.4 | 256 | K-quant 3-bit |
| 12 | Q4_K | 4.5 | 256 | K-quant 4-bit |
| 13 | Q5_K | 5.5 | 256 | K-quant 5-bit |
| 14 | Q6_K | 6.5 | 256 | K-quant 6-bit |
| 30 | BF16 | 16 | 1 | Brain float 16 |

---

## Section 4: Tensor Data

The final and largest section contains the actual tensor weights. Data is aligned for efficient memory-mapped access.

### Alignment and Padding

```mermaid
flowchart TB
    subgraph ALIGN["DATA ALIGNMENT"]
        direction TB
        
        END_META["End of Tensor Info Section"]
        PAD["ALIGNMENT PADDING<br/>Padded to 32-byte boundary"]
        DATA["TENSOR DATA START<br/>Aligned address"]
        
        END_META --> PAD --> DATA
    end
    
    subgraph LAYOUT["TENSOR DATA LAYOUT"]
        direction LR
        T1D["Tensor 1 Data"]
        T2D["Tensor 2 Data"]
        T3D["Tensor 3 Data"]
        TND["Tensor N Data..."]
    end
    
    DATA --> LAYOUT
    T1D --> T2D --> T3D --> TND
    
    style PAD fill:#fc8181,stroke:#c53030,color:#fff
    style DATA fill:#48bb78,stroke:#2f855a,color:#fff
```

### Data Section Properties

- **Alignment**: Data starts at a 32-byte aligned offset (configurable, default 32)
- **Tensor Order**: Tensors are stored in the order their info appears
- **No Padding Between Tensors**: Tensors are packed contiguously (type-specific alignment may apply)
- **Offsets**: Each tensor's offset is relative to the start of the data section

---

## Complete File Layout

```mermaid
flowchart TB
    subgraph FILE["COMPLETE GGUF FILE BYTE LAYOUT"]
        direction TB
        
        subgraph H["BYTES 0-23: HEADER"]
            H1["0x00-0x03: Magic 'GGUF'"]
            H2["0x04-0x07: Version"]
            H3["0x08-0x0F: Tensor Count"]
            H4["0x10-0x17: KV Count"]
        end
        
        subgraph M["VARIABLE: METADATA"]
            M1["KV Pair 1"]
            M2["KV Pair 2"]
            M3["..."]
            MN["KV Pair N"]
        end
        
        subgraph T["VARIABLE: TENSOR INFOS"]
            T1["Tensor Info 1"]
            T2["Tensor Info 2"]
            T3["..."]
            TN["Tensor Info N"]
        end
        
        subgraph P["PADDING"]
            P1["Alignment Padding<br/>to 32-byte boundary"]
        end
        
        subgraph D["BULK: TENSOR DATA"]
            D1["Tensor 1 Weights"]
            D2["Tensor 2 Weights"]
            D3["..."]
            DN["Tensor N Weights"]
        end
    end
    
    H --> M --> T --> P --> D
    
    style H fill:#4a90d9,stroke:#2c5282,color:#fff
    style M fill:#48bb78,stroke:#276749,color:#fff
    style T fill:#ed8936,stroke:#c05621,color:#fff
    style P fill:#a0aec0,stroke:#718096,color:#fff
    style D fill:#9f7aea,stroke:#6b46c1,color:#fff
```

---

## Memory Mapping

GGUF is designed for efficient memory-mapped loading:

```mermaid
flowchart LR
    subgraph DISK["DISK FILE"]
        DF["GGUF File"]
    end
    
    subgraph MMAP["MEMORY MAP"]
        VM["Virtual Memory<br/>Pages"]
    end
    
    subgraph RAM["PHYSICAL RAM"]
        PM["Loaded Pages<br/>On Demand"]
    end
    
    DF -->|"mmap()"| VM
    VM -->|"Page Fault"| PM
    
    style DF fill:#4299e1,stroke:#2b6cb0,color:#fff
    style VM fill:#ed8936,stroke:#c05621,color:#fff
    style PM fill:#48bb78,stroke:#2f855a,color:#fff
```

### Benefits of Memory Mapping

1. **Fast Load Times**: Only metadata is read initially
2. **On-Demand Loading**: Tensor pages loaded only when accessed
3. **Shared Memory**: Multiple processes can share the same mapped file
4. **Low Memory Overhead**: No need to copy data from disk buffer

---

## Typical Model Architecture in GGUF

```mermaid
flowchart TB
    subgraph MODEL["TRANSFORMER MODEL TENSORS"]
        direction TB
        
        subgraph EMB["EMBEDDINGS"]
            TE["token_embd.weight"]
        end
        
        subgraph BLOCKS["TRANSFORMER BLOCKS × N"]
            subgraph ATTN["ATTENTION"]
                QW["blk.N.attn_q.weight"]
                KW["blk.N.attn_k.weight"]
                VW["blk.N.attn_v.weight"]
                OW["blk.N.attn_output.weight"]
            end
            
            subgraph FFN["FEED FORWARD"]
                G["blk.N.ffn_gate.weight"]
                U["blk.N.ffn_up.weight"]
                D["blk.N.ffn_down.weight"]
            end
            
            subgraph NORM["LAYER NORMS"]
                AN["blk.N.attn_norm.weight"]
                FN["blk.N.ffn_norm.weight"]
            end
        end
        
        subgraph OUT["OUTPUT"]
            ON["output_norm.weight"]
            OL["output.weight"]
        end
    end
    
    EMB --> BLOCKS --> OUT
    
    style TE fill:#4299e1,stroke:#2b6cb0,color:#fff
    style OL fill:#48bb78,stroke:#2f855a,color:#fff
```

---

## Reading GGUF in Python

Here's a minimal example for reading GGUF headers on Windows (CPU only):

```python
import struct

def read_gguf_header(filepath):
    """Read GGUF file header information."""
    with open(filepath, 'rb') as f:
        # Read magic
        magic = f.read(4)
        if magic != b'GGUF':
            raise ValueError(f"Invalid GGUF magic: {magic}")
        
        # Read version (uint32, little-endian)
        version = struct.unpack('<I', f.read(4))[0]
        
        # Read tensor count (uint64, little-endian)
        tensor_count = struct.unpack('<Q', f.read(8))[0]
        
        # Read metadata kv count (uint64, little-endian)
        metadata_kv_count = struct.unpack('<Q', f.read(8))[0]
        
        return {
            'magic': magic.decode('ascii'),
            'version': version,
            'tensor_count': tensor_count,
            'metadata_kv_count': metadata_kv_count
        }

# Usage
# header = read_gguf_header("model.gguf")
# print(header)
```

---

## Tools for Working with GGUF

| Tool | Purpose | Platform |
|------|---------|----------|
| `llama.cpp` | Inference engine | Windows/Linux/Mac |
| `gguf-py` | Python GGUF library | Cross-platform |
| `llama-quantize` | Quantization tool | Windows/Linux/Mac |
| `llama-gguf` | GGUF manipulation | Windows/Linux/Mac |

---

## Summary

```mermaid
flowchart TB
    subgraph SUMMARY["GGUF FORMAT SUMMARY"]
        direction LR
        
        S1["🔒 Self-Contained<br/>Single file distribution"]
        S2["⚡ Memory Mapped<br/>Fast loading"]
        S3["📊 Quantized<br/>Reduced size"]
        S4["📋 Metadata Rich<br/>Complete model info"]
    end
    
    style S1 fill:#4299e1,stroke:#2b6cb0,color:#fff
    style S2 fill:#48bb78,stroke:#2f855a,color:#fff
    style S3 fill:#ed8936,stroke:#c05621,color:#fff
    style S4 fill:#9f7aea,stroke:#6b46c1,color:#fff
```

**Key Takeaways:**

1. **Header (24 bytes)**: Magic number, version, counts
2. **Metadata**: Key-value pairs for model configuration and tokenizer
3. **Tensor Info**: Names, dimensions, types, and offsets for each tensor
4. **Tensor Data**: Aligned, quantized weight data

GGUF enables efficient distribution and inference of large language models on consumer hardware, including CPU-only systems like your Lenovo Yoga with 32GB RAM.

---

*Guide generated for GGUF format version 3*

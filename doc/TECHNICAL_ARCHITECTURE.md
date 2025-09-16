# Maude Technical Architecture Documentation

## Overview

Maude is a high-performance reflective language and system supporting both equational and rewriting logic specification and programming. This document provides a comprehensive technical architecture overview of the Maude interpreter, including system components, data flows, and architectural patterns.

## System Architecture

Maude is implemented as a modular C++ system with clear separation of concerns across multiple subsystems. The architecture follows a layered approach with core rewriting logic at the center, surrounded by parsing, built-in theories, and user interface layers.

```mermaid
graph TB
    subgraph "User Interface Layer"
        CLI[Command Line Interface]
        REPL[Interactive REPL]
        FILE[File Processing]
    end
    
    subgraph "Language Frontend"
        PARSER[Parser & Lexer]
        MIXFIX[Mixfix Parser]
        SYNTAX[Syntax Analysis]
    end
    
    subgraph "Core Engine"
        CORE[Core Rewriting Engine]
        UNIFY[Unification Engine]
        MATCH[Pattern Matching]
        MEMO[Memoization]
    end
    
    subgraph "Theory Modules"
        ACU[ACU Theory]
        AU[AU Theory]
        CUI[CUI Theory]
        FREE[Free Theory]
        S[S Theory]
        NA[NA Theory]
    end
    
    subgraph "Built-in Systems"
        BUILTIN[Built-in Types]
        OBJECT[Object System]
        META[Meta-level]
        STRATEGY[Strategy Language]
    end
    
    subgraph "External Integration"
        SMT[SMT Solvers]
        IO[I/O Operations]
        TEMPORAL[Temporal Logic]
    end
    
    CLI --> PARSER
    REPL --> PARSER
    FILE --> PARSER
    
    PARSER --> MIXFIX
    MIXFIX --> SYNTAX
    SYNTAX --> CORE
    
    CORE --> UNIFY
    CORE --> MATCH
    CORE --> MEMO
    
    CORE <--> ACU
    CORE <--> AU
    CORE <--> CUI
    CORE <--> FREE
    CORE <--> S
    CORE <--> NA
    
    CORE <--> BUILTIN
    CORE <--> OBJECT
    CORE <--> META
    CORE <--> STRATEGY
    
    CORE --> SMT
    CORE --> IO
    CORE --> TEMPORAL
```

## Module Dependencies

The Maude system is organized into 25 distinct modules, each with specific responsibilities and dependencies:

```mermaid
graph TD
    subgraph "Foundation Layer"
        UTIL[Utility]
        INTERFACE[Interface]
        VARIABLE[Variable]
    end
    
    subgraph "Parsing Layer"
        PARSER[Parser]
        MIXFIX[Mixfix]
    end
    
    subgraph "Core Logic Layer"
        CORE[Core]
        HIGHER[Higher]
    end
    
    subgraph "Theory Layer"
        NA_THEORY[NA_Theory]
        FREE_THEORY[FreeTheory]
        S_THEORY[S_Theory]
        CUI_THEORY[CUI_Theory]
        ACU_THEORY[ACU_Theory]
        AU_THEORY[AU_Theory]
        ACU_PERSISTENT[ACU_Persistent]
        AU_PERSISTENT[AU_Persistent]
    end
    
    subgraph "Extension Layer"
        BUILTIN[BuiltIn]
        OBJECT_SYSTEM[ObjectSystem]
        META[Meta]
        STRATEGY_LANG[StrategyLanguage]
        TEMPORAL[Temporal]
        SMT[SMT]
        IO_STUFF[IO_Stuff]
        FULL_COMPILER[FullCompiler]
    end
    
    subgraph "Application Layer"
        MAIN[Main]
    end
    
    %% Foundation dependencies
    PARSER --> UTIL
    PARSER --> INTERFACE
    MIXFIX --> PARSER
    MIXFIX --> INTERFACE
    
    %% Core dependencies
    CORE --> UTIL
    CORE --> INTERFACE
    CORE --> VARIABLE
    HIGHER --> CORE
    
    %% Theory dependencies
    NA_THEORY --> CORE
    FREE_THEORY --> CORE
    S_THEORY --> CORE
    CUI_THEORY --> CORE
    ACU_THEORY --> CORE
    AU_THEORY --> CORE
    ACU_PERSISTENT --> ACU_THEORY
    AU_PERSISTENT --> AU_THEORY
    
    %% Extension dependencies
    BUILTIN --> HIGHER
    OBJECT_SYSTEM --> BUILTIN
    META --> BUILTIN
    STRATEGY_LANG --> META
    TEMPORAL --> META
    SMT --> CORE
    IO_STUFF --> BUILTIN
    FULL_COMPILER --> META
    
    %% Application dependencies
    MAIN --> MIXFIX
    MAIN --> STRATEGY_LANG
    MAIN --> TEMPORAL
    MAIN --> SMT
    MAIN --> IO_STUFF
    MAIN --> FULL_COMPILER
```

## Data Flow Architecture

The data flow in Maude follows a clear pipeline from source code to execution:

```mermaid
flowchart TD
    SOURCE[Maude Source Code] --> LEX[Lexical Analysis]
    LEX --> PARSE[Syntactic Analysis]
    PARSE --> MIXFIX_PARSE[Mixfix Parsing]
    MIXFIX_PARSE --> AST[Abstract Syntax Tree]
    
    AST --> MODULE_BUILD[Module Construction]
    MODULE_BUILD --> SORT_CHECK[Sort Checking]
    SORT_CHECK --> THEORY_SELECT[Theory Selection]
    
    THEORY_SELECT --> REWRITE_PREP[Rewrite Preparation]
    REWRITE_PREP --> PATTERN_COMPILE[Pattern Compilation]
    PATTERN_COMPILE --> EXEC_ENGINE[Execution Engine]
    
    subgraph "Execution Context"
        EXEC_ENGINE --> UNIFICATION[Unification]
        EXEC_ENGINE --> MATCHING[Pattern Matching]
        EXEC_ENGINE --> REDUCTION[Term Reduction]
        EXEC_ENGINE --> STRATEGY_EXEC[Strategy Execution]
    end
    
    subgraph "Theory Dispatch"
        UNIFICATION --> ACU_ALG[ACU Algorithms]
        UNIFICATION --> AU_ALG[AU Algorithms]
        UNIFICATION --> FREE_ALG[Free Algorithms]
        UNIFICATION --> CUI_ALG[CUI Algorithms]
    end
    
    subgraph "Built-in Processing"
        REDUCTION --> BUILTIN_OPS[Built-in Operations]
        REDUCTION --> META_OPS[Meta Operations]
        REDUCTION --> OBJECT_OPS[Object Operations]
    end
    
    EXEC_ENGINE --> RESULT[Execution Result]
    RESULT --> OUTPUT[User Output]
    
    %% Feedback loops
    STRATEGY_EXEC -.-> EXEC_ENGINE
    META_OPS -.-> MODULE_BUILD
    
    %% External interactions
    EXEC_ENGINE --> SMT_SOLVE[SMT Solving]
    EXEC_ENGINE --> IO_OPS[I/O Operations]
```

## Build System Architecture

Maude uses GNU Autotools for its build system, providing portability across Unix-like systems:

```mermaid
graph TB
    subgraph "Configuration Phase"
        CONFIGURE_AC[configure.ac]
        M4_MACROS[M4 Macros]
        AUTOCONF[autoconf]
        CONFIGURE[configure script]
    end
    
    subgraph "Makefile Generation"
        MAKEFILE_AM[Makefile.am files]
        AUTOMAKE[automake]
        MAKEFILES[Generated Makefiles]
    end
    
    subgraph "Dependency Libraries"
        GMP[GNU MP Library]
        BUDDY[BuDDy BDD Library]
        TECLA[Tecla Line Editor]
        LIBSIGSEGV[libsigsegv]
        YICES2[Yices2 SMT Solver]
        CVC4[CVC4 SMT Solver]
    end
    
    subgraph "Compilation Units"
        UTIL_LIB[Utility Library]
        PARSER_LIB[Parser Library]
        CORE_LIB[Core Library]
        THEORY_LIBS[Theory Libraries]
        BUILTIN_LIB[Built-in Library]
        MAIN_EXEC[Main Executable]
    end
    
    subgraph "Testing"
        TEST_SUITE[Test Suite]
        CORNER_TESTS[Corner Case Tests]
        BUILTIN_TESTS[Built-in Tests]
        META_TESTS[Meta Tests]
        STRATEGY_TESTS[Strategy Tests]
    end
    
    CONFIGURE_AC --> AUTOCONF
    M4_MACROS --> AUTOCONF
    AUTOCONF --> CONFIGURE
    
    MAKEFILE_AM --> AUTOMAKE
    AUTOMAKE --> MAKEFILES
    
    CONFIGURE --> MAKEFILES
    
    MAKEFILES --> UTIL_LIB
    MAKEFILES --> PARSER_LIB
    MAKEFILES --> CORE_LIB
    MAKEFILES --> THEORY_LIBS
    MAKEFILES --> BUILTIN_LIB
    
    UTIL_LIB --> MAIN_EXEC
    PARSER_LIB --> MAIN_EXEC
    CORE_LIB --> MAIN_EXEC
    THEORY_LIBS --> MAIN_EXEC
    BUILTIN_LIB --> MAIN_EXEC
    
    GMP --> MAIN_EXEC
    BUDDY --> MAIN_EXEC
    TECLA --> MAIN_EXEC
    LIBSIGSEGV --> MAIN_EXEC
    YICES2 -.-> MAIN_EXEC
    CVC4 -.-> MAIN_EXEC
    
    MAIN_EXEC --> TEST_SUITE
    TEST_SUITE --> CORNER_TESTS
    TEST_SUITE --> BUILTIN_TESTS
    TEST_SUITE --> META_TESTS
    TEST_SUITE --> STRATEGY_TESTS
```

## Core Components

### 1. Utility Layer
- **Location**: `src/Utility/`
- **Purpose**: Fundamental data structures, memory management, and utility functions
- **Key Features**: Vector templates, memory allocation, basic algorithms

### 2. Interface Layer  
- **Location**: `src/Interface/`
- **Purpose**: Core interfaces and abstract base classes
- **Key Features**: Term interfaces, symbol definitions, sort hierarchies

### 3. Parser Subsystem
- **Location**: `src/Parser/`
- **Purpose**: Lexical and syntactic analysis of Maude source code
- **Key Features**: Flex/Bison based parsing, mixfix operator handling

### 4. Core Engine
- **Location**: `src/Core/`
- **Purpose**: Central rewriting logic engine
- **Key Features**: 
  - Pattern matching and unification
  - Term rewriting
  - Strategy execution
  - Memory management for terms
  - Module system

### 5. Variable Management
- **Location**: `src/Variable/`
- **Purpose**: Variable binding and substitution management
- **Key Features**: Variable abstraction, substitution composition

### 6. Theory Modules
Each theory provides specialized algorithms for specific algebraic structures:

- **ACU Theory** (`src/ACU_Theory/`, `src/ACU_Persistent/`): Associative-Commutative-Unity
- **AU Theory** (`src/AU_Theory/`, `src/AU_Persistent/`): Associative-Unity  
- **CUI Theory** (`src/CUI_Theory/`): Commutative-Unity-Idempotent
- **Free Theory** (`src/FreeTheory/`): Free algebra
- **S Theory** (`src/S_Theory/`): Special theory handling
- **NA Theory** (`src/NA_Theory/`): Non-associative theory

### 7. Built-in Systems
- **Built-in Types** (`src/BuiltIn/`): Integers, strings, floating point, etc.
- **Object System** (`src/ObjectSystem/`): Object-oriented programming support  
- **Meta Level** (`src/Meta/`): Reflection and meta-programming
- **Strategy Language** (`src/StrategyLanguage/`): Strategy definition and execution

### 8. External Integration
- **SMT Integration** (`src/SMT/`): Interface to SMT solvers (Yices2, CVC4)
- **I/O Operations** (`src/IO_Stuff/`): File and network I/O
- **Temporal Logic** (`src/Temporal/`): Temporal logic model checking

### 9. Advanced Features
- **Full Compiler** (`src/FullCompiler/`): Experimental compilation features
- **Mixfix Parser** (`src/Mixfix/`): Advanced operator parsing
- **Higher Level** (`src/Higher/`): Higher-order features

## Memory Management

Maude implements sophisticated memory management for term structures:

- **Hash Consing**: Ensures structural sharing of identical subterms
- **Garbage Collection**: Automatic memory reclamation for unreachable terms  
- **Memoization**: Caches computation results for performance
- **Copy-on-Write**: Efficient term copying through delayed copying

## Concurrency and Performance

- **Single-threaded Design**: Current implementation is single-threaded
- **Optimized Data Structures**: Carefully tuned for rewriting performance
- **Theory-specific Algorithms**: Specialized algorithms for each algebraic theory
- **Lazy Evaluation**: Deferred computation where beneficial

## Extension Points

The architecture provides several extension mechanisms:

1. **New Theories**: Add support for additional algebraic theories
2. **Built-in Types**: Extend with new built-in data types  
3. **SMT Solvers**: Interface to additional SMT solvers
4. **I/O Modules**: Add new I/O capabilities
5. **Strategy Language**: Extend strategy language features

## Configuration and Portability

The build system supports extensive configuration:

- **Optional Dependencies**: SMT solvers, line editing, signal handling
- **Compiler Optimization**: Platform-specific optimization flags
- **Debug Support**: Configurable debug and profiling support
- **Cross-platform**: Supports multiple Unix-like operating systems

## Future Architecture Considerations

Potential architectural improvements:

1. **Parallelization**: Multi-threaded execution for independent computations
2. **Modularization**: Further separation of concerns
3. **Plugin Architecture**: Dynamic loading of extensions  
4. **Modern C++**: Migration to newer C++ standards
5. **Memory Optimization**: Reduced memory footprint for large computations

---

This architecture provides a solid foundation for a high-performance rewriting logic system while maintaining modularity and extensibility for future enhancements.
# Maude API and Interfaces Documentation

## Overview

This document describes the key APIs and interfaces in the Maude system, focusing on the programmatic interfaces between major components and extension points for developers.

## Core Interfaces

### Term Interface
The fundamental abstraction for all terms in Maude.

```mermaid
classDiagram
    class Term {
        <<interface>>
        +Symbol* symbol()
        +Sort* getSort()
        +bool ground()
        +int compare(Term* other)
        +Term* instantiate(Substitution& subst)
    }
    
    class DagNode {
        +Term* term2Dag()
        +bool isReduced()
        +void reduce(RewritingContext& context)
        +bool equal(DagNode* other)
    }
    
    class Symbol {
        <<abstract>>
        +int arity()
        +Sort* domainComponent(int)
        +Sort* rangeComponent()
        +bool canRewriteToItself()
        +DagNode* makeDagNode(Vector~DagNode*~& args)
    }
    
    class ConnectedComponent {
        +int nrSorts()
        +Sort* sort(int index)
        +int nrMaximalSorts()
        +int fastNrSorts()
    }
    
    Term --> Symbol
    Term --> ConnectedComponent
    DagNode --> Term
    Symbol --> ConnectedComponent
```

### Module Interface
Represents a Maude module containing sorts, operators, and statements.

```mermaid
classDiagram
    class Module {
        <<abstract>>
        +int nrUserSorts()
        +Sort* findSort(int name)
        +Symbol* findSymbol(int name, Vector~Sort*~& domain, Sort* range)
        +bool insertSort(Sort* sort)
        +bool insertSymbol(Symbol* symbol)
        +void closeSortSet()
        +void closeSignature()
    }
    
    class PreModule {
        +void addSortDecl(SortDecl* decl)
        +void addOpDecl(OpDecl* decl)
        +void addStatement(Statement* stmt)
        +Module* flatten()
    }
    
    class SortTable {
        +int nrSorts()
        +Sort* index2Sort(int index)
        +int sort2Index(Sort* sort)
        +void computeTable()
    }
    
    Module --> SortTable
    PreModule --> Module
```

### Rewriting Context Interface
Manages the execution context for rewriting operations.

```mermaid
classDiagram
    class RewritingContext {
        +Module* getModule()
        +void reduce(DagNode* node)
        +bool rewrite(DagNode* node, int gas)
        +void addInCount(DagNode* node)
        +int getCount()
        +void setObjectMode(ObjectSystem::ObjectMode mode)
    }
    
    class LocalBinding {
        +DagNode* value(int index)
        +void bind(int index, DagNode* value)
        +int nrFragileBindings()
        +void unbind()
    }
    
    class Substitution {
        +DagNode* value(int index)
        +void bind(int index, DagNode* value)
        +Substitution* clone()
        +bool occurs(VariableSymbol* var, DagNode* term)
    }
    
    RewritingContext --> LocalBinding
    RewritingContext --> Substitution
```

## Theory-Specific Interfaces

### ACU Theory Interface
Associative-Commutative-Unity theory implementation.

```mermaid
classDiagram
    class ACU_Symbol {
        +bool isAssociative()
        +bool isCommutative()
        +DagNode* getIdentity()
        +ACU_DagNode* makeCanonical(Vector~DagNode*~& args)
        +bool unify(DagNode* pattern, DagNode* subject, Substitution& subst)
    }
    
    class ACU_DagNode {
        +int nrArgs()
        +DagNode* getArg(int index)
        +int getMultiplicity(int index)
        +void insertArg(DagNode* arg, int multiplicity)
        +bool normalize()
    }
    
    class ACU_UnificationSubproblem {
        +bool solve(bool findFirst, RewritingContext& solution)
        +Subproblem* makeClone()
        +void markReachableNodes()
    }
    
    ACU_Symbol --> ACU_DagNode
    ACU_Symbol --> ACU_UnificationSubproblem
```

### Built-in Type Interfaces
Interfaces for built-in data types.

```mermaid
classDiagram
    class BuiltinSymbol {
        <<abstract>>
        +bool isConstructor()
        +void computeBaseSort(DagNode* subject)
        +bool eqRewrite(DagNode* subject, RewritingContext& context)
    }
    
    class NumberSymbol {
        +const mpz_class& getValue(DagNode* dagNode)
        +DagNode* makeNumber(const mpz_class& value)
        +bool isNat(DagNode* dagNode)
    }
    
    class StringSymbol {
        +string& getValue(DagNode* dagNode) 
        +DagNode* makeString(const string& value)
        +int length(DagNode* dagNode)
    }
    
    class FloatSymbol {
        +double getValue(DagNode* dagNode)
        +DagNode* makeFloat(double value)
        +bool isNaN(DagNode* dagNode)
    }
    
    BuiltinSymbol <|-- NumberSymbol
    BuiltinSymbol <|-- StringSymbol  
    BuiltinSymbol <|-- FloatSymbol
```

## Parser and Mixfix Interfaces

### Parser Interface
Core parsing functionality.

```mermaid
classDiagram
    class MixfixParser {
        +bool parse(TokenSequence& sequence)
        +Term* getTerm()
        +void insertProduction(int nonTerminal, Production production)
        +void insertBubbleSpec(int purpose, int lowerBound, int upperBound)
    }
    
    class TokenSequence {
        +int length()
        +Token getToken(int index)
        +void contractToFit(int maxTokens)
        +void append(Token token)
    }
    
    class Production {
        +int lhs()
        +Vector~int~ rhs()
        +int prec()
        +Vector~int~ gather()
        +int special()
    }
    
    MixfixParser --> TokenSequence
    MixfixParser --> Production
```

## Meta-Level Interface
Reflection and meta-programming capabilities.

```mermaid
classDiagram
    class MetaModule {
        +DagNode* upModule(Module* module)
        +Module* downModule(DagNode* metaModule)
        +DagNode* upTerm(DagNode* term)
        +DagNode* downTerm(DagNode* metaTerm)
        +DagNode* metaRewrite(DagNode* metaModule, DagNode* metaTerm)
    }
    
    class MetaLevel {
        +DagNode* upRewriteCount(Int64 count)
        +bool isNat(DagNode* dagNode)
        +Int64 downBound(DagNode* dagNode)
        +bool isTermType(DagNode* dagNode) 
    }
    
    class ObjectSystemRewritingContext {
        +void bufferMessage(DagNode* target, DagNode* message)
        +void executeMessages()
        +bool offerMessageExternally(DagNode* message)
    }
    
    MetaModule --> MetaLevel
    MetaModule --> ObjectSystemRewritingContext
```

## Strategy Language Interface

### Strategy Definition Interface
Programming with strategies.

```mermaid
classDiagram
    class StrategyExpression {
        <<abstract>>
        +StrategyStackMachine::Instruction* compile()
        +bool check(VariableInfo& indices)
        +void process()
    }
    
    class CallStrategy {
        +int getName()
        +Vector~Term*~ getArguments()
        +StrategyDefinition* getDef()
    }
    
    class RewriteStrategy {
        +Rule* getRule()
        +bool getTop()
        +ConditionFragment* getCondition()
    }
    
    class StrategyDefinition {
        +Term* getLhs()
        +StrategyExpression* getRhs()
        +int getNrVariables()
        +VariableInfo getVariableInfo()
    }
    
    StrategyExpression <|-- CallStrategy
    StrategyExpression <|-- RewriteStrategy
    CallStrategy --> StrategyDefinition
```

## SMT Interface
Integration with SMT solvers.

```mermaid
classDiagram
    class SMT_Manager {
        +bool assertFormula(DagNode* formula)
        +SMT_Result checkSat()
        +DagNode* getModel(DagNode* variable)
        +void push()
        +void pop()
        +void clearAssertions()
    }
    
    class SMT_Symbol {
        +bool canHandleConstraints()
        +DagNode* constructSMT_Formula(DagNode* dagNode)
        +bool isBaseSMT_Symbol()
    }
    
    class SMT_RewriteSequenceSearch {
        +bool findNextRewrite()
        +Rule* getRule()
        +DagNode* getStateDagNode()
        +const Substitution& getSubstitution()
    }
    
    SMT_Manager --> SMT_Symbol
    SMT_Manager --> SMT_RewriteSequenceSearch
```

## I/O and External Interface

### I/O Operations Interface
File and network operations.

```mermaid
classDiagram
    class StreamManager {
        +int openFile(const string& fileName, const string& mode)
        +bool closeFile(int fd)
        +string readLine(int fd)
        +bool writeLine(int fd, const string& line)
        +bool good(int fd)
    }
    
    class SocketManager {
        +int createClientTcpSocket(const string& hostname, int portNr)
        +int createServerTcpSocket(int portNr, int queueLength)
        +int acceptClient(int serverSocket)
        +bool send(int socket, const string& message)
        +string receive(int socket)
    }
    
    class ProcessManager {
        +int createProcess(const string& command, Vector~string~& args)
        +bool waitForProcess(int pid)
        +int getExitCode(int pid)
        +bool signalProcess(int pid, int signal)
    }
    
    StreamManager --> SocketManager
    StreamManager --> ProcessManager
```

## Extension Points

### Adding New Built-in Types

To add a new built-in type:

1. **Inherit from BuiltinSymbol**:
```cpp
class MyTypeSymbol : public BuiltinSymbol {
    bool eqRewrite(DagNode* subject, RewritingContext& context);
    void computeBaseSort(DagNode* subject);
    // ... implement required methods
};
```

2. **Create corresponding DagNode class**:
```cpp  
class MyTypeDagNode : public DagNode {
    MyTypeValue value;
    // ... implement storage and operations
};
```

3. **Register with module system**:
```cpp
module->insertSymbol(new MyTypeSymbol(/* parameters */));
```

### Adding New Theories

To implement a new algebraic theory:

1. **Create theory-specific Symbol class**:
```cpp
class MyTheorySymbol : public Symbol {
    bool unify(DagNode* pattern, DagNode* subject, Substitution& subst);
    // ... implement theory-specific algorithms  
};
```

2. **Implement specialized DagNode**:
```cpp
class MyTheoryDagNode : public DagNode {
    // ... theory-specific representation
};
```

3. **Add unification algorithms**:
```cpp
class MyTheoryUnificationSubproblem : public UnificationSubproblem {
    bool solve(bool findFirst, RewritingContext& solution);
};
```

### Extending the Meta-Level

Add new meta-level operations:

1. **Extend MetaModule**:
```cpp
DagNode* MetaModule::myNewMetaOperation(DagNode* args) {
    // ... implement meta-operation
}
```

2. **Register operation**:
```cpp
insertEqRewrite(myMetaOp, &MetaModule::myNewMetaOperation);
```

## Thread Safety and Concurrency

**Current Status**: The Maude system is **not thread-safe**. Key considerations:

- **Single-threaded Design**: All operations assume single-threaded execution
- **Global State**: Many components maintain global state
- **Memory Management**: Hash consing and garbage collection are not thread-safe
- **Future Work**: Thread-safety would require significant architectural changes

## Performance Considerations

### Critical Paths
- **Term Construction**: Hash consing for structural sharing
- **Pattern Matching**: Optimized for common cases  
- **Rewriting**: Memoization and incremental evaluation
- **Unification**: Theory-specific optimizations

### Memory Usage
- **Term Sharing**: Aggressive structural sharing reduces memory
- **Garbage Collection**: Reference counting with cycle detection
- **Cache Management**: LRU policies for memoization caches

### Optimization Guidelines
1. **Minimize Term Construction**: Reuse existing terms when possible
2. **Leverage Memoization**: Cache expensive computations  
3. **Choose Appropriate Theories**: Select most specific theory for performance
4. **Batch Operations**: Group related operations to improve locality

---

This API documentation provides the foundation for understanding and extending the Maude system architecture.
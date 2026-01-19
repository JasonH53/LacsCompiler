# Lacs Compiler
Coursework for advanced version of the compiler course (CS241E). Due to the nature of the course, it contains code snippets from Dr. Ondrej Lhotak. Hence, it cannot be shared publically but the code can be accessed via a private SSH key on request.

## Compiler Frontend

### Scanning
Used DFAs + a [Maximal Munch](https://en.wikipedia.org/wiki/Maximal_munch) scanner where it scans greedily to find the longest token before continuing to the next input. Consider an output sequence of non-empty words $w_1, w_2, \dots w_n$ where w is a word and L is the language of valid tokens

- Concatenate $w_1w_2 \dots w_n = w$
- Each word is valid for all $i \in L$
- Each w_i is the longest prefix remainder of input $w_iw_{i+1} \dots w_n$ in L

We can do so by running the following algorithm

- Run DFA for L on remaining input until DFA gets stuck or end of input
- If the state is non accepting, backtrack DFA to last accepted state. If no accepted state and it is all visited, return no solution
- We output the token to an accepting state
- Reset DFA and go back to step 1

### Parsing and building AST
We use a [CYK](https://en.wikipedia.org/wiki/CYK_algorithm) parser where it takes an input context-free grammar G and it determines whether the word is language G and constructs the associated parse tree. The following is the pseudocode of the algorithm used

```
parse(α, x) =
    if (α is empty) {
        if (x is empty) return empty parse tree
        else return false
    }
    elif (α = aβ where a is a terminal symbol and β is a sequence of symbols) {
        if (x = az && parse(β, z)) // check first letter of x and the rest of tokens
            return create a leaf node in the parse tree
        else return false
    }
    elif (α = A where A is a non-terminal) {
        for each production A → γ ∈ P { // A ⇒ γ ⇒* x
            if (parse(γ, x)) return a tree with A being the parent and parse(γ, x) as subtrees
        }
        return false
    } 
    else { // α = Aβ (starts with a non-terminal)
        for each way to split x into substrings x = x_1x_2 {
            if (parse(A, x_1) && parse(β, x_2)) return concatenation of two subtrees
        }
        return false
    }
```
We also memoize this to ensure that we do not repeat computations. 

### Type Checking
To check if a parse tree is valid corresponding to the Lacs specification:
For each Lacs grammar rule that could be the production rule expanding the root node of tree E, there's only one type inference rule that could be applied
- It looks at the grammar production rule in root node of E and implements the inference rule corresponding to production rule
- When implementing the inference rule, the type checker determines values for meta-variables from the tree and symbol table
- The symbol table maps names to variables/procedures and their types

### IR Translator
Based on the parse tree from the parser, typer and the context-sensitive information from the symbol table. IR is generated and is passed to the middle end of the compiler.

## Back End

### Variable Elimination
Variables are abstracted as offsets from the frame pointer. We use `eliminateVarAccesses` by differentiating it into various types
- Local variables: Access via relative frame pointer (offset)
- Parameters: Access through parameter pointer and offset of the parameter chunk
- Nested Procedures: Following the static link n times where n = depth(current procedure) - depth(procedure declaring the var)

### Register Allocation
The register allocation phase also assign variables to MIPS register or memory by
Liveness Analysis : Determine the live range of a variable
Graph Coloring : Creating an interference graph and greedily assign smallest available color to each vertex

### Code Transformation
Transforms the IR into a sequence of MIPS instructions by:
Eliminating procedure, closure calls
Eliminating control structures (If / while) using jumps and labels
Eliminating labels by replacing it to numeric PC offsets
Eliminating VarAccesses as aforementioned

### Tail Call Optimization
If procedure f calls procedure g and the last call is a tail call, the frame and parameter chunk of f can be popped before calling g to reduce stack space. It is [implemented](https://en.wikipedia.org/wiki/Tail_call) by modifying how we create instructions for procedure calls and changing from `JALR` to a jump instruction.

### Memory Management
Procedures creating closure or with nested procedures may outlive their creators. Hence, we created a garbage collector for memory management. It is created using [Cheney’s copying garbage collector](https://en.wikipedia.org/wiki/Cheney%27s_algorithm). We augment each procedure, parameter chunk with size and a pointer counter to implement the GC






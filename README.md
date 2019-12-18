#### By: Brooke Peterson brooke.o.peterson@gmail.com

# Project 1 - Lexical Analyzer

Read in a file word by word and tokenize: i.e. assign a value to the received word. Values will be keywords, variables, integers, and symbols. This is done by dropping each word into a series of ```machines``` until the word is recognized and categorized, or an error is returned. 

**Warning:** Make sure your lexical analyzer returns just one token each time __getNextToken()__ is called, or, you'll end up re-writing project 1.   

# Project 2 - Syntax Analyzer

The goal of project 2 is to create a parser: that is when it receives a token from the tokenizer, its gonna pop the token in a giant case statement machine and the token will trickle down until it finds the slot it's supposed to be at, i.e. __recursive descent parsing__. 

**Step 1:** Massage your grammar into LL(1) grammar:
1. Remove Ambiguity: eliminate nullable (epsilon) productions
2. Remove Immediate Left Recursion 
3. Remove Deep Left Recursion
4. Left Factor

Then create a parse table with the firsts and follows of each production. 

**Step 2:** Code your parser


For the parser, you'll need three main functions (__parse()__, __match()__, and __synch()__ ) to help you in addition to a function for each of your rows in the parse table (a function for program, program_prime, program_double_prime, etc...). Here are the functions I had at the end of P2 (remember, I'm an even year so odd years will have a slight variation):

```
void parse();
void match(int token);
void synch(int synchset [], int length);
void program();
void program_prime();
void program_double();
void id_list();
void id_list_prime();
void declarations();
void declarations_prime();
int type();
int standard_type();
void sub_declarations();
void sub_declarations_prime();
void sub_declaration();
void sub_declaration_prime();
void sub_declaration_double();
void sub_head();
void sub_head_prime();
void arguments();
void parameter_list();
void parameter_list_prime();
void compound_statement();
void compound_statement_prime();
void optional_statements();
void statement_list();
void statement_list_prime();
void statement();
void statement_prime();
int variable();
int variable_prime(int inputType);
void procedure_statement();
void procedure_statement_prime();
void expression_list();
void expression_list_prime();
int expression();
int expression_prime(int inputType);
int simple_expression();
int simple_expression_prime(int inputType);
int term();
int term_prime(int inputType);
int factor();
int factor_prime(int inputType);
void sign();
```

Seems like a lot of functions, but they're all pretty much identical. Project 3 entails modifying these SLIGHTLY, after you're done with P2, you've done 80% of the coding for the compiler. 

To make things easier, every time I grab a token in the parser, ie each time getNextToken() is called, I make it a global variable (global struct actually) for easy reference (Convenience is king!... Except when you're paid to program safely). Thus each "token" reference is referring to my global struct.


## The Parse() function...

The __parse()__ function is the easiest to write. All it does is call ```getNextToken()``` from your tokenizer, call the __program()__ method which you'll write shortly, call __match(EOF)__ and then return to __main()__. Here's mine for reference:

```
void parse()
{
    token = getNextToken();   // Get next token from tokenizer
    program();                // Go immediately to program() to begin the recursive descent parsing
    match(EOF);               // Look to see if the current token is EOF (done parsing if it returns TRUE)
    return;                   // Return to main() when done. 
}
```

## The Match() function...

The __match()__ function simply takes the token that is currently being processed, compares it against the expected token.type, and will print one of three things:

1. You've got an EOF token! You're done parsing!
2. You've got the right token! Lets grab the next token...
3. You've got the wrong token! ERROR: Got X when expecting Y! DON'T GET THE NEXT TOKEN. 

In the below function, ```match``` is the token tpye of the token currently being processed, and, ```token.type``` is a field of my handy global token struct. 


```
void match(int match)
{
    if(match == EOF)
    {
        printf("[+] Parse Complete!");
        return;
    }

    if(match == token.type)
    {    
        // Got the expected match! Get next token
        token = getNextToken();
        return;
    }
       
    if(match != token.type){}
        // Didn't match on the expected token, print ERROR message instead
        fprintf(lp, "SYNERR: Expecting %s got %s instead\n",tokenName(match), tokenName(token.type));
    return;
}
```

## The Synch() function...

The __synch()__ function is an error recovery mode which keeps getting tokens until it matches with a token in its synch set. Your synch set is going to be your follows plus EOF. So the synch set of __program__ will be ```EOF```, and the synch set for __type__ will be ```;```, ```)```, and ```EOF```. More on that shortly. 

The __synch()__ method takes two arguments: the array of the synch set items, and the length of said array (at least mine does, now that I think about it, you probably don't need the length, but that's what I chose to do). Then, the __synch()__ function is basically a while loop until it recovers by finding a token in the synch set, or fails until hitting an EOF.

```

void synch(int synchset [], int length){

    // Cycle through input file until either a match is found, or EOF is received
    while (token.type != EOF)
    {
        // Check current token against synch set
        for (int i= 0; i < length; i++) 
        {
            // If match is found in synch set, return!
            if ((synchset[i] == token.type))
                return;
        }
        // Match not found in synch set, get next token.
        token = getNextToken();
    }
    return;
}
```

## The rest of the functions....

The remainder of the functions for the parser are of identical format. Each simply has a case statement where the cases are the firsts, within the case you call the tokens in your grammar, the default case is calling the synch function. 

So each production gets its own case statement in the following form:
```
switch(current_token)
{
    case FIRST_#1:
        // follow production calls ...
    case FIRST_#2:
        // follow production calls ...
    case FIRST_#3:
    .
    .
    .
    default:
        print("SYN ERROR: Expecting {list firsts...}, got {X} instead!");
        int array_of_follows[#follows] = {FOLLOW_#1, FOLLOW_#2, FOLLOW_#3 ... EOF};
        synch(array_of_follows, #follows);
}
```

Here are a few examples for comparison, it's trivial...

### program()

```Positions for reference...1.....2..3...4....5.6.....7...............   ```
* 1.1.1 ```<program> -->  program  id  (  ID_List  )  ;  <program>'```  
* Firsts: ```program``` 
* Follows: ```EOF```

```
void program()
{
    switch(token.type)
    {
        case PROGRAM: // 1.1.1         // CASE: FIRST of 1.1.1!
            match(PROGRAM);            // Token #1 of 1.1.1
            match(ID);                 // Token #2 of 1.1.1
            match(OPENPAREN);          // Token #3 of 1.1.1
            id_list();                 // Item #4 of 1.1.1, not a token, its another production, call its function instead!
            match(CLOSEPAREN);         // Token #5 of 1.1.1
            match(SEMICOLON);          // Token #6 of 1.1.1
            program_prime();           // Item #7 of 1.1.1, not a token, call the corresponding production function instead! 
            break;                     // We done. Get outta here.
        default:
            // No mwatch found in the case statement(s)! Time to go through the synch set....
            // print the error
            fprintf(lp, "SYNERR: Expecting PROGRAM. Got %s instead\n",tokenName(token.type));
            // Create the synch set from the follows list
            int synchset[1] = {EOF};
            // call synch
            synch(synchset, 1);
            return;
    }
}

```

### program_prime()

* 1.2.1 ```<program>' -->  <sub_decs>  <compound_statement> .```  
* 1.2.2 ```<program>' -->  <compound_statement> .```  
* 1.2.3 ```<program>' -->  <declarations> <program>''```  
* Firsts: ```procedure, begin, var``` 
* Follows: ```EOF```

```
void program_prime()
{
    switch(token.type)
    {
        case PROCEDURE:  // 1.2.1
            sub_declarations();
            compound_statement();
            match(DOT);
            break;
        case BEGIN: // 1.2.2
            compound_statement();
            match(DOT);
            break;
        case VAR: // 1.2.3
            declarations();
            program_double();
            break;
        default:
            fprintf(lp, "SYNERR: Expecting PROCEDURE, BEGIN, or VAR. Got %s instead\n",tokenName(token.type));
            int synchset[1] = {EOF};
            synch(synchset, 1);
            return;
    }
}
```


### program_double() 

* 1.3.1 ```<program>'' -->  <sub_decs>  <compound_statement> .```  
* 1.3.2 ```<program>'' -->  <compound_statement> .```  
* Firsts: ```procedure, begin``` 
* Follows: ```EOF```

```
void program_double()
{
    switch(token.type)
    {
        case PROCEDURE:  // 1.3.1
            sub_declarations();
            compound_statement();
            match(DOT);
            break;
        case BEGIN: // 1.3.2
            compound_statement();
            match(DOT);
            break;
        default:
            fprintf(lp,"SYNERR: Expecting PROCEDURE or BEGIN. Got %s instead\n",tokenName(token.type));
            int synchset[1] = {EOF};
            synch(synchset, 1);
            return;
    }
}
```


### type()

* 4.1.1 ```<type> --> array { num .. num ] of <starndard_type> ```       
* 4.1.2 ```<type> -->  <starndard_type>```      
* Firsts: ```integer, real, array, long (I added long for some reason, you may not need it)```    
* Follows: ```;  ) EOF```   

Notice I use the case fall through to handle the three cases belonging to INTEGER, REAL, and LONG by the same statement since they're identical.

```
int type()
{
    switch(token.type)
    {
        case ARRAY: // 4.1.2
            match(ARRAY);
            match(OPENBRACK);
            match(INT);
            match(DOTDOT);
            match(INT);
            match(CLOSEBRACK);
            match(OF);
            standard_type();
            break;
        case INTEGER: // 4.1.1
        case REAL: // 4.1.1
        case LONG:
            standard_type();
            break;
        default:
            fprintf(lp,"SYNERR: Expecting INTEGER, REAL, or ARRAY. Got %s instead\n",tokenName(token.type));
            int synchset[3] = {SEMICOLON, CLOSEPAREN, EOF};
            synch(synchset, 3);   
    }
    return returnType;
}
```

# Project 3 - Type and Scope Checking

## Type Checking

Type checking is the verification that operations are preformed on the proper types. In other words, "like operates on like". For the RELOP (<, >, =<, >=...) You cannot compare an int with a real, or an int with a boolean. For an ADDOP (+, -), you can't add a boolean to an int, or a boolean to a boolean, only ints or reals should be called with an ADDOP! Type checking verifies that any time an operator is called on a variable, the operation being preformed can be "legally" applied to the types of objects being worked on. 

Good news: There are only a few cases where this needs to be done in the grammar. These are the truth tables that have been drawn out in class. 

I suggest working backwards for the type checking.

You'll do this for (in even years at least):
* factor
* factor_prime
* simple_expression
* simple_expression_prime
* term
* term_prime
* expression
* expression_prime
* var
* var_prime
* statement
* standard_type
* type   
__Note:__ Notice in my list of functions written for Project 2, these are all the functions that return an ```int``` rather than ```void```

### factor()

The item that needs type checking preformed in __factor()__ is the first ```NOT```. NOT is a boolean operator, so, logically, whatever token follows __MUST__ be a boolean, or, an error has occurred. 

**Truth table for factor():**
| Type      | factor() |
| ----------- | ----------- |
| BOOL      | BOOL       |
| ERR       | ERR           |
| ERR*   | OTHER      |
The asterisk means "print this error as a SEMERR"

Notice that the case with the NOT key word has a recursive call in the production:
* 21.1.3:  ``` <factor> --> not <factor>``` 

This is because we make two passes with this particular function: the first time it falls through to the NOT keyword, indicating that the next token should be a boolean if there are no errors. So, within the NOT case, __factor()__ is called again on the next token, and, the type of that next token is returned as an int (because the function returns an INT, not VOID).

So, pass #1, match NOT keyword, pass #2, get the type of the token following NOT and preform type checking.

How this looks within the __factor()__ function under the NOT case:

```
int factor()
{
    int returnType = ERROR;  // Default return type: if no type is set, an ERROR is returned
    int idType;              // Ignore for now, for scope checking

    switch(token.type)
    {
        case INT: // 21.1.1 NUM
            returnType = token.type;  // Return INT type!
            match(INT);
            break;
        case REAL: // 21.1.1 NUM
            returnType = token.type;  // Return REAL type!
            match(REAL);
            break;
        case LONG: // 21.1.1 NUM     // Return LONG type! 
            returnType = token.type;
            match(LONG);
            break;
        case OPENPAREN: // 21.1.2
            match(OPENPAREN);
            returnType = expression();  // Call expression() which will return type BOOL if valid
            match(CLOSEPAREN);
            break;
        case NOT: // 21.1.3                             
            match(NOT);                    // Match NOT, will get next token during match() call
            int factorType = factor();     // Recursive call to get type of next token
            if(factorType == ERROR)
            {}                             // If type ERROR, do nothing
            else if(factorType == BOOL)    // If type BOOL, set return type to BOOL, all is well
                returnType = BOOL;
            else                          // Else print SEMERR message of other type
                fprintf(lp, "SEMERR: %s is not of type BOOL. (factor)\n", tokenName(factorType));
            break;
        case ID: // 21.1.4
            // IGNORE THIS CASE STATEMENT FOR NOW, THIS IS PART OF SCOPE CHECKING
            idType = getType(token, currLexeme);
            if(check_scope(currLexeme, token.type) == 0)
                fprintf(lp, "SEMERR: %s has not been declared within this scope.\n", currLexeme);
            match(ID);
            returnType = factor_prime(idType);
            break;
        default:
            fprintf(lp, "SYNERR: Expecting NUM, OPENPAREN, NOT, or ID. Got %s instead\n",tokenName(token.type));
            int synchset[12] = {DO, THEN, CLOSEBRACK, CLOSEPAREN, COMMA, ELSE, SEMICOLON, END, RELOP, ADDOP, MULOP, EOF};
            synch(synchset, 12);     
    }
    return returnType;
}
```

### factor_prime()

The ```factor_prime()``` production has the reference to an array. In Pascal, an array is declared in the format: ```[type_of_elements .. num_of_elements]```. Type of elements can be INTs or REALs, and have been dubbed by Dr. Shenoi as AINT or AREAL, "A" standing for "array", but the number of elements MUST be an INT.

Thus the truth table for the OPENBRACK (since an array follows an ```[``` ) case of ```factor_prime()``` is:


**Truth table for factor_prine():**
| Type      | inherit    | factor() |
| ----------| -----------|----------- |
| ERR*      | ! AINT ...     | --
| ERR*      | --         | ! INT
| INT       | AINT       | INT
| REAL      | AREAL      | INT
| ERR       | ERR        | or ERR
The asterisk means "print this error as a SEMERR"

Unlike the recursive nature of the ```factor()``` example, the expression following the brackets will inherit it's type from ```expression()``` (which calls ```simple_expression()```, which calls ```term()```......ultimately the)

```standard_type()``` whose result (INT or REAL) allow ```type()``` to set the array's return type as AINT or AREAL

## Scope Checking 

# Project 4 - Memory Address Computations

It's trivial...this should take you less than an hour. Writeup to come...












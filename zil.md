# ZIL

NOTE: WIP notes regarding ZIL, taken from:
https://archive.org/details/Learning_ZIL_Steven_Eric_Meretzky_1995

---

## The Basics
### Handler
- Accepts user input
- Prints game response

### Parser
- Parses and examines user input
    1. Can parse?
    2. Parse **verb** _(PRSA)_, **direct object** _(PRSO)_, and **indirect object** _(PRSI)_

### Handling Sequence
- Determine handling sequence
    1. PRSI
    2. PRSO
    3. PRSA (default / last resort)


## Creating Rooms
### Typical Room Definition
```
    <ROOM LIVING-ROOM
        (LOC ROOMS)
        (DESC "Living Room")
        (EAST TO KITCHEN)
        (WEST TO STRANGE-PASSAGE IF CYCLOPS-FLED ELSE
            "The wooden door is nailed shut.")
        (ACTION LIVING ROOM-F)
        (FLAGS RLANDBIT ONBIT SACREDBIT)
        (GLOBAL STAIRS)
        (THINGS <> NAILS NAILS-PSEUDO) >
```
Description:
```
    <ROOM ROOM-NAME
        (INTERNAL LOCATION)
        (EXTERNAL NAME / DESCRIPTION)
        (EXITS)
        (...)
        (ROOM'S ACTION ROUTINE)
        (FLAGS)
        (GLOBALS)
        (OBJECTS) >
```


## Creating Objects
### Typical Object Definition
```
    <OBJECT LANTERN
        (LOC LIVING-ROOM)
        (SYNONYM LAMP LANTERN LIGHT)
        (ADJECTIVE BRASS)
        (DESC "brass lantern")
        (FLAGS TAKEBIT LIGHTBIT)
        (ACTION LANTERN-F)
        (FDESC "A battery-powered lantern is on the trophy case.")
        (LDESC "there is a brass lantern (battery-powered) here.")
        (SIZE 15) >
```


## Routines
### Basic Routine Structure
```
    <ROUTINE ROUTINE-NAME (argument-list)
        <guts of the routine>>
```
### Example  simple routines
```
    <ROUTINE TURN-OFF-HOUSE-LIGHTS ()
        <FCLEAR ,LIVING-ROOM ,ONBIT>
        <FCLEAR ,DINING-ROOM ,ONBIT>
        <FCLEAR ,KITCHEN, ONBIT>>

    <ROUTINE INCREMENT-SCORE (NUM)
        <SETG SCORE <+ ,SCORE .NUM>>
        <COND (,SCORE-NOTIFICATION-ON
            <TELL "[Your score has just gone up by "
                N .NUM ".]" CR>)>>
```

### Calling a routine
```
    <ROUTINE ROUTINE-A ()
        <TELL "Routine-B is about to be callsed by Routine-A." CR>
        <ROUTINE-B>
        <TELL "Routine-B just called by Routine-A." CR>>
```

### Conditionals
#### Simple if-then
```
    <COND (<if-this-is-true>
           <then-do-this>)>
```
#### Multiple predicates
A COND may have more than one predicate clause; if so, the COND
will continue until one of the predicates proves to be true, and
then skip any remaining predicate clauses.
```
    <COND (<predicate-1>
           <do-stuff-1>)
          (<predicate-2>
           <do-stuff-2>)
          (<predicate-3>
           <do-stuff-3>)>
```
A routine may have more than one COND:
```
    <COND (<predicate-1>
           <do-stuff-1>)>
    <COND (<predicate-2>
           <do-stuff-2>)>
```

### Exiting a routine
All routines return a value to the routine that called them. That value
is the last thing that the routine did. You can also force a routine to
return a specific value.
```
        <COND (<IN? ,CANDY-BAR ,HERE>
               <RETURN ,CANDY-BAR>)>
        <COND (<IN? ,CANDY-BAR ,HERE>
               <RETURN "candy bar">)>
```


## ZIL Instructions (op-codes)
Examples:
```
    <FCLEAR ,SEARCHLIGHT ,ONBIT>
    <SET .LAP-COUNTER 12>
    <RANDOM 100>
    <SAVE>
    <QUIT>
```

---

## Object Action Routines
Simple Example: Object AVOCADO with property: 
```
    (ACTION AVOCADO-F)
```
calls:
```
    <ROUTINE AVOCADO-F ()
        <COND (<VERB? EAT>
               <REMOVE ,AVOCADO>
               <TELL "The avocado is so delicious that you eat it all." CR>)
              (<VERB? CUT OPEN>
               <FSET ,AVOCADO ,OPENBIT>
               <MOVE ,AVOCADO-PIT ,AVOCADO>
               <TELL "You halve the avocado, revealy a gnarly pit." CR>)>>
```

### Common predicates
- VERB?
- EQUAL?
- FSET?
- IN?
- Global boolean vars (,CYCLOPS-FLED)
- Routines (<IS-OTTO-ON-ELBA?>)

### More complicated predicates
- NOT
- AND
- OR

### More complicated object action routine
```
    <ROUTINE AVOCADO-F ()
        <COND (<VERB? EAT>
               <COND (,AVOCADO-POISONED
                    <SETG PLAYER-POISONED T>
                    <REMOVE ,AVOCADO>
                    <TELL "You begin to feel sick." CR>)
                    (<AND <EQUAL? ,HERE ,GARDEN-OF-EDEN>
                        <IN? ,SNAKE ,TREE-OF-KNOWLEDGE>>
                            <MOVE ,APPLE ,HERE>
                            <TELL "The avocado is so unappetizing. Suddenly a seductive voice beckons from the tree. A moment later, a succulent apple lands at your feet." CR>)
                            (T
                             <REMOVE ,AVOCADO>
                             <MOVE ,AVOCADO-PIT ,PLAYER>
                             <TELL "You eat the entire avocado. It was filling, if not tasty." CR>)>)
                    (<AND <VERB? THROW>
                          <EQUAL? ,HERE ,MIDWAY>
                          <NOT ,BALLOON-POPPED>
                          <HAWKER-AT-CIRCUS>>
                     <MOVE ,AVOCADO ,MIDWAY-BOOTH>
                     <TELL "The avocado bounces off the balloon. The hawker sneers. \"You'd have more luck with a dart, kiddo! Only two bits!\"" CR>)
                    (<AND <VERB? COOK>
                        <OR <NOT EQUAL?? ,HERE ,KITCHEN>>
                            <NOT <IN? ,COOKPOT ,OVEN>>>>
                     <TELL "Even a master chef couldn't cook an avocado with what you've got!" CR>)>>
```


## Room Action Routines
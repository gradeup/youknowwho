# youknowwho

Rule engine for most of generic decisions and flow control.

Rules consists of Conditions and Actions. Action of a rule is applied when its concerned conditions evaluate to true. 

## Rules

- *Id*            : Unique identification of a rule

- *Name*          : Not used anywhere except for Logging

- *Tags*          : Each rule can be associated with many tags. They help in applying selective rules.

- *Description*   : Personal comments for a rule

- *External Reference*    : If rules are created by some other entity, this is used to create that bonding

- *Status*    : Enabled / Disabled

- *Priority*  : An integer value, which decides the order of rule execution. Lower the integer value, higher will be its priority and Earlier will be the execution of rule.

- *Conditional Operator* : Operator Applied to all conditions to calculate final True/False value for conditions.

	* || if all conditions are to be || (ORed)
	* && if all conditions are to be && (ANDed)
  * Custom value E.g. <%= c[0] %> && <%= c[1] %> || <%= c[2] %> && <%= c[3] %> : which dictates how conditions are manipulated

- *Conditions* : They are applied sequentially, and are blocking in manner. Next condition is applied only after previous has been evaluated.

- *Actions* : They are applied only after conditions evaluate to `True` according to conditional operator's value. They are sequential, and blocking in manner. Next Action is applied only after previous has been evaluated.

### Rule Conditions

Each condition has 4 parts

1. **Condition**

Possible options are 

* **Check Variable** : Default option which uses key value operation

2. **Key**           : Value / dictionary path which will be compared
3. **Value**         : which will be compared to the key
4. **Operation**     : How to compare. Possible options are

* ( '=', 'Equals ( = )')

* ( '!=', 'Not Equals ( != )'),

* ( '>', 'Great than Integer ( > )')

  Before comparing, the value on both sides is parsed as float using [parseFloat][parsefloat-link].

* ( '>=', 'Great than Equals Integer( >= )')

	Before comparing, the value on both sides is parsed as float using [parseFloat][parsefloat-link].

* ( '<', 'Less than Integer( < )')

	Before comparing, the value on both sides is parsed as float using [parseFloat][parsefloat-link].

* ( '<=', 'Less than Equals Integer(<= )')

	Before comparing, the value on both sides is parsed as float using [parseFloat][parsefloat-link].

* ('range', 'In Numerical Range ( range )')

* ('!range', 'Not In Numerical Range ( !range )')

	E.g. : 1,2, *4~5*, *6~10*,11,*12~*


* ('datetimerange', 'In DateTime Range (datetimerange )'), ('!datetimerange', 'Not In DateTime Range (!datetimerange )'),

	E.g. : 2015-06-11 00:00:00~ 2015-07-12 00:00:00

	**NOTE** 
        * ALWAYS specify datetime in YYYY-MM-DD HH:mm:ss format
        * If datetimeformat is invalid then that condition is given benefit of doubt and marked as null. Same is true of condition value is also not correct datetime value

* ('timerange', 'In Time Range ( timerange )'), ('!timerange', 'Not In Time Range ( !timerange )'),

    **NOTE**
        * Format HH:mm:ss~HH:mm:ss
        * If time is invalid then that condition is given benefit of doubt and marked as null. Same is true of condition value is also not correct datetime value

* ('regex', 'In Regex ( regex )'), ('!regex', 'Not In Regex ( !regex)'),

    E.g. ["a", "b", "c"]

* ('stringrange', 'In String Array ( stringrange)'), ('!stringrange', 'Not In String Array ( !stringrange )'),

    E.g. [1, 2, 3]

* ('set', 'Is a part of the Set (set)'), ('!set', 'Is not a part of the set (!set)'),

* ('includesint', 'Does array contains the given integer ( includesint )'), ('!includesint', 'Does array not contains the given integer ( !includesint )'),

* ('includesstring', 'Does array contains the given string ( includesstring )'), ('!includesstring', 'Does array not contains the given string ( !includesstring )'),

The value to the condition can be a static value or something that belongs to the input message of the rule.
To provide dynamic input values, use the [lodash template](https://lodash.com/docs/4.17.4#template) syntax. like 

```
<%= input.variable1 %>
```

### Rule Actions

Each Action has 3 parts

1.  **Action** : Possible options are

* **Set Variable** : Sets a variable in the source message

* **Set Number** : Sets a variable as Number in the source message

* **Stop Processing more rules** :  Stop Processing more rule after this action. This will continue processing all actions of the rule which is being processed, and will exit after that. This will **NOT** stop the next action. Hence it should ideally be the last action of a rule. Generally used to handle those cases when we want to return from a function on meeting a particular condition

* **DANGEROUS_EVAL** : This uses `eval` to evaluate the key and overwrite it. The value should be a valid expression like

```
Math.log(9) * 4
```

2. **Key** : For Set variable, Set Number or according to action

3. **Value** : For set variable or according to action. See Rule Action Value for options

### Rule Action Value

- **boolean** : "true" or "false" is converted to boolean TRUE/FALSE

- **template string** : Specify string in lodash format `<%= variable %>` and it will be treated as a lodash template. The variables in the template are picked from message.

- **normal string** : any string other than above two will be treated as a normal string.


## Custom Function in Actions ( Just in theory, NOT IMPLEMENTED as of now)
The function is expected to be in following format

```sh
function custom(callback, arg) {
    
    .... do something with arg()
    callback();
} 
```

 - Each function's 1st argument will be the callback it is expected to call once its execution is over.
 - There is no differentiation in blocking and non-blocking functions, and also in sync and async functions, since we are anyway passing a callback.

 - **arg** : arg is the argument which can be passed to function by previous rules/actions. Each message will have a key :=> **msg.__execargs__** which will have key for each function. E.g. if user wishes to pass an object as an argument to function *f1* , it will set it in **msg.__execargs__.f1** . **NOTE** : At the end , __execargs__ key will be deleted from message, hence try not to put anything of persistence there.

 - **function context** : Rule engine will expose a *execContext* which will be the object which will hold all function definitions and will be supplied as the context (this) in function execution.

 - **Error handling/ exceptions** : All functions will be called within a try-catch exception handling and rule engine is not expected to handle any error/exception.


### Logs and Debugging

( Tag 0.0.7 ) UPDATED : wont emit logs anymore. This will be taken care by Meta object

### Usage

Load the Rules first( and again and again ...) and simply apply them .

```
# loaded_hash is hash generated for the list of rules.
# This can be used to differentiate the set of rules.

loaded_hash = ruleEngineObject.loadRules(arrayOfRules);
```

Applying Rules

```
/*
    msg : the object which needs to be changed. This object is passed by reference and if user wishes to keep the original object sane then he/she needs to clone the object before passing here ( using lodash/underscore or similar )

    ruletag : single tag. TODO : support for multiple tags

    generateMeta : (boolean) If you want to generate meta information of rule execution ( will be 50% slower to execute a rule ). Default false
*/

meta_object = ruleEngineObject.applyRules(msg, ruletag, generateMeta);

```

### Array of Rules Example

```
Var arrayOfRules = [
    {
    "id": 1, // rule ID , has to be unique 
    "name": "Natural Number ", // Dont care
    "external_reference": "", // Dont care
    "conditionsOperator": "&&", // very important
    "priority": 170001,
    "tags": [
        "natural",
        ""
    ],
    "conditions": [
        {
            "id": 1,
            "key": "integer",
            "operation": ">",
            "value": "0"
        }
    ],
    "actions": [
        {
            "id": 2,
            "action": "SET_VARIABLE",
            "key": "is_natural",
            "value": 1
        }
    ]
}
];

```


### Rule Engine Hash ( loaded_hash )

This calculates a hash of loaded rules , which helps in audit. This is a simple md5 hash.

### Rule Engine Cache assist

It might be possible for the user to cache the input and output of the rule engine. An input object can contain any number of keys but the keys on which conditions are applied are the actual keys which give the rule engine its dynamic nature. After rules are loaded user can call `getLoadedMeta` to get loaded rules hash and array of condition keys. 
User is expected to create a sub object consisting of these condition keys only , hash it and use it for cache.
Use RULES HASH + HASH of object having condition keys only as Cache Key. value can be Output of the rule engine.

E.g. If condition keys are [key1, key2] and object which is generally passed to rule engine is { key1: 1, key2 : 2, key3 : 3}, then user should create a smaller object { key1: 1, key2: 2} , take hash of this, and append loaded-rules-hash in it and use that as the key of cache.

```
ruleEngineObject.getLoadedMeta()
// will return an object like this

{ rules_load:
   { hash: '3f2ddc875c24b0aabe238a21d9da8e0a',
     load_start: '1469687197716',
     load_end: '1469687197727',
     load_exec_time: 11,
     uniqueConditionKeys : ['key1', 'key2'],
     uniqueActionKeys : ['keyA', 'keyB']
    } 
 }
```

### Meta Object

Meta object saves the state of each rule , condition and action with variups required timestamps. It is not in the scope of this project to analyze the performace/metrics of the engine/rules.
This can help in re-arranging conditions, removing redundant /slow rules, etc. . Idea should be to minimize the number of conditions/rules for each message.
Schema of meta object

Rules, conditions and actions are dictionary based to have easy accessibility

```
    {
    "rules": { // Rules object, each key: val is rule id : data
        "1": { // rule ID
            "ruleid": 1, // rule id
            "exec_order": 1, // Execution order of rules by RE. starts with 0
            "conditions": { // conditions object
                "1": { // condition ID
                    "cid": 1, // condition ID
                    "lval": -1, // Left value of condition
                    "op": ">", // Operation applied
                    "rval": "0", // Right value of operation
                    "d": false // decision for this condition
                }
            },
            "applied": false, // Rules was applied or not
            "actions": {} // actions
        },
    },
    "rules_load": {
        "hash": "feb65a88a9f346494ad5a12de14dc7ec", // hash of loaded rules
        "load_start": "1463996594477", // Rules load start time
        "load_end": "1463996594479", // Rules loading end time
        "load_exec_time": 2 // milliseconds for rule loading
    },
    "startTime": "1463996594492", // start time of RE execution
    "endTime": "1463996594494", // end time ( Unix Milliseconds )
    "execTime": 2 //exec time for RE
}
```


### Benchmarks : uses benchmark.js
```
$ npm run benchmark

Start Benchmark Set 1 : with 1 Rule : natural number
ruleengine x 123,730 ops/sec ±2.79% (82 runs sampled)
native x 39,966,923 ops/sec ±2.55% (83 runs sampled)
Fastest is native
```



### Theory 

* Why will we never support 'calling rules' with multiple tag options ? 

	Then conditional operators among tags will be a major requirement and tags are essentially rule groups.

* Changing Rule Engine execution from sync to async :

	While sync offered a lot more performance benefit, ease of usage and code simplicity, it lacked a majority extensibility, i.e. executing custom functions in code flows. Conditions, while sync, are forcefully made async to keep the options open, and to keep performance realistic.


### Todo / improvements / known Bugs
- Support for Custom Blocking/non-blocking sync/async functions is still debatable and is not added as of now
- Rule Snapshots ? Rule Audits ?
- How to define a common Rules language ? Currently Rules are picked from DB. Is that standard way , or should we define an API for this ?
- Give a GUI to manage Rules/ get status/ get active Rules, etc...
- Performance benchmark


[parsefloat-link]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/parseFloat
